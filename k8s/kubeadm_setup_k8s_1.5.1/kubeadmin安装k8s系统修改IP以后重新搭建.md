使用kubeadmin安装k8s系统修改IP以后重新搭建
---

1 . 停掉master节点docker 服务

```txt
[root@master ~]# systemctl stop docker
```
2 . 删除etcd配置文件目录 (使用kubeadm安装etcd配置文件一般在master节点上，手工安装的删除所有etcd Server上面的配置文件)

```txt
etcd配置we文件默认路径
[root@master ~]# rm -rf /var/lib/etcd/*
```
3 . 再启动docker服务

```txt
[root@master ~]# systemctl start docker
```
4 . 每个节点使用kubeadm reset 命令重置系统(这步最重要)

```txt
[root@master ~]# kubeadm reset
```
5 . 清理每台主机docker 容器

```txt
[root@master ~]# docker stop $(docker ps -aq)
[root@master ~]# docker rm -v $(docker ps -aq)
```
6 . 使用docker ps再也没有容器生成的情况下使用kubeadm重新搭建k8s系统

```txt
[root@master ~]# kubeadm init --use-kubernetes-version v1.5.1 --api-advertise-addresses=192.168.100.34 --pod-network-cidr 10.1.0.0/16
```
经验：一定要用kubeadm reset来清理系统，否则容器会不断生成

备注：

- etcd 是存储K8s整个系统所有配置文件的，把这个目录删了，系统就不知道创建pod、Service、namespace等所有的配置信息了

- kubelet 是master 和node节点通讯的关键，停掉后node和master就失去通讯