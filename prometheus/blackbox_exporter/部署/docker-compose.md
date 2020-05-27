## docker-compose

### 00. 直接使用docker执行

> 先创建配置文件

```sh
docker run --rm -d -p 9115:9115 --name blackbox_exporter -v `pwd`:/config prom/blackbox-exporter:master --config.file=/config/blackbox.yml
```

### 01. 创建项目目录

```sh
mkdir blackbox_exporter
```

### 02. 创建配置文件

> 按需创建配置文件
>
> [如果需要配置ipv6支持，需要配置docker服务](https://docs.docker.com/config/daemon/ipv6/)
>
> 可以参考项目目录中的 blackbox.yml 和 example.yml

**blackbox.yml**

```yaml
modules:
  http_2xx:  # http 监测模块
    prober: http
    http:
  http_post_2xx: # http post 监测模块
    prober: http
    http:
      method: POST
  tcp_connect: # tcp 监测模块
    prober: tcp
  ping: # icmp 检测模块
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"
```

### 03. 创建 docker-compose.yml

**docker-compose.yml**

```yml
version: '2.0'
services:

  blackbox:
    image: prom/blackbox-exporter:master
    container_name: my_blackbox_exporter
    restart: always
    command: "--config.file=/config/blackbox.yml"
    ports:
      - 9115:9115
    volumes:
      - ./:/config
```

### 04. 启动服务

> 可先尝试前台启动，去掉参数 -d

```yml
docker-compose up -d blackbox
```

### 05. 测试

```sh
curl 'http://localhost:9115/probe?target=baidu.com&module=http_2xx'
```

