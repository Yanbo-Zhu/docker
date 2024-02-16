
# 1 热备冷备温备容灾

- 热备： 就是 多个机器之前已经开器， leader 机器 挂了， 就马上让另一个机器当leader 
- 冷备： 除了 leader 机器 ， 其他机器都没有已经开启、 leader 机器 挂了之后，启动另外的机器， 让他当leader 机器
- 温备：  leader 机器  运行一段时间后， 将其他的机器开启， 把leader机器上的内容同步过去， 然后关闭其他的机器. 每隔一段时间就运行一次这样的流程 


Swarm 的manager 节点 采取热备方式 

![](image/Pasted%20image%2020240216123659.png)

# 2 容灾能力 

那个manager node 被选为 leader  

![](image/Pasted%20image%2020240216123647.png)

# 3 容灾模拟













