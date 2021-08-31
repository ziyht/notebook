# 概述

Rclone是一款的命令行工具，支持在不同对象存储、网盘间同步、上传、下载数据。

支持的主流对象存储：

Google Drive
Amazon S3
Openstack Swift / Rackspace cloud files / Memset Memstore
Dropbox
Google Cloud Storage
Amazon Drive
Microsoft One Drive
Hubic
Backblaze B2
Yandex Disk
The local filesystem

# 安装

### Linux下安装

#### 下载安装包

```
curl -O http://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
```

#### 安装

```
sudo cp rclone /usr/sbin/
sudo chown root:root /usr/sbin/rclone
sudo chmod 755 /usr/sbin/rclone
```

#### 配置

```
rclone config
```

### MacOS下安装

#### 下载安装包

```
cd && curl -O http://downloads.rclone.org/rclone-current-osx-amd64.zip
unzip -a rclone-current-osx-amd64.zip && cd rclone-*-osx-amd64
```

### 安装

```
sudo mv rclone /usr/local/bin/
cd .. && rm -rf rclone-*-osx-amd64 rclone-current-osx-amd64.zip
```

### 配置

```
rclone config
```

# 使用

## 操作命令

rclone命令的语法格式：

```
Syntax: [options] subcommand <parameters> <parameters...>
```



常用的rclone命令有：

- rclone config - 以控制会话的形式添加rclone的配置，配置保存在.rclone.conf文件中。
- rclone copy - 将文件从源复制到目的地址，跳过已复制完成的。
- rclone sync - 将源数据同步到目的地址，只更新目的地址的数据。
- rclone move - 将源数据移动到目的地址。
- rclone delete - 删除指定路径下的文件内容。
- rclone purge - 清空指定路径下所有文件数据。
- rclone mkdir - 创建一个新目录。
- rclone rmdir - 删除空目录。
- rclone check - 检查源和目的地址数据是否匹配。
- rclone ls - 列出指定路径下所有的文件以及文件大小和路径。
- rclone lsd - 列出指定路径下所有的目录/容器/桶。
- rclone lsl - 列出指定路径下所有文件以及修改时间、文件大小和路径。
- rclone md5sum - 为指定路径下的所有文件产生一个md5sum文件。
- rclone sha1sum - 为指定路径下的所有文件产生一个sha1sum文件。
- rclone size - 获取指定路径下，文件内容的总大小。.
- rclone version - 查看当前版本。
- rclone cleanup - 清空remote。
- rclone dedupe - 交互式查找重复文件，进行删除/重命名操作。

### rclone config

开启一个交互式的配置会话。命令格式如下：

```
rclone config
```



### rclone copy

将文件从源复制到目的地址，跳过已复制完成的。命令格式如下：

```
rclone copy source:sourcepath dest:destpsth
```

注：
1、rclone copy复制总是指定路径下的数据；而不是当前目录。
2、–no-traverse 标志用于控制是否列出目的地址目录。
3、-P 显示拷贝进度
4、--dry-run 测试，不执行拷贝
5、--create-empty-src-dirs 将在文件拷贝完成后在目的地创建源中的空目录

### rclone sync

```
rclone sync source:path dest:path
```

注：
1、同步数据时，可能会删除目的地址的数据；建议先使用–dry-run标志来检查要复制、删除的数据。
2、同步数据出错时，不会删除任何目的地址的数据。
3、rclone sync同步的始终是path目录下的数据，而不是path目录。（空目录将不会被同步）

### rclone move

```
rclone move source:path dest:path
```

注：
1、同步数据时，可能会删除目的地址的数据；建议先使用–dry-run标志来检查要复制、删除的数据。

### rclone purge

清空path目录和数据。命令格式如下：

```
rclone purge remote:path
```



注：
1、此命令，include/exclude 过滤器失效。
2、删除path目录下部分数据，请使用rclone delete 命令

### rclone mkdir

创建path目录。命令格式如下：

```
rclone mkdir remote:path
```



### rclone rmdir

删除一个空目录。命令格式如下：

```
rclone rmdir remote:path
```



注：
1、不能删除非空的目录，删除非空目录请使用 rclone purge。

### rclone check

检查源和目标地址文件是否匹配。命令格式如下：

```
rclone check source:path dest:path
```



注：
1、–size-only标志用于指定，只比较大小，不比较MD5SUMs。

### rclone ls

列出指定path下，所有的文件以及文件大小和路径。命令格式如下：

```
rclone ls remote:path
```



### rclone lsd

列出指定path下，所有目录、容器、桶。命令格式如下：

```
rclone lsd remote:path
```



### rclone delete

删除指定目录的内容。命令格式如下：

```
rclone delete remote:path
```



注：
1、不同于rclone purge，rclone delete 可使用 include/exclude 过滤器选择删除文件内容。
eg：
删除文件大小大于100M的文件

```
# 先检查哪些文件将被删除
rclone --min-size 100M lsl remote:path                  # 使用rclone lsl 列出大于100M的文件
rclone --dry-run --min-size 100M delete remote:path    # 使用--dry-run 检查将要被删除的文件
	
# 使用 rclone delete 进行文件删除
rclone --min-size 100M delete remote:path
```



### rclone size

获取指定path下所有数据文件的总大小。命令格式如下：

```
rclone size remote:path
```



### more

更多rclone命令，详见http://rclone.org/commands/ 。

## 配置rclone

rclone 提供了交互式配置会话的方式来进行rclone的配置；每次配置的信息存储在.rclone.conf文件中。

### 配置示例

在这里，我们使用共享存储KS3示例，来进行rclone的配置：

```
# 执行rclone config 开启配置会话
$ rclone config 
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n                     # 输入n，配置一个新的remote
name> ks3-remote             # 输入remote的名字，这里命名为 ks3-remote
Type of storage to configure.
Choose a number from below, or type in your own value
 1 / Amazon Drive
   \ "amazon cloud drive"
 2 / Amazon S3 (also Dreamhost, Ceph, Minio)
   \ "s3"
 3 / Backblaze B2
   \ "b2"
 4 / Dropbox
   \ "dropbox"
 5 / Encrypt/Decrypt a remote
   \ "crypt"
 6 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
 7 / Google Drive
   \ "drive"
 8 / Hubic
   \ "hubic"
 9 / Kingsoft Cloud KS3
   \ "ks3"
10 / Local Disk
   \ "local"
11 / Microsoft OneDrive
   \ "onedrive"
12 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
13 / Yandex Disk
   \ "yandex"
Storage> 9                   # 选择9，也可直接输入ks3
Get KS3 credentials from runtime (environment variables). Only applies if access_key_id and secret_access_key is blank.
Choose a number from below, or type in your own value
 1 / Enter KS3 credentials in the next step
   \ "false"
 2 / Get KS3 credentials from the environment (env vars)
   \ "true"
env_auth> 1                 # 选择1，直接配置access_key_id 和 secret_access_key  
AWS Access Key ID - leave blank for anonymous access or runtime credentials.
access_key_id> sgyO124X+/ZRbmgrbP7d    # 键入ks3的access_key_id
AWS Secret Access Key (password) - leave blank for anonymous access or runtime credentials.
secret_access_key> p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH    # 键入ks3的secret_access_key
Region to connect to.
Choose a number from below, or type in your own value
 1 / Beijing Region(the default region).
   \ "ks3-cn-beijing"
 2 / Hangzhou Region.
   \ "ks3-cn-hangzhou"
 3 / Shanghai Region.
   \ "ks3-cn-shanghai"
 4 / HongKong Region.
   \ "ks3-cn-hk-1"
 5 / US West Region.
   \ "ks3-us-west-1"
region> 1               # 选择ks3的region，这里选择北京的region
Endpoint for KS3 API.
Leave blank if using KS3 to use the default endpoint for the region.
endpoint>            # 不使用外部的endpoint，直接回车略过
An internal endpoint or the public endpoint for KS3 access. The default is false.
Choose a number from below, or type in your own value
 1 / Internal endpoint.
   \ "false"
 2 / Public endpoint.
   \ "true"
internal> 1          # 选择1，从公网访问，不使用内网加速。 在金山云的云主机中建议选择2，使用内网加速，且不计流量。
Canned ACL used when creating buckets and/or storing objects in KS3.
Choose a number from below, or type in your own value
 1 / Owner gets FULL_CONTROL. No one else has access rights (default).
   \ "private"
 2 / Owner gets FULL_CONTROL. The AllUsers group gets READ access.
   \ "public-read"
   / Owner gets FULL_CONTROL. The AllUsers group gets READ and WRITE access.
 3 | Granting this on a bucket is generally not recommended.
   \ "public-read-write"
acl> 1    # 选择1（私有控制）
The server-side encryption algorithm used when storing this object in S3.
Choose a number from below, or type in your own value
 1 / None
   \ ""
 2 / AES256
   \ "AES256"
server_side_encryption> 2  # 选择2，使用AES256加密。
Remote config
--------------------
[ks3-remote]
env_auth = false
access_key_id = sgyO124X+/ZRbmgrbP7d
secret_access_key = p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH
region = ks3-cn-beijing
endpoint = 
internal = false
acl = private
server_side_encryption = AES256
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y   # 检查生成的配置，确认无误，选择y确认
Current remotes:

Name                 Type
====                 ====
ks3-remote           ks3

e) Edit existing remote
n) New remote
d) Delete remote
s) Set configuration password
q) Quit config
e/n/d/s/q> q  # 配置完成，选择q退出。
```

配置完成，检查生成的配置文件~/.rclone.conf:

```
$ more ~/.rclone.conf
[ks3-remote]          # 远程存储的名称：ks3-remote
type = ks3            # 远程存储的类型：ks3
env_auth = false      # 是否从环境变量中读取access_key和secret_key 
access_key_id = sgyO124X+/ZRbmgrbP7d   # 当前ks3的access_key
secret_access_key = p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH  # 当前ks3的secret_key
region = ks3-cn-beijing  # 所使用ks3的region名称
endpoint =            # 是否使用外部endpoint
internal = false      # 是否从内网访问
acl = private         # 访问控制策略：私有控制
server_side_encryption = AES256 # 加密
```



### 测试配置

查看上述配置远程存储所有的桶：

```
$ rclone lsd ks3-remote:
          -1 2016-06-07 07:26:04        -1 dockertest2
          -1 2016-11-21 08:24:52        -1 ks3-fs-test
2016/11/28 15:26:25 
Transferred:      0 Bytes (0 Bytes/s)
Errors:                 0
Checks:                 0
Transferred:            0
Elapsed time:       200ms
```



## 一些实用场景

### 将本地目录同步到ks3

一般，由于对象存储具有廉价、高可用、安全、可靠等特性；我们通常会用作本地数据（日志、图片、视频）的存储或是备份。通常在使用对象存储存储数据时，往往需要调用官方的SDK，需要一定的开发量。尤其当我们仅仅是需要备份某个盘或是目录下的日志时，这时候使用rclone这样的命令行工具，能方便把数据存储（或备份）到远程存储上，且不受文件大小的限制（比如ks3单个文件上传大小不能超过5G）。

比如，将/home/local/directory同步到远程存储的某个bucket中，操作命令如下：

```
$ rclone sync /home/local/directory remote:bucket
```



### AWS s3 迁移到 KS3

由于s3的region多建在国外，s3的资费相对较高。那么将一部分数据迁移到KS3，是比较经济的。除了使用KS3官方提供的镜像服务，可以别的远程存储迁移到KS3；rclone为此提供了另外一种可能。

比如将远程存储s3的桶s3-log，迁移到远程存储ks3的桶ks3-log:

```
$ rclone sync s3-remote:s3-log ks3-remote:ks3-log
```



远程存储s3-remote、ks3-remote的配置信息如下：

```
[s3-remote]
type = s3-remote
env_auth = false
access_key_id = AKIAJTT3FZHMRQSXFZFQ
secret_access_key = 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
region = ap-northeast-2
endpoint = 
location_constraint = ap-northeast-2
acl = private
server_side_encryption = AES256
storage_class = 

[ks3-remote]
type = ks3
env_auth = false
access_key_id = sgyO124X+/ZRbmgrbP7d
secret_access_key = p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH
region = ks3-cn-beijing
endpoint = 
internal = false
acl = private
server_side_encryption = AES256
```



### KS3 bucket之间的同步

KS3对象存储不支持使用KS3存储的桶作为另一个桶的镜像。

比如：我们需要搭建一个大型的docker hub，为了保证不同地域用户下载和上传速度（虽然KS3自带CDN），需要在北京、上海分别建立两个镜像中心；这样在拉取镜像时，南方用户只需要去上海的镜像中心拉取数据，北方用户则到北京镜像中心拉取数据；极大地提高数据的访问速度。
那么类似这样的需求，我们不得不保持两个镜像中心的数据一致性；在不适用官方SDK的情况下，我们可以使用rclone sync同步不同region下bucket中的数据：

```
$ rclone sync ks3-beijing:bucket1 ks3-shanghai:bucket2
```



注：
即便bucket1与bucket2共用一套账户体系，即ks3-beijing 和 ks3-shanghai的access_key_id和secret_access_key是一样的。仍然需要分别建立 ks3-beijing 和 ks3-shanghai 两个远程存储的配置。
例如：

```
$ more ~/.rclone
[ks3-beijing]                                                                                               
type = ks3  
env_auth = false
access_key_id = sgyO124X+/ZRbmgrbP7d
secret_access_key = p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH
region = ks3-cn-beijing
endpoint =  
internal = true
acl = private
server_side_encryption = AES256
              
              
[ks3-shanghai]  
type = ks3  
env_auth = false
access_key_id = sgyO124X+/ZRbmgrbP7d
secret_access_key = p9/6yLzbU2FZEPw5vCPf2/buQOz1SVNdqqBtzscH
region = ks3-cn-shanghai
endpoint =  
internal = true
acl = private
server_side_encryption = AES256
```