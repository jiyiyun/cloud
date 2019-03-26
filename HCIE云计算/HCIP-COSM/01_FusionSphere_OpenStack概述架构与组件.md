OpenStack项目概述
===

OpenStack是一个云操作系统，用于控制整个数据中心海量计算、存储和网络资源，借助统一视图或OpenStack API进行管理

OpenStack基本设计思想
- 开放 开源，并尽可能重用已有开源项目；不要“重复造轮子”，而是要“站在巨人肩上”
- 灵活 不使用任何不可代替的私有/商业组件；大量使用插件化方式进行架构设计与实现
- 可拓展 由多个独立项目组成；每个项目包含多个独立服务组件；无中心架构；无状态架构

OpenStack与传统虚拟化
===

OpenStack不是虚拟化
- OpenStack只是系统的控制面；OpenStack不包括系统的数据面组件，如Hypervisor、存储和网络设备等

云和虚拟化关键区别
- 云计算(IT能力服务化；按需使用，按量计费；多租户隔离)
- 虚拟化(环境隔离，资源复用；降低隔离损耗，提升运行效率；提供高级虚拟化特性)

虚拟化是实现云计算的技术支撑手段之一，并非云计算核心关注点。

OpenStack不是云
- OpenStack只是构建云的关键组件(内核、骨干、框架、总线)，构建云还需要(cloud Console、Cloud BSS、Cloud OSS)

OpenStack架构与组件
===

OpenStack服务简介
```txt
服务                   项目名称    描述
Block Storage         Cinder     为运行实例而提供的持久性块存储，可插拔驱动架构有助于创建和管理块存储设备
Identity Service      Keystone   为其它服务提供认证和授权服务，为所有服务提供一个端点目录
Image Service         Glance     存储和检索虚拟机磁盘镜像，实例部署时使用此服务
Telemetry Service     Ceilometer 计费、基准、拓展性以及统计等目的提供检测和计量
Orchestration Service Heat       即可使用本地模板，也可以使用AWS CloudFormation模板格式编排多个综合的云应用
```
OpenStack的项目分层
```txt
IaaS+服务        Swift、Trove、Sahara、...... 
系统管理及自动化   Ceilometer、Heat、...... 
IaaS服务         Nova、Glance、Cinder、Neutron、Ironic 
基础公共组件      Database、Keystone、Message Queue
```

Nova综述
---

- Nova是什么？  OpenStack中提供计算资源服务的项目
- Nova负责什么? 1.虚拟机生命周期管理； 2.其它计算资源生命周期管理
- Nova不负责什么？ 1.承载虚拟机的物理主机自身的管理； 2. 全面的系统状态监控
- Nova是OpenStack事实上的最核心项目 1.历史最长，首批两个项目之一；2.功能最复杂，代码量最大；3.大部分集成项目和Nova之间都存在配合关系

Nova管理下的资源类型
- 主要资源：虚拟机(KVM、Xen、Hyper-V、vCenter/vSpere)
- 其它资源：物理机(通过Ironic)、容器(LXC、docker)

Nova逻辑架构-KVM场景
```txt
Nova-api  Nova-conductor Nova-scheduler
              Message Queue
Compute Manager |Nova-compute |Q  |虚
libvirt Driver  |Nova-compute |E  |拟
           Libvirt            |MU |机
              Linux OS    KVM
                 物理主机
```
Nova部署示例
- 无中心结构
- 各组件无本地持久化状态
- 可水平扩展
- 通常将nova-api、nova-scheduler、nova-conductor组件合并部署在控制节点上
- 通过部署多个控制节点实现HA和负载均衡
- 通过增加控制节点和计算节点实现简单方便的系统扩容

Cinder综述
---

- OpenStack的一个组件，从folsom版本从Nova-volume中分离出来
- 为云平台提供统一接口，按需分配的，持久化的块存储服务
- 通过驱动的方式接入不同种类的后端存储(本地存储、网络存储、FCSAN、IPSAN)

Cinder支持的存储后端
```txt
                                                |--DFS  包括ClusterFS、GPFS、PBD(ceph)
                          |--File system based--|
         Software based --|                     |--NFS  通过NFS支持NAS存储
                |         |--Block based    通过LVM支持本地存储
Cinder Plugins -|         
                |         |--Fibre Channel  Cinder通过添加不同厂商的Drivers来支持不同类型和
        Hardware based----|--iSCSI          型号的商业存储设备IBM SVC/Storwize，EMC...
                          |--NFS
```
Cinder逻辑架构
- Cinder client封装cinder提供的rest接口，以CLI形式提供用户使用
- Cinder API对外提供rest API,对操作需求解析，对API进行路由寻找相应的处理方法，包含卷的增删改查、快照增删改查备份、volume type管理，挂载/卸载(Nova调用)等
- Cinder scheduler负责收集backend上报的容量、能力信息，根据设定的算法完成卷到指定cinder-volume的调度
- Cinder volume多节点部署，使用不同的配置文件，接入不同的backend设备，由各厂商插入driver代码与设备交互完成设备容量和能力信息收集、卷操作
- Cinder backup实现将卷的数据备份到其它存储介质(目前SWIFT/Ceph/TSM提供了驱动)
- SQL DB 提供存储卷、快照、备份、service等数据，支持Mysql、PG、MSSQL等SQL数据库

Cinder部署示例：以传统存储为例
- Cinder-API,Cinder-Scheduler,Cinder-Volume可以部署到一个节点上，也可以分别部署
- API采用AA模式，Haproxy作为LB,分别发送请求到多个Cinder-API
- Scheduler也采用AA模式，有rabbitmq以负载均衡模式向3个节点分发任务，并同时从rabbitmq收取cinder volume上报的能力信息，调度时，scheduler通过在DB中预留资源从而保证数据一致性
- Cinder-Volume 也采用AA模式，同时上报同一个backend容量和能力信息，并同时接受请求并进行处理
- Rabbitmq支持主bei主备或集群
- MySQL支持主备或集群

Neutron综述
---

- OpenStack子项目，为VM提供“Network as a Service”服务,在Folsom版本成为核心项目
- OpenStack "三驾马车"之一(计算Nova,块存储Cinder,网络Neutron)

Neutron的价值与优势
- OpenStack里“一切皆服务”，Neutron就是“网络即服务”
- Neutron带来更多可能性。1. 网络类型丰富(Flat,VLAN,GRE,VxLAN); 2.支持复杂拓扑，租户灵活组网； 3.服务与后端解耦，方便引入SDN等新技术。 4. 可扩展框架。 5. 告警网络服务(Router,LB,VPN,FW等)； 6. 持续快速发展

Neutron逻辑架构：主要组件
- Neutron-Server
- Core Plugin
- 各种Advanced Service Plugin(L3 Service Plugin、LB service Plugin、Firewall、VPN)
- 各种Agent(L2(ovs-agent),L3 Agent,DHCP Agent)

Neutron逻辑架构：逻辑层次
```txt
Core REST API、                Extension A REST API、       Extension ... REST API
AuthN、 AuthZ、 Input validation、 Output view
Core Plugin Interface         Service A Plugin Interface、 Service ...Plugin Interface
          |                           |                              |
Core Plugin(Vendor specitic)、 Service A Plugin、            Service ... Plugin
```

Neutron部署示例
```txt
                           |  External Network 192.168.122.0/24
-----------------------------------------------------------
    |               |                      |                  |
   eth0            eth0                   eth0              eth0
Controller Node   Network Node         Compute Node1     Compute Node2

Nova  Keystone  Neutron Server         Neutron openvswitch-agent
Glance Horizon  Neutron ML2 Plugin     Nova-compute      Neutron openvswitch-agent
                Neutron metadata-agent                   Nova-compute 
                Neutron L3/dhcp-agent
eth1   eth2        eth1   eth2           eth1   eth2       eth1  eth2
  |                  |      |              |     |           |     |
  --------------------------|------- ------------|------------     |
 Management 192.168.20.0/24 |                    |                 |
                            ---------------------------------------|
                                      Data 192.168.10.0/24
```
