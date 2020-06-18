## 获取数据



```go
http.Get("")
```





```go
req, err := http.NewRequest("GET", p.rangePrefix, nil)
if err != nil {
  return nil, err
}

q := req.URL.Query()
q.Add("query", promql)
q.Add("start", start)
q.Add("end", end)
q.Add("step", step)

req.URL.RawQuery = q.Encode()

client := &http.Client{}
return client.Do(req)
```





