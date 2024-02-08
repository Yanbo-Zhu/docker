

# 1 常规步骤 
·	搜索镜像
·	拉取镜像
·	查看镜像
·	启动镜像
·	服务端口映射
·	停止容器
·	移除容器



# 2 安装tomcat


![](image/Pasted%20image%2020240208141956.png)



# 3 安装 mysql 

![](image/Pasted%20image%2020240208142658.png)


先查看 mysql 是否之前已经在 server 上 安装了 避免 窗口占用 
ps -e | grep mysql 


·	使用mysql镜像
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
docker ps
docker exec -it 容器ID /bin/bash
mysql -uroot -p

建库建表插入数据

![](image/Pasted%20image%2020240208143233.png)


## 3.1 删除容器后，里面的mysql数据如何办, 

一起被删除了. 要想不被删除 

docker run -d -p 3306:3306 --privileged=true -v /zzyyuse/mysql/log:/var/log/mysql -v /zzyyuse/mysql/data:/var/lib/mysql -v /zzyyuse/mysql/conf:/etc/mysql/conf.d -e 
![](image/Pasted%20image%2020240208143630.png)


## 3.2 新建my.cnf
·	通过容器卷同步给mysql容器实例
```
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8

```

![](image/Pasted%20image%2020240208143730.png)



# 4 安装redis

![](image/Pasted%20image%2020240208144722.png)

## 4.1 设置 容器卷
Docker挂载主机目录Docker访问出现cannot open directory .: Permission denied
解决办法：在挂载目录后多加一个--privileged=true参数即可

1 在CentOS宿主机下新建目录/app/redis
建目录
  mkdir -p /app/redis


2 	将一个redis.conf文件模板拷贝进/app/redis目录下

![](image/Pasted%20image%2020240208144555.png)


拷贝配置文件 将准备好的redis.conf文件放进/app/redis目录下
app/redis目录下修改redis.conf文件

- 开启redis验证    可选
    - requirepass 123
- 允许redis外地连接  必
    - 注释掉 # bind 127.0.0.1
- daemonize no
    - 将daemonize yes注释起来或者 daemonize no设置，因为该配置和docker run中-d参数冲突，会导致容器一直启动失败
-   开启redis数据持久化  appendonly yes  可选



3 使用redis6.0.8镜像创建容器(也叫运行镜像)

docker run  -p 6379:6379 --name myr3 --privileged=true -v /app/redis/redis.conf:/etc/redis/redis.conf -v /app/redis/data:/data -d redis:6.0.8 redis-server /etc/redis/redis.conf
![](image/Pasted%20image%2020240208144905.png)

试redis-cli连接上来

![](image/Pasted%20image%2020240208144925.png)

 docker exec -it 运行着Rediis服务的容器ID redis-cli
