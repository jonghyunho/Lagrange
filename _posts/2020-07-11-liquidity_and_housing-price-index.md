---
layout: post
title: "유동성과 서울 부동산"
author: "Jonghyun Ho"
categories: Data Analysis
tags: [Crawling, Python, 부동산, 아파트, 매매가격지수]
image: posts/20200711/liquidity-housing-comparison-graph.png
---

이번 포스팅에서는 현금 유동성과 서울 부동산의 관계를 살펴보고자 한다.


## M1, M2, Lf

- M1(협의통화)은 현금통화와 요구불예금 수시입출식예금(투신사 MMF 포함)의 합계

- M2(광의통화)는 M1과 만기 2년 미만 금융상품(예적금, 시장형 및 실적배당형, 금융채 등)의 합계

- Lf(금융기관 유동성, 종전 M3)는 M2와 2년 이상 유동성 상품, 생보사 보험계약준비금 등의 합계

이 중에서 `M1(협의통화)/M2(광의통화)`의 비율을 살펴보면 즉시 투입될 수 있는 현금의 비중을 살펴볼 수 있어 현금의 유동성을 대표할 수 있는 수치라 할 수 있다.

출처 : [M1, M2, Lf](https://terms.naver.com/entry.nhn?docId=20082&cid=43659&categoryId=43659)


## 한국은행 OpenAPI

[한국은행 경제통계시스템 OpenAPI](http://ecos.bok.or.kr/jsp/openapi/OpenApiController.jsp)에 접속하여 회원 가입 후 `서비스 이용` > `인증키 신청` 메뉴에서 개발에 필요한 인증키를 발급받는다.

인증키를 발급받은 후,`개발 가이드` > `통계코드검색` 메뉴에서 API 호출에 필요한 코드를 파악할 수 있다.

![통계코드검색](/assets/img/posts/20200711/statistics-code.png)

위의 `통계코드검색` 페이지를 살펴보면

`M1`의 통계자료 코드는 `010Y002`, 통계항목 코드는 `AAAA16` 이고,

`M2`의 통계자료 코드는 `010Y002`, 통계항목 코드는 `AAAA18` 임을 알 수 있다.


## 한국은행 데이터 크롤링

아래 `getDataFromBankOfKorea`함수는 `code`와 `sub_code` 라는 변수의 이름으로 위의 코드를 입력으로 받아 한국은행 API를 호출하여 시계열 데이터를 얻을 수 있다.

``` python
import requests
import json
from datetime import datetime
import pandas as pd
import matplotlib.pyplot as plt

endpoint_url = 'http://ecos.bok.or.kr/api'
dev_key = 'XXXXXXXXXXXXXXXXXX' 
start_date = '199501'
end_date = '202007'

def getDataFromBankOfKorea(name, code, sub_code):
    url = endpoint_url + '/StatisticSearch/' + dev_key + '/json/kr/1/2000/' + code + '/MM/' + start_date + '/' + end_date + '/' + sub_code

    req = requests.get(url)

    results = json.loads(req.text)
    results = results['StatisticSearch']['row']

    series = pd.Series(dtype=float)

    for result in results:
        time = result['TIME']
        time = datetime(year=int(time[0:4]), month=int(time[4:6]), day=1)

        value = float(result['DATA_VALUE'])

        series[time] = value

    df = pd.DataFrame()
    df[name] = series
    return df
```

위 함수를 이용하여 `M1/M2` 유동성 비중을 계산하고, 서울 아파트 매매가격지수의 데이터를 합산하여 하나의 데이터 프레임을 생성한다.

``` python
def getM1M2():
    df_M1 = getDataFromBankOfKorea('M1', '010Y002', 'AAAA16')
    df_M2 = getDataFromBankOfKorea('M2', '010Y002', 'AAAA18')

    df = pd.DataFrame()
    df['M1'] = df_M1['M1']
    df['M2'] = df_M2['M2']
    df['M1/M2'] = df['M1'] / df['M2']
    return df

df = getM1M2()

from kbstar.get_house_price_index import get_house_price_index
house_price = get_house_price_index('../../datasets/★(월간)KB주택가격동향_시계열(2020.06).xlsx', '매매종합')
house_price = house_price['서울']['서울']

df['Seoul APT price index'] = house_price
print(df)
```

`get_house_price_index` 함수는 [주택매매가격 종합지수](https://jonghyunho.github.io/data/analysis/housing-purchase-price-composite-indices.html) 의 글에서 구현된 함수를 재사용하였다.

실행 결과는 다음과 같다.

``` bash
                   M1         M2     M1/M2  Seoul APT price index
1986-01-01    14883.1    43133.6  0.345047              30.043817
1986-02-01    15038.9    43492.4  0.345782              30.043817
1986-03-01    15573.8    44587.1  0.349289              30.002377
1986-04-01    15891.8    45188.7  0.351676              29.836618
1986-05-01    16227.3    46197.5  0.351259              29.587979
...               ...        ...       ...                    ...
2019-12-01   927098.5  2912434.1  0.318324             102.559631
2020-01-01   945103.8  2929009.2  0.322670             103.055966
2020-02-01   957889.6  2954603.8  0.324202             103.416594
2020-03-01   988826.3  2984304.3  0.331342             103.905202
2020-04-01  1012290.1  3015816.3  0.335660             104.072285

[412 rows x 4 columns]
```


## 데이터 시각화

위에서 생성된 데이터프레임을 이용하여 정규화 과정을 거쳐 그래프로 시각화한 결과이다.

``` python
df = df[['Seoul APT price index', 'M1/M2']]
df = (df - df.mean()) / df.std()
df.plot()
plt.grid()
plt.show()
```

![liquidity-housing-comparison-graph](/assets/img/posts/20200711/liquidity-housing-comparison-graph.png)


## 결론

그래프를 확인해보면, 1998년 IMF 위기와 2008년 리먼브러더스 위기 때, `M1/M2` 비중이 최저점을 찍은 후 부동산 매매가격지수도 일정 구간 상승하지 못하고 횡보하는 것을 확인할 수 있다.

반대로 2000년 초반, 2014년을 기점으로 `M1/M2` 비중이 급격히 증가하는데, 부동산 매매가격지수도 함께 지속적인 상승 곡선을 보이는 것을 확인할 수 있다.

유동성은 2019년에 다소 하락하지만 2020년에는 다시 증가하고 있어 집값 상승 가능성이 커지고 있다고 볼 수 있다.