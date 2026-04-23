
# 1 Service 操作 

## 1.1 service 创建 
1 docker service create --name toms --replicas 3 -p 9000:8080 tomcat:8.5.49 
创建了 3 个任务 

![](image/Pasted%20image%2020240216124604.png)


![](image/Pasted%20image%2020240216124704.png)

--- 
不同 option 的用法

```
docker service create \
--name toms \
--replicas 10 \
--update-parallelism 2 \ # 一次更新两个
--update-delay 3s \
--update-max-failure-ratio 0.2 \
--update-failure-action rollback \ # 更新失败了就会滚
--rollback-parallelism 2 \
--rollback-delay 3s \
--rollback-max-failure-ratio 0.2 \
--rollback-failre-action continue \
-p 9000:8000\
tomcat:8.5.39 
```

![](image/Pasted%20image%2020240216150613.png)

---

4 创建 7 个 service 看在那个node 放着
整体人物的分配也是基于负载均衡的 
![](image/Pasted%20image%2020240216125030.png)


![](image/Pasted%20image%2020240216150628.png)


## 1.2 service list
2 docker service ls 查看 有啥service 

![](image/Pasted%20image%2020240216124748.png)

## 1.3 查看 某个 service 的 ps
3 docker service ps toms

![](image/Pasted%20image%2020240216124816.png)



## 1.4 查服务的详情  `docker service inspect <servicename>`

![](image/Pasted%20image%2020240216125453.png)

6 查任务的详情 

某个 service 有 replicas 3个， 就代表 这个 service 下有 3个任务 
![](image/Pasted%20image%2020240216125639.png)

## 1.5 查 service 的logs 
`docker service logs <servicename>`

## 1.6 去查 service  下的某个任务 
用 `docker service inspect <taskname> or <taskID>` 去查 service  下的某个任务 是不行的 

![](image/Pasted%20image%2020240216125752.png)

用 `docker service logs <taskID>` 去查 service  下的某个任务 的日志 是没问题的。 就是查这个容器的日志 
用 `docker service logs <taskname>` 是会报错的， 因为 task的 name  是不唯一的 . task 的 id 是唯一的 

![](image/Pasted%20image%2020240216130134.png)


## 1.7 查看 本机 node 下的 进程   docker node ps 

![](image/Pasted%20image%2020240216130412.png)

## 1.8 查另一个 机器上的 node 的进程 ： docker node ps docker2

![](image/Pasted%20image%2020240216130533.png)

# 2 Service的负载均衡

1 每个 docker node 上 都运行2个 task

![](image/Pasted%20image%2020240216131514.png)


![](image/Pasted%20image%2020240216131732.png)


OSI七层模型与TCP/IP五层模型. OSI分层（7层）：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层

应用层为 第7层 
其中 2，3，4，7 有负载均衡 

service 的负载聚恒 是 网络层的传输 


---

## 2.1 例子操作 

下载 whoami 镜像

![](image/Pasted%20image%2020240216132102.png)


# 3 task 的扩容

有 五个docker node , 3个 task (属于同一个 service)
如何 扩容 使得 这个 service 下的 task 变多 

![](image/Pasted%20image%2020240216133410.png)

两个方法 
<mark>1 docker service update --replicas 4 toms </mark>
<mark> 2 docker service scale toms=7 </mark>



----
一个虚拟机上 可以有多个 docker node . 但是 一个虚拟机上只有有一个激活状态的node
一个 node 上可以跑多个任务

---

## 3.1 暂停节点 Docker node update Node --availability pause 
![](image/Pasted%20image%2020240216133841.png)

暂停某个节点， 这个节点上就不被分有task 了 。 但是原来这个node 上的 task 还在这个 task 上面跑。 但是不在分有新的 task 了
Docker node update Node --availability pause 

## 3.2 将节点上的跑的task 排空task，  docker node update --availability drain docker2 



# 4 task 缩容
Docker node update Node --availability pause  只会影响扩容， 不会影响缩容. 对 缩容不起作用

docker service scale toms=5, 减少这个 service 的 task 数量

![](image/Pasted%20image%2020240216134258.png)

也可以使用 docker node update --availability drain docker2 
将将节点上的跑的task 排空task。 但是总任务量不会减少。 因为这个 任务 会在 其他的 node 上被重启， 任务名相同， 但是 任务id 不同了
![](image/Pasted%20image%2020240216134739.png)

再启动 这个 task : docker start taskID, 这个任务 也不会被抢过来， 到 另一个 docker node 上面跑了。 执行了 docker start taskID 后， 这个任务也不会在被启动 

# 5 docker service update 

## 5.1 docker service update --image tomcat:8.5.39 toms 

更新这个 service 用的 image 

![](image/Pasted%20image%2020240216145802.png)


----


```
docker service create \
--name toms \
--replicas 10 \
--update-parallelism 2 \ # 一次更新两个
--update-delay 3s \
--update-max-failure-ratio 0.2 \
--update-failure-action rollback \ # 更新失败了就会滚
--rollback-parallelism 2 \
--rollback-delay 3s \
--rollback-max-failure-ratio 0.2 \
--rollback-failre-action continue \
-p 9000:8000\
tomcat:8.5.39 
```

![](image/Pasted%20image%2020240216150613.png)


每次更新两个 每次停3s
![](image/Pasted%20image%2020240216150723.png)

## 5.2 docker service update --rollback toms

接着上一个例子

归滚到 之前 image 的版本
![](image/Pasted%20image%2020240216150915.png)


![](image/Pasted%20image%2020240216150921.png)



# 6 服务全局部署

global 模式 和 replicated 模式 
![](image/Pasted%20image%2020240216151550.png)

![](image/Pasted%20image%2020240216151602.png)

1 global 模式
服务全局模式 就看 cluster 中有几个节点， 有几个节点 就 产生这个 service 的几个副本
docker service create --name toms --mode global -p 9000:8080 tomcat8.5.49 

想要扩容 增加 replicas ， 进入副本模式。 在全局模式下是不允许的 
![](image/Pasted%20image%2020240216151344.png)


2  replicated 模式 
1 docker service create --name toms --replicas 3 -p 9000:8080 tomcat:8.5.49 


replicated 模式 如果 某个 节点上没有被分配上 task, 那么这个节点能够访问通吗? 
答案： 可以访问到 详细见 overlay 网络

docker2 的 可用性设置为 pause 
![](image/Pasted%20image%2020240216151958.png)
