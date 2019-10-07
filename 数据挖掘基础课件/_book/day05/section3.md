# 股票时间序列数据处理

## 学习目标

- 目标
  - 了解时间序列数据(股票)的分析方法
  - 了解移动窗口的定义
  - 知道移动平均线的作用
  - 知道移动平均线按照周期和算法划分的类别
  - 记忆sma的计算公式
  - 应用pd.rolling_mean实现K线图的均线图绘制
  - 知道wma和ewma的计算方式
  - 应用pd.ewma实现指数平滑移动平均线绘制
- 应用
  - 股票的日线数据移动平均线观察

对于时间序列类型，有特有的分析方法。同样股票本身也是一种时间序列类型，我们就以股票的数据来进行时间序列的分析

## 1、什么是时间序列分析

时间序列分析( time series analysis)方法,强调的是通过对一个区域进行一定时间段内的连续观察计算，提取相关特征，并分析其变化过程。

**时间序列分析主要有确定性变化分析和随机性变化分析**

* **确定性变化分析：移动平均法， 移动方差和标准差、移动相关系数**
* 随机性变化分析：AR、ARMA模型

## 2、移动平均法

### 2.1 移动窗口

主要用在时间序列的数组变换， 不同作用的函数将它们统称为移动窗口函数

![移动窗口](/images/移动窗口.png)

### 2.2 移动平均线

那么会有各种观察窗口的方法，其中最常用的就是移动平均法

* 移动平均线(Moving Average)简称均线, **将某一段时间的收盘价之和除以该周期**

![移动平均方法](/images/移动平均方法.png)



#### 2.2.1移动平均线的分类 

* 移动平均线依计算周期分为短期(5天)、中期(20天)和长期(60天、120天)，移动平均线没有固定的界限
* 移动平均线依据算法分为算数、加权法和指数移动平均线

> 注：不同的移动平均线方法不一样

### 2.3 简单移动平均线

**简单移动平均线(SMA)，又称“算数移动平均线”，是指特定期间的收盘价进行平均化比如说，5日的均线SMA=(C1+ C2 + C3 + C4 + C5) / 5**

![公式](/images/公式.png)

例子：

![简单移动平均例子](/images/简单移动平均例子.png)

### 案例：对股票数据进行移动平均计算

![简单移动平均线效果](/images/简单移动平均线效果.png)

#### 1、拿到股票数据，画出K线图

```python
# 拿到股票K线数据
stock_day = pd.read_csv("./data/stock_day/stock_day.csv")
stock_day = stock_day.sort_index()
stock_day["index"] = [i for i in range(stock_day.shape[0])]
arr = stock_day[['index', 'open', 'close', 'high', 'low']]
values = arr.values[:200]
# 画出K线图
fig, axes = plt.subplots(nrows=1, ncols=1, figsize=(20, 8), dpi=80)
candlestick_ochl(axes, values, width=0.2, colorup='r', colordown='g')
```

#### 2、计算移动平均线

* pandas.rolling_mean(arg, window, min_periods=None, freq=None, center=False, how=None, **kwargs)
  Moving mean.

  Parameters:	

  * arg : Series, DataFrame
  * window : 计算周期

```python
# 直接对每天的收盘价进行求平均值， 简单移动平局线(SMA)
# 分别加上短期、中期、长期局均线
pd.rolling_mean(stock_day["close"][:200], window=5).plot()
pd.rolling_mean(stock_day["close"][:200], window=10).plot()
pd.rolling_mean(stock_day["close"][:200], window=20).plot()
pd.rolling_mean(stock_day["close"][:200], window=30).plot()
pd.rolling_mean(stock_day["close"][:200], window=60).plot()
pd.rolling_mean(stock_day["close"][:200], window=120).plot()
```

### 2.3 加权移动平均线 (WMA) 

加权移动平均线 (WMA)将过去某特定时间内的价格取其平均值，**它的比重以平均线的长度设定，愈近期的收市价，对市况影响愈重要。**

![加权移动平均公式](/images/加权移动平均公式.png)



正因加权移动平均线强调将愈近期的价格比重提升，故此当市况倒退时，加权移动平均线比起其它平均线更容易预测价格波动。但是我们还是不会轻易使用加权，应为他的比重过大！！！！

### 2.4 指数平滑移动平均线(EWMA)

是因应移动平均线被视为落后指标的缺失而发展出来的，为解决一旦价格已脱离均线差值扩大，而平均线未能立即反应，EXPMA可以减少类似缺点。

![指数平滑移动平均线公式](/images/指数平滑移动平均线公式.png)

总结：

![指数平均线公式总结](/images/指数平均线公式总结.png)

* pd.ewma(com=None,   span=one)
  * 指数平均线
  * span:时间间隔

```python
# 画出指数平滑移动平均线
pd.ewma(stock_day['close'][:200], span=10).plot()
pd.ewma(stock_day['close'][:200], span=30).plot()
pd.ewma(stock_day['close'][:200], span=60).plot()
```

## 3、移动方差和标准差 和 移动相关系数

* 方差和标准差：反应某一时期的序列的稳定性

![移动方差图](/images/移动方差图.png)

```python
# 求出指定窗口大小的收盘价标准差和方差
pd.rolling_var(stock_day['close'][:200], window=10).plot()
pd.rolling_std(stock_day['close'][:200], window=10).plot()
```

* 移动相关系数：有些统计运算(如相关系数和协方差)需要在两个时间序列上执行. 

> 相关系数：后面会介绍，目前我们只需知道他是反应两个序列之间的关系即可

![移动相关系数](/images/移动相关系数.png)

```python
# 显示两个序列之间的关系（相关系数和协方差）
pd.rolling_corr(stock_day["close"][:200], stock_day["volume"][:200], window=10).plot()
```

## 案例：移动平均线数据本地保存

