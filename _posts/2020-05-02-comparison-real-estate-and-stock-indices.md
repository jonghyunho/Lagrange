---
layout: post
title: "서울 부동산과 KOSPI 지수의 관계"
author: "Jonghyun Ho"
categories: Data Analysis
tags: [Crawling, Python, Pandas, DataFrame, KOSPI, 부동산, 아파트, 매매가격지수]
image: posts/20200502/comparison-housing-and-kospi.png
---

보통 부동산의 가격이 오르면 주식의 가격이 내리고, 부동산의 가격이 내리면 주식의 가격이 오른다는 이야기를 듣곤 한다.

부동산과 주식, 두 지수에는 어떠한 관계가 있는지 살펴보고자 한다. 세부적인 비교를 위해 `서울 부동산`과 `KOSPI 지수`를 비교 대상으로 하였다.


## Python 을 이용한 분석

필요한 모듈을 임포트 한다.

`get_house_price_index` 함수는 [주택매매가격 종합지수](https://jonghyunho.github.io/data/analysis/housing-purchase-price-composite-indices.html) 의 글에서 구현된 형태로 재사용한다.

``` python
import datetime
import pandas as pd
import matplotlib.pyplot as plt

import yfinance as yf
from kbstar.get_house_price_index import get_house_price_index
```

`get_stock_prices` 는 [How to get stock data using Yahoo Finance Python API
](https://jonghyunho.github.io/data/crawling/how-to-get-stock-data-using-yahoo-finance-python-api.html) 에서 사용한 방법과 동일하지만 일간 단위의 데이터를 월별 데이터로 변환하였다.

``` python
def get_stock_prices(code, column_name, period='5y', is_monthly=True):
    stock = yf.Ticker(code)
    stock = stock.history(period=period)
    stock = stock[['Close']]
    stock.rename(columns={'Close': column_name}, inplace=True)

    if is_monthly:
        monthly = []
        for index in stock.index:
            monthly.append(datetime.datetime(index.year, index.month, 1))

        stock['Monthly'] = monthly
        stock = stock.groupby('Monthly').mean()

    return stock
```

월별로 정리된 데이터는 다음과 같다.
``` python
kospi = get_stock_prices('^KS11', 'kospi', 'max')  # KOSPI Composite Index
print(kospi)
```
```python
                  kospi
Monthly                
1997-07-01   751.725455
1997-08-01   741.660500
1997-09-01   676.228947
1997-10-01   581.165909
1997-11-01   497.304000
...                 ...
2019-12-01  2147.013500
2020-01-01  2203.442500
2020-02-01  2167.123500
2020-03-01  1786.746364
2020-04-01  1849.589000

[274 rows x 1 columns]
```

datasets 폴더에 시계열 데이터가 저장된 엑셀 파일에서 `매매종합` 시트의 값을 읽는다. `KOSPI 지수`와 비교하기 위해 `부동산 지수`는 `서울`의 데이터를 사용하였다.

``` python
house_price = get_house_price_index('datasets/★(월간)KB주택가격동향_시계열(2020.04).xlsx', '매매종합')
house_price = house_price['서울']['서울']
print(house_price)
```
``` python
1986-01-01     30.043817
1986-02-01     30.043817
1986-03-01     30.002377
1986-04-01     29.836618
1986-05-01     29.587979
                 ...    
2019-12-01    102.559631
2020-01-01    103.055966
2020-02-01    103.416594
2020-03-01    103.905202
2020-04-01    104.072285
Name: 서울, Length: 412, dtype: float64
```

두 지수의 데이터를 하나의 `DataFrame` 에 합치고, 비교를 위해 정규화를 하였다.

``` python
df = pd.DataFrame()
df['housing'] = house_price
df['kospi'] = kospi.kospi
df = df.dropna()
df = (df - df.mean()) / df.std()
print(df)
```
``` python
             housing     kospi
1997-07-01 -1.457585 -1.118629
1997-08-01 -1.453304 -1.134642
1997-09-01 -1.442603 -1.238745
1997-10-01 -1.440463 -1.389991
1997-11-01 -1.446884 -1.523417
...              ...       ...
2019-12-01  1.621952  1.101293
2020-01-01  1.647585  1.191072
2020-02-01  1.666210  1.133288
2020-03-01  1.691445  0.528103
2020-04-01  1.700074  0.628086

[274 rows x 2 columns]
```

1997년부터 현재까지 비교된 상관관계 지수는 0.9 로, `서울 부동산`과 `코스피 지수`는 매우 밀접한 관계를 갖고 있는 것을 확인할 수 있었다.

``` python
print(df.corr())
```
``` python
          housing     kospi
housing  1.000000  0.914871
kospi    0.914871  1.000000
```


## 구간별로 비교하여도 강한 상관관계를 가지는가?

아래의 코드로 두 개의 그래프가 그려지는데, 첫 번째 그래프는 `서울 부동산`과 `코스피 지수`의 등락을 함께 표시하였고, 두 번째 그래프는 시간의 흐름에 따른 상관관계의 변화를 표시하였다.

``` python
ax1 = plt.subplot(211)
df.plot(ax = ax1, grid=True)

ax2 = plt.subplot(212)
df_corr = pd.DataFrame(df['kospi'].rolling(10).corr(df['housing']), columns=['correlation'])
df_corr.plot(ax=ax2, grid=True)

plt.tight_layout(True)
plt.show()
```

![correlation-housing-and-kospi](/assets/img/posts/20200502/correlation-housing-and-kospi.png)

2015년 이후의 데이터로 비교한 결과는 다음과 같다.

![correlation-housing-and-kospi-5y](/assets/img/posts/20200502/correlation-housing-and-kospi_5y.png)

첫 번째 그래프에서 긴 시간을 놓고 보면 두 지수가 밀접한 관계를 가지는 반면, 두번째 `correlation` 그래프에서는 밀접한 관계를 가질 때도 있고 역관계를 가질 때도 있는 시기를 반복하며 시간의 흐름에 따라 관계성이 달라지는 것을 확인할 수 있었다.


## 주식과 부동산 어느 것이 더 투자에 유리한가?

긴 시간 동안 두 지수가 0.9 이상의 상관 관계를 갖는다는 것은 결국 물가 상승률 만큼 혹은 그 이상으로 동일하게 상승하고 있다는 것을 의미하고, 구간별로 상관 관계가 변화한다는 것은 둘 중 하나의 지수가 상대적으로 더 비싸고, 싸고를 반복하며 주기를 이룬다는 의미가 될 수도 있을 것 같다.

그래서 정규화된 두 지수의 차를 계산하여 그래프로 시각화하여 확인하였다.

`코스피 지수`에서 `서울 부동산` 지수의 차를 계산하였으므로 양수일 경우 `서울 부동산`이 가격적으로 더 저렴하여 투자에 더 용이하고, 반대로 음수일 경우 `코스피 지수`가 더 투자에 유리하다고 볼 수 있을 것 같다.

``` python
ax1 = plt.subplot(211)
df.plot(ax=ax1, grid=True)

ax2 = plt.subplot(212)
df_diff = df['kospi'] - df['housing']
ax2.plot(df_diff.index, df_diff.values, '-k', label='comparison')
ax2.fill_between(df_diff.index, df_diff.values, 0.0, where=df_diff.values > 0.0, facecolor='b', alpha=0.1)
ax2.fill_between(df_diff.index, df_diff.values, 0.0, where=df_diff.values < 0.0, facecolor='r', alpha=0.1)
ax2.legend()
ax2.grid()

plt.show()
```

![comparison-housing-and-kospi](/assets/img/posts/20200502/comparison-housing-and-kospi.png)

두 번째 그래프에서 파란색으로 칠해진 구간이 `부동산`이 유리한 구간, 빨간색으로 칠해진 구간이 `주식`이 유리한 구간으로 해석해 볼 수 있을 것 같다.

실제로 2009년 빨간색으로 칠해진 구간에는 `리먼 브러더스 글로벌 금융 위기`로 인해 주가가 폭락하여 `주식`이 유리한 구간이 있었고, 2014년 이후 파란색으로 칠해진 구간에는 부동산 가격이 급등했던 시기로 `부동산`이 유리한 구간이었다고 볼 수 있다.

최근에는 코로나 바이러스로 인해 주가가 폭락하여 2009년 `리먼 브러더스 글로벌 금융 위기`때 만큼 하락하여 `주식`에 유리한 구간이 되어 있다.

향후 이 그래프의 방향성에 대해 주시해보는 것도 흐름을 예측하는 데 도움이 될 것 같다.

