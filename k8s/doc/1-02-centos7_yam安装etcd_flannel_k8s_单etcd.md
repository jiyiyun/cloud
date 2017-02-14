《CentOS 7.2 yum 安装kubernetes,etcd,flannel并配置》《单etcd》 
===

经验总结： 
---
1. 在Renhat或者CentOS上面安装k8s,第一次启用pod的时候会下载registry.access.redhat.com/rhel7/pod-infrastructure:latest镜像，相当于二进制安装需要的pause镜像，这个镜像200M以上，下载需要一段时间，可以提前下载，导入到每个node节点

2. 检查并关闭selinux，firewall


一、安装etcd并配置 （仅一台主机安装，本示例etcd和k8s_master安装在100.11上）
===

1. 安装etcd
---

``` shell
[root@k8s_master ~]# yum install etcd –y
```
2. 配置etcd
---

``` shell
[root@k8s_master ~]# vi /etc/etcd/etcd.conf

# [member]
ETCD_NAME=etcd1
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_WAL_DIR=""
#ETCD_SNAPSHOT_COUNT="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
#ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_PEER_URLS="http://192.168.100.11:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.100.11:2379,http://localhost:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#
#[cluster]
#ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.100.11:2380"
# if you use different ETCD_NAME (e.g. test), set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
#ETCD_INITIAL_CLUSTER="default=http://localhost:2380"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.100.11:2380"
#ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_STATE="new"
#ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.100.11:2379"
#ETCD_DISCOVERY=""
```
3. 启动etcd
---

``` shell
[root@k8s_master ~]# systemctl start etcd
```

二、安装flannel并配置 (master和所有node都安装)
===

1. yum安装flannel
---

``` shell
[root@k8s_master ~]# yum install flannel           
```
2. 修改每台主机的flaneld 配置文件
---

``` shell
[root@k8s_master ~]#vi /etc/sysconfig/flannel

# Flanneld configuration options

# etcd url location.  Point this to the server where etcd runs
FLANNEL_ETCD="http://192.168.100.11:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
#FLANNEL_ETCD_KEY="/atomic.io/network"
FLANNEL_ETCD_KEY="/atomic.io/network"

# Any additional options that you want to pass
#FLANNEL_OPTIONS=""
FLANNEL_OPTIONS="-iface=eno16777736"
```
3. 设置etcd网段 (仅在安装了etcd服务的主机上运行此命令)
---

``` shell
[root@k8s_master ~]# etcdctl set /atomic.io/network/config  '{ "Network": "10.1.0.0/16", "Backend": { "Type": "vxlan", "VNI": 1 } }'
```
4. 启动flannel
---

``` shell
[root@k8s_master ~]# systemctl start flanneld
```
5. flannel设置为开机启动
---

``` shell
[root@k8s_master ~]# systemctl enable flanneld
```
6. 重启docker
---

``` shell
[root@k8s_master ~]# systemctl daemon-reload
[root@k8s_master ~]# systemctl restart docker
```
7. 查看docker是否获取了flannel的IP
---

``` shell
[root@k8s_master ~]# ip a
2: eno16777736: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:6f:00:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.11/24 brd 192.168.100.255 scope global eno16777736
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe6f:4/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:9b:63:6f:c3 brd ff:ff:ff:ff:ff:ff
    inet 10.1.42.1/24 scope global docker0
       valid_lft forever preferred_lft forever
4: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN 
    link/ether 92:1d:cc:42:e7:7d brd ff:ff:ff:ff:ff:ff
    inet 10.1.42.0/16 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::901d:ccff:fe42:e77d/64 scope link 
       valid_lft forever preferred_lft forever
```
8. 测试flannel是否生效
---
``` shell
[root@master ~]# ping 10.1.100.1
PING 10.1.100.1 (10.1.100.1) 56(84) bytes of data.
64 bytes from 10.1.100.1: icmp_seq=1 ttl=64 time=1.67 ms
64 bytes from 10.1.100.1: icmp_seq=2 ttl=64 time=0.987 ms
64 bytes from 10.1.100.1: icmp_seq=3 ttl=64 time=0.697 ms
^C
--- 10.1.100.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.697/1.120/1.677/0.411 ms
[root@master ~]# ping 10.1.46.1
PING 10.1.46.1 (10.1.46.1) 56(84) bytes of data.
64 bytes from 10.1.46.1: icmp_seq=1 ttl=64 time=4.14 ms
64 bytes from 10.1.46.1: icmp_seq=2 ttl=64 time=1.17 ms
^C
--- 10.1.46.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 1.179/2.660/4.141/1.481 ms

两台node节点都ok了
```
9. 配置flannel总结
---
- [root@k8s_master ~]# vi /etc/sysconfig/flanneld                 #这个文件修改2项
- [root@k8s_master ~]# vi /lib/systemd/system/flanneld.service    #这个文件不用配置

三、 安装k8s并配置 (master和node 都安装，设置不同)
===

1. 安装k8s 
---

``` shell
[root@k8s_master ~]# yum install kubernetes -y
```
2. 配置k8s master
---

*  编辑config 文件

``` shell
[root@k8s_master ~]# vi /etc/kubernetes/config

###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://192.168.100.11:8080"
```
*  编辑# vi /etc/kubernetes/apiserver

``` shell
[root@k8s_master ~]# vi /etc/kubernetes/apiserver
###
# kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--insecure-bind-address=0.0.0.0"

# The port on the local server to listen on.
# KUBE_API_PORT="--port=8080"

# Port minions listen on
# KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://192.168.100.11:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.1.0.0/16"

# default admission control policies
KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS="--client-ca-file=/k8s/ca.crt \
               --tls-private-key-file=/k8s/server.key \
               --tls-cert-file=/k8s/server.crt"
```

* 编辑# vi /etc/kubernetes/controller-manager

``` shell
[root@k8s_master ~]#vi /etc/kubernetes/controller-manager
###
# The following values are used to configure the kubernetes controller-manager

# defaults from config and apiserver should be adequate

# Add your own!
KUBE_CONTROLLER_MANAGER_ARGS="--service-account-private-key-file=/k8s/server.key \
                              --root-ca-file=/k8s/ca.crt"
```
* 编辑# vi /etc/kubernetes/scheduler

``` shell
[root@k8s_master ~]# vi /etc/kubernetes/scheduler
###
# kubernetes scheduler config

# default config should be adequate

# Add your own!
KUBE_SCHEDULER_ARGS=""
```

* 将kube-apiserver、kube-controller-manager、kube-scheduler设置为开机启动

``` shell
[root@k8s_master ~]# systemctl enable kube-apiserver
[root@k8s_master ~]# systemctl enable kube-controller-manager
[root@k8s_master ~]# systemctl enable kube-scheduler
```
3. 配置k8s node端
---

*  编辑# vi /etc/kubernetes/config

``` shell
[root@k8s_node1 ~]# vi /etc/kubernetes/config
###
# kubernetes system config
#
# The following values are used to configure various aspects of all
# kubernetes services, including
#
#   kube-apiserver.service
#   kube-controller-manager.service
#   kube-scheduler.service
#   kubelet.service
#   kube-proxy.service
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://192.168.100.11:8080"
```

* 编辑# vi /etc/kubernetes/kubelet

``` shell
[root@k8s_node1 ~]# vi /etc/kubernetes/kubelet
###
# kubernetes kubelet (minion) config

# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=192.168.100.12"

# The port for the info server to serve on
# KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=192.168.100.12"

# location of the api-server
KUBELET_API_SERVER="--api-servers=http://192.168.100.11:8080"

# pod infrastructure container
#KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"

# Add your own!
KUBELET_ARGS=""
```

*  编辑# vi /etc/kubernetes/proxy

``` shell
[root@k8s_node1 ~]# vi /etc/kubernetes/proxy
###
# kubernetes proxy config

# default config should be adequate

# Add your own!
KUBE_PROXY_ARGS=""
```
* 将kubelet 和proxy设置为开机启动

``` shell
[root@k8s_node1 ~]# systemctl start kubelet
[root@k8s_node1 ~]# systemctl start kube-proxy
```
四、验证k8s安装情况
===
1、检查node获取
``` shell
[root@k8s_master ~]# kubectl get nodes
NAME             STATUS    AGE
192.168.100.12   Ready     1h
192.168.100.13   Ready     1h
```
五、附属
===

1、创建证书文件 (master主机上)
--- 
``` shell
[root@k8s_master ~]#mkdir –p /k8s/
[root@k8s_master ~]#cd /k8s/
[root@k8s_master ~]#openssl genrsa -out ca.key 2048
[root@k8s_master ~]#openssl req -x509 -new -nodes -key ca.key -subj "/CN=cloudsoar.com" -days 5000 -out ca.crt
[root@k8s_master ~]#openssl genrsa -out server.key 2048
[root@k8s_master ~]#openssl req -new -key server.key -subj "/CN=k8s_master" -out server.csr
[root@k8s_master ~]#openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 5000
```
2 、安装1个镜像 (每个node节点上)
---
pause 镜像：是k8s自带的pod中容器间互联的容器，pause创建pod要用
- gcr.io/google_containers/pause-amd64:3.0 

- 或者（二者选一，推荐使用k8s的pause 因为redhat的pod-infrastructure太大）

pod-infrastructure redhat安装k8s默认的连接容器的容器，如果使用这个镜像，kubelet配置文件的# pod infrastructure container选项就不用注释掉了
- registry.access.redhat.com/rhel7/pod-infrastructure:latest
CentOS 系统自动下拉这个镜像，这个镜像很大，第一次创建pod用时较长

``` shell
[root@k8s_node1 ~]# docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
registry.access.redhat.com/rhel7/pod-infrastructure   latest              7d5548e9fb99        2 weeks ago         205.3 MB
gcr.io/google_containers/pause-amd64                  3.0                 99e59f495ffa        6 months ago        746.9 kB
```
