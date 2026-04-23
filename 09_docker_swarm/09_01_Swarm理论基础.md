
# 1 简介


![](image/Pasted%20image%2020240215213642.png)

Docker Swarm is built into the Docker Engine. Do not Docker Swarm mod with Docker Classic Swarm which is no longer actively developed 

# 2 工作模式

节点和服务

一个虚拟机上 可以有多个 docker node . 但是 一个虚拟机上只有有一个激活状态的node
一个 node 上可以跑多个任务

![](image/Pasted%20image%2020240213180311.png)

Worker 去 实际做工作的 

Manager nodes
Manager nodes handle cluster management tasks:

    Maintaining cluster state
    Scheduling services
    Serving Swarm mode HTTP API endpoints

Worker nodes
Worker nodes are also instances of Docker Engine whose sole purpose is to execute containers. Worker nodes don't participate in the Raft distributed state, make scheduling decisions, or serve the swarm mode HTTP API.


# 3 Service

To deploy an application image when Docker Engine is in Swarm mode, you create a service. Frequently a service is the image for a microservice within the context of some larger application. Examples of services might include an HTTP server, a database, or any other type of executable program that you wish to run in a distributed environment.

When you create a service, you specify which container image to use and which commands to execute inside running containers. You also define options for the service including:

    The port where the swarm makes the service available outside the swarm
    An overlay network for the service to connect to other services in the swarm
    CPU and memory limits and reservations
    A rolling update policy
    The number of replicas of the image to run in the swarm


When you deploy the service to the swarm, the swarm manager accepts your service definition as the desired state for the service. Then it schedules the service on nodes in the swarm as one or more replica tasks. The tasks run independently of each other on nodes in the swarm.

![](image/Pasted%20image%2020240215215012.png)

## 3.1 Tasks and scheduling
The diagram below shows how Swarm mode accepts service create requests and schedules tasks to worker nodes.

![](image/Pasted%20image%2020240215215232.png)

## 3.2 Replicated and global services
服务怎么部署到 节点上的

There are two types of service deployments, replicated and global.

![](image/Pasted%20image%2020240215220052.png)




# 4 例子

1 
docker swarm init --advertise-addr 172.24.82.149

172.24.82.149 是本机私网的地址 

![](image/Pasted%20image%2020240213181256.png)


2 docker swarm join , 加入 管理者 ， 工作者 

获取令牌
docker swarm join-token manager   add a manager to this swarm 
docker swarm join-token worker    join a worker 

3 docker node ls 
![](image/Pasted%20image%2020240213181814.png)

4 创造一个 主节点的 token 令牌 

![](image/Pasted%20image%2020240213182021.png)

![](image/Pasted%20image%2020240213181952.png)
