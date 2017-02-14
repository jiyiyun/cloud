K8S 单etcd搭建
===
在网上看一篇技术资料，etcd可以不用集群模式，可以使用单etcd模式，这样稳定性更好更容易搭建.

*  1、出现问题：etcd server 和node 时间不同步问题，无论是单节点etcd还是集群etcd 都需要时间同步，否则etcd server 一直会报错，搭建个ntp server ，所有主机时钟同步到这台服务器，问题解决。
*  2、跟etcd集群模式设置的不同，

>    1. k8s集群中只有一台host装etcd即可，或者将etcd单独在k8s集群之外单独的etcd服务器或者集群。
>    2. 创建flannel.conf   参数FLANNELD_ETCD_ENDPOINTS=http://192.168.100.160:2379  就只填写这一台etcd即可，每台都这样设置
>    3. master 主机的apiserver 文件KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.100.160:2379"   就这一台etcd



一、架构
---
master  Ubuntu16.04  192.168.3.160
node1  Ubuntu16.04   192.168.15.160
node2  Ubuntu16.04   192.168.15.161
node2  Opensuse42.1  192.168.0.161

二、安装配置etcd
---
略

三、安装flannel
---
由于flannel目前在ubuntu上面没有apt直接安装包，所以必须创建.service 文件


参考资料
----
- [在Ubuntu16.04集群上手工部署Kubernetes] http://blog.csdn.net/linuxgo/article/details/52105068
