

https://www.bilibili.com/video/BV1sb411X7oe?p=174


# 1 网络模型

![](image/Pasted%20image%2020240216152238.png)

![](image/Pasted%20image%2020240216152517.png)



最底下是宿主机的网络， 之上是 docker 虚拟出的 brige 网路， 这之上的 ingress 网络

两个 node 之间通信：
ingress 网路， 通向底层，构建一个隧道 然后 到达另一个node 的 ingress 网路

![](image/Pasted%20image%2020240216152746.png)


# 2 docker_gwbrige网络基础信息


# 3 ingress网络基础信息



# 4 宿主机的NAT过程


# 5 ingress_sbox的负载均衡


# 6 VXLAN





