# jsoniter - go
[toc]
## 遍历数组

```go
      //
      //  {"metrics": [
      //    {"time": "", ...}
      //    ...
      //    ]
      //  }
      //

      body := []byte(msg.Body)
			iter := jsoniter.ParseString(jsoniter.ConfigDefault, jsoniter.Get(body, "metrics").ToString()) // 这里应该有优化的空间
			for iter.ReadArray() {
				metric := iter.ReadAny()
				logtime, err := parseDataString(metric.Get("time").ToString(), time.Stamp, "2006-01-02T15:04:05.999")
				if err != nil {
					...
				}
			}
```



## 遍历字典

