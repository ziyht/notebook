## MinIO

[doc](https://docs.min.io/cn/distributed-minio-quickstart-guide.html)



### 设置文件链接永久有效

#### 1. 安装 mc

```sh
wget https://dl.minio.io/client/mc/release/linux-amd64/mc
chmod +x mc
```

#### 2. 添加服务端

```sh
./mc config host add share http://xxx.xxx.xxx.xxx:9000 access-key  secret-key
```

* 如果此时已经不知道access-key和secret-key是多少的话，就把minio重启，minio重启后会打印出access-key和secret-key。
* share是设置的名称，可自行更改。

#### 3. 配置下载策略

```sh
./mc policy set public share/packages
```

* 这个命令的作用是将 server 端的 test 桶设置为开放管理，可以直接通过 url 进行下载。
* [桶名]/[路径]可以一直拼接到具体的文件夹或文件
  * 类似于 http://xxx.xxx.xxx.xxx:9000/test/9079799689686434242，可用浏览器直接从此URL访问下载。
  * 但是没有一个专门的页面，这点不是很方便