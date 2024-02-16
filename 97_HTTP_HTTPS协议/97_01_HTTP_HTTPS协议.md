
# 1 HTTP/HTTPS协议

![](image/Pasted%20image%2020240216165526.png)

![](image/Pasted%20image%2020240216170154.png)

# 2 HTTPS协议
## 2.1 SSL/TLS

SSL: Secure Sockets Layer 安全套接字 协议
TLS: Transport Layer Security  传输层安全协议 

他们主要用于 在 internet 上数据传输的安全性和完整性 


## 2.2 加密验证方式 

![](image/Pasted%20image%2020240216172206.png)

![](image/Pasted%20image%2020240216172256.png)

## 2.3 公钥和私钥


![](image/Pasted%20image%2020240216172503.png)#

公钥和私钥都可以用于 加密解密
公钥加密，私钥解密： 称为信息加密与信息解密
私钥加密， 公钥解密： 称为数字签名与签名验证

# 3 HTTPS通信过程 
## 3.1 明文通信过程 

![](image/Pasted%20image%2020240216183921.png)


## 3.2 使用 数字签名加密

![](image/Pasted%20image%2020240216184208.png)


![](image/Pasted%20image%2020240216184304.png)

## 3.3 钓鱼问题


![](image/Pasted%20image%2020240216185223.png)


## 3.4 使用数字证书通讯 

整个通信过程包含三个阶段： 通信基础构建阶段， 通信关系建立阶段与通信阶段 

![](image/Pasted%20image%2020240216185802.png)

![](image/Pasted%20image%2020240216185844.png)

张三的数字证书中包含： 张三的个人信息以及张三的公钥


![](image/Pasted%20image%2020240216190221.png)
上图中第二个4 是多余的 



























