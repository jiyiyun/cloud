OpenStack为管理虚拟机、容器和裸机提供了一个软件平台
课程目标
1. 了解Neutron组件与网络虚拟化原理
2. 了解Cinder组件与创卷流程
3. 了解Nova服务原理与虚拟机创建流程
4. 了解Ironic服务原理与裸金属实例申请流程

1. Neutron服务与网络虚拟化实现原理
===

Neutron逻辑架构
-            Neutron-server
- service plugin    service plugin ......  <-----> DB
-                        Queue
- Neutron-sriov-nic-agent  Neutron-openvswitch-agent  Neutron-dhcp-agent  Neutron-Metadata-agent Neutron-vc-vswitch-agent Neutron-L3-ac-agent Neutron-huawei-ac-agent LB-agent vcenter
- AC2.0

Neutron组件说明

组件  功能
- Neutron-server           接收REST请求，向keystone鉴权，与数据库交互，提供网络对象的API
- Neutron-dbcp-agent       提供DHCP服务
- Neutron-metadata-agent   为虚拟机访问metadata服务提供网络通道
- Neutron-openvswitch-agent配置openvswitch流表，提供二层转发路径
- Neutron-vc-vswitch-agent 对接vmware,将neutron网络模型转换为vmware的网络模型，形成统一虚拟网络
- Neutron-sriov-nic-agent  支持SRIOV网卡虚拟化
- Neutron-huawei-ac-agent  对接AC2.0控制器，将Neutron中L2网络信息传递给AC2.0控制器
- Neutron-l3-ac-agent      对接AC2.0控制器，将Neutron中L3网络信息传递给AC2.0控制器

网络虚拟化实现原理
```txt
虚拟化视角
192.168.1.5/24  192.168.1.10/24  192.168.1.5/24  192.168.1.10/24
       VM              VM              VM             VM
vlan100------------------------  -------------------------------vlan200

物理视角
       VM              VM              VM             VM
    linux-br        linux-br        linux-br        linux-br
           br-int                            br-int
     br-nic1         br-nic2         br-nic3         br-nic4
         |              |               |                |
Phy-nic1----------------|--------------------------------|--- vlan 100-199
Phy-nic2 ---------------|--------------------------------|--- vlan 200-299
```
Neutron将虚拟网络对象模型在物理网络上实现
1. 在linux-br上配置iptables规则，实现安全组
2. 在openvswitch网桥上，配置流表规则，为不同的端口设置不同的VLAN tag,实现虚拟机的流量VLAN隔离
3. 为网卡命名，Neutron将虚拟网络的流量导出网卡

Neutron VLAN隔离
```txt
       VM              VM              VM             VM
vlan100------------------------  -------------------------------vlan200

------------------------------------------------------------

       VM              VM              VM             VM
            vSwitch                         vSwitch
vlan100        |    vlan200            vlan100 | vlan200
---------------|-------------------------------|-------------

1. 物理网络使用vlan隔离
2. Neutron通过配置vSwitch，将一个虚拟网络映射到一个VLAN网络
3. 虚拟机报文，经过vSwitch，添加上VLAN Tag,发送到物理网络或vSwitch内部交换，达到虚拟网络隔离效果
```
Neutron Vxlan网络隔离
```txt
       VM              VM              VM             VM
VNI10000----------------------  -----------------------------VNI20000
提供虚拟网络

------------------------------  -------------------------------

       VM              VM              VM             VM
            vSwitch                         vSwitch
    vlan100    |    vlan200            vlan300 | vlan400
VTEP网络       TOR                             TOR
    VNI10000       VNI20000          VNI10000       VNI20000

Neutron将虚拟网络映射到物理网络
```
1. 租户网络在HOST内部用VLAN隔离
2. TOR交换机由AC2.0控制，完成VLAN和VxLAN的相互转换，相同的VNI在不同HOST可以映射不同的VLANID


Neutron DHCP服务
```txt

        VM          VM           DHCP Server
         |           |                |        提供DHCP服务
-------------------------------------------------------
    
        VM                          VM namespace
     vSwitch                           vSwitch
HOST     |                         HOST   |
VLAN 100 |                        VLAN 100| DHCP报文交互
---------|--------------------------------|-------------------

Neutron使用dnsmasq软件为虚拟机提供DHCP服务
```
1. 每一个Network对象对应一个DHCP服务(一个命名空间)
2. Neutron在network上创建DHCP端口
3. 命名空间内，使用dnsmasq监听DHCP端口
4. Neutron配置dnsmasq配置文件，将MAC、IP、路由、网关等信息保存

Neutron对接KVM
```txt
               云服务
-----------------------------------------------------------------|
Keystone             OpenStack商用版                    Heat      |
Glance        Nova            Cinder       Neutron    Ceilometer |
Swift     plugin driver  plugin driver  plugin driver Ironic     |
-----------------------------------------------------------------|

                         虚拟化平台KVM
    Server1         server2         server3
Neutron openvswitch-agent
              Neutron openvswitch-agent
                                Neutron dhcp-agent
                                Neutron metadata agent
```
1. 在计算节点上部署Neutron openvswitch-agent
2. 在网络节点上，部署DHCP服务
3. 支持vlan类型网络
4. 支持SRIOV网卡

PCI-SIG 组织发布了 SR-IOV （Single Root I/O Virtualization and sharing） 规范，它定义了一个标准化的机制用以原生地支持实现多个客户机共享一个设备。不过，目前 SR-IOV （单根 I/O 虚拟化）最广泛地应用还是网卡上

AC实现业务下发时网络配置的自动化
- OpenStack与SDN控制器对接，并通过SDN实现传统网络设备管理与自动化部署
- 负载均衡配置下发：直接通过F5插件实现
- 防火墙配置下发：通过FWaaS、Neutron配置下发给AC，AC实现防火墙统一配置

单机房模块总体部署方式

- 1个AC集群节点管理150台交换机
- 1套OpenStack管理512~1k台服务器

Cinder服务原理与创建卷流程
===

- Cinder在folsom版本开始，从Nova中的部分持久性块存储功能nova-volume分离出来
- Cinder核心功能是对卷的管理，允许对卷、卷类型，卷快照，卷备份进行处理。它为后端不同的存储设备提供了统一的接口，不同的块设备服务上在cinder中实现其驱动支持与OpenStack进行整合

cinder逻辑架构
---
```txt
            cinder client
             REST |
SQL DB         Cinder-api----AMPQ---->Cinder-backup
            AMPQ  |
              Cinder-volume---AMPQ--->Cinder-scheduler
```
- Cinder client封装cinder提供的rest接口，以CLI形式供用户使用。
- Cinder API对外提供rest API，对操作需求进行解析，对API进行路由寻找相应的处理方法，包含卷的增删改查(包括从源卷、镜像、快照创建)、快照增删改查、备份、volume type管理、挂载/卸载(nova调用等)
- Cinder scheduler负责收集backend上报的容量、能力信息，根据设定的算法完成卷到指定cinder-volume的调度
- Cinder volume多节点部署，使用不同的配置文件，接入不同的backend设备，由各存储厂商插入driver代码与设备交互完成设备容量和能力信息收集、卷操作
- Cinder backup实现将卷的数据备份到其它存储介质(目前Swift/ceph/TSM提供了驱动)
- SQL DB提供存储卷、快照、备份、service等数据，支持MySQL,PG,MSSQL等SQL数据库

Cinder组件
- Cinder API :cinder模块对外唯一的入口，cinder的endpoint,接收和处理rest请求
- cinder-scheduler:根据调度过滤策略以及权重计算策略，选择出合适的后端来处理任务
- cinder-volume:负责与后端存储进行对接，通过各厂商提供的driver将OpenStack操作转换为存储操作
```txt
                 cinder-api
                     |
               cinder-scheduler
      _______________|_________________
      |              |                |
cinder-colume   cinder-colume   cinder-colume
      |              |                |
      |     LVM volume-driver  VRM volume-driver
Huawei SAN Storage   |                |
               local volume         FC 集群

cinder-volume执行卷、快照相关业务，通过调用不同driver管理不同存储后端
```
Cinder创建卷的流程
1. 用户下发创卷请求{“name”:"test","size":"100","type":"IPSAN"}
2. 校验用户是否具有权限，以及做一些基本校验，quota预占等操作，异步返回卷信息(生成卷id)
3. 将创卷消息投递到schedule队列中
4. Schedule从自己消息队列中消费创卷消息，根据各个volume定期上报能力以及卷信息，选择一个主机进行创卷
5. schedule调度到主机后，消息投递到相应的volume队列中
6. volume从自己的消息队列中消费创卷消息，调用driver的接口进行创卷，更新数据库

KVM场景下使用阵列时挂卷流程
1. 阵列侧添加主机和lun的映射
2. 主机侧扫描scsi总线
3. 多路径生成虚拟磁盘
4. nova调用libvirt接口将磁盘信息添加到xml中



2. Nova服务原理与虚拟机创建流程
===


3. Ironic服务原理与裸金属实例申请流程
===