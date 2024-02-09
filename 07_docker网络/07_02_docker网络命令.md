

# 1 总览

![](image/Pasted%20image%2020240209113307.png)

查看网络
·	docker network ls


删除网络
·	docker network rm XXX网络名字

创建网络连接 
docker newwork create aa_network 
![](image/Pasted%20image%2020240209113330.png)



# 2 查看网络源数据 docker network inspect 
·	docker network inspect  XXX网络名字

例子 docker network inspect brige 
得到的结果是 
![](image/Pasted%20image%2020240209115537.png)

![](image/Pasted%20image%2020240209115521.png)

