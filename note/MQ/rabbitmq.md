1.docker 安装 rabbitmq

```sh
docker pull rabbitmq:3-management

docker run -d --name rabbit -p 15672:15672 -p 5672:5672 rabbitmq:3-management
```

web管理台地址:node3:15672

