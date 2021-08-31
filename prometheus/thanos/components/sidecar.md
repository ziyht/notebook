## Upload compacted blocks [#](https://thanos.io/tip/components/sidecar.md/#upload-compacted-blocks)

If you want to migrate from a pure Prometheus setup to Thanos and have to keep the historical data, you can use the flag `--shipper.upload-compacted`. This will also upload blocks that were compacted by Prometheus. Values greater than 1 in the `compaction.level` field of a Prometheus block’s `meta.json` file indicate level of compaction.

To use this, the Prometheus compaction needs to be disabled. This can be done by setting the following flags for Prometheus:

- `--storage.tsdb.min-block-duration=2h`
- `--storage.tsdb.max-block-duration=2h`

如果您想从纯 Prometheus 迁移到 Thanos 并且需要保存历史数据，您可以使用标志 `--shipper.upload-compacted`。这也将上传由 Prometheus 压缩的块。Prometheus 块描述文件 `meta.json` 的字段 `compaction.level` 大于 1 表示压缩级别。

要使用它，需要禁用 Prometheus 压缩。这可以通过为 Prometheus 设置以下标志来完成：

- `--storage.tsdb.min-block-duration=2h`
- `--storage.tsdb.max-block-duration=2h`

## Flags 

```$
usage: thanos sidecar [<flags>]

Sidecar for Prometheus server.

Flags:
      --grpc-address="0.0.0.0:10901"  
                                 gRPC的监听 ip:port (StoreAPI). 确保当前地址可被其它组件路由.
      --grpc-grace-period=2m     Time to wait after an interrupt received for
                                 GRPC Server.
      --grpc-server-tls-cert=""  TLS Certificate
      --grpc-server-tls-client-ca=""  
      --grpc-server-tls-key=""   TLS Key 
      --hash-func=               Specify which hash function to use when
                                 calculating the hashes of produced files. If no
                                 function has been specified, it does not
                                 happen. This permits avoiding downloading some
                                 files twice albeit at some performance cost.
                                 Possible values are: "", "SHA256".
  -h, --help                     显示帮助 (也可以尝试 --help-long 和 --help-man).
      --http-address="0.0.0.0:10902"   HTTP 监听 host:port
      --http-grace-period=2m     Time to wait after an interrupt received for
                                 HTTP Server.
      --http.config=""           [实验] Path to the configuration file
                                 that can enable TLS or authentication for all
                                 HTTP endpoints.
      --log.format=logfmt        日志格式. 可设置为 logfmt 或 json.
      --log.level=info           日志等级.
      --min-time=0000-01-01T00:00:00Z  
                                 服务的开始时间限制，Thanos sidecar 将只发送大于此时刻的metrics，此项配置格式可以设置为 
                                 RFC3339 格式，或者相对于当前时间的区间如：-1d 或 2h45m，合法的单位有 ms, s, m, h, d, w, y
      --objstore.config=<content>  
                                 同 --objstore.config-file（两选一）, yaml格式的远程对象存储配置
                                 详细的配置格式可以查阅：
                                   https://thanos.io/tip/thanos/storage.md/#configuration
      --objstore.config-file=<file-path>  
                                 远程对象存储配置的路径, YAML格式
                                 详细的配置格式可以查阅：
                                   https://thanos.io/tip/thanos/storage.md/#configuration
      --prometheus.ready_timeout=10m
                                 等待 prometheus 正常提供服务的最大时间
      --prometheus.url=http://localhost:9090
                                 需要关联的Prometheus提供服务的 URL，最好使用本地网络（部署在同一个节点上）
      --receive.connection-pool-size=RECEIVE.CONNECTION-POOL-SIZE
                                 控制最大连接数，默认为0，不做限制
      --receive.connection-pool-size-per-host=100
                                 控制每个host的最大连接数
      --reloader.config-envsubst-file=""
                                 外部环境配置文件，来覆盖配置文件
                                 Output file for environment variable
                                 substituted config file.
      --reloader.config-file=""  被reloader监听的配置文件
      --reloader.retry-interval=5s
                                 如果发生错误，reloader的检查间隔
   
      --reloader.rule-dir=RELOADER.RULE-DIR ...
                                 需要被reloader监控的规则文件目录(可配置多个)
      --reloader.watch-interval=3m
                                 reloader 重新加载 config 和 rules 的间隔
      --request.logging-config=<content>  
                                 Alternative to 'request.logging-config-file'
                                 flag (mutually exclusive). Content of YAML file
                                 with request logging configuration. 
                                 详情请查看:
                                 https://gist.github.com/yashrsharma44/02f5765c5710dd09ce5d14e854f22825
      --request.logging-config-file=<file-path>  
                                 带有请求日志记录配置的YAML文件的路径
                                 详情请查看:
                                 https://gist.github.com/yashrsharma44/02f5765c5710dd09ce5d14e854f22825
      --shipper.upload-compacted 
                                 如果设置为 true，shipper 将会尝试发送已被 prometheus 压缩的 blocks, 
                                 迁移时，这将非常有用。它只在 prometheus 的 compaction 关闭的时候生效，
                                 在迁移完毕后请及时关闭这个配置
      --tracing.config=<content>  
                                 Alternative to 'tracing.config-file' flag
                                 (mutually exclusive). Content of YAML file with
                                 tracing configuration. See format details:
                                 https://thanos.io/tip/thanos/tracing.md/#configuration
      --tracing.config-file=<file-path>  
                                 Path to YAML file with tracing configuration.
                                 See format details:
                                 https://thanos.io/tip/thanos/tracing.md/#configuration
      --tsdb.path="./data"       需要关联的 prometheus 的 data 目录.
      --version                  Show application version.
```

