## `文档`

[官方文档](https://prometheus.io/docs/introduction/overview/)
[Prometheus操作指南1](https://github.com/yunlzheng/prometheus-book)
[Prometheus操作指南2](https://github.com/1046102779/prometheus)
[配置指南](https://github.com/Alrights/prometheus)

## `基本概念`

可以认为是一个时间序列化数据库，它可以主动从相关的组件中定时拉取数据并存储，为每项数据打上标签，以时间维度存储，并提供相关 API 来查询（支持查询时的聚合和计算）。

![Short-lived jobs  Pushgateway  pull metrics  Jobs / Exporters  Prometheus Server  Service Discovery  • DNS  Kubernetes  • Consul  • Custom integratic  find  targets  Prometheus Server  PagerDuty  Email  notify  Alertmanager  push alerts  PromQL  Retrieval  Node  Storage  Web UI  Grafana  API clients  HDD / SSD ](C:\Users\EDZ\Desktop\notebook\prometheus\prometheus\.asserts\1.文档和概念\clip_image001.png)