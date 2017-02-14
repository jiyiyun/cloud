ubuntu14下的docker是没有service服务。去除每次sudo运行docker命令，需要添加组：
sudo groupadd docker
sudo gpasswd -a ${USER} docker

ubuntu14的febootstrap没有-i命令
Dockerfile中的EXPOSE、docker run --expose、docker run -p之间的区别
Dockerfile的EXPOSE相当于docker run --expose，提供container之间的端口访问。docker run -p允许container外部主机访问container的端口
