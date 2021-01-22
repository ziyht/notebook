[toc]

# datetime

## datetime.datetime

```
time.mktime(datetime.timetuple()) # to timestamp
```

**datetime库使用**

**一、操作当前时间**

1.获取当前时间

```
>>> import datetime
>>> print datetime.datetime.now()
2019-07-11 14:24:01.954000
```

时间格式化输出：

```
>>> print datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
2019-07-11 14:25:33
>>> print datetime.datetime.now().strftime("%Y%m%d")
20190711
>>> print datetime.datetime.now().strftime("%Y-%m-%d %H:%M")
2019-07-11 14:25
```

使用**timedelta**方法对**当前时间**进行加减

加 一分钟

```
>>> print (datetime.datetime.now()+datetime.timedelta(minutes=1)).strftime("%Y-%m-%d %H:%M:%S")
2019-07-11 14:29:46
```

减 一分钟

```
>>> print (datetime.datetime.now()+datetime.timedelta(minutes=-1)).strftime("%Y-%m-%d %H:%M:%S")
2019-07-11 14:29:32
```

加 一天

```
>>> print (datetime.datetime.now()+datetime.timedelta(days=1)).strftime("%Y-%m-%d %H:%M:%S")
2019-07-12 14:32:37
```

加 一小时

```
>>> print (datetime.datetime.now()+datetime.timedelta(hours=1)).strftime("%Y-%m-%d %H:%M:%S")
2019-07-11 15:33:37
```

也可以使用**timedelta**方法对**指定时间**进行加减：首先对指定时间进行处理

```
strTime = '2019-07-11 11:03'  # 给定一个时间，此是个字符串
startTime = datetime.datetime.strptime(strTime, "%Y-%m-%d %H:%M")  # 把strTime转化为时间格式,后面的秒位自动补位的
print startTime
print startTime.strftime("%Y-%m-%d %H:%M")  # 格式化输出，保持和给定格式一致
# startTime时间加 一分钟
startTime2 = (startTime + datetime.timedelta(minutes=2)).strftime("%Y-%m-%d %H:%M")
print startTime2
```

输出:

```
2019-07-11 11:03:00
2019-07-11 11:03
2019-07-11 11:05

Process finished with exit code 0
```

 循环加时间

```
startTime = '2019-07-11 23:30:00'  # 输入一个时间，此是个字符串
# endTime = '2019-07-11 15:35'
for i in range(3):
    endTime = (datetime.datetime.strptime(startTime, "%Y-%m-%d %H:%M:%S") + datetime.timedelta(
        days=1)).strftime("%Y-%m-%d %H:%M:%S")
    print startTime,endTime
    startTime = endTime

# 参数days=1（天+1） 可以换成 minutes=1（分钟+1）、seconds=1（秒+1）
```

```
输出：
2019-07-11 23:30:00 2019-07-12 23:30:00
2019-07-12 23:30:00 2019-07-13 23:30:00
2019-07-13 23:30:00 2019-07-14 23:30:00

Process finished with exit code 0
```

**转换为时间戳**

```python
timestamp = (datetime.datetime.now() - datetime.datetime(1970, 1, 1)).total_seconds()
```

