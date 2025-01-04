
# 1 Docker 运行的流程

Docker 的后台 有 Docker daemon 守护进程 
Docker 是一个 C/S 模式的架构，后端是一个松耦合架构，众多模块各司其职

![](image/Pasted%20image%2020240207160432.png)


![](image/Pasted%20image%2020240207160334.png)

![](image/Pasted%20image%2020250104110937.png)

Docker manages images and containers
- Images are stored in registries and layered, that means they build upon each other incrementally; this reduces transferred data significantly if layers are re-used.
- Client and host are not necessarily the same, the client can connect to remotely available API-hosts

Issues
- Images, which contain untrusted software
- Updating „base-images“ to avoid exploitable security holes
- General-purpose images nullify the advantages of having more light-weight deployment artifacts


![](image/Pasted%20image%2020240207160544.png)


![](image/Pasted%20image%2020240207160518.png)



# 2 Docker run 到底干了什么
![](image/Pasted%20image%2020240207162602.png)
