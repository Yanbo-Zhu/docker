

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


# 5 mysql 主从复制 


![](image/Pasted%20image%2020240208161309.png)

1 主mysql 中: 
master容器实例内创建数据同步用户
·	CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
·	GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';

2 
副mysql 中 
在从数据库中配置主从复制
change master to master_host='宿主机ip', master_user='slave', master_password='123456', master_port=3307, master_log_file='mall-mysql-bin.000001', master_log_pos=617, master_connect_retry=30;

![](image/Pasted%20image%2020240208161642.png)


主从复制命令参数说明
master_host：主数据库的IP地址；
master_port：主数据库的运行端口；
master_user：在主数据库创建的用于同步数据的用户账号；
master_password：在主数据库创建的用于同步数据的用户密码；
master_log_file：指定从数据库要复制数据的日志文件，通过查看主数据的状态，获取File参数；
master_log_pos：指定从数据库从哪个位置开始复制数据，通过查看主数据的状态，获取Position参数；
master_connect_retry：连接失败重试的时间间隔，单位为秒。


3
在主数据库中查看 主从同步 
show master status; 

![](image/Pasted%20image%2020240208162633.png)


在从机中数据库中查看主从同步状态
show slave status \\G;

![](image/Pasted%20image%2020240208162652.png)

![](image/Pasted%20image%2020240208161705.png)

在从数据库中开启主从同步
![](image/Pasted%20image%2020240208161823.png)

查看从数据库状态发现已经同步
![](image/Pasted%20image%2020240208161834.png)

·	主从复制测试
·	主机新建库-使用库-新建表-插入数据，ok
·	从机使用库-查看记录，ok

# 6 安装redis集群模式-docker版 

![](image/Pasted%20image%2020240208162020.png)


2亿条记录就是2亿个k,v，我们单机不行必须要分布式多机，假设有3台机器构成一个集群，用户每次读写操作都是根据公式：

2亿条记录就是2亿个k,v，我们单机不行必须要分布式多机，假设有3台机器构成一个集群，用户每次读写操作都是根据公式：
hash(key) % N个机器台数，计算出哈希值，用来决定数据映射到哪一个节点上。

优点：
简单粗暴，直接有效，只需要预估好数据规划好节点，例如3台、8台、10台，就能保证一段时间的数据支撑。使用Hash算法让固定的一部分请求落到同一台服务器上，这样每台服务器固定处理一部分请求（并维护这些请求的信息），起到负载均衡+分而治之的作用。

缺点：
原来规划好的节点，进行扩容或者缩容就比较麻烦了额，不管扩缩，每次数据变动导致节点有变动，映射关系需要重新进行计算，在服务器个数固定不变时没有问题，如果需要弹性扩容或故障停机的情况下，原来的取模公式就会发生变化：Hash(key)/3会变成Hash(key) /?。此时地址经过取余运算的结果将发生很大变化，根据公式获取的服务器也会变得不可控。
某个redis机器宕机了，由于台数数量变化，会导致hash取余全部数据重新洗牌。


## 6.1 一致性Hash算法背景

分布式缓存数据变动和映射问题，某个机器宕机了，分母数量改变了，自然取余数不OK了。

提出一致性Hash解决方案。 目的是当服务器个数发生变动时， 尽量减少影响客户端到服务器的映射关系

1 算法构建一致性哈希环
一致性哈希环
一致性哈希算法必然有个hash函数并按照算法产生hash值，这个算法的所有可能哈希值会构成一个全量集，这个集合可以成为一个hash空间`[0,2^32-1]`，这个是一个线性空间，但是在算法中，我们通过适当的逻辑控制将它首尾相连(0 = 2^32),这样让它逻辑上形成了一个环形空间。
它也是按照使用取模的方法，前面笔记介绍的节点取模法是对节点（服务器）的数量进行取模。而一致性Hash算法是对2^32取模，简单来说，一致性Hash算法将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为0-2^32-1（即哈希值是一个32位无符号整形），整个哈希环如下图：整个空间按顺时针方向组织，圆环的正上方的点代表0，0点右侧的第一个点代表1，以此类推，2、3、4、……直到2^32-1，也就是说0点左侧的第一个点代表2^32-1， 0和2^32-1在零点中方向重合，我们把这个由2^32个点组成的圆环称为Hash环。

![](image/Pasted%20image%2020240208162537.png)


2 服务器IP节点映射
节点映射
将集群中各个IP节点映射到环上的某一个位置。
将各个服务器使用Hash进行一个哈希，具体可以选择服务器的IP或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。假如4个节点NodeA、B、C、D，经过IP地址的哈希函数计算(hash(ip))，使用IP地址哈希后在环空间的位置如下：  

![](image/Pasted%20image%2020240208162544.png)

3 
key落到服务器的落键规则
当我们需要存储一个kv键值对时，首先计算key的hash值，hash(key)，将这个key使用相同的函数Hash计算出哈希值并确定此数据在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器，并将该键值对存储在该节点上。
如我们有Object A、Object B、Object C、Object D四个数据对象，经过哈希计算后，在环空间上的位置如下：根据一致性Hash算法，数据A会被定为到Node A上，B被定为到Node B上，C被定为到Node C上，D被定为到Node D上。

![](image/Pasted%20image%2020240208162719.png)

## 6.2 一致性Hash算法的 优点 

优点: 一致性哈希算法的容错性
假设Node C宕机，可以看到此时对象A、B、D不会受到影响，只有C对象被重定位到Node D。一般的，在一致性Hash算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即沿着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。简单说，就是C挂了，受到影响的只是B、C之间的数据，并且这些数据会转移到D进行存储。

![](image/Pasted%20image%2020240208162749.png)


优点: 一致性哈希算法的扩展性
数据量增加了，需要增加一台节点NodeX，X的位置在A和B之间，那收到影响的也就是A到X之间的数据，重新把A到X的数据录入到X上即可，
不会导致hash取余全部数据重新洗牌。

![](image/Pasted%20image%2020240208162804.png)


缺点: 一致性哈希算法的数据倾斜问题
Hash环的数据倾斜问题
一致性Hash算法在服务节点太少时，容易因为节点分布不均匀而造成数据倾斜（被缓存的对象大部分集中缓存在某一台服务器上）问题，
例如系统中只有两台服务器：

![](image/Pasted%20image%2020240208162906.png)


总结 

为了在节点数目发生改变时尽可能少的迁移数据
 
将所有的存储节点排列在收尾相接的Hash环上，每个key在计算Hash后会顺时针找到临近的存储节点存放。
而当有节点加入或退出时仅影响该节点在Hash环上顺时针相邻的后续节点。  
 
优点
加入和删除节点只影响哈希环中顺时针方向的相邻的节点，对其他节点无影响。
 
缺点 
数据的分布和节点的位置有关，因为这些节点不是均匀的分布在哈希环上的，所以数据在进行存储时达不到均匀分布的效果。

## 6.3 哈希槽分区

![](image/Pasted%20image%2020240208163333.png)

哈希槽实质就是一个数组，数组`[0,2^14 -1]`形成hash slot空间。

2 能干什么
解决均匀分配的问题，在数据和节点之间又加入了一层，把这层称为哈希槽（slot），用于管理数据和节点之间的关系，现在就相当于节点上放的是槽，槽里放的是数据。
![](image/Pasted%20image%2020240208163350.png)

槽解决的是粒度问题，相当于把粒度变大了，这样便于数据移动。
哈希解决的是映射问题，使用key的哈希值来计算所在的槽，便于数据分配。

3 多少个hash槽
一个集群只能有16384个槽，编号0-16383（0-2^14-1）。这些槽会分配给集群中的所有主节点，分配策略没有要求。可以指定哪些编号的槽分配给哪个主节点。集群会记录节点和槽的对应关系。解决了节点和槽的关系后，接下来就需要对key求哈希值，然后对16384取余，余数是几key就落入对应的槽里。slot = CRC16(key) % 16384。以槽为单位移动数据，因为槽的数目是固定的，处理起来比较容易，这样数据移动问题就解决了。
 
4 哈希槽计算
 
Redis 集群中内置了 16384 个哈希槽，redis 会根据节点数量大致均等的将哈希槽映射到不同的节点。当需要在 Redis 集群中放置一个 key-value时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，也就是映射到某个节点上。如下代码，key之A 、B在Node2， key之C落在Node3上

![](image/Pasted%20image%2020240208163423.png)


![](image/Pasted%20image%2020240208163429.png)



## 6.4 3主3从redis集群扩缩容配置

![](image/Pasted%20image%2020240208163905.png)

![](image/Pasted%20image%2020240208163950.png)


1 新建6个docker容器redis实例

```

docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6382
 
docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6383
 
docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6384
 
docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6385
 
docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis:6.0.8 --cluster-enabled yes --appendonly yes --port 6386

```

![](image/Pasted%20image%2020240208164142.png)

·	命令分步解释
·	docker run
·	创建并运行docker容器实例

·	--name redis-node-6
·	容器名字

·	--net host
·	使用宿主机的IP和端口，默认

·	--privileged=true
·	获取宿主机root用户权限

·	-v /data/redis/share/redis-node-6:/data
·	容器卷，宿主机地址:docker内部地址

·	redis:6.0.8
·	redis镜像和版本号

·	==--cluster-enabled yes==
·	开启redis集群
·	--appendonly yes
·	开启持久化
·	--port 6386
·	redis端口号
·	进入容器redis-node-1并为6台机器构建集群关系



2 构建主从关系
//注意，进入docker容器后才能执行一下命令，且注意自己的真实IP地址
redis-cli --cluster create 192.168.111.147:6381 192.168.111.147:6382 192.168.111.147:6383 192.168.111.147:6384 192.168.111.147:6385 192.168.111.147:6386 --cluster-replicas 1
 
--cluster-replicas 1 表示为每个master创建一个slave节点
 

![](image/Pasted%20image%2020240208164303.png)

![](image/Pasted%20image%2020240208164314.png)


![](image/Pasted%20image%2020240208164318.png)


3 进入容器
docker exec -it redis-node-1 /bin/bash

4 查看 集群状态 

cluster info
cluster nodes

链接进入6381作为切入点，查看集群状态
链接进入6381作为切入点，查看节点状态



![](image/Pasted%20image%2020240208164451.png)

![](image/Pasted%20image%2020240208164455.png)



## 6.5 主从容错切换迁移


·	数据读写存储
·	启动6机构成的集群并通过exec进入
·	对6381新增两个key
·	防止路由失效加参数-c并新增两个key

![](image/Pasted%20image%2020240208164600.png)

加入参数-c，优化路由
![](image/Pasted%20image%2020240208164621.png)

·	查看集群信息
redis-cli --cluster check 192.168.111.147:6381

![](image/Pasted%20image%2020240208164646.png)

![](image/Pasted%20image%2020240208164654.png)


## 6.6 主从扩容

不看了 自己去看 word
## 6.7 主从缩容


不看了 自己去看 word