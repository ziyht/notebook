### webui

直接在浏览器中使用 `ip:port` 加载页面即可，如：

 http://36.103.238.142:9090

然后输出需要的 metric 进行查询，如：

```
http_requests_total
http_requests_total{code="200"}
count(http_requests_total)
```

