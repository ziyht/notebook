[toc]

## 1. du -hs /var/lib/docker/ 命令查看磁盘使用情况。

    sudo du -hs /var/lib/docker/
    
    237G /var/lib/docker/

### docker system df命令，类似于Linux上的df命令，用于查看Docker的磁盘使用情况:

    docker system df
    
    TYPE TOTAL ACTIVE SIZE RECLAIMABLE
    Images 7 2 122.2GB 79.07GB (64%)
    Containers 2 2 61.96GB 0B (0%)
    Local Volumes 0 0 0B 0B
    Build Cache 0 0 0B 0B

### docker system prune命令可以用于清理磁盘，删除关闭的容器、无用的数据卷和网络，以及dangling镜像(即无tag的镜像)。

    docker system prune
    
    WARNING! This will remove:
            - all stopped containers
            - all networks not used by at least one container
            - all dangling images
            - all build cache
    
    Are you sure you want to continue? [y/N] y
    
    Total reclaimed space: 0B

> `docker system prune -a` 命令清理得更加彻底，可以将没有容器使用Docker镜像都删掉。注意，这两个命令会把你暂时关闭的容器，以及暂时没有用到的Docker镜像都删掉了…所以使用之前一定要想清楚.。我没用过，因为会清理 没有开启的 Docker 镜像。



## 2 迁移 /var/lib/docker 目录。

### 2.1 停止docker服务。

```
systemctl stop docker
systemctl stop docker.socket
```

### 2.2 创建新的docker目录，执行命令df -h,找一个大的磁盘。 我在 /home目录下面建了 /home/docker/lib目录，执行的命令是：

```
mkdir -p /home/docker/lib
```

### 2.3 迁移/var/lib/docker目录下面的文件到 /home/docker/lib：

```
rsync -avz /var/lib/docker /home/docker/lib/
```

### 2.4 配置 /etc/systemd/system/docker.service.d/devicemapper.conf。查看 devicemapper.conf 是否存在。如果不存在，就新建。

    sudo mkdir -p /etc/systemd/system/docker.service.d/
    
    sudo vi /etc/systemd/system/docker.service.d/devicemapper.conf

然后在 devicemapper.conf 写入：（同步的时候把父文件夹一并同步过来，实际上的目录应在 /home/docker/lib/docker ）

    [Service]
    ExecStart=
    ExecStart=/usr/bin/dockerd --graph=/home/docker/lib/docker

4.6 重新加载 docker
```
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```
4.7 为了确认一切顺利，运行

```
docker info
```

命令检查Docker 的根目录.它将被更改为 /home/docker/lib/docker

    ...
    Docker Root Dir: /home/docker/lib/docker
    Debug Mode (client): false
    Debug Mode (server): false
    Registry: https://index.docker.io/v1/
    ...

4.8 启动成功后，再确认之前的镜像还在：

    linlf@dacent:~$ docker images
    REPOSITORY TAG IMAGE ID CREATED SIZE
    AAA/AAA v2 7331b8651bcc 27 hours ago 3.85GB
    BBB/BBB v1 da4a80dd8424 28 hours ago 3.47GB

4.9 确定容器没问题后删除/var/lib/docker/目录中的文件。

