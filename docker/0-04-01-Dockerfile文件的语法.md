Dockerfile文件的语法
===
Dockerfile文件注释都是以#开头的，每一行一条指令
一般情况下，Dockerfile文件由4部分组成：
- 基础镜像             FROM <image>:<tag>
- 维护者信息           MAINTAINER  your-name yourname@email.com
- 镜像操作指令         RUN apt-get update && apt-get install -y nginx
- 容器启动指令         CMD /usr/sbin/nginx

Docekrfile文件是一种被docker程序解释的脚本，Dockerfile由一条条指令组成，Docker程序将Dockerfile指令翻译成为真正的Linux命令
相对于image这种黑盒子，Dockerfile更能让人看清楚镜像是怎么产生的
语法
---
Dockerfile忽略大小写，建议用大写，用#作为注释，每行只支持一条命令，每条命令可以携带多个参数。
Dockerfile指令根据作用分为两种，构建指令和设置指令。构建指令用于构建image,其指定的操作不会在运行image的容器上执行；设置指令用于设置image的属性，其指定的操作将在运行image的容器中执行。
(1) FROM （指定基础image)
---
构建指令，必须指定且需要在dockerfile其他指令的前面，后续指令依赖与此基础image,FROM指定的镜像可以是官方仓库也可以是本地仓库
格式： FROM <image>:<tag>

(2) MAINTAINER  (用来指定创建者信息)
---
格式： MAINTAINER <name>

(3) RUN 指令
---
格式： RUN <command>
该命令是用来执行shell命令的，当遇到RUN指令的时候，Docker会将该指令翻译为 "/bin/sh-c "xxx"" ，其中xxx 为RUN后面的shell命令

(4) expose指令
---
格式： EXPOSE <port>[<port> ...]
将容器端口暴露出来，也可以用"docker run -p" 实现服务器端口映射

(5) CMD 命令
---
该指令有三种形式
- CMD ["executable","param1",....] 使用exec执行
- CMD command param1,... 在/bin/sh 中执行，提供给需要交互的应用
- CMD ["param1",...] 提供给ENTRYPOINT 的默认参数
启动容器时执行的命令，每个Dockerfile只能有一条CMD命令，如果指定了多条只执行最后一条。如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的命令。

(6) ENTRYPOINT 指令
---
有两种格式: 
- ENTRYPOINT ["executable","param1" ...]
- ENTRYPOINT command param1 param2 （shell中执行）
每个Dockkerfile 中只能有一个ENTRYPOINT ,指定多个时，最后一个有效

(7) VOLUME 指令
---
格式: VOLUME ["/data"]
一般用来存放需要永久保留的数据，数据库等
如果是本机目录，使用 docker run -v $HOSTPATH:$CONTAINERPATH

(8) ENV 指令
---
格式： ENV <key><value>
指定一个环境变量，会被后续RUN指令使用，并在容器内保持

(9) ADD 指令
---
格式： ADD <src><dest>
复制指定的<src> 到<dest> 其中<src>可以是Dockerfile所在目录的一个相对路径，也可以是一个URL,还可以是一个tar文件<自动解压目录>

(10) COPY 指令
---
格式：COPY <src><dest>
复制本地的<src> 到容器的<dest>,当使用本地目录为源目录时，推荐使用COPY

参考资料
---
- 《docker 进阶与实战》华为docker实践小组
- [如何使用Dockerfile构建镜像 ](http://blog.csdn.net/we_shell/article/details/38445979)http://blog.csdn.net/we_shell/article/details/38445979
