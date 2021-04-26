```sh
#!/bin/sh

curl -X POST http://zw6-8:9093/-/reload
echo "--"
docker-compose logs --tail=10 am
```

