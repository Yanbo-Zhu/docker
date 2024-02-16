

# 1 下载地点 

![](image/Pasted%20image%2020240216161617.png)



# 2 安装步骤

直接安装 官网的 安装步骤 

![](image/Pasted%20image%2020240216162707.png)


## 2.1 设置下载仓库 

![](image/Pasted%20image%2020240216162744.png)


![](image/Pasted%20image%2020240216162830.png)

![](image/Pasted%20image%2020240216162858.png)

## 2.2 安装



## 2.3 在线安装 

![](image/Pasted%20image%2020240216163031.png)

安装了四部分 
docker-ce  就是  docker 引擎
docker-ce-cli
containerd
docker-compose-plugin  作用是 让docker 引擎可以支持 docker compose 

![](image/Pasted%20image%2020240216163122.png)



--- 
也可以 安装 指定version

![](image/Pasted%20image%2020240216163155.png)

## 2.4 手动安装 rpm 

![](image/Pasted%20image%2020240216163254.png)

## 2.5 用脚本安装 

![](image/Pasted%20image%2020240216163309.png)


# 3 查看 docker 状态 

![](image/Pasted%20image%2020240216163933.png)

systemctl enable  docker : 设置为 docker 会自启动， 随着 host机器 重启 


# 4 卸载 docker引擎

三大步

![](image/Pasted%20image%2020240216164438.png)

1 uninstall Docker Engine 
2 delete images , containers, and volumes
3 delete the edited configuration files manually 







