[toc]

[doc](https://docs.min.io/docs/minio-client-complete-guide.html) [下载](https://min.io/download#/linux)

## 简介

minIO客户端(mc)为UNIX命令提供了一种新的替代方法，如ls、cat、cp、mirror、diff、find等。支持文件系统和兼容Amazon S3的云存储服务(AWS Signature v2和v4)。

```
alias       set, remove and list aliases in configuration file
ls          list buckets and objects
mb          make a bucket
rb          remove a bucket
cp          copy objects
mirror      synchronize object(s) to a remote site
cat         display object contents
head        display first 'n' lines of an object
pipe        stream STDIN to an object
share       generate URL for temporary access to an object
find        search for objects
sql         run sql queries on objects
stat        show object metadata
mv          move objects
tree        list buckets and objects in a tree format
du          summarize disk usage recursively
retention   set retention for object(s)
legalhold   set legal hold for object(s)
diff        list differences in object name, size, and date between two buckets
rm          remove objects
encrypt    manage bucket encryption config
event       manage object notifications
watch       listen for object notification events
undo        undo PUT/DELETE operations
policy      manage anonymous access to buckets and objects
tag         manage tags for bucket(s) and object(s)
ilm         manage bucket lifecycle
version     manage bucket versioning
replicate   configure server side bucket replication
admin       manage MinIO servers
update      update mc to latest release
```

### alias - 添加，删除，修改服务

```
$ ./mc alias --help
NAME:
  mc alias - set, remove and list aliases in configuration file

USAGE:
  mc alias COMMAND [COMMAND FLAGS | -h] [ARGUMENTS...]

COMMANDS:
  set, s      set a new alias to configuration file
  list, ls    list aliases in configuration file
  remove, rm  remove an alias from configuration file
  
FLAGS:
  --config-dir value, -C value  path to configuration folder (default: "/share/home/qos/.mc")
  --quiet, -q                   disable progress bar display
  --no-color                    disable color theme
  --json                        enable JSON lines formatted output
  --debug                       enable debug output
  --insecure                    disable SSL certificate verification
  --help, -h                    show help
```

#### set

```
mc alias set <ALIAS> <YOUR-S3-ENDPOINT> <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY> --api <API-SIGNATURE> --path <BUCKET-LOOKUP-TYPE>
```

```sh
mc alias set minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
mc ls minio # 测试列出文件
```

#### ls

```
./mc alias ls
gcs  
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns

local
  URL       : http://localhost
  AccessKey : admin
  SecretKey : qos@minio@zw6-8|abc
  API       : s3v4
  Path      : auto

minio
  URL       : http://localhost:19000
  AccessKey : admin
  SecretKey : qos@minio@zw6-8|abc
  API       : S3v4
  Path      : auto
```



### mirror - 同步

```
USAGE:
   mc mirror [FLAGS] SOURCE TARGET

FLAGS:
  --overwrite                        overwrite object(s) on target if it differs from source
  --fake                             perform a fake mirror operation
  --watch, -w                        watch and synchronize changes
  --remove                           remove extraneous object(s) on target
  --region value                     specify region when creating new bucket(s) on target (default: "us-east-1")
  --preserve, -a                     preserve file system attributes and bucket policy rules on target bucket(s)
  --exclude value                    exclude object(s) that match specified object name pattern
  --older-than value                 filter object(s) older than N days (default: 0)
  --newer-than value                 filter object(s) newer than N days (default: 0)
  --storage-class value, --sc value  specify storage class for new object(s) on target
  --encrypt value                    encrypt/decrypt objects (using server-side encryption with server managed keys)
  --encrypt-key value                encrypt/decrypt objects (using server-side encryption with customer provided keys)
  --help, -h                         show help

ENVIRONMENT VARIABLES:
   MC_ENCRYPT:      list of comma delimited prefixes
   MC_ENCRYPT_KEY:  list of comma delimited prefix=secret values
```

