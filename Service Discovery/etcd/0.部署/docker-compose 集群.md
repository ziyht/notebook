[TOC]

### docker-compose 单节点部署etcd集群

#### 1. 创建项目目录

```sh
mkdir etcd_cluster
cd etcd_cluster
```

#### 2. 创建 docker-compose.yml 文件

```yml
version: '2'

services:
  c_etcd1:
    image: quay.io/coreos/etcd
    container_name: etcd1
    # command: etcd -name etcd2 -advertise-client-urls http://0.0.0.0:2379 -listen-client-urls http://0.0.0.0:2379 -listen-peer-urls http://0.0.0.0:2380 -initial-cluster-token etcd-cluster -initial-cluster "etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380" -initial-cluster-state new
    environment:
      - ETCD_NAME=etcd1
      - ETCD_ADVERTISE_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_DATA_DIR=/var/lib/etcd
    volumes:
      - ./data/c_etcd1:/var/lib/etcd   # 注意目录
    ports:
      - 2379
      - 2380
    networks:
      - byfn

  c_etcd2:
    image: quay.io/coreos/etcd
    container_name: etcd2
    environment:
      - ETCD_NAME=etcd2
      - ETCD_ADVERTISE_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_DATA_DIR=/var/lib/etcd
    ports:
      - 2379
      - 2380
    volumes:
      - ./data/c_etcd2:/var/lib/etcd
    networks:
      - byfn

  c_etcd3:
    image: quay.io/coreos/etcd
    container_name: etcd3
    environment:
      - ETCD_NAME=etcd3
      - ETCD_ADVERTISE_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379
      - ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380
      - ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster
      - ETCD_INITIAL_CLUSTER=etcd1=http://etcd1:2380,etcd2=http://etcd2:2380,etcd3=http://etcd3:2380
      - ETCD_INITIAL_CLUSTER_STATE=new
      - ETCD_DATA_DIR=/var/lib/etcd
    ports:
      - 2379
      - 2380
    volumes:
      - ./data/c_etcd3:/var/lib/etcd
    networks:
      - byfn
   
networks:
  byfn:
```

> 如果再多节点上使用docker-compose部署，则需要注意 ip 地址的 expose 和 编写

参数说明：

- `--name`：节点名称，默认为 default。
- `--data-dir`：服务运行数据保存的路径，默认为`${name}.etcd`。
- `--snapshot-count`：指定有多少事务（transaction）被提交时，触发截取快照保存到磁盘。
- `--heartbeat-interval`：leader 多久发送一次心跳到 followers。默认值是 100ms。
- `--eletion-timeout`：重新投票的超时时间，如果 follow 在该时间间隔没有收到心跳包，会触发重新投票，默认为 1000 ms。
- `--listen-peer-urls`：和同伴通信的地址，比如`http://ip:2380`，如果有多个，使用逗号分隔。需要所有节点都能够访问，所以不要使用 localhost！
- `--listen-client-urls`：对外提供服务的地址：比如`http://ip:2379,http://127.0.0.1:2379`，客户端会连接到这里和 etcd 交互。
- `--advertise-client-urls`：对外公告的该节点客户端监听地址，这个值会告诉集群中其他节点。
- `--initial-advertise-peer-urls`：该节点同伴监听地址，这个值会告诉集群中其他节点。
- `--initial-cluster`：集群中所有节点的信息，格式为`node1=http://ip1:2380,node2=http://ip2:2380,…`，注意：这里的 node1 是节点的 --name 指定的名字；后面的 ip1:2380 是 --initial-advertise-peer-urls 指定的值。
- `--initial-cluster-state`：新建集群的时候，这个值为 new；假如已经存在的集群，这个值为 existing。
- `--initial-cluster-token`：创建集群的 token，这个值每个集群保持唯一。这样的话，如果你要重新创建集群，即使配置和之前一样，也会再次生成新的集群和节点 uuid；否则会导致多个集群之间的冲突，造成未知的错误。

各个参数可以使用ETCD_前缀的方式用环境变量的方式设定

#### 3. 创建数据目录

```sh
mkdir -p data/c_etcd1
mkdir -p data/c_etcd2
mkdir -p data/c_etcd3
chmod 777 data/c_etcd1
chmod 777 data/c_etcd2
chmod 777 data/c_etcd3
```

#### 4. 测试启动

```sh
docker-compose up
```

#### 5. 测试

```sh
$ docker-compose ps
Name          Command         State                        Ports                      
--------------------------------------------------------------------------------------
etcd1   /usr/local/bin/etcd   Up      0.0.0.0:32769->2379/tcp, 0.0.0.0:32768->2380/tcp
etcd2   /usr/local/bin/etcd   Up      0.0.0.0:32771->2379/tcp, 0.0.0.0:32770->2380/tcp
etcd3   /usr/local/bin/etcd   Up      0.0.0.0:32773->2379/tcp, 0.0.0.0:32772->2380/tcp

$ curl -L http://127.0.0.1:32769/v2/keys/foo -XPUT -d value="Hello foo"
{"action":"set","node":{"key":"/foo","value":"Hello foo","modifiedIndex":8,"createdIndex":8}}
$ curl -L http://127.0.0.1:32769/v2/keys/foo1/foo1 -XPUT -d value="Hello foo1"
{"action":"set","node":{"key":"/foo1/foo1","value":"Hello foo1","modifiedIndex":9,"createdIndex":9}}

$ curl -L http://127.0.0.1:32771/v2/keys/foo
{"action":"get","node":{"key":"/foo","value":"Hello foo","modifiedIndex":8,"createdIndex":8}}

$ docker-compose exec c_etcd1 etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=true
$ docker-compose exec c_etcd2 etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=true
$ docker-compose exec c_etcd3 etcdctl member list
ade526d28b1f92f7: name=etcd1 peerURLs=http://etcd1:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
bd388e7810915853: name=etcd3 peerURLs=http://etcd3:2380 clientURLs=http://0.0.0.0:2379 isLeader=false
d282ac2ce600c1ce: name=etcd2 peerURLs=http://etcd2:2380 clientURLs=http://0.0.0.0:2379 isLeader=true
```

#### 6. 启动

```sh
docker-compose up -d
```



### 场景：移除节点

### API 说明和 etcdctl 命令说明

etcd REST API 说明（v2 版本）：

| 命令                                                         | 说明                      |
| ------------------------------------------------------------ | ------------------------- |
| curl -L http://127.0.0.1:2379/version                        | 查看版本                  |
| curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello world" | 添加键值                  |
| curl http://127.0.0.1:2379/v2/keys/message                   | 获取键值                  |
| curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello etcd" | 更新键值                  |
| curl http://127.0.0.1:2379/v2/keys/message -XDELETE          | 删除键值                  |
| curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl=5 | 添加 TTL 键值（过期时间） |
| curl http://127.0.0.1:2379/v2/keys/foo                       | 获取 TTL 键值             |
| curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -d ttl= -d prevExist=true | 更新 TTL 键值             |
| curl http://127.0.0.1:2379/v2/keys/foo?wait=true             |                           |
| curl http://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar    |                           |
| curl 'http://127.0.0.1:2379/v2/keys/foo?wait=true&waitIndex=7' |                           |
| curl http://127.0.0.1:2379/v2/keys/queue -XPOST -d value=Job1 |                           |
| curl -s 'http://127.0.0.1:2379/v2/keys/queue?recursive=true&sorted=true' |                           |
| curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true |                           |
| curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d ttl=30 -d dir=true -d prevExist=true |                           |
| curl 'http://127.0.0.1:2379/v2/keys/dir?wait=true'           |                           |
| curl http://127.0.0.1:2379/v2/keys/dir -XPUT -d dir=true     | 创建目录                  |
| curl http://127.0.0.1:2379/v2/keys/foo_dir/foo -XPUT -d value=bar | 在目录下添加键值          |
| curl http://127.0.0.1:2379/v2/keys/?recursive=true           | 获取目录下的键值          |
| curl 'http://127.0.0.1:2379/v2/keys/foo_dir?dir=true' -XDELETE | 删除目录                  |
| curl http://10.0.0.10:2379/v2/members                        | 查看集群成员              |
| curl http://10.0.0.10:2379/v2/members -XPOST -H "Content-Type: application/json" -d '{"peerURLs":["[http://10.0.0.10:2380](http://10.0.0.10:2380/)"]}' | 添加集群成员              |
| curl http://10.0.0.10:2379/v2/members/272e204152 -XDELETE    | 删除集群成员              |
| curl http://10.0.0.10:2379/v2/members/272e204152 -XPUT -H "Content-Type: application/json" -d '{"peerURLs":["[http://10.0.0.10:2380](http://10.0.0.10:2380/)"]}' | 更新集群成员              |

更多 API 请查看：[etcd API](https://coreos.com/etcd/docs/latest/v2/api.html) 和 [Members API](https://coreos.com/etcd/docs/latest/v2/members_api.html)

| 命令                     | 说明                       |
| ------------------------ | -------------------------- |
| etcdctl set key value    | 添加键值                   |
| etcdctl get key          | 获取键值                   |
| etcdctl update key value | 更新键值                   |
| etcdctl rm key           | 删除键值                   |
| etcdctl mkdir dirname    | 添加目录（不存在的话创建） |
| etcdctl setdir           | 添加目录（都创建）         |
| etcdctl updatedir        | 更新目录                   |
| etcdctl rmdir            | 删除目录                   |
| etcdctl ls               | 列出目录                   |
| etcdctl watch            | 监控键值                   |
| etcdctl exec-watch       | 监控键值（执行命令）       |
| etcdctl list             | 查看集群成员               |
| etcdctl member add       | 添加集群成员               |
| etcdctl remove           | 移除集群成员               |