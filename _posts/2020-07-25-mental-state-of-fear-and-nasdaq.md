---
layout: post
title: "나스닥과 공포 심리의 관계"
author: "Jonghyun Ho"
categories: Data Analysis
tags: [Crawling, Python, Covid, Fear, 공포 심리, Nasdaq, 나스닥]
image: posts/20200725/fear-and-stock.png
---

현재 미국 내 코로나 바이러스 일 신규 확진자 수는 7만명을 넘어서고 있고 누적 확진자 수는 400만명을 넘고 있다.

코로나 바이러스의 공포 심리로 인해 미국 증시는 3월에 큰 폭으로 하락하였지만 그 이후 현재 증시는 어느 정도 회복되었다.

사실 확진자 수는 3월보다 현재가 더 많은 상황이지만 증시는 더 이상 하락하지 않고 다시 상승하고 있다.

증시가 신규 확진자 수와 관련이 있다면 3월보다 현재 지수가 더 하락해야 하지만 상승한다는 것은 실제 확진자 수와는 크게 관련이 깊지는 않은 것처럼 보인다.

코로나 바이러스가 장기화 될 것이라고 생각되고, 생활 방역으로 경제 활동을 지속하는 것이 뉴노멀이 되어 바이러스 발생 초기 만큼의 공포 심리를 가지고 있지 않아서일까?

이러한 관점에서 증시와 공포 심리는 어떤 관계를 갖고 있을지 살펴보려고 한다.

공포 심리를 파악하기 위해 미국 일 신규 확진자 수, 구글 트렌드의 `covid` 검색 결과, `VIX` 지수 데이터 등을 이용하고 증시는 나스닥 데이터를 이용하고자 한다.


## 미국 일 신규 확진자 수

미국 일 신규 확진자 수는 [미국 질병통제예방센터](https://www.cdc.gov/coronavirus/2019-ncov/cases-updates/cases-in-us.html) 에서 데이터를 확인할 수 있다.

![US New Cases by Day](/assets/img/posts/20200725/us_new_cases_by_day.png)

``` python
import requests
import json
import numpy as np
import pandas as pd
from dateutil.parser import parse
import matplotlib.pyplot as plt

def getUSCovidNewCases():
    req = requests.get('https://www.cdc.gov/coronavirus/2019-ncov/json/new-cases-chart-data.json')
    new_cases = json.loads(req.text)

    dates = np.array(new_cases[0][1:])
    num_cases = np.array(new_cases[1][1:], dtype=float)

    series = pd.Series(dtype=float)
    series.index.name='date'
    for i, date in enumerate(dates):
        date = parse(date)
        num_case = num_cases[i]
        series[date] = num_case

    df = pd.DataFrame(series, columns=['US Covid New Cases'])
    df.rename(index={'': 'date'})
    return df
	
df_covid_new_cases = getUSCovidNewCases()
print(df_covid_new_cases)
```

`getUSCovidNewCases` 함수의 실행 결과는 다음과 같다.

``` python
            US Covid New Cases
date                          
2020-01-22                 1.0
2020-01-23                 0.0
2020-01-24                 1.0
2020-01-25                 0.0
2020-01-26                 3.0
...                        ...
2020-07-19             63201.0
2020-07-20             57777.0
2020-07-21             63028.0
2020-07-22             70106.0
2020-07-23             72219.0

[184 rows x 1 columns]
```

이를 그래프로 시각화 해보자.

```
df_covid_new_cases.plot()
plt.grid()
plt.show()
```

![Crawling US new cases by day](/assets/img/posts/20200725/crawling_us_new_cases_by_day.png)


## 구글 트렌드의 `covid` 검색 결과

아래는 [Google Trends](https://trends.google.com/trends/explore?q=covid&geo=US) 에서 `covid` 를 검색한 결과이다.

![Google Trends US Covid](/assets/img/posts/20200725/google_trends_us_covid.png)

최근 일 신규 확진자 수는 지속적으로 늘어나고 있는 반면, `covid` 검색량은 오히려 증가하다가 최근에는 다시 줄어들고 있는 추세를 보인다.

검색 엔진에서 `covid` 단어를 검색한 빈도는 실제 일 신규 확진자 수와 유사하겠지만 어느 정도 사람들의 심리가 반영되지 않았을까 하는 추측을 해본다.

관심이 있는 만큼 검색을 해보지 않을까 싶어서이다.

얼마나 큰 상관관계를 가지고 있는지 파악하기 위해 이 데이터도 함께 분석에 사용한다.

``` python
from pytrends.request import TrendReq

def getGoogleTrend(keyword, column_name):
    request = TrendReq(hl='en-US', tz=360)
    kw_list = [keyword]
    request.build_payload(
         kw_list=kw_list,
         cat=0,
         timeframe='today 12-m',
         geo='',
         gprop='')
    df = request.interest_over_time()
    df = df[[keyword]].astype(float)
    df = df.rename(columns={keyword: column_name})
    return df

df_covid_trend = getGoogleTrend('covid', 'US Covid Trend')
print(df_covid_trend)
```

`getGoogleTrend` 함수를 실행한 결과는 다음과 같다.

``` python
            US Covid Trend
date                      
2019-07-28             0.0
2019-08-04             0.0
2019-08-11             0.0
2019-08-18             0.0
2019-08-25             0.0
2019-09-01             0.0
2019-09-08             0.0
2019-09-15             0.0
2019-09-22             0.0
2019-09-29             0.0
2019-10-06             0.0
2019-10-13             0.0
2019-10-20             0.0
2019-10-27             0.0
2019-11-03             0.0
2019-11-10             0.0
2019-11-17             0.0
2019-11-24             0.0
2019-12-01             0.0
2019-12-08             0.0
2019-12-15             0.0
2019-12-22             0.0
2019-12-29             0.0
2020-01-05             0.0
2020-01-12             0.0
2020-01-19             0.0
2020-01-26             0.0
2020-02-02             0.0
2020-02-09             1.0
2020-02-16             1.0
2020-02-23             4.0
2020-03-01             8.0
2020-03-08            31.0
2020-03-15            78.0
2020-03-22           100.0
2020-03-29            93.0
2020-04-05            86.0
2020-04-12            79.0
2020-04-19            71.0
2020-04-26            68.0
2020-05-03            67.0
2020-05-10            66.0
2020-05-17            61.0
2020-05-24            56.0
2020-05-31            51.0
2020-06-07            51.0
2020-06-14            53.0
2020-06-21            59.0
2020-06-28            62.0
2020-07-05            63.0
2020-07-12            67.0
2020-07-19            65.0
```

이를 그래프로 시각화 해보자.

``` python
df_covid_trend.plot()
plt.grid()
plt.show()
```

![Crawling Google Trends US Covid](/assets/img/posts/20200725/crawling_google_trends_us_covid.png)

그리고 `df` 라는 이름으로 두 개의 데이터 프레임을 합친다.

``` python
df = df_covid_new_cases.merge(df_covid_trend, on='date')
print(df)
```

다음은 실행 결과이다.

``` python
            US Covid New Cases  US Covid Trend
date                                          
2020-01-26                 3.0             0.0
2020-02-02                 0.0             0.0
2020-02-09                 0.0             1.0
2020-02-16                 0.0             1.0
2020-02-23                 0.0             4.0
2020-03-01                 6.0             8.0
2020-03-08               147.0            31.0
2020-03-15              1237.0            77.0
2020-03-22              8821.0           100.0
2020-03-29             18251.0            92.0
2020-04-05             26065.0            86.0
2020-04-12             29145.0            78.0
2020-04-19             25995.0            71.0
2020-04-26             29256.0            68.0
2020-05-03             29763.0            64.0
2020-05-10             23792.0            65.0
2020-05-17             13284.0            59.0
2020-05-24             15342.0            55.0
2020-05-31             26177.0            50.0
2020-06-07             17919.0            51.0
2020-06-14             21957.0            52.0
2020-06-21             27616.0            59.0
2020-06-28             41390.0            61.0
2020-07-05             44361.0            62.0
2020-07-12             60469.0            66.0
2020-07-19             63201.0            63.0
```


## VIX

`VIX(Volatility Index)`는 시카고옵션거래소에 상장된 S&P 500 지수옵션의 향후 30일간의 변동성에 대한 시장의 기대를 나타내는 지수로, 증시 지수와는 반대로 움직이는 특징이 있다.

예를 들어, VIX지수가 최고치에 이른다는 것은 투자자들의 불안 심리가 극에 달했다는 것으로 주식시장에서 팔 사람은 모두 팔아 치우게 돼 지수가 반등 여지를 마련했다는 것을 의미한다.

‘공포지수’라고도 불린다.

출처 : [VIX](https://terms.naver.com/entry.nhn?docId=5698336&cid=43659&categoryId=43659)

`Yahoo Finance`를 이용하여 나스닥 지수와 VIX 지수를 얻을 수 있다.

``` python
import yfinance as yf

def GetYahooFinance(name, code):
    ticker = yf.Ticker(code)
    ticker = ticker.history(period='1y')
    ticker = ticker[['Close']]
    ticker.rename(columns={'Close': name}, inplace=True)
    ticker.index.name = 'date'
    return ticker
```

나스닥 지수의 데이터 프레임 생성

``` python
df_nasdaq = GetYahooFinance('Nasdaq', '^DJI')
df_nasdaq = df_nasdaq[df_nasdaq.index >= df.index[0]]
print(df_nasdaq)
```

나스닥 지수 데이터 결과

``` python
[126 rows x 1 columns]
              VIX
date             
2020-01-27  18.23
2020-01-28  16.28
2020-01-29  16.39
2020-01-30  15.49
2020-01-31  18.84
...           ...
2020-07-20  24.46
2020-07-21  24.84
2020-07-22  24.32
2020-07-23  26.08
2020-07-24  25.84
```

`VIX` 지수의 데이터 프레임 생성

``` python
df_vix = GetYahooFinance('VIX', '^VIX')
df_vix = df_vix[df_vix.index >= df.index[0]]
print(df_vix)
```

`VIX` 지수 데이터 결과

``` python
              Nasdaq
date                
2020-01-27  28535.80
2020-01-28  28722.85
2020-01-29  28734.45
2020-01-30  28859.44
2020-01-31  28256.03
...              ...
2020-07-20  26680.87
2020-07-21  26840.40
2020-07-22  27005.84
2020-07-23  26652.33
2020-07-24  26469.89
```

## 모든 데이터의 상관관계

미국 일 신규 확진자 수, 구글 트렌드의 `covid` 검색 결과, `VIX` 지수 데이터, 나스닥 지수 데이터 등을 모든 데이터를 합친다.

데이터 정규화를 하고, 누락된 데이터는 interpolation 을 통해 값을 채웠다.

``` python
df = pd.concat([df_nasdaq['Nasdaq'], df['US Covid New Cases'], df['US Covid Trend'], df_vix['VIX']], axis=1)
df = (df - df.mean()) / df.std()
df = df.interpolate()
print(df)
```

모든 데이터를 합친 결과이다.

``` python
              Nasdaq  US Covid New Cases  US Covid Trend       VIX
date                                                              
2020-01-26       NaN           -1.102948       -1.673008       NaN
2020-01-27  1.344532           -1.102976       -1.673008 -1.117156
2020-01-28  1.420651           -1.103003       -1.673008 -1.249661
2020-01-29  1.425371           -1.103030       -1.673008 -1.242187
2020-01-30  1.476235           -1.103058       -1.673008 -1.303343
...              ...                 ...             ...       ...
2020-07-20  0.589677            2.354868        0.396771 -0.693818
2020-07-21  0.654597            2.354868        0.396771 -0.667996
2020-07-22  0.721922            2.354868        0.396771 -0.703331
2020-07-23  0.578062            2.354868        0.396771 -0.583736
2020-07-24  0.503819            2.354868        0.396771 -0.600044

[152 rows x 4 columns]
```

이들 간의 상관관계를 계산해보자.

``` python
print(df.corr())
```

상관관계는 다음과 같다.

``` python
                      Nasdaq  US Covid New Cases  US Covid Trend       VIX
Nasdaq              1.000000           -0.080119       -0.845421 -0.892276
US Covid New Cases -0.080119            1.000000        0.531151 -0.087459
US Covid Trend     -0.845421            0.531151        1.000000  0.689749
VIX                -0.892276           -0.087459        0.689749  1.000000
```

`Nasdaq` 지수와 `US Covid Trend` 구글 검색량의 관계는 `-0.84`로 강한 상관관계를 갖는다.

`covid` 검색량이 늘어날수록 주가가 하락한다고 볼 수 있다.

`Nasdaq` 지수와 `VIX` 지수 또한 상관관계가 `-0.89`로 검색량보다 더 큰 상관관계로 연관되어 있다.

이 또한, `VIX`로 대표되는 공포 심리가 증가할수록 증시가 하락한다고 볼 수 있다.


## 데이터 시각화

위에서 생성된 데이터프레임을 이용하여 정규화 과정을 거쳐 그래프로 시각화한 결과이다.

``` python
df.plot()
plt.grid()
plt.show()
```

![Fear and Stock](/assets/img/posts/20200725/fear-and-stock.png)

모든 데이터가 하나로 합쳐지면 시각적으로 비교가 잘 되지 않아, 나스닥과 각 지표들간의 비교를 분리해보았다.

![Nasdaq and Covid new cases](/assets/img/posts/20200725/nasdaq_and_covid_new_cases.png)

![Nasdaq and Google Trends](/assets/img/posts/20200725/nasdaq_and_covid_trend.png)

![Nasdaq and VIX](/assets/img/posts/20200725/nasdaq_and_vix.png)

## 결론

`Nasdaq` 지수에 영향을 미치는 요인은 실제 일 신규 확진자 수보다 `VIX` 지표나 `covid` 구글 검색량이 더 큰 상관관계를 가지는 것을 확인할 수 있었다.

만일 제 2차 코로나 팬데믹이 발생한다면, `VIX` 지표 혹은 `covid` 검색량이 코로나 1차 팬데믹 당시의 수치보다 높거나 같아지는 시점이 아닐까 하는 예상을 해본다.

물론 이 예상이 반드시 맞는다고 볼 수는 없지만 적어도 이러한 공포 심리를 대표하는 데이터와의 관계를 통해 증시의 방향성을 가늠해볼 수 있지 않을까?