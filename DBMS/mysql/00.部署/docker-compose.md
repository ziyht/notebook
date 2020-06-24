## docker-compose

### 1. 创建项目目录

```sh
mkdir mysql
cd mysql
```

### 2. 创建docker-compose.yml

vim docker-compose.yml

```yaml
version: '2.0'

services:

  db1:
    image: mysql
    container_name: qos-mysql1
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    ports: 
      - 3306:3306
    volumes:
      #- ./etc:/etc/mysql
      - ./data/db1/logs:/var/log/mysql
      - ./data/db1/data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: hsjhfqgd
  
  # adminer 为一个快速查看mysql的web UI工具，非常好用
  adminer:
    image: adminer
    container_name : qos-adminer
    restart: always
    ports:
      - 8082:8080
```

有必要的情况下，在docker自动创建 data 目录后

### 3. 创建客户端登陆脚本

vim run_client_it

```sh
#/bin/sh

docker run -it --net mysql_default --rm mysql mysql -hqos-mysql1 -uroot -phsjhfqgd
```

