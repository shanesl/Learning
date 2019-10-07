# 行情数据

## 学习目标

- 目标
  - 说明股票的K线图结构
  - 了解什么是除权数据
- 应用
  - 应用resample实现交易数据重采样

回到我们刚才说的行情数据，最常用的是交易数据

## 1、交易数据

股票在流通市场上的价格，才是完全意义上的股票的市场价格，一般称为股票市价或股票行市。股票市价表现为开盘价、收盘价、最高价、最低价等形式。其中收盘价最重要，是分析股市行情时采用的基本数据。



![K线图](/images/K线图.png)

## 2、股票K线图

K线图这种图表源处于日本德川幕府时代，被当时日本米市的商人用来记录米市的行情与价格波动，后因其细腻独到的标画方式而被引入到股市及期货市场。

### 2.1K线图基本形态

![K线图基本形态](/images/K线图基本形态.png) 

### 2.2K线图的计算周期

### 不同的时间间隔看待股票数据变化，会有不一样的发现！

**K线的计算周期可将其分为日K线，周K线，月K线，年K线**

很多网站提供了日线、周K线、月K线等周期数据，但是最原始的只有日K线的数据。我们需要自己去生成计算不同频率的数据

## 3、案例：股票K线数据重采样

* DataFrame.resample(rule, how=None, axis=0, fill_method=None, closed=None,kind=None,)
  * 频率转换和时间序列重采样，对象必须具有类似日期时间的索引（DatetimeIndex，PeriodIndex或TimedeltaIndex）

|        参数        |                             作用                             |
| :----------------: | :----------------------------------------------------------: |
|        rule        | 表示重采样频率，例如周’W’,月’M’,季度’Q’,5分钟’5min’,12天’12D’ |
|        how         | **用于产生聚合值的函数名或数组函数，例如‘mean’、‘ohlc’、np.max等，默认是‘mean’，其他常用的值由：‘first’、‘last’、‘median’、‘max’、‘min’** |
|       axis=0       |                  默认是纵轴，横轴设置axis=1                  |
|  closed = ‘right’  | 在降采样时，各时间段的哪一段是闭合的，‘right’或‘left’，默认‘right’ |
| fill_method = None |           升采样时如何插值，比如‘ffill’、‘bfill’等           |
|    kind = None     | 聚合到时期（‘period’）或时间戳（‘timestamp’），默认聚合到时间序列的索引类型 |

- 日K周K对比：

![日周K线对比](/images/日周K线对比.png)

**那么日线、周线、月线等怎么切换标准？？**

>  注：周K线是指以周一的开盘价，周五的收盘价，全周最高价和全周最低价来画的K线图
>
> 大部分周线的指标是这个日线指标在这一周最后一个交易日的值。比如周线的’close’应该等于这一周最后一天日线数据的‘close’,但是有的指标是例外，比如周线的’high’应该等于这一周所有日线‘high’中的最大值



接下来我们还是使用之前stock_day当中的某个股票的行情数据

### 3.1 分析

* 将索引转换成DatetimeIndex类型
* 对不同指标进行重采样

```python
stock_day = pd.read_csv("./data/stock_day/stock_day.csv")
stock_day = stock_day.sort_index()
# 对每日交易数据进行重采样 （频率转换）
stock_day.index


# 1、必须将时间索引类型编程Pandas默认的类型
stock_day.index = pd.to_datetime(stock_day.index)

# 2、进行频率转换日K---周K,首先让所有指标都为最后一天的价格
period_week_data = stock_day.resample('W').last()

# 分别对于开盘、收盘、最高价、最低价进行处理
period_week_data['open'] = stock_day['open'].resample('W').first()
# 处理最高价和最低价
period_week_data['high'] = stock_day['high'].resample('W').max()
# 最低价
period_week_data['low'] = stock_day['low'].resample('W').min()
# 成交量 这一周的每天成交量的和
period_week_data['volume'] = stock_day['volume'].resample('W').sum()
```

* 对于其中存在的缺失值

![K线缺失值](/images/K线缺失值.png)

```python
period_week_data.dropna(axis=0)
```

我们可以将计算出来的周K和原先的日K画图显示出来

* 画出K线图显示

```python
from matplotlib.finance import candlestick_ochl
import matplotlib.pyplot as plt

# 先画日K线
fig, axes = plt.subplots(nrows=1, ncols=1, figsize=(20, 8), dpi=80)
# 准备数据， array数组
stock_day['index'] = [i for i in range(stock_day.shape[0])]
day_k = stock_day[['index', 'open', 'close', 'high', 'low']]
candlestick_ochl(axes, day_k.values, width=0.2, colorup='r', colordown='g')
plt.show()


# 周K线图数据显示出来
period_week_data['index'] = [i for i in range(period_week_data.shape[0])]
week_k = period_week_data[['index', 'open', 'close', 'high', 'low']]
candlestick_ochl(axes, week_k.values, width=0.2, colorup='r', colordown='g')

plt.show()
```

## 4、什么是除权数据以及复权操作

上市公司**会时不时的发生现金分红、送股等一系列股本变动，这会造成股价的非正常变化，**导致我们不能直接通过股价来计算股票的涨跌幅。这种数据我们也称之为除权数据。

![除权数据](/images/除权数据.png)

所以我们要对这种数据做处理，也称之为复权数据。怎么进行复权呢？

```
简单的一种方式：
原始数据：
1号：100  2号：50 3号：53 4号：51
复权后：
100 / 50 = 2 比例
1号：100  2号：100 3号：106 4号：102
```

