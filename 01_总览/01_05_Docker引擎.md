

# 1 Docker引擎组成
Docker Client, Dockerd, Containerd , Runc， Shim

![](image/Pasted%20image%2020240216160241.png)

![](image/Pasted%20image%2020240216153949.png)

![](image/Pasted%20image%2020240216154358.png)

![](image/Pasted%20image%2020240216155930.png)

Dockerd:  
docker build , docker pull. dopcker run等 与镜像相关的命令， docker 网络， docker compose ,扽等等 都是 Dockerd 来处理的 

Containerd: 
客户端 提交 docker run, 会直接 交割 Containerd, 然后 contained fork 处 一个  runc 进程， 然后把  image 转换成  OCI (开放容器倡议基金会) 样式 ， 

OCI 样式 就是 (开放容器倡议基金会)  这个组织规定的样式

Runc: 
容器进程是 由runc进程 派生出的。 容器进程 是 runc 进程的 子进程 


Shim 垫片



# 2 Docker 引擎发展历史 


![](image/Pasted%20image%2020240216160617.png)

![](image/Pasted%20image%2020240216160754.png)


# 3 Docker 引擎分类 

![](image/Pasted%20image%2020240216160948.png)








