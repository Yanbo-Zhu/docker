
# 1 解释 

Dockerfile 文件名 D 必须大写 

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

![](image/Pasted%20image%2020240208202403.png)


构建三步骤
·	编写Dockerfile文件
·	docker build命令构建镜像
·	docker run依镜像运行容器实例

# 2 DockerFile构建过程解析

Dockerfile内容基础知识
1：每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2：指令按照从上到下，顺序执行
3：#表示注释. ==大家注意，那个注释不要写在指令右边，要写在上面，也就是不同行，否则会被视为参数==
4：每条指令都会创建一个新的镜像层并对镜像进行提交

Docker执行Dockerfile的大致流程
（1）docker从基础镜像运行一个容器
（2）执行一条指令并对容器作出修改
（3）执行类似docker commit的操作提交一个新的镜像层
（4）docker再基于刚提交的镜像运行一个新容器
（5）执行dockerfile中的下一条指令直到所有指令都执行完成

![](image/Pasted%20image%2020240208213907.png)


小总结
从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，
*  Dockerfile是软件的原材料
*  Docker镜像是软件的交付品
*  Docker容器则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例
Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

# 3 DockerFile常用保留字指令
参考tomcat8的dockerfile入门:   https://github.com/docker-library/tomcat·	

![](image/Pasted%20image%2020240208203328.png)

| x | x |
| ---- | ---- |
| FROM | 基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from<br><br>继承的镜像 不要处于运行状态 <br>FORM 的 image , 最好在本机中已经存在了 <br> |
| MAINTAINER | 镜像维护者的姓名和邮箱地址 |
| RUN | 容器构建时需要运行的命令 <br>RUN是在 docker build时运行<br><br>两种格式<br><br>shell格式<br>RUN yum -y install vim<br>![](image/Pasted%20image%2020240208203143.png)<br>exec格式<br>![](image/Pasted%20image%2020240208203248.png)<br> |
| EXPOSE | 当前容器对外暴露出的端口 |
| WORKDIR | 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点<br><br>docker run -it -p 8080:8080 \<imageID\> bash<br>以交互打开 docker 容器后, 路径直接变为 Workdir 中给入的值<br><br><br> |
| USER | 指定该镜像以什么样的用户去执行，如果都不指定，默认是root |
| ENV | 用来在构建镜像过程中设置环境变量<br><br>ENV MY_PATH /usr/mytest<br>这个环境变量可以在后续的在 dockerfile 中 任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；<br>也可以在其它指令中直接使用这些环境变量，<br>比如：WORKDIR $MY_PATH<br> |
| ADD | 将宿主机目录下的文件拷贝进镜像, 且会自动处理URL和解压tar压缩包 |
| COPY | <br>类似ADD，拷贝文件和目录到镜像中。 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置<br>·	COPY src dest<br>·	COPY ["src", "dest"]<br>·	<src源路径>：源文件或者源目录<br>·	<dest目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。<br><br> |
| VOLUME | 容器数据卷，用于数据保存和持久化工作 |
| CMD | 指定容器启动后的要干的事情<br><br>![](image/Pasted%20image%2020240208203748.png)<br><br>注意 Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，之前那些都不会被执行 <br>CMD 会被 docker run 之后的参数替换<br><br><br>它和前面RUN命令的区别<br>CMD是在docker run 时运行。<br>RUN是在 docker build时运行。<br><br>docker run -it xxx bin/bash <br>相当于 在 dockerfile 的最后一行 加上了 `CMD["bin/bash", "run"]`. 所以原本在 dockerfile 中定义的 最后一行的cmd 不会被执行了. <br><br> |
| ENTRYPOINT | 在执行 docker build 的时候,  ENTRYPOINT中的内容会被自动执行 <br><br>也是用来指定一个容器启动时要运行的命令<br><br>![](image/Pasted%20image%2020240208204150.png)<br><br>![](image/Pasted%20image%2020240209105440.png)<br>上面相当于执行 java -jar zzyy_docker.jar <br><br>类似于 CMD 指令，但是ENTRYPOINT不会被docker run后面的命令覆盖， 而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序<br><br>在执行 docker run 的时候 可以指定 ENTRYPOINT 运行所需要的参数. <br>如果dockerfile 中 存在 多个 ENTRYPOint 指令, 仅最后一个生效<br><br>![](image/Pasted%20image%2020240208221209.png)<br><br><br>ENTRYPOINT可以和CMD一起用，一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参。<br><br>当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是将CMD的内容作为参数传递给ENTRYPOINT指令，他两个组合会变成  `<ENTRYPOINT> <CMD>`<br> |
|  |  |


参考官网Tomcat的dockerfile演示讲解 
官网最后一行命令
我们演示自己的覆盖操作
![](image/Pasted%20image%2020240208203904.png)

## 3.1 ENTRYPOINT 命令格式和案例说明


![](image/Pasted%20image%2020240208204150.png)

ENTRYPOINT可以和CMD一起用，一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参。

当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是将CMD的内容作为参数传递给ENTRYPOINT指令，他两个组合会变成  `<ENTRYPOINT> <CMD>`
![](image/Pasted%20image%2020240208204204.png)

案例如下：假设已通过 Dockerfile 构建了 nginx:test 镜像：
![](image/Pasted%20image%2020240208204213.png)

|  |  |  |
| ---- | ---- | ---- |
| 是否传参 | 按照dockerfile编写执行 | 传参运行 |
| Docker命令 | docker run  nginx:test | docker run  nginx:test -c /etc/nginx/new.conf |
| 衍生出的实际命令 | nginx -c /etc/nginx/nginx.conf<br><br>如 上面那个截图中给出的  | nginx -c /etc/nginx/new.conf |

优点: 在执行docker run的时候可以指定 ENTRYPOINT 运行所需的参数。
注意:  如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

# 4 虚悬镜像

仓库名、标签都是`<none>`的镜像，俗称dangling image

构建或者删除镜像的时候, 出现错误, 到时 repository 和 tag 的值 都是 nonne
强制删除正在运行中的镜像的images文件会出现虚悬镜像情况


Dockerfile写一个

1 vim Dockerfile
from ubuntu
CMD echo 'action is success'
 
2 然后 docker build .   不加 -t xxxx, 然后查询得到的 image 
![](image/Pasted%20image%2020240208211120.png)

3 查看 虚悬镜像
docker image ls -f dangling=true

![](image/Pasted%20image%2020240208211149.png)


4 删除
 
docker image prune: 只删除虚悬镜像 
使用docker image prune无法删除的，可以使用docker ps -a查找并删除使用该镜像的容器，然后在使用docker image prune删除虚悬镜像
docker image prune -a清理无容器使用的镜像

所有镜像都删除，包括虚悬镜像。所有的 docker system prune -a

虚悬镜像已经失去存在价值，可以删除

![](image/Pasted%20image%2020240208211255.png)

# 5 例子

## 5.1 自定义镜像mycentosjava8

要求: Centos7镜像具备vim+ifconfig+jdk8

1 
准备编写Dockerfile文件
![](image/Pasted%20image%2020240208210608.png)

```
# 若是报错可以下不用最新的centos  里面FROM centos可以改为例如: FROM centos:7.6.1810
FROM centos
MAINTAINER zzyy<zzyybs@126.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java

#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
#把 当前路径下 jdk-8u171-linux-x64.tar.gz 解压, 打进 路径 /usr/local/java/ 中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/  

#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash

```


大写字母D

2 
构建:   docker build -t 新镜像名字:TAG .
注意，上面TAG后面有个空格，有个点
==.号是指镜像构建时打包上传到Docker引擎中的文件的目录,不是本机目录==,  将当前目录的所有文件 都 打包这个 一个 docker image
`docker build -t centosjava8:1.5 .`


![](image/Pasted%20image%2020240208210650.png)

![](image/Pasted%20image%2020240208210659.png)


3 
docker run -it 新镜像名字:TAG
 docker run -it centosjava8:1.5 /bin/bash
![](image/Pasted%20image%2020240208210957.png)

4 再体会下UnionFS（联合文件系统）
UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



## 5.2 自定义镜像myubuntu
准备编写DockerFile文件

```
FROM ubuntu
MAINTAINER zzyy<zzyybs@126.com>
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
RUN apt-get update
RUN apt-get install net-tools
#RUN apt-get install -y iproute2
#RUN apt-get install -y inetutils-ping
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "install inconfig cmd into ubuntu success--------------ok"
CMD /bin/bash

```

构建: docker build -t 新镜像名字:TAG .
运行:  docker run -it 新镜像名字:TAG



## 5.3 jar 包 和 java 环境打包在一起 


```

# 基础镜像使用java
FROM java:8
# 作者
MAINTAINER zzyy

# VOLUME 指定临时文件目录为/tmp，在主机/var/lib/docker目录下创建了一个临时文件并链接到容器的/tmp
VOLUME /tmp

# 将宿主机当前路径下面的 jar包, 添加到容器中, 并更名为zzyy_docker.jar
ADD docker_boot-0.0.1-SNAPSHOT.jar zzyy_docker.jar

# 运行jar包
RUN bash -c 'touch /zzyy_docker.jar'
# 在执行 docker build 的时候,  ENTRYPOINT中的内容会被自动执行 
ENTRYPOINT ["java","-jar","/zzyy_docker.jar"]  

#暴露6001端口作为微服务
EXPOSE 6001

```


将微服务jar包和Dockerfile文件上传到同一个目录下/mydocker
docker build -t zzyy_docker:1.6 .

运行容器
docker run -d -p 6001:6001 zzyy_docker:1.6
