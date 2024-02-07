

https://docs.docker.com/engine/install/centos/
# 1 前期准备

查看自己的内核
uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。


![](image/Pasted%20image%2020240207161850.png)

# 2 配置 阿里云镜像加速

·	选择容器镜像服务
![](image/Pasted%20image%2020240207162417.png)

·	获取加速器地址
![](image/Pasted%20image%2020240207162427.png)


·	粘贴脚本直接执行
mkdir -p /etc/docker
vim  /etc/docker/daemon.json
```
 
 
 #阿里云
{
  "registry-mirrors": ["https://｛自已的编码｝.mirror.aliyuncs.com"]
}

```


重启服务器
·	systemctl daemon-reload
·	systemctl restart docker





