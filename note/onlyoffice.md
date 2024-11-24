# OnlyOffice使用

## 环境搭建

1. 方式一：

```yaml
version: '3.3'
services:
    documentserver:
        ports:
            - '8085:80'
        restart: always
        volumes:
            - '/app/onlyoffice/DocumentServer/logs:/var/log/onlyoffice'
            - '/app/onlyoffice/DocumentServer/data:/var/www/onlyoffice/Data'
            - '/app/onlyoffice/DocumentServer/lib:/var/lib/onlyoffice'
            - '/app/onlyoffice/DocumentServer/db:/var/lib/postgresql'
        image: 'onlyoffice/documentserver:7.0 '
```

2. 方式二：

```yaml
version: '3'
services:
  onlyoffice-documentserver:
    image: 'onlyoffice/documentserver:7.0'
    build:
      context: .
    container_name: onlyoffice-documentserver
    depends_on:
      - onlyoffice-postgresql
      - onlyoffice-rabbitmq
    environment:
      - DB_TYPE=postgres
      - DB_HOST=onlyoffice-postgresql
      - DB_PORT=5432
      - DB_NAME=onlyoffice
      - DB_USER=onlyoffice
      - AMQP_URI=amqp://guest:guest@onlyoffice-rabbitmq
      # Uncomment strings below to enable the JSON Web Token validation.
      #- JWT_ENABLED=true
      #- JWT_SECRET=secret
      #- JWT_HEADER=Authorization
      #- JWT_IN_BODY=true
    ports:
      - '8088:80'
      - '443:443'
    stdin_open: true
    restart: always
    stop_grace_period: 60s
    volumes:
      - /var/www/onlyoffice/Data
      - /var/log/onlyoffice
      - /var/lib/onlyoffice/documentserver/App_Data/cache/files
      - /var/www/onlyoffice/documentserver-example/public/files
      - /usr/share/fonts

  onlyoffice-rabbitmq:
    container_name: onlyoffice-rabbitmq
    image: rabbitmq
    restart: always
    expose:
      - '5672'
    ports:
      - '5672:5672'

  onlyoffice-postgresql:
    container_name: onlyoffice-postgresql
    image: postgres:9.5
    environment:
      - POSTGRES_DB=onlyoffice
      - POSTGRES_USER=onlyoffice
      - POSTGRES_HOST_AUTH_METHOD=trust
    restart: always
    expose:
      - '5432'
    ports:
      - '5432:5432'
    volumes:
      - postgresql_data:/var/lib/postgresql

volumes:
  postgresql_data:
```

##  前端配置

1. 前端配置
2. 

## 后端配置

1. callback接口编写
2. 