## Ulimit

有时候在命令行中执行了 ulimit -n 655350 后，使用 systemd 的服务时依然会出现 too many openfiles，这是因为 systemd 有单独的配置

vim /etc/systemd/system.conf   - 系统程序使用

vim /etc/systemd/user.conf     - 用户程序使用

```
DefaultLimitCORE=infinity
DefaultLimitNOFILE=100000
DefaultLimitNPROC=100000
```

上述配置需重启后生效

顺便提一下，CentOS7自带的/etc/security/limits.d/20-nproc.conf文件里面默认设置了非root用户的最大进程数为4096，会覆盖掉/etc/security/limits.conf 

通过 `systemctl show` 可以查看当前生效的配置

```sh
# 所有用户创建的进程数
ps h -Led -o user | sort | uniq -c | sort -n
```



## 设置永久生效

编辑/etc/security/limits.conf

```cfg
* soft nofile 655350  #任何用户可以打开的最大的文件描述符数量，默认1024，这里的数值会限制tcp连接
* hard nofile 655350
* soft nproc  655350  #任何用户可以打开的最大进程数
* hard nproc  650000

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

# End of file
@<groupname> soft nproc 65535
@<groupname> hard nproc 65535
@<groupname> soft nofile 60000
@<groupname> hard nofile 65535
```

设置后，重新登录 ssh 就会生效