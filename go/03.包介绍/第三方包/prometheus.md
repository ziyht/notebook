```go
package metrics

import (
	"github.com/gogf/gf/net/ghttp"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"qos_query_service/app/model/metrics"
)

// Metrics is to get prometheus metrics of running stats for this app
func Metrics(r *ghttp.Request) {
	promhttp.HandlerFor(metrics.Registry, promhttp.HandlerOpts{}).ServeHTTP(r.Response.ResponseWriter, r.Request)
}
```



```go
package metrics

import (
	"github.com/prometheus/client_golang/prometheus"
)

var Registry  *prometheus.Registry

func init(){
	Registry = prometheus.NewRegistry()
	Registry.Register(prometheus.NewProcessCollector(prometheus.ProcessCollectorOpts{}))
	Registry.Register(prometheus.NewGoCollector())
}
```

