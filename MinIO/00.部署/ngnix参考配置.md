### 简单配置

```
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://localhost:9000;
    }
}
```

### 添加存储处理

```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m
use_temp_path=off;
server {
    # ...
    location / {
        proxy_cache      my_cache;
        proxy_set_header Host $http_host;
        proxy_pass       http://localhost:9000;
    }
}
```

### loadbalance

```
 upstream minio {
    server 127.0.0.1:9001 weight=20 max_fails=2 fail_timeout=30s;
    server 127.0.0.1:9002 weight=10 max_fails=2 fail_timeout=30s;
    server 127.0.0.1:9003 weight=10 max_fails=2 fail_timeout=30s;
    server 127.0.0.1:9004 weight=10 max_fails=2 fail_timeout=30s;
}
location / {
    # proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header Host $http_host;
        client_body_buffer_size 10M;
        client_max_body_size 10G;
        proxy_buffers 1024 4k;
        proxy_read_timeout 300;
        proxy_next_upstream error timeout http_404;
        proxy_pass http://minio;
}
```