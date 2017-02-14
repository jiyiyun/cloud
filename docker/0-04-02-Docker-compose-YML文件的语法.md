Compose配置简介
===
Compose是对docker命令的封装，默认使用docker-compose.yml文件来指定docker各个命令中所需的参数
示例
---
``` shell
web:
    build: ./web
    ports:
    - "5000:5000"     #把容器5000端口映射到了主机5000端口
    volumes:
    - .:/code         #挂载当前目录
    links             #Web服务通过链接Redis容器来访问后台Redis数据库
    - redis
redis:
    image: redis      #redis数据库服务是通过运行Redis镜像来提供的
```
- 在Docker-compose文件中每个定义的服务至少包含build和image两个命令中的一个，其它命令都是可选的
- 在Docker-compose文件中,build命令对应的是docker build命令
- 在Docker-compose文件中，ports标记对应docker run命令中的-p 选项，volune对应docker run中的-v,links对应 --links选项
通过运行docker-compose build和docker-compose up命令docker-compose.yml文件定义的服务就会运行起来
