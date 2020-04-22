---
layout: post
title: "Backtrader 를 이용한 트레이딩 시뮬레이션"
author: "Jonghyun Ho"
categories: Data Analysis
tags: [Stock, KOSPI, Crawling, Python, Backtrader, Simulation]
use_math: true
image: posts/20200423/backtrader_sma_simulation.png
---
<script id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>

트레이딩을 할 때 투자 전략을 정하고 계획한 전략이 효과적으로 잘 동작하는지에 대해 검증하거나, 얼마나 수익률이 발생하는지 확인할 수 있다면 유용할 것이다.

`Backtrader` 를 이용하여 전략을 시뮬레이션 해보자.


## Backtrader

[Backtrader](https://www.backtrader.com/) 는 `Python` 언어 기반의 트레이딩 백테스트 기능을 제공한다.

[Zipline](https://www.zipline.io/index.html) 이라는 백테스트 툴도 존재하지만, 사용 가능한 `Python` 언어의 버전이 최신 버전을 지원하지 않아 `Backtrader` 를 사용하는 것이 적합할 것 같다.

참고 : [Backtrader](https://www.backtrader.com/)


## Backtrader 설치하기

설치는 [Anaconda](https://www.anaconda.com/distribution/) `Windows` 환경의 `Python 3.7` 버전을 사용하였다.

``` bash
> conda create -n stock python=3.7
> conda activate stock
```

`backtrader`를 포함하여 필요한 모듈을 설치한다.

``` bash
(stock) > pip install backtrader requests matplotlib
```


## 이동평균선을 활용한 전략은 유효한가?

[코스피 지수 이동평균선](https://jonghyunho.github.io/data/crawling/%EC%BD%94%EC%8A%A4%ED%94%BC-%EC%A7%80%EC%88%98-%EC%9D%B4%EB%8F%99%ED%8F%89%EA%B7%A0%EC%84%A0.html) 에서 `골든크로스`와 `데드크로스`에 대해 언급하였다.

`골든크로스에 사고 데드크로스에 팔아라` 라는 전략은 얼마나 유효한지 `Backtrader`를 이용해 확인해 보려고 한다.

`backtrader`를 포함하여 필요한 모듈을 임포트 한다.

``` python
from datetime import datetime
import backtrader as bt
import locale

locale.setlocale(locale.LC_ALL, 'ko_KR')
```

`backtrader` 의 `Strategy` 클래스를 상속받아 분석에 필요한 지표와 로직을 구현한다. `5일 이동평균선`과 `30일 이동평균선`을 지표로 사용할 예정이다.

``` python
# Create a subclass of Strategy to define the indicators and logic
class SmaCross(bt.Strategy):
    # list of parameters which are configurable for the strategy
    params = dict(
        pfast=5,  # period for the fast moving average
        pslow=30  # period for the slow moving average
    )
```

클래스 초기화 부분에는 두 개의 `이동평균선`을 이용하여 `CrossOver` 시그널을 만든다. `0`보다 크면 `골든크로스`, `0`보다 작으면 `데드크로스`를 의미한다.

``` python
    def __init__(self):
        sma1 = bt.ind.SMA(period=self.p.pfast)  # fast moving average
        sma2 = bt.ind.SMA(period=self.p.pslow)  # slow moving average
        self.crossover = bt.ind.CrossOver(sma1, sma2)  # crossover signal

        self.holding = 0
```

`next` 함수는 지정된 기간 동안 액션을 취하기 위해 순차적으로 호출되는 함수이다.

현재의 주가를 얻어오고, 매수자의 현금 잔액을 얻어오면 매수 가능한 주식의 수를 알 수 있는데, `available_stocks`에 그 수를 저장하였다.

`buy` 함수를 호출할 때 `available_stocks` 를 인자로 전달하면 전량 매수가 되겠지만 예제에서는 1주씩 매수, 매도하기로 한다.


``` python
    def next(self):
        current_stock_price = self.data.close[0]

        if not self.position:  # not in the market
            if self.crossover > 0:  # if fast crosses slow to the upside
                available_stocks = self.broker.getcash() / current_stock_price
                self.buy(size=1)
```

`데드크로스`의 경우에는 `close` 함수를 호출하여 전량 매도하도록 하였다. `sell` 함수를 사용하면 매도하고자 하는 주식의 수를 지정할 수 있다.

``` python
        elif self.crossover < 0:  # in the market & cross to the downside
            self.close()  # close long position
```

주문이 체결될 때 `notify_order` 함수가 호출되는데, 주문이 발생할 때마다 `매수`, `매도`, `주식 가격`, `보유 현금`, `자산 가치`, `보유 주식의 수` 등의 로그를 출력하도록 하였다.

``` python
    def notify_order(self, order):
        if order.status not in [order.Completed]:
            return

        if order.isbuy():
            action = 'Buy'
        elif order.issell():
            action = 'Sell'

        stock_price = self.data.close[0]
        cash = self.broker.getcash()
        value = self.broker.getvalue()
        self.holding += order.size

        print('%s[%d] holding[%d] price[%d] cash[%.2f] value[%.2f]'
              % (action, abs(order.size), self.holding, stock_price, cash, value))
```

`Cerebro` 엔진을 생성하고, 초기 현금과 수수료를 설정한다. 0.002 는 0.2% 수수료를 설정한 것을 의미한다.

```
cerebro = bt.Cerebro()  # create a "Cerebro" engine instance
cerebro.broker.setcash(100000)
cerebro.broker.setcommission(0.002)
```

[삼성전자](https://finance.yahoo.com/quote/005930.KS) 주가를 사용하고, `Yahoo Finance` 에서 데이터를 얻어오도록 하였다.

```
# Create a data feed
data = bt.feeds.YahooFinanceData(dataname='005930.KS',
                                 fromdate=datetime(2019, 1, 1),
                                 todate=datetime.now())

cerebro.adddata(data)  # Add the data feed

cerebro.addstrategy(SmaCross)  # Add the trading strategy
```

시뮬레이션을 실행하고 결과를 확인한다.

```
start_value = cerebro.broker.getvalue()
cerebro.run()  # run it all
final_value = cerebro.broker.getvalue()

print('* start value : %s won' % locale.format_string('%d', start_value, grouping=True))
print('* final value : %s won' % locale.format_string('%d', final_value, grouping=True))
print('* earning rate : %.2f %%' % ((final_value - start_value) / start_value * 100.0))

cerebro.plot()  # and plot it with a single command
```


## Backtrader 에 문제가 있다.

실행을 해보면 다음과 같이 에러를 발생하며 현재 버전에서 동작하지 않는다. (2020/4/23 기준)

``` python
(stock) > python sma.py
Traceback (most recent call last):
  File "sma.py", line 64, in <module>
    cerebro.run()  # run it all
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\cerebro.py", line 1127, in run
    runstrat = self.runstrategies(iterstrat)
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\cerebro.py", line 1210, in runstrategies
    data._start()
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\feed.py", line 203, in _start
    self.start()
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\feeds\yahoo.py", line 352, in start
    super(YahooFinanceData, self).start()
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\feeds\yahoo.py", line 94, in start
    super(YahooFinanceCSVData, self).start()
  File "C:\Anaconda3\envs\stock\lib\site-packages\backtrader\feed.py", line 674, in start
    self.f = io.open(self.p.dataname, 'r')
FileNotFoundError: [Errno 2] No such file or directory: '005930.KS'
```

[Backtrader Community](https://community.backtrader.com/topic/2363/errno-2-no-such-file-or-directory)의 글에 따르면 `yahoo` API 의 응답이 변경되어 발생하는 것으로 `yahoo.py` 파일의 수정이 필요하다.

``` python
            ctype = resp.headers['Content-Type']
            if 'text/csv' not in ctype:
                self.error = 'Wrong content type: %s' % ctype
                continue  # HTML returned? wrong url?
```

위의 코드를 다음과 같이 수정해 주어야 한다.

``` python
diff --git a/backtrader/feeds/yahoo.py b/backtrader/feeds/yahoo.py
index abfe97d..bd1f6ea 100644
--- a/backtrader/feeds/yahoo.py
+++ b/backtrader/feeds/yahoo.py
@@ -330,7 +330,7 @@ class YahooFinanceData(YahooFinanceCSVData):
                 continue

             ctype = resp.headers['Content-Type']
-            if 'text/csv' not in ctype:
+            if ctype not in ['text/csv', 'text/plain']:
                 self.error = 'Wrong content type: %s' % ctype
                 continue  # HTML returned? wrong url?
```

## 전체 코드

위의 코드 조각의 전체 코드는 다음과 같다.

``` python
from datetime import datetime
import backtrader as bt
import locale

locale.setlocale(locale.LC_ALL, 'ko_KR')

# Create a subclass of Strategy to define the indicators and logic
class SmaCross(bt.Strategy):
    # list of parameters which are configurable for the strategy
    params = dict(
        pfast=5,  # period for the fast moving average
        pslow=30  # period for the slow moving average
    )

    def __init__(self):
        sma1 = bt.ind.SMA(period=self.p.pfast)  # fast moving average
        sma2 = bt.ind.SMA(period=self.p.pslow)  # slow moving average
        self.crossover = bt.ind.CrossOver(sma1, sma2)  # crossover signal

        self.holding = 0

    def next(self):
        current_stock_price = self.data.close[0]

        if not self.position:  # not in the market
            if self.crossover > 0:  # if fast crosses slow to the upside
                available_stocks = self.broker.getcash() / current_stock_price
                self.buy(size=1)

        elif self.crossover < 0:  # in the market & cross to the downside
            self.close()  # close long position

    def notify_order(self, order):
        if order.status not in [order.Completed]:
            return

        if order.isbuy():
            action = 'Buy'
        elif order.issell():
            action = 'Sell'

        stock_price = self.data.close[0]
        cash = self.broker.getcash()
        value = self.broker.getvalue()
        self.holding += order.size

        print('%s[%d] holding[%d] price[%d] cash[%.2f] value[%.2f]'
              % (action, abs(order.size), self.holding, stock_price, cash, value))

cerebro = bt.Cerebro()  # create a "Cerebro" engine instance
cerebro.broker.setcash(100000)
cerebro.broker.setcommission(0.002)

# Create a data feed
data = bt.feeds.YahooFinanceData(dataname='005930.KS',
                                 fromdate=datetime(2019, 1, 1),
                                 todate=datetime.now())

cerebro.adddata(data)  # Add the data feed

cerebro.addstrategy(SmaCross)  # Add the trading strategy

start_value = cerebro.broker.getvalue()
cerebro.run()  # run it all
final_value = cerebro.broker.getvalue()

print('* start value : %s won' % locale.format_string('%d', start_value, grouping=True))
print('* final value : %s won' % locale.format_string('%d', final_value, grouping=True))
print('* earning rate : %.2f %%' % ((final_value - start_value) / start_value * 100.0))

cerebro.plot()  # and plot it with a single command
```

## 시뮬레이션 결과 확인

10만원을 설정하여 시뮬레이션을 해 보았지만, 여러 차례 매수와 매도를 거듭 했음에도 불구하고 수익률은 0.25% 에 불과했다.

``` bash
(stock) > python sma.py
Buy[1] holding[1] price[45350] cash[55160.50] value[100510.50]
Sell[1] holding[0] price[45050] cash[100270.10] value[100270.10]
Buy[1] holding[1] price[46950] cash[54027.80] value[100977.80]
Sell[1] holding[0] price[44650] cash[98189.30] value[98189.30]
Buy[1] holding[1] price[44800] cash[53800.70] value[98600.70]
Sell[1] holding[0] price[44950] cash[98261.60] value[98261.60]
Buy[1] holding[1] price[46900] cash[51718.70] value[98618.70]
Sell[1] holding[0] price[51300] cash[103514.90] value[103514.90]
Buy[1] holding[1] price[50300] cash[52212.50] value[102512.50]
Sell[1] holding[0] price[50400] cash[103010.70] value[103010.70]
Buy[1] holding[1] price[54700] cash[48401.70] value[103101.70]
Sell[1] holding[0] price[58900] cash[105387.50] value[105387.50]
Buy[1] holding[1] price[60400] cash[44165.30] value[104565.30]
Sell[1] holding[0] price[57900] cash[100252.90] value[100252.90]
* start value : 100,000 won
* final value : 100,252 won
* earning rate : 0.25 %
```

아래 그래프를 통해 거래 상황을 좀 더 명확히 확인할 수 있다.

![Backtrader SMA Test](/assets/img/posts/20200423/backtrader_sma_simulation.png)

`이동평균선` 전략이 잘 맞는 상황도 있겠지만 현재 상황에서는 그다지 효율적이지 않은 것을 `Backtrader` 를 통해 확인할 수 있었다. `Backtrader`를 이용하여 다양한 전략을 구상해보고 적용해보는 재미가 있을 것 같다. 또한, 유용하고 다양한 기능을 제공하는 만큼 `backtrader` 의 오픈소스가 잘 관리되기를 기대해본다.