```yml
version: '2.0'
services:

  am:
    image: prom/alertmanager:v0.21.0
    restart: always
    command: 
    - --config.file=/etc/alertmanager.yml
    - --web.listen-address=:9093
    - --storage.path=/var/lib/alertmanager
    user : '1000'                      # user qos
    network_mode: host       
    volumes:
      - ./alertmanager.yml:/etc/alertmanager.yml
      - ./data/am:/var/lib/alertmanager
```

