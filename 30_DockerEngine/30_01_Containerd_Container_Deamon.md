

# 1 Konfiguration: transparenten pull through cache

damit lassen sich images beispielsweise von dockerhub benutzen, ohne dass ==extra die URL des caches(local Mirror)== vorangestellt werden muss.


## 1.1 Local Mirror for Docker

“Local Mirror for Docker” 指的是在本地或局域网内搭建一个镜像仓库（Registry）或者缓存代理，来加速 Docker 镜像的拉取过程。它的主要作用是：

1. **加速镜像下载**  
    Docker 官方镜像仓库（Docker Hub）有时因为网络问题访问慢，尤其是在中国或网络环境不佳的地方。通过本地镜像仓库，镜像下载变成局域网内速度很快的操作，大幅提升构建和部署效率。
    
2. **节省带宽和流量**  
    本地缓存了常用镜像后，重复使用时无需每次都从远程仓库拉取，减少对外网的依赖，节省带宽资源。
    
3. **提高稳定性和可靠性**  
    当外网不稳定或访问不到时，本地镜像仓库依然可以保证镜像拉取正常，避免业务中断。
    
4. **安全与合规**  
    通过控制镜像来源和版本，避免下载被篡改的镜像，提高安全性，也方便符合企业合规要求。


典型的“Local Mirror for Docker”实现方式有：
- **私有 Docker Registry**  
    使用 Docker 官方的 Registry 镜像（registry:2）搭建私有仓库，直接存储和管理镜像。
- **缓存代理镜像仓库（如 Harbor、Nexus）**  
    这些工具不仅支持私有仓库，还能作为远程镜像的缓存代理，即首次从远程拉取镜像，缓存到本地，后续再拉就走本地。
- **官方 Docker Hub 镜像加速器**  
    一些云厂商（阿里云、腾讯云、华为云等）提供了 Docker Hub 的加速器地址，可以设置 Docker 客户端使用加速器来拉取镜像。


如何配置一个本地镜像代理示例（简化）

1
- 启动一个本地 Registry 容器： docker run -d -p 5000:5000 --name registry registry:2
- 设置 Docker 客户端使用本地 Registry 镜像：
    - 拉取镜像时写成：docker pull localhost:5000/my-image:tag

2
- 或者通过配置 daemon.json 来使用加速器。



## 1.2 确定 是否从 Local Mirror(Cache Registry) 还是从 source Registry 捉取的 image 

虽然 local Cache Registry 中 已经有了 cached image, 
运行docker pull 的时候  containerd (Container Engine ) 仍然 会先从 source Registry  捉取 metadaten, mache metadatenabgleich . ( in meinem naiven verständnis würde es sinn ergeben, dass die container engine noch metadaten und digests gegen die source registry prüft, denn sonst könnte ein pull-through cache ja beliebige artefakte "injecten", die es eigentlich nicht upstream gibt )

### 1.2.1 方法
1 通过 tcpdump
通过 tcpdumo 进行 Netzwerkverkehr mitschneiden , 然后 看 header 
Ich habe auf meinem Single-Node-Testsystem für DEVOPS-3980 mal den Netzwerkverkehr angeschaut. Auch wenn mein Container-Image aus dem Nexus (Port 14444) geholt wird, ist weiterhin Datenverkehr zu allen 3 IP-Adresse von "registry-1.docker.io" (https) festzustellen. Finde ich gerade etwas überraschend 


2 通过 containerd logs



### 1.2.2 Problem : 不从 Local Mirror(Cache Registry) , 而是直接从 source Registry 捉取的 image 

erste vermutung: containerd macht noch irgendwas mit den hostnames und scheinbar kann schon ein trailing / zu unerwartetem verhalten führen: [https://github.com/containerd/containerd/issues/6634](https://github.com/containerd/containerd/issues/6634 "https://github.com/containerd/containerd/issues/6634")



## 1.3 Einrichten transparenten pull through cache in local machine 

das ist der gängige weg, mit containerd einen transparenten pull through cache einzurichten: (tl;dr: eintrag in hosts.toml des nodes)
[https://github.com/containerd/containerd/blob/main/docs/hosts.md#setup-a-local-mirror-for-docker](https://github.com/containerd/containerd/blob/main/docs/hosts.md#setup-a-local-mirror-for-docker "https://github.com/containerd/containerd/blob/main/docs/hosts.md#setup-a-local-mirror-for-docker")

---

创造  hosts.toml file in the xx host namespace 
Here is a simple example for a default registry hosts configuration. Set config_path = "/etc/containerd/certs.d" in your config.toml for containerd.  Make a directory tree at the config path that includes docker.io as a directory representing the host namespace to be configured. Then add a hosts.toml file in the docker.io to configure the host namespace. It should look like this:
```toml
$ tree /etc/containerd/certs.d
/etc/containerd/certs.d
└── docker.io
    └── hosts.toml

$ cat /etc/containerd/certs.d/docker.io/hosts.toml
server = "https://docker.io"

[host."https://registry-1.docker.io"]
  capabilities = ["pull", "resolve"]
```

---

in hosts.toml 
```
server = "https://registry-1.docker.io"    # Exclude this to not use upstream

[host."https://public-mirror.example.com"]
  capabilities = ["pull"]                  # Requires less trust, won't resolve tag to digest from this host
[host."https://docker-mirror.internal"]
  capabilities = ["pull", "resolve"]
  ca = "docker-mirror.crt"                 # Or absolute path /etc/containerd/certs.d/docker.io/docker-mirror.crt
```


damit lassen sich images beispielsweise von dockerhub benutzen, ohne dass extra die URL des caches vorangestellt werden muss.





## 1.4 Einrichten transparenten pull through cache in EKS  

mit EKS ist das nicht so straight forward, weil wir managed nodes mit defaults benutzen wollen und keine umwege wie custom AMI o.ä. gehen wollen.

----


1 通过 daemonset 
turns out: wir können einfach ein daemonset definieren, das die containerd config updated: [https://rlevchenko.com/2025/01/26/solving-docker-hub-rate-limits-in-kubernetes-with-containerd-registry-mirrors/](https://rlevchenko.com/2025/01/26/solving-docker-hub-rate-limits-in-kubernetes-with-containerd-registry-mirrors/ "https://rlevchenko.com/2025/01/26/solving-docker-hub-rate-limits-in-kubernetes-with-containerd-registry-mirrors/")

das ist deklarativ, k8s native, und benötigt keine zusätzlichen tools. therefore i like it.



---

还可以通过 

- **DaemonSets** that run predefined configuration of each your k8s node
- For AWS-based clusters, **EC2 Launch Template** and it’s user data input
- For AWS-based clusters, **AWS Systems Manager** and _aws:runShellScript_ actio
- You can update the config manually, however, in most cases the cluster nodes have a short lifetime due to autoscaler (_use the shell script from daemonset below, containerd service restart is not required_)






