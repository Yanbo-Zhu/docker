
# 1 集群搭建

需求

![](image/Pasted%20image%2020240215220228.png)


克隆主机

![](image/Pasted%20image%2020240215220302.png)

## 1.1 创建集群步骤

![](image/Pasted%20image%2020240215221548.png)

---


1 
docker info 查 swarm 有没有开启

![](image/Pasted%20image%2020240215220602.png)

---
2 
docker swarm init 
第一次创建的时候， 这个 docker node 就自动为 manager 

![](image/Pasted%20image%2020240215221025.png)


**添加 worker**
在其他的 docker node 中运行 这个 docker swarm join --token 就是 将这个 docker node 作为 worker 加入 这个 被当作 manager 的 docker node 中 
![](image/Pasted%20image%2020240215221311.png)
在其他的 docker node 中运行 docker warm join-token worker 也可以
![](image/Pasted%20image%2020240215221749.png)


**添加manager**
在 其他的 docker node 运行 docker swarm join-tokenmanager 将这个 docker node 作为 manager  加入 这个 被当作 manager 的 docker node  的 swarm 中 
![](image/Pasted%20image%2020240215221446.png)

![](image/Pasted%20image%2020240215221454.png)

---

3 在运行 docker info

可以看到 三个 manager 的 地址 
![](image/Pasted%20image%2020240215221538.png)



---
4 docker node ls 

这个命令 只能在 swarm manager 上运行 
![](image/Pasted%20image%2020240215222321.png)

![](image/Pasted%20image%2020240215222122.png)

星号表示当前主机 


# 2 node 退出swarm cluster 以及再次加入 

## 2.1 worker节点的操作

1 退群
docker swarm leave


![](image/Pasted%20image%2020240215223901.png)


2 再次加入
![](image/Pasted%20image%2020240215223922.png)


3 删除节点 docker node rm nodeID


## 2.2 manager 节点的操作 


1 需要使用 docker swarm init --force-new-cluster 才能使得一个 manager node 退出 swarm cluster 
docker swarm leave --force 可以强制使得 一个 manager node 退出 一个 cluster, 但是不建议这样做 
![](image/Pasted%20image%2020240215224257.png)
还可以 让manager 降成 worker, 这样退出 node 就不会报错了 



# 3 Swarm 自动锁定

![](image/Pasted%20image%2020240215225011.png)

下面的操作 都是针对 某个 机器上的 docker 上的操作。 就是针对某个docker 节点的操作

## 3.1 查是否自动锁定了 

docker info 中的 autolock manager 

![](image/Pasted%20image%2020240215230051.png)

## 3.2 开启自动锁定

在 dock swarm init 的时候 可以开启 --autolock=true 

![](image/Pasted%20image%2020240215230138.png)

---
在 docker swarm init 没有开启 autolock 
可以通过 docker swarm update --autolock=true 添加 
![](image/Pasted%20image%2020240215230320.png)

结果的密钥 需要记好。 
没有他 就不能重启 manager
![](image/Pasted%20image%2020240215230401.png)

## 3.3 关闭自动锁定 
docker swarm update --autolock=true



## 3.4 unlock-key 的值查询
只能在 manager node 上面 运行 
docker swarm unlock-key 

![](image/Pasted%20image%2020240215231045.png)


## 3.5 swarm 解锁

autolock=true 后 
执行 systemctl start docker, 然后 再执行 systemctl stop docker ， 就会在 这个 docker node 上出现 swarm: locker 的状态 

swarm : locker 后 
![](image/Pasted%20image%2020240215231152.png)


需要执行 docker swarm unlock ，　然后输入key 值

![](image/Pasted%20image%2020240215231239.png)



# 4 swarm节点维护

## 4.1 升级为 manager docker node promote NodeID 

![](image/Pasted%20image%2020240215232337.png)


## 4.2 降级为worker  docker node demote NodeID

![](image/Pasted%20image%2020240215232416.png)

## 4.3 docker node udpate 

![](image/Pasted%20image%2020240215232552.png)


1 --role 
将 docker2 提升为 manager
docker node update --role manager docker2

将 docker2 降为 worker
docker node update --role worker docker2


2 label
加上标签， 获得更多节点的信息 

docker node update --label-add auth=zs --label-add email=zs@163.x docker2
docker node update --label-rm auth docker2

## 4.4 docker node insepct NodeID

![](image/Pasted%20image%2020240215233025.png)


可以看到 Label: {}


## 4.5 docker node rm NodeID
这个命令 不是 真正的 让这个 Node 退出 swarm cluster
只有 命令 docker swarm leave -f 或者 docker node rm NodeID -f 才是真正的让这个 node 推出了 swarm cluster 
docker node rm -f 会使得一个节点强制退群， 而 docker swarm leave 命令 是使得当前的 docker 主机关闭 swarm 模式 


### 4.5.1 有问题的删除
1
active 的node 无法用 docker node rm, 会报错 
down 状态的 node 才能用 docker node rm 
使用 systemctel stop docker 后 docker 状态的变为 down 

然后 这个 删除的 node 的 状态是 
swarm: pending 

![](image/Pasted%20image%2020240215233636.png)


2 
也可以强制删除 docker node rm -f NodeID
但是 删除后， 虽然 这个 node 推出了 swarm cluster , 但是 
他的 swarm的状态 为 active 
![](image/Pasted%20image%2020240215233805.png)


### 4.5.2 正确的删除
3 不管节点的 swarm 状态是 pending 还是 acitve ,  这个 node 的信息 还存在留在 swarm cluster 中。 这时候 想要 再将这个 node 加入到 cluster 中， 就需要 先 
在这个已经 rm 的docker node 上 先 docker swarm leave, 将 swarm 状态变为 inactive 
然后再 docker swarm join-token worker 

docker node rm -f 会使得一个节点强制退群， 而 docker swarm leave 命令 是使得当前的 docker 主机关闭 swarm 模式 



### 4.5.3 强制删除  docker swarm leave -f 


docker node rm -f 会使得一个节点强制退群， 而 docker swarm leave 命令 是使得当前的 docker 主机关闭 swarm 模式 



 docker swarm leave -f  只能删除 worker 节点。 对于 manager 节点 ， 强制删除也不能删除 
![](image/Pasted%20image%2020240215235259.png)


