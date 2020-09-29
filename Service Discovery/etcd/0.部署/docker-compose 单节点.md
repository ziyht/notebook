## docker-compose 单节点部署etcd

### 1. 创建项目目录

```sh
mkdir etcd
cd etcd
```

### 2. 创建 docker-compose.yml

```sh
vim docker-compose.yml
```

```yaml
version: '2'

networks:
  etcd:
    driver: bridge

services:
  etcd1:
    container_name: qos_etcd1
    image         : bitnami/etcd:3.4.9
    environment:
      - ALLOW_NONE_AUTHENTICATION=yes
      - ETCD_ADVERTISE_CLIENT_URLS=http://etcd1:2379
      - ETCD_NAME=etcd1
      - ETCD_DATA_DIR=/var/lib/etcd
    volumes:
      - ./data/etcd1:/var/lib/etcd
    ports:
      - 2379:2379
      - 2380:2380
    networks:
      - etcd
```

相关的配置项查看：https://etcd.io/docs/v3.4.0/op-guide/configuration/

### 3. 创建数据目录

```sh
mkdir -p data/etcd1
chmod 777 data/etcd1
```

### 4. 测试启动etcd

```sh
docker-compose up
```

```
...
etcd1_1_9568ceb7e04a | raft2020/07/07 06:58:31 INFO: raft.node: 8e9e05c52164694d elected leader 8e9e05c52164694d at term 3
etcd1_1_9568ceb7e04a | 2020-07-07 06:58:31.211613 I | etcdserver: published {Name:etcd1 ClientURLs:[http://etcd1:2379]} to cluster cdf818194e3a8c32
etcd1_1_9568ceb7e04a | 2020-07-07 06:58:31.211655 I | embed: ready to serve client requests
etcd1_1_9568ceb7e04a | 2020-07-07 06:58:33.377666 N | embed: serving insecure client requests on [::]:2379, this is strongly discouraged!
```

### 5. 尝试 put 和 get

**创建脚本**

```sh
touch etcdctl
chmod +x etcdctl
vim etcdctl
```

```sh
#!/bin/bash

docker exec qos_etcd1 etcdctl $*
```

**put**

```sh
[qos@rndstorage etcd]$ ./etcdctl put key1 value1
OK
[qos@rndstorage etcd]$ ./etcdctl put key2 value2
OK
[qos@rndstorage etcd]$ ./etcdctl put keys/key1 value1
OK
[qos@rndstorage etcd]$ ./etcdctl put keys/key2 value2
OK
```

**get**

```sh
[qos@rndstorage etcd]$ ./etcdctl get key1
key1
value1
[qos@rndstorage etcd]$ ./etcdctl get key2
key2
value2
[qos@rndstorage etcd]$ ./etcdctl get keys --prefix
keys/key1
value1
keys/key2
value2
[qos@rndstorage etcd]$ ./etcdctl get keys/key1
keys/key1
value1
[qos@rndstorage etcd]$ ./etcdctl get keys/key2
keys/key2
value2
```

**del**

```sh
[qos@rndstorage etcd]$ ./etcdctl del key1
1
[qos@rndstorage etcd]$ ./etcdctl del key2
1
[qos@rndstorage etcd]$ ./etcdctl del keys --prefix
2
```

### 6. 启动etcd

```sh
docker-compose up -d
```

