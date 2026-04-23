
docker-compose -h                           # 查看帮助

docker-compose build    # `docker-compose build` will read your `docker-compose.yml`, look for all services containing the `build:` statement and run a `docker build` for each one.  只build image, 不 run image

docker-compose up                           # 启动所有docker-compose服务
docker-compose up -d                        # 启动所有docker-compose服务并后台运行, 需要在 docker-compose.yaml 同步目录下执行

- **`-d`（detached mode）**：让容器在后台运行，这样终端不会被占用，你可以继续执行其他命令。
- -f 指定 docker-compose 文件位置 docker-compose -f /root/docker-compose/docker-compose.yml up -d

docker-compose down                         # 停止并删除容器、网络、卷、镜像。
docker-compose exec  yml里面的服务id                 # 进入容器实例内部  docker-compose exec docker-compose.yml文件中写的服务id /bin/bash

docker-compose ps                      # 展示当前docker-compose编排过的运行的所有容器
docker-compose top                     # 展示当前docker-compose编排过的容器进程
 
docker-compose logs  yml里面的服务id     # 查看容器输出日志
docker-compose config     # 检查配置
docker-compose config -q  # 检查配置，有问题才有输出

docker-compose restart   # 重启服务
docker-compose start     # 启动服务
docker-compose stop      # 停止服务


# 1 docker-compose build 

https://stackoverflow.com/questions/50230399/what-is-the-difference-between-docker-compose-build-and-docker-build

So basically `docker-compose build` will read your `docker-compose.yml`, look for all services containing the `build:` statement and run a `docker build` for each one.

[Each `build`](https://docs.docker.com/compose/compose-file/build/) can specify a `Dockerfile`, a context and args to pass to docker.

To conclude with an example `docker-compose.yml` file:

```yaml
version: '3.2'

services:
  database:
    image: mariadb
    restart: always
    volumes:
      - ./.data/sql:/var/lib/mysql

  web:
    build:
      dockerfile: Dockerfile-alpine
      context: ./web
    ports:
      - 8099:80
    depends_on:
      - database 
```

When calling `docker-compose build`, only the `web` target will need an image to be built. The `docker build` command would look like:

```yaml
docker build -t web_myproject -f Dockerfile-alpine ./web
```
