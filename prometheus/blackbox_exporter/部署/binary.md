## binary

### 从以下地址下载对应版本

https://github.com/prometheus/blackbox_exporter/releases

### 解压后直接运行

```sh
blackbox_exporter --web.listen-address=:9115 --config.file=blackbox.yml
```

一般情况下都会以非root用户运行`blackbox_exporter`，这里使用的prometheus用户，Wie了使用icmp prober，需要设置`CAP_NET_RAW`，即对可执行文件`blackbox_exporter`执行下面的命令：

```sh
setcap cap_net_raw+ep blackbox_exporter
```

---

\>>> **`配置文件`和`后续步骤`请参考 `docker-compose`** 

