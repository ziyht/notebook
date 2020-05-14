### docker-compose

#### 1. 拉取镜像

```sh
docker pull yandex/clickhouse-server:20.3
docker pull yandex/clickhouse-client:20.3
```

#### 2. 建立项目目录

```sh
mkdir clickhouse-server
cd clickhouse-server
```

#### 3. 创建 docker-compose.yml 文件

```sh
vim docker-compose.yml
```

```yaml
version: '2.0'
services:

  server1:
    image: yandex/clickhouse-server:20.3
    restart: always
    container_name: qos-clickhouse-server1
    #user: "1008"
    ports:
      - 8123:8123
      - 9000:9000           
    volumes:
      - ./etc:/etc/clickhouse-server
      - ./data/server1/data:/var/lib/clickhouse
      - ./data/server1/logs:/var/log/clickhouse-server
    networks:
      clickhouse:
        aliases:
          - server1
  
  adminer:
    image: adminer
    container_name: qos-clickhouse-adminer
    restart: always
    ports:
      - 8083:8080
    networks:
      clickhouse:
        aliases:
          - server1

networks:
  clickhouse:
    driver: bridge
```

#### 4. 创建配置文件

在当前目录下创建 etc 目录，并依次创建相关的配置文件

```sh
mkdir etc
```

vim etc/[config.xml](./etc/config.xml)

vim etc/[server-test.xml](./etc/server-test.xml)

vim etc/[users.xml](./etc/users.xml)

如果要创建新用户，可使用如下命令创建随机密码，然后在`users.xml`文件中设置即可：

```sh
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
```

#### 5. 启动服务

```sh
docker-compose up -d
```

#### 6. 连接clickhouse

1. 使用 docker 运行 clickhouse-client 进行连接

```sh
docker run -it --rm --link qos-clickhouse-server1:server1 --net=clickhouse-server_clickhouse yandex/clickhouse-client:20.3 --host server1
```

---

2. 先使用如下命令获取容器的相关信息，得到ip地址：

```sh
docker inspect qos-clickhouse-server1
```

然后运行（须先安装 clickhouse-client）：

```sh
clickhouse-client --host ${ip}
```







