# Pandas时间类型序列

## 学习目标

- 目标

  - 了解Pandas的时间类型
  - 应用pd.to_datetime和pd.DatetimeIndex实现时间类型转换
  - 应用pandas.tseries.offsets实现时间的偏移
  - 应用DatetimeIndex.shift实现时间的偏移
  - 应用pd.date_range实现时间序列数据的生成

- 应用

  - 无


前面其实我们已经使用一些时间序列接口，回忆我们在股票涨跌幅使用的生成一个时间列表，接下来我们就去看看Pandas当中的时间类型等。

## 1、datetime模块

```python
datetime(2018, 3, 2) # datetime的类型
```

## 2、Pandas的时间类型

* pd.to_datetime()：转换成pandas的时间类型 Timestamp('2018-03-02 00:00:00')

```python
# pd将时间数据转换成pandas时间类型
# 1、填入时间的字符串，格式有几种, "2018-01-01" ，”01/02/2018“
pd.to_datetime("01/02/2017")

# 2、传入datetime的时间格式
pd.to_datetime(datetime(2018, 3, 2))
```

如果我们传入的是多个时间点，那么会是什么样的？

## 3、Pandas的时间序列类型

* 1、转换时间序列类型

```python
# 传入时间的列表
pd.to_datetime(["2017-01-01", "2017-02-01", "2017-03-01"])

# 或者
date = [datetime(2018, 3, 1), datetime(2018, 3, 2), datetime(2018, 3, 3), datetime(2018, 3, 4), datetime(2018, 3, 5)]
date = pd.to_datetime(date)

# 如果其中有空值
date = [datetime(2018, 3, 1), datetime(2018, 3, 2), np.nan, datetime(2018, 3, 4), datetime(2018, 3, 5)]
date = pd.to_datetime(date)
# 结果会变成NaT类型
DatetimeIndex(['2018-03-01', '2018-03-02', 'NaT', '2018-03-04', '2018-03-05'], dtype='datetime64[ns]', freq=None)
```

* 2、Pandas的时间序列类型：DatetimeIndex

```python
# DateTimeIndex
pd.to_datetime(date)
DatetimeIndex(['2018-03-01', '2018-03-02', '2018-03-03', '2018-03-04',
               '2018-03-05'],
              dtype='datetime64[ns]', freq=None)

pd.to_datetime(date).values

array(['2018-03-01T00:00:00.000000000', '2018-03-02T00:00:00.000000000',
       '2018-03-03T00:00:00.000000000', '2018-03-04T00:00:00.000000000',
       '2018-03-05T00:00:00.000000000'], dtype='datetime64[ns]')
```

其中可以看到一种**datetime64的类型，这个是numpy的一种时间类型**。

我们也可以通过DatetimeIndex来转换

* 3、通过pd.DatetimeIndex进行转换

```python
pd.DatetimeIndex(date)
```

#### 知道了时间序列类型，所以我们可以用这个当做索引，获取数据

## 4、Pandas的基础时间序列结构

```python
# 最基础的pandas的时间序列结构,以时间为索引的，Series序列结构
# 以时间为索引的DataFrame结构
series_date = pd.Series(3.0, index=date)

# 以下两种方法的结果会有区别
pd.to_datetime(series_date)
pd.DatetimeIndex(series_date)
```

pandas时间序列series的index必须是DatetimeIndex

* **DatetimeIndex的属性**
  * year,month,weekday,day,hour….

```python
time.year
time.month
time.weekday
```

## 5、时间的偏移

* frompandas.tseries.offsets import Hour, Minute,Day, MonthEnd…

```
# 计算总分钟
# pandas支持不同时间单位的运算，包括时间一些偏移
Hour(5) + Minute(30)
```

* DatetimeIndex.shift(n, freq=)
  * n:偏移的大小
  * freq:偏移的频率
  * Returns:DatetimeIndex

```
# 针对时间序列进行统一的运算
# shift相当于左右偏移的函数，时间向前，向后偏移
# 30天为一个单位，偏移3次
date.shift(3, freq="30D")
```

## 6、Pandas生成指定频率的时间序列

* pandas.date_range(start=None, end=None, periods=None, freq='D', tz=None, normalize=False, name=None, closed=None, **kwargs)
  * Returna fixed frequency DatetimeIndex, with day (calendar) as the default frequency
  * start:开始时间
  * end:结束时间
  * periods:产生多长的序列
  * freq:频率 D,H,Q等
  * tz:时区

|        参数         |        含义        |
| :-----------------: | :----------------: |
|          D          |        每日        |
|          B          |      每工作日      |
|   H、T或者min、S    |     时、分、秒     |
|          M          |    每月最后一天    |
|         BM          | 每月最后一个工作日 |
| WOM-1MON,  WOM-3FRI | 每月第几周的星期几 |

```
# 生成指定的时间序列
# 1、生成2017-01-02~2017-12-30，生成频率为1天, 不跳过周六周日
pd.date_range("2017-01-02", "2017-12-30", freq="D")

# 2、生成2017-01-02~2017-12-30，生成频率为1天, 跳过周六周日, 能够用在金融的数据，日线的数据
pd.date_range("2017-01-02", "2017-12-30", freq="B")

# 3、只知道开始时间日期，我也知道总共天数多少，生成序列， 从"2016-01-01", 共504天，跳过周末
pd.date_range("2016-01-01", periods=504, freq="B")

# 4、生成按照小时排列的时间序列数据
pd.date_range("2017-01-02", "2017-12-30", freq='H')

# 5、按照3H去进行生成
pd.date_range("2017-01-02", "2017-12-30", freq='3H')

# 6、按照1H30分钟去进行生成时间序列
pd.date_range("2017-01-02", "2017-12-30", freq='1H30min')

# 7、按照每月最后一天
pd.date_range("2017-01-02", "2017-12-30", freq='BM')

# 8、按照每个月的第几个星期几
pd.date_range("2017-01-02", "2017-12-30", freq='WOM-3FRI')
```