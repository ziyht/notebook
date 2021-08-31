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