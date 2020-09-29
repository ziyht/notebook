## docker-compose 部署 Superset

### 1. clone 官方项目

```sh
git clone https://github.com/apache/incubator-superset

mv incubator-superset superset

cd superset
```

### 2. docker_compose 说明

#### 1. 服务

docker-compose.yml 文件位于项目目录下

在这个 docker-compose.yml  中，会启动三个服务

* redis
  * 此服务用以做superset的缓存服务
* db
  * 此服务用以做superset的数据持久化服务，存储用户，表格等等
  * 配置的数据库为 postgres
* superset
  * superset 服务本身
  * 并没有指定镜像，而是指定了构建规则

#### 2. volumes

在 docker-compose.yml 的最下面定义了需要存储数据的地方

```yaml
volumes:
  superset_home:
    external: false
  db_home:
    external: false
  redis:
    external: false
```

可以根据需要修改配置，一般情况下，我们都希望持久化的存储在宿主机上：

```yaml
volumes:
  superset_home:
    external: ./data/superset
  db_home:
    external: ./data/db
  redis:
    external: ./data/redis
```

### 3. 配置说明

在superset中，需要定义一个包含在python_path中的 py 文件来覆盖原有配置，在docker-compose.yml中已进行了进行了相关目录的挂载：

```yaml
x-superset-volumes: &superset-volumes
  # /app/pythonpath_docker will be appended to the PYTHONPATH in the final container
  - ./docker/docker-init.sh:/app/docker-init.sh
  - ./docker/pythonpath_dev:/app/pythonpath            # <--
  - ./superset:/app/superset
  - ./superset-frontend:/app/superset-frontend
  - superset_home:/app/superset_home
```

在项目目录下有 ./docker/pythonpath_dev 目录，在此目录下已有一个示例的配置 py 文件 superset_config.py，需要做配置修改的话，直接修改此文件即可:

```python
#---------------------------------------------------------
# Superset specific config
#---------------------------------------------------------
ROW_LIMIT = 5000

SUPERSET_WEBSERVER_PORT = 8088
#---------------------------------------------------------

#---------------------------------------------------------
# Flask App Builder configuration
#---------------------------------------------------------
# Your App secret key
SECRET_KEY = '\2\1thisismyscretkey\1\2\e\y\y\h'

# The SQLAlchemy connection string to your database backend
# This connection defines the path to the database that stores your
# superset metadata (slices, connections, tables, dashboards, ...).
# Note that the connection information to connect to the datasources
# you want to explore are managed directly in the web UI
SQLALCHEMY_DATABASE_URI = 'sqlite:////path/to/superset.db'

# Flask-WTF flag for CSRF
WTF_CSRF_ENABLED = True
# Add endpoints that need to be exempt from CSRF protection
WTF_CSRF_EXEMPT_LIST = []
# A CSRF token that expires in 1 year
WTF_CSRF_TIME_LIMIT = 60 * 60 * 24 * 365

# Set this API key to enable Mapbox visualizations
MAPBOX_API_KEY = ''
```

具体说明可以查看：https://superset.incubator.apache.org/installation.html#configuration



### 1. 创建独立的目录供使用

```sh
mkdir superset
cd superset
```

### 2. 一些准备工作

#### 方式1

##### 1.准备自己的dockerfile生成镜像

```sh
vim dockerfile
```

```dockerfile
FROM preset/superset
# Switching to root to install the required packages
USER root
# Example: installing the MySQL driver to connect to the metadata database
# if you prefer Postgres, you may want to use `psycopg2-binary` instead
RUN pip install mysqlclient
# Example: installing a driver to connect to Redshift
# Find which driver you need based on the analytics database
# you want to connect to here:
# https://superset.incubator.apache.org/installation.html#database-dependencies
RUN pip install sqlalchemy-redshift

RUN pip install sqlalchemy-clickhouse
# Switching back to using the `superset` user
USER superset
```

这里主要是为了安装必要的数据库驱动：[列表](https://superset.incubator.apache.org/installation.html#database-dependencies)

##### 2. 创建版本文件

```sh
vim version
```

```
1.0.0
```

##### 3. 创建构建脚本

```sh
vim build_image
chmod +x build_image
```

```sh
#!/bin/sh

dir=$(cd $(dirname $0); pwd)

version=`cat $dir/version`

# 注意这里的用户改成自己的
docker build -t ziyht/superset:$version $dir
```

##### 4. 最终的结构如下

```sh
superset/
├── build_image
├── dockerfile
└── version
```

##### 5. 执行build_image构建镜像

> 因为拉取的镜像非常大，构建前最好先用 docker info 查看当前 docker 可用空间是否足够

```sh
./build_image
```





---

#### 1. 使用第三方项目

[https://github.com/amancevice/superset](https://github.com/amancevice/superset)

vim docker-compose.yml

```yaml
version: '3'
services:
  redis:
    image: redis
    restart: always
    volumes:
      - redis:/data
  superset:
    image: amancevice/superset
    restart: always
    depends_on:
      - redis
    environment:
      MAPBOX_API_KEY: ${MAPBOX_API_KEY}
      SUPERSET_HOME: /etc/superset
    ports:
      - "8088:8088"
    volumes:
      - ./superset_config.py:/etc/superset/superset_config.py
      - superset:/var/lib/superset

volumes:
  redis   : ./data/redis
  superset: ./data/superset
```

```sh
mkdir -p data/redis
mkdir -p data/superset
chmod -R 777 data
```









### 2.创建目录和配置





