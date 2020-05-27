# docker-compose

Compose 项目是Docker官方的开源项目，负责实现Docker容器集群的快速编排，开源代码在https://github.com/docker/compose 上
	
我们知道使用Dockerfile模板文件可以让用户很方便的定义一个单独的应用容器，其实在工作中，经常会碰到需要多个容器相互配合来完成的某项任务情况，例如工作中的web服务容器本身，往往会在后端加上数据库容器，甚至会有负责均衡器，比如LNMP服务

Compose 就是来做这个事情的，它允许用户通过一个单独的`docker-compose.yml`模板文件(YAML格式)来定义一组相关联的应用容器为一个项目(project)	

Compose 中有两个重要的概念：

* 服务(`service`):一个应用的容器，实际上可以包括若干运行相同镜像的容器实例
* 项目(`project`):由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml中定义

Compose 项目是由Python编写的，实际上就是调用了Docker服务提供的API来对容器进行管理，因此，只要所在的操作系统的平台支持Docker API，就可以在其上利用Compose来进行编排管理.

## 安装

### centos

#### 1. 二进制包安装（推荐）

```sh
[root@operation ~]# curl -L https://github.com/docker/compose/releases/download/1.23.0-rc2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
Dload  Upload   Total   Spent    Left  Speed
100   617    0   617    0     0    373      0 --:--:--  0:00:01 --:--:--   373
100 11.1M  100 11.1M    0     0   368k      0  0:00:31  0:00:31 --:--:--  444k
[root@operation ~]# chmod +x /usr/local/bin/docker-compose
[root@operation ~]# docker-compose version
docker-compose version 1.23.0-rc2, build 350a555e
docker-py version: 3.5.0
CPython version: 3.6.6
OpenSSL version: OpenSSL 1.1.0f  25 May 2017
```

#### 2. pip安装

```sh
[root@operation ~]# pip install docker-compose
[root@operation ~]# ln -s /usr/bin/docker-compose /usr/local/bin/  # 安装完需要做个软链接
[root@operation ~]# docker-compose version
docker-compose version 1.22.0, build f46880f
docker-py version: 3.5.0
CPython version: 2.7.5
OpenSSL version: OpenSSL 1.0.1e-fips 11 Feb 2013
```

#### 3. 容器安装

```sh
[root@operation ~]# curl -L https://github.com/docker/compose/releases/download/1.23.0-rc2/run.sh > /usr/local/bin/docker-compose
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100   596    0   596    0     0    158      0 --:--:--  0:00:03 --:--:--   158
100  1670  100  1670    0     0    343      0  0:00:04  0:00:04 --:--:-- 1630k
[root@operation ~]# chmod +x /usr/local/bin/docker-compose
[root@operation ~]# docker-compose
Unable to find image 'docker/compose:1.23.0-rc2' locally
1.23.0-rc2: Pulling from docker/compose
3489d1c4660e: Pull complete
2e51ed086e7d: Pull complete
07d7b41c67a1: Pull complete
Digest: sha256:14f5ad3c2162b26b3eaafe870822598f80b03ec36fd45126952c891fd5e5a59a
# 实际上就是下的镜像(可以看下下载的run.sh脚本)
[root@operation ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker/compose      1.23.0-rc2          dc59a0b5e981        5 days ago          45.6MB
alpine              latest              196d12cf6ab1        4 weeks ago         4.41MB
[root@operation ~]# docker-compose version
docker-compose version 1.23.0-rc2, build 350a555e
docker-py version: 3.5.0
CPython version: 3.6.6

OpenSSL version: OpenSSL 1.1.0f  25 May 2017
```

### macos

下载安装 [docker toolbox](https://github.com/docker/toolbox/releases)

## 命令

### 参数选项

Compose 大部分命令的对象即可以是项目的本身，也可以是指定为项目中的服务或者容器
执行`docker-compose [COMMAND] --help` 或者`docker-compose help [COMMAND]`可以查看命令的帮助信息

具体的使用格式：

```
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS]
```

参数选项

* -f,--file file指定模板文件，默认是docker-compose.yml模板文件,可以多次指定
* -p,--project-name name指定项目名称，默认使用所在目录名称作为项目名称
* --x-networking 使用Docker的后端可插拔网络特性
* --x-networking-driver driver指定网络的后端驱动，默认使用bridge
* --verbose 输入更多的调试信息
* -v,--version 输出版本信息

### 命令

```
	build              Build or rebuild services (构建项目中的服务容器)
	bundle             Generate a Docker bundle from the Compose file (从Compose文件生成分布式应用程序包)
	config             Validate and view the Compose file (验证并查看Compose文件)
	create             Create services (为服务创建容器)
	down               Stop and remove containers, networks, images, and volumes (停止容器并删除由其创建的容器，网络，卷和图像up)
	events             Receive real time events from containers (为项目中的每个容器流式传输容器事件)
	exec               Execute a command in a running container (这相当于docker exec。使用此子命令，您可以在服务中运行任意命令。默认情况下，命令分配TTY，因此您可以使用命令docker-compose exec web sh来获取交互式提示。)
	help               Get help on a command (获得一个命令的帮助)
	images             List images ()
	kill               Kill containers (通过发送SIGKILL信号来强制停止服务容器)
	logs               View output from containers (查看服务容器的输出)
	pause              Pause services (暂停一个容器)
	port               Print the public port for a port binding (打印某个容器端口所映射的公共端口)
	ps                 List containers (列出项目中目前所有的容器)
	pull               Pull service images (拉取服务依赖镜像)
	push               Push service images (推送服务镜像)
	restart            Restart services (重启项目中的服务)
	rm                 Remove stopped containers (删除所有停止状态的服务容器)
	run                Run a one-off command (在指定服务上执行一个命令)
	scale              Set number of containers for a service (设置指定服务执行的容器个数)
	start              Start services (启动已存在的服务容器)
	stop               Stop services (停止已存在的服务容器)
	top                Display the running processes (显示容器正在运行的进程)
	unpause            Unpause services (恢复处于暂停状态的容器)
	up                 Create and start containers (自动完成包括构建镜像、创建服务、启动服务并关联服务相关容器的一系列操作)
	version            Show the Docker-Compose version information (输出版本)
```



## 环境变量

​	环境变量可以用来配置 Compose 的行为,以DOCKER_开头的变量和用来配置 Docker 命令行客户端的使用一样。如果使用 boot2docker , $(boot2docker shellinit) 将会设置它们为正确的值：

| --                   | --                                                           |
| -------------------- | ------------------------------------------------------------ |
| COMPOSE_PROJECT_NAME | 设置通过 Compose 启动的每一个容器前添加的项目名称，默认是当前工作目录的名字。 |
| COMPOSE_FILE         | 设置要使用的 docker-compose.yml 的路径。默认路径是当前工作目录。 |
| DOCKER_HOST          | 设置 Docker daemon 的地址。默认使用 unix:///var/run/docker.sock，与 Docker 客户端采用的默认值一致。 |
| DOCKER_TLS_VERIFY    | 如果设置不为空，则与 Docker daemon 交互通过 TLS 进行。       |
| DOCKER_CERT_PATH     | 配置 TLS 通信所需要的验证（ca.pem、cert.pem 和 key.pem）文件的路径，默认是 ~/.docker |

## 模板文件 docker-compose.yml

模板文件时Compose的核心，涉及的指令关键字比较多，但是大部分的指令与docker run相关的参数的含义是类似的
	
默认的模板名是`docker-compose.yml`
	
[官网链接](https://docs.docker.com/compose/compose-file/#compose-file-structure-and-examples)
	
Compose 和 Docker兼容性：
Compose 文件格式有3个版本,分别为1, 2.x 和 3.x
目前主流的为 3.x 其支持 docker 1.13.0 及其以上的版本

| Compose file format | Docker Engine |
| ------------------- | ------------- |
| 1                   | 1.9.0+        |
| 2.0                 | 1.10.0+       |
| 2.1                 | 1.12.0+       |
| 2.2, 3.0, 3.1, 3.2  | 1.13.0+       |
| 2.3, 3.3, 3.4, 3.5  | 17.06.0+      |
| 2.4                 | 17.12.0+      |
| 3.6                 | 18.02.0+      |
| 3.7                 | 18.06.0+      |

**示例**

```yaml
version: "3"
    services:
 
      redis:
        image: redis:alpine
        ports:
          - "6379"
        networks:
          - frontend
        deploy:
          replicas: 2
          update_config:
            parallelism: 2
            delay: 10s
          restart_policy:
            condition: on-failure
 
      db:
        image: postgres:9.4
        volumes:
          - db-data:/var/lib/postgresql/data
        networks:
          - backend
        deploy:
          placement:
            constraints: [node.role == manager]

```



### 日志配置

```yaml
logging:
  options:
    max-size: '12m'
    max-file: '5'
  driver: json-file
```







```yaml
version           # 指定 compose 文件的版本
services          # 定义所有的 service 信息, services 下面的第一级别的 key 既是一个 service 的名称
	[svc_name]        # service 的名称
	  build                 # 指定包含构建上下文的路径, 或作为一个对象，该对象具有 context 和指定的 dockerfile 文件以及 args 参数值
	    context             # context: 指定 Dockerfile 文件所在的路径
	    dockerfile          # dockerfile: 指定 context 指定的目录下面的 Dockerfile 的名称(默认为 Dockerfile)
	    args                # args: Dockerfile 在 build 过程中需要的参数 (等同于 docker container build --build-arg 的作用)
	    cache_from          # v3.2中新增的参数, 指定缓存的镜像列表 (等同于 docker container build --cache_from 的作用)
	    labels              # v3.3中新增的参数, 设置镜像的元数据 (等同于 docker container build --labels 的作用)
	    shm_size            # v3.5中新增的参数, 设置容器 /dev/shm 分区的大小 (等同于 docker container build --shm-size 的作用)
	 
	  command               # 覆盖容器启动后默认执行的命令, 支持 shell 格式和 [] 格式
	  configs               # 不知道怎么用
	  cgroup_parent         # 不知道怎么用
	  container_name        # 指定容器的名称 (等同于 docker run --name 的作用)
	  credential_spec       # 不知道怎么用
	 
	  deploy                # v3 版本以上, 指定与部署和运行服务相关的配置, deploy 部分是 docker stack 使用的, docker stack 依赖 docker swarm
	    endpoint_mode         # v3.3 版本中新增的功能, 指定服务暴露的方式
	      vip                   # Docker 为该服务分配了一个虚拟 IP(VIP), 作为客户端的访问服务的地址
	      dnsrr                 # DNS轮询, Docker 为该服务设置 DNS 条目, 使得服务名称的 DNS 查询返回一个 IP 地址列表, 客户端直接访问其中的一个地址
	    labels                # 指定服务的标签，这些标签仅在服务上设置
	    mode                  # 指定 deploy 的模式
	      global                # 每个集群节点都只有一个容器
	      replicated            # 用户可以指定集群中容器的数量(默认)
	    placement             # 不知道怎么用
	    replicas              # deploy 的 mode 为 replicated 时, 指定容器副本的数量
	    resources             # 资源限制
	      limits                # 设置容器的资源限制
	        cpus: "0.5"           # 设置该容器最多只能使用 50% 的 CPU
	        memory: 50M           # 设置该容器最多只能使用 50M 的内存空间
	      reservations          # 设置为容器预留的系统资源(随时可用)
	        cpus: "0.2"           # 为该容器保留 20% 的 CPU
	        memory: 20M           # 为该容器保留 20M 的内存空间
	    restart_policy        # 定义容器重启策略, 用于代替 restart 参数
	      condition             # 定义容器重启策略(接受三个参数)
	        none                  # 不尝试重启
	        on-failure            # 只有当容器内部应用程序出现问题才会重启
	        any                   # 无论如何都会尝试重启(默认)
      delay                 # 尝试重启的间隔时间(默认为 0s)
	      max_attempts          # 尝试重启次数(默认一直尝试重启)
	      window                # 检查重启是否成功之前的等待时间(即如果容器启动了, 隔多少秒之后去检测容器是否正常, 默认 0s)
	    update_config         # 用于配置滚动更新配置
	      parallelism           # 一次性更新的容器数量
	      delay                 # 更新一组容器之间的间隔时间
	      failure_action        # 定义更新失败的策略
	        continue              # 继续更新
	        rollback              # 回滚更新
	        pause                 # 暂停更新(默认)
	      monitor               # 每次更新后的持续时间以监视更新是否失败(单位: ns|us|ms|s|m|h) (默认为0)
	      max_failure_ratio     # 回滚期间容忍的失败率(默认值为0)
        order                 # v3.4 版本中新增的参数, 回滚期间的操作顺序
	        stop-first            #旧任务在启动新任务之前停止(默认)
	        start-first           #首先启动新任务, 并且正在运行的任务暂时重叠
	    rollback_config       # v3.7 版本中新增的参数, 用于定义在 update_config 更新失败的回滚策略
	      parallelism           # 一次回滚的容器数, 如果设置为0, 则所有容器同时回滚
	      delay                 # 每个组回滚之间的时间间隔(默认为0)
	      failure_action        # 定义回滚失败的策略
	        continue              # 继续回滚
	        pause                 # 暂停回滚
	      monitor               # 每次回滚任务后的持续时间以监视失败(单位: ns|us|ms|s|m|h) (默认为0)
	      max_failure_ratio     # 回滚期间容忍的失败率(默认值0)
	      order                 # 回滚期间的操作顺序
	        stop-first            # 旧任务在启动新任务之前停止(默认)
	        start-first           # 首先启动新任务, 并且正在运行的任务暂时重叠
	               
	          注意：
	                支持 docker-compose up 和 docker-compose run 但不支持 docker stack deploy 的子选项
	                security_opt  container_name  devices  tmpfs  stop_signal  links    cgroup_parent
	                network_mode  external_links  restart  build  userns_mode  sysctls

    devices               # 指定设备映射列表 (等同于 docker run --device 的作用)
	 
	  depends_on            # 定义容器启动顺序 (此选项解决了容器之间的依赖关系， 此选项在 v3 版本中 使用 swarm 部署时将忽略该选项)    
	      示例：
	          docker-compose up 以依赖顺序启动服务，下面例子中 redis 和 db 服务在 web 启动前启动
	          默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系
	                version: '3'
	                services:
	                  web:
	                    build: .
	                    depends_on:
	                      - db     
	                      - redis 
	                  redis:
	                    image: redis
	                  db:
	                    image: postgres                            
	 
    dns                   # 设置 DNS 地址(等同于 docker run --dns 的作用)
	  dns_search            # 设置 DNS 搜索域(等同于 docker run --dns-search 的作用)
	  tmpfs                 # v2 版本以上, 挂载目录到容器中, 作为容器的临时文件系统(等同于 docker run --tmpfs 的作用, 在使用 swarm 部署时将忽略该选项)
	  entrypoint            # 覆盖容器的默认 entrypoint 指令 (等同于 docker run --entrypoint 的作用)
	  env_file              # 从指定文件中读取变量设置为容器中的环境变量, 可以是单个值或者一个文件列表, 如果多个文件中的变量重名则后面的变量覆盖前面的变量, environment 的值覆盖 env_file 的值
	          文件格式：
	              RACK_ENV=development
	 
	  environment           # 设置环境变量， environment 的值可以覆盖 env_file 的值 (等同于 docker run --env 的作用)
	  expose                # 暴露端口, 但是不会和宿主机建立映射关系, 类似于 Dockerfile 的 EXPOSE 指令，暴露给其它容器
    external_links        # 连接不在 docker-compose.yml 中定义的容器或者不在 compose 管理的容器(docker run 启动的容器, 在 v3 版本中使用 swarm 部署时将忽略该选项)
	  extra_hosts           # 添加 host 记录到容器中的 /etc/hosts 中 (等同于 docker run --add-host 的作用)
	  healthcheck           # v2.1 以上版本, 定义容器健康状态检查, 类似于 Dockerfile 的 HEALTHCHECK 指令
	    test                  # 检查容器检查状态的命令, 该选项必须是一个字符串或者列表, 第一项必须是 NONE, CMD 或 CMD-SHELL, 如果其是一个字符串则相当于 CMD-SHELL 加该字符串
	      NONE                  # 禁用容器的健康状态检测
	      CMD                   # test: ["CMD", "curl", "-f", "http://localhost"]
	      CMD-SHELL             # test: ["CMD-SHELL", "curl -f http://localhost || exit 1"] 或者　test: curl -f https://localhost || exit 1
	    interval: 1m30s       # 每次检查之间的间隔时间
	    timeout: 10s          # 运行命令的超时时间
	    retries: 3            # 重试次数
	    start_period: 40s     # v3.4 以上新增的选项, 定义容器启动时间间隔
	    disable: true         # true 或 false, 表示是否禁用健康状态检测和　test: NONE 相同
	         
	  image                 # 指定 docker 镜像, 可以是远程仓库镜像、本地镜像
	  init                  # v3.7 中新增的参数, true 或 false 表示是否在容器中运行一个 init, 它接收信号并传递给进程
	  isolation             # 隔离容器技术, 在 Linux 中仅支持 default 值
	  labels                # 使用 Docker 标签将元数据添加到容器, 与 Dockerfile 中的 LABELS 类似
	  links                 # 链接到其它服务中的容器, 该选项是 docker 历史遗留的选项, 目前已被用户自定义网络名称空间取代, 最终有可能被废弃 (在使用 swarm 部署时将忽略该选项)
	         
	  logging               # 设置容器日志服务
	    driver                # 指定日志记录驱动程序, 默认 json-file (等同于 docker run --log-driver 的作用)
	    options               # 指定日志的相关参数 (等同于 docker run --log-opt 的作用)
	      max-size              # 设置单个日志文件的大小, 当到达这个值后会进行日志滚动操作
	      max-file              # 日志文件保留的数量
	 
	  network_mode          # 指定网络模式 (等同于 docker run --net 的作用, 在使用 swarm 部署时将忽略该选项)        
	  networks              # 将容器加入指定网络 (等同于 docker network connect 的作用), networks 可以位于 compose 文件顶级键和 services 键的二级键
	    aliases               # 同一网络上的容器可以使用服务名称或别名连接到其中一个服务的容器
	      ipv4_address      # IP V4 格式
	      ipv6_address      # IP V6 格式
	 
	            示例:
	                version: '3.7'
	                services:
	                  test:
	                    image: nginx:1.14-alpine
	                    container_name: mynginx
	                    command: ifconfig
	                    networks:
	                      app_net:                                # 调用下面 networks 定义的 app_net 网络
	                        ipv4_address: 172.16.238.10
	                networks:
	                  app_net:
	                    driver: bridge
	                    ipam:
	                      driver: default
	                      config:
	                        - subnet: 172.16.238.0/24
	 
	  pid: 'host'           # 共享宿主机的 进程空间(PID)
	 
	  ports                 # 建立宿主机和容器之间的端口映射关系, ports 支持两种语法格式
	    # SHORT 语法格式示例:
	    - "3000"                            # 暴露容器的 3000 端口, 宿主机的端口由 docker 随机映射一个没有被占用的端口
	    - "3000-3005"                       # 暴露容器的 3000 到 3005 端口, 宿主机的端口由 docker 随机映射没有被占用的端口
	    - "8000:8000"                       # 容器的 8000 端口和宿主机的 8000 端口建立映射关系
	    - "9090-9091:8080-8081"
	    - "127.0.0.1:8001:8001"             # 指定映射宿主机的指定地址的
	    - "127.0.0.1:5000-5010:5000-5010"  
	    - "6060:6060/udp"                   # 指定协议
	 
	    # LONG 语法格式示例:(v3.2 新增的语法格式)
	    ports:
	      - target: 80                    # 容器端口
	        published: 8080               # 宿主机端口
	        protocol: tcp                 # 协议类型
	        mode: host                    # host 在每个节点上发布主机端口,  ingress 对于群模式端口进行负载均衡
	 
	  secrets               # 不知道怎么用
	  security_opt          # 为每个容器覆盖默认的标签 (在使用 swarm 部署时将忽略该选项)
	  stop_grace_period     # 指定在发送了 SIGTERM 信号之后, 容器等待多少秒之后退出(默认 10s)	 
	  stop_signal           # 指定停止容器发送的信号 (默认为 SIGTERM 相当于 kill PID; SIGKILL 相当于 kill -9 PID; 在使用 swarm 部署时将忽略该选项)
	  sysctls               # 设置容器中的内核参数 (在使用 swarm 部署时将忽略该选项)
	  ulimits               # 设置容器的 limit
	  userns_mode           # 如果Docker守护程序配置了用户名称空间, 则禁用此服务的用户名称空间 (在使用 swarm 部署时将忽略该选项)
	 
	  volumes               # 定义容器和宿主机的卷映射关系, 其和 networks 一样可以位于 services 键的二级键和 compose 顶级键, 如果需要跨服务间使用则在顶级键定义, 在 services 中引用
	    SHORT 语法格式示例:
	    volumes:
	      - /var/lib/mysql                # 映射容器内的 /var/lib/mysql 到宿主机的一个随机目录中
	      - /opt/data:/var/lib/mysql      # 映射容器内的 /var/lib/mysql 到宿主机的 /opt/data
	      - ./cache:/tmp/cache            # 映射容器内的 /var/lib/mysql 到宿主机 compose 文件所在的位置
	      - ~/configs:/etc/configs/:ro    # 映射容器宿主机的目录到容器中去, 权限只读
	      - datavolume:/var/lib/mysql     # datavolume 为 volumes 顶级键定义的目录, 在此处直接调用
	             
	    LONG 语法格式示例:(v3.2 新增的语法格式)
	      version: "3.2"
        services:
	        web:
	          image: nginx:alpine
	          ports:
	            - "80:80"
	          volumes:
	            - type: volume                # mount 的类型, 必须是 bind、volume 或 tmpfs
	              source: mydata              # 宿主机目录
	              target: /data               # 容器目录
	              volume:                     # 配置额外的选项, 其 key 必须和 type 的值相同
	                nocopy: true                # volume 额外的选项, 在创建卷时禁用从容器复制数据
	            - type: bind                    # volume 模式只指定容器路径即可, 宿主机路径随机生成; bind 需要指定容器和数据机的映射路径
	              source: ./static
	              target: /opt/app/static
	              read_only: true             # 设置文件系统为只读文件系统
        volumes:
	        mydata:                           # 定义在 volume, 可在所有服务中调用
	                 
    restart               # 定义容器重启策略(在使用 swarm 部署时将忽略该选项, 在 swarm 使用 restart_policy 代替 restart)
      no                    # 禁止自动重启容器(默认)
      always                # 无论如何容器都会重启
      on-failure            # 当出现 on-failure 报错时, 容器重新启动
	 
	  ## 其他选项：
    domainname
    hostname
    ipc
    mac_address
    privileged
    read_only
    shm_size
    stdin_open
    tty
    user
    working_dir
	    ## 上面这些选项都只接受单个值和 docker run 的对应参数类似

	       对于值为时间的可接受的值：
	            2.5s
	            10s
	            1m30s
	            2h32m
	            5h34m56s
	 
	            时间单位: us, ms, s, m， h
	 
	       对于值为大小的可接受的值：
	            2b
	            1024kb
	            2048k
	            300m
	            1gb
	 
	            单位: b, k, m, g 或者 kb, mb, gb
	 
	 
	 
	 
	 
networks          # 定义 networks 信息
	driver                # 指定网络模式, 大多数情况下, 它 bridge 于单个主机和 overlay Swarm 上
	  bridge                # Docker 默认使用 bridge 连接单个主机上的网络
	  overlay               # overlay 驱动程序创建一个跨多个节点命名的网络
	  host                  # 共享主机网络名称空间(等同于 docker run --net=host)
	  none                  # 等同于 docker run --net=none
	 
	driver_opts           # v3.2以上版本, 传递给驱动程序的参数, 这些参数取决于驱动程序
	 
	attachable            # driver 为 overlay 时使用, 如果设置为 true 则除了服务之外，独立容器也可以附加到该网络; 如果独立容器连接到该网络，则它可以与其他 Docker 守护进程连接到的该网络的服务和独立容器进行通信
	 
	ipam                  # 自定义 IPAM 配置. 这是一个具有多个属性的对象, 每个属性都是可选的
	  driver                # IPAM 驱动程序, bridge 或者 default
	  config                # 配置项
	    ubnet                # CIDR格式的子网，表示该网络的网段       
	external              # 外部网络, 如果设置为 true 则 docker-compose up 不会尝试创建它, 如果它不存在则引发错误
	name                  # v3.5 以上版本, 为此网络设置名称
```

