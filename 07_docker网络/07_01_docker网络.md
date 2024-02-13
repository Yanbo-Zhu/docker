
NAT：NAT服务器和DHCP服务器会对IP进行地址转换，虚拟机通过共享主机IP可以访问外部网络，而外网是不能访问到虚拟机的。

docker 网络到底能干什么 
·	容器间的互联和通信以及端口映射
·	容器IP变动时候可以通过服务名直接网络通信而不受到影响, 不要写死ip 

指定不同的容器 跑在 指定的 网络里面 


# 1 一些辅助命令 

1  ip addr 查看 容器内的网络设置 
docker exec -it tomcat01 ip addr, 不用打开 bash shell, 直接在容器内运行 ip addr 
![](image/Pasted%20image%2020240213152733.png)

2 docker exec -it tomcat02 ping tomcat01
容器之间ping

3  docker exec -it tomcat02 cat /etc/hosts

4 docker run --net 或者 --network



# 2 Docker网络

1 docker不启动，默认网络情况
有三个 
ens33: 为宿主机 linux本地的网卡  192.168.111.167/24
lo:  本地回环链路 127.0.0.1/8 
virbr0: 虚拟网桥链路 

在CentOS7的安装过程中如果有选择相关虚拟化的的服务安装系统后，启动网卡时会发现有一个以网桥连接的私网地址的virbr0网卡(virbr0网卡：它还有一个固定的默认IP地址192.168.122.1)，是做虚拟机网桥的使用的，其作用是为连接其上的虚机网卡提供 NAT访问外网的功能。
 
我们之前学习Linux安装，勾选安装系统的时候附带了libvirt服务才会生成的一个东西，如果不需要可以直接将libvirtd服务卸载，
yum remove libvirt-libs.x86_64

![](image/Pasted%20image%2020240208213649.png)


2 docker 启动后 的网络情况 
我们每启动一个docker容器, docker就会给docker容器分配一个ip, 我们只要安装了docker, 就会有一个网卡docker0
会产生一个名为docker0的虚拟网桥
桥接模式, 就是evth-pair技术 
使用 docker0 : 172.17.0.1 通过 这个网桥 使得 docker 和宿主机联系, 已经 docker 容器之间的相互联系. 


![](image/Pasted%20image%2020240209112443.png)

3 查看docker网络模式命令

![](image/Pasted%20image%2020240209112528.png)



# 3 网络模式

![](image/Pasted%20image%2020240209113506.png)

·	bridge模式：使用--network  bridge指定，默认使用docker0
·	host模式：使用--network host指定
·	none模式：使用--network none指定
·	container模式：使用--network container:NAME或者容器ID指定    . 使用另一个容器的 ip 地址 (用的少 局限很大)

----


==docker容器内部的ip是有可能会发生改变的==

1 先启动两个ubuntu容器实例
![](image/Pasted%20image%2020240209125726.png)

2 docker inspect 容器ID or 容器名字
![](image/Pasted%20image%2020240209125738.png)

3  关闭u2实例，新建u3，查看ip变化
![](image/Pasted%20image%2020240209125748.png)


## 3.1 bridge (veth-pair技术)

<mark>docker0 的问题: 默认 不支持容器名连接访问. 使用 --link 可以打通连接</mark>

我们每启动一个docker容器, docker就会给docker容器分配一个ip, 我们只要安装了docker, 就会有一个网卡docker0
Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为docker0，它在内核层连通了其他的物理或虚拟网卡，==这就将所有容器和本地主机都放到同一个物理网络==, 在同一个网段的话 相互之间就可以自由通信了。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，让主机和容器之间可以通过网桥相互通信。
 
查看 bridge 网络的详细信息，并通过 grep 获取名称项
docker network inspect bridge | grep name
![](image/Pasted%20image%2020240209125935.png)

ifconfig
![](image/Pasted%20image%2020240209125944.png)

![](image/Pasted%20image%2020240213160224.png)

1 Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0)，Docker启动一个容器时会根据Docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。
 
2 docker run 的时候，没有指定network的话默认使用的网桥模式就是bridge，使用的就是docker0。在宿主机ifconfig,就可以看到docker0和自己create的network(后面讲)eth0，eth1，eth2……代表网卡一，网卡二，网卡三……，lo代表127.0.0.1，即localhost，inet addr用来表示网卡的IP地址
 
3 网桥docker0创建一对对等虚拟设备接口一个叫veth，另一个叫eth0，成对匹配。
   3.1 整个宿主机的网桥模式都是docker0，类似一个交换机有一堆接口，每个接口叫veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫veth pair）；
   3.2 每个容器实例内部也有一块网卡 (估计是虚拟出的网卡)，每个接口叫eth0；
   3.3 docker0上面的每个veth匹配某个容器实例内部的eth0，两两配对，一一匹配。

> evth-pair 就是一堆虚拟设备接口, 他们都是承兑出现的, 一段连着协议, 一段彼此相连. evth-pair 充当桥梁 , 连接各种虚拟网络设备 

4 所有容器在不制定网络的情况下, 都是由 docker0路由的, docker 会给容器自动分配一个默认可用的ip 

5 contianer 之间 不是直接联系, 而是要通过 docker0 联系 

6 只要容器已删除, 这个网桥对 就没有 

 通过上述，将宿主机上的所有容器都连接到这个内部网络上，两个容器在同一个网络下,会从这个网关下各自拿到分配的ip，此时两个容器的网络是互通的。
![](image/Pasted%20image%2020240209130420.png)

eth 是 ethernet 的缩写 
ens33 是本机的网卡 
个人理解,大家仔细看会发现桥接连接的是容器和主机的网卡,而网卡内部是有ip地址,主机利用桥接提供的容器ip访问容器,容器利用桥接的主机ip访问主机,达到主机和容器的通信
docker0相当于是一个交换机，每个容器就是一台装了linux操作系统的主机


![](image/Pasted%20image%2020240213154359.png)



![](image/Pasted%20image%2020240213155059.png)
---	
验证两两匹配
docker run -d -p 8081:8080   --name tomcat81 billygoo/tomcat8-jdk8
docker run -d -p 8082:8080   --name tomcat82 billygoo/tomcat8-jdk8

输入 ip addr 
![](image/Pasted%20image%2020240209140811.png)

23 对应 vethxxxx22 

![](image/Pasted%20image%2020240209140723.png)


## 3.2 host

直接使用宿主机的 IP 地址与外界进行通信，不再需要额外进行NAT 转换。

容器将不会获得一个独立的Network Namespace， 而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口。

![](image/Pasted%20image%2020240209141124.png)


----
例子
·	docker run -d -p 8083:8080 --network host --name tomcat83 billygoo/tomcat8-jdk8
![](image/Pasted%20image%2020240209141152.png)

注意上图 port 处 , 为空了 

问题：
     docke启动时总是遇见标题中的警告
原因：
    docker启动时指定--network=host或-net=host，如果还指定了-p映射端口，那这个时候就会有此警告，
并且通过-p设置的参数将不会起到任何作用，端口号会以主机端口号为主，重复时则递增。
解决:
    解决的办法就是使用docker的其他网络模式，例如--network=bridge，这样就可以解决问题，或者直接无视。。。。O(∩_∩)O哈哈~
docker run -d  --network host --name tomcat83 billygoo/tomcat8-jdk8   (不写 -p 8083:8080 了)

无之前的配对显示了，看容器实例内部
![](image/Pasted%20image%2020240209141503.png)

可以和 下图中 用 brige 模式 创造出来的 container 进行比较 
![](image/Pasted%20image%2020240209141811.png)

docker run -d  --network host --name tomcat83 billygoo/tomcat8-jdk8   (不写 -p 8083:8080 了)
上面没有设置-p的端口映射了，如何访问启动的tomcat83？？
http://宿主机IP:8080/    相当于在住加上装了 tomcat 
 
在CentOS里面用默认的火狐浏览器访问容器内的tomcat83看到访问成功，因为此时容器的IP借用主机的，
所以容器共享宿主机网络IP，这样的好处是外部主机与容器可以直接通信。

## 3.3 none

在none模式下，并不为Docker容器进行任何网络配置。 没有 网卡 等信息 
也就是说，这个Docker容器没有网卡、IP、路由等信息，只有一个lo, localhost  表示为本地回环 
需要我们自己为Docker容器添加网卡、配置IP等。
·	禁用网络功能，只有lo标识(就是127.0.0.1表示本地回环)

driver: null , 就是根本没有任何的网络

![](image/Pasted%20image%2020240209135549.png)


----
例子
docker run -d -p 8084:8080 --network none --name tomcat84 billygoo/tomcat8-jdk8
docker run -d -p 8084:8080 --network none --name tomcat84 billygoo/tomcat8-jdk8

 进入容器内部查看
![](image/Pasted%20image%2020240209142145.png)
只有一个lo, 表示为本地回环

在容器外部查看
![](image/Pasted%20image%2020240209142157.png)

## 3.4 container 

container⽹络模式 
新建的容器和已经存在的一个容器共享一个网络ip配置而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。

![](image/Pasted%20image%2020240209142701.png)



-----
例子1

·	docker run -d -p 8085:8080                                     --name tomcat85 billygoo/tomcat8-jdk8
·	docker run -d -p 8086:8080 --network container:tomcat85 --name tomcat86 billygoo/tomcat8-jdk8

![](image/Pasted%20image%2020240209142744.png)

上图显示 相当于tomcat86和tomcat85公用同一个ip同一个端口，导致端口冲突
本案例用tomcat演示不合适。。。演示坑。。。。。。o(╥﹏╥)o
 
换一个镜像给大家演示，


----
例子2
Alpine操作系统是一个面向安全的轻型 Linux发行版
 
 
Alpine Linux 是一款独立的、非商业的通用 Linux 发行版，专为追求安全性、简单性和资源效率的用户而设计。 可能很多人没听说过这个 Linux 发行版本，但是经常用 Docker 的朋友可能都用过，因为他小，简单，安全而著称，所以作为基础镜像是非常好的一个选择，可谓是麻雀虽小但五脏俱全，镜像非常小巧，不到 6M的大小，所以特别适合容器打包。

·	docker run -it                                               --name alpine1  alpine /bin/sh
·	docker run -it --network container:alpine1 --name alpine2  alpine /bin/sh

运行结果，验证共用搭桥
![](image/Pasted%20image%2020240209142857.png)

假如此时关闭alpine1，再看看alpine2
![](image/Pasted%20image%2020240209142913.png)

15: eth0@if16: 消失了。。。。。。关闭alpine1，再看看alpine2
![](image/Pasted%20image%2020240209142926.png)

# 4 容器互联 --link
--link就是在hosts文件内加了一行解析
已经不推荐使用了 

docker run -d -P --name tomcat03 --link tomcat02 tomcat
docker exec -it tomcat03 ping tomcat02, 可以ping通

![](image/Pasted%20image%2020240213155953.png)

但是 反向ping 无法ping通, 因为没有配置 
docker exec -it tomcat02 ping tomcat03 返回无法ping 

查看 host docker exec -it tomcat03 cat /etc/hosts
![](image/Pasted%20image%2020240213161250.png)



# 5 自定义网络 

![](image/Pasted%20image%2020240209143743.png)

自定义的网络, 支持 ping hostname 和 ip了
docker0 的话, 只支持 ping ip, 不支持ping hostname

## 5.1 如何定义一个网络 docker network create xx

```
docker network create --driver bridge --subnet 192.168.0.0/16 --gatewat 192.168.0.0 mynet 
```

![](image/Pasted%20image%2020240213163548.png)

![](image/Pasted%20image%2020240213163918.png)

## 5.2 不定义自定义网络

例子1 

·	docker run -d -p 8081:8080   --name tomcat81 billygoo/tomcat8-jdk8
·	docker run -d -p 8082:8080   --name tomcat82 billygoo/tomcat8-jdk8
·	上述成功启动并用docker exec进入各自容器实例内部

按照IP地址ping是OK的
![](image/Pasted%20image%2020240209144349.png)

![](image/Pasted%20image%2020240209144356.png)

---
因为 ip地址 是变化的, 按照服务名 ping , 不通 

![](image/Pasted%20image%2020240209144400.png)

![](image/Pasted%20image%2020240209144405.png)

## 5.3 使用自定义网路

自定义桥接网络, 自定义网络默认使用的是桥接网络bridge

新建自定义网络
![](image/Pasted%20image%2020240209144455.png)

新建容器加入上一步新建的自定义网络
·	docker run -d -p 8081:8080 --network zzyy_network  --name tomcat81 billygoo/tomcat8-jdk8
·	docker run -d -p 8082:8080 --network zzyy_network  --name tomcat82 billygoo/tomcat8-jdk8
·	互相ping测试

![](image/Pasted%20image%2020240209144519.png)

![](image/Pasted%20image%2020240209144523.png)

问题结论
·	自定义网络本身就维护好了主机名和ip的对应关系（ip和域名都能通）


# 6 不同网段互联 docker network connect 


connect事实上就是写了一个路由条目，写完之后，可以在linux路由表里应该查得到
docker network connect mynet tomcat01, 将mynet 这个网络和 tomcat01 这个 容器 联通 
连通之后 就是将 tomcat01 放到了 mynet 网络下
实质就是 一个容器 两个ip地址, 一个公网ip 一个 私网ip 


---

之间连不同网段 之前的容器 是不可能的 
![](image/Pasted%20image%2020240213165319.png)


解决方式: 
docker network connect 
![](image/Pasted%20image%2020240213165441.png)

![](image/Pasted%20image%2020240213165514.png)

docker network connect mynet tomcat01, 将mynet 这个网络和 tomcat01 这个 容器 联通 
连通之后 就是将 tomcat01 放到了 mynet 网络下

docker network inspect mynet 
![](image/Pasted%20image%2020240213170020.png)


# 7 Docker平台架构图解

整体说明
从其架构和运行流程来看，Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职。 
 
Docker 运行的基本流程为：
1 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。
2 Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Docker Server 的功能使其可以接受 Docker Client 的请求。
3 Docker Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
4 Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 Graph driver将下载镜像以Graph的形式存储。
5 当需要为 Docker 创建网络环境时，通过网络管理驱动 Network driver 创建并配置 Docker 容器网络环境。
6 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 Execdriver 来完成。
7 Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作


![](image/Pasted%20image%2020240209144642.png)


 
