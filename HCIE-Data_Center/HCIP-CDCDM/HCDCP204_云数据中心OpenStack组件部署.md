FusionSphere 解决方案软件构成

|部件|功能|
|:---|:---|
|FusionCompute|FusionCompute是云操作系统软件，主要负责硬件资源虚拟化，以及对虚拟资源、业务资源、用户资源的集中管理。它采用虚拟计算、虚拟存储、虚拟网络技术，完成计算资源、存储资源、网络资源的虚拟化|
|FusionStorage|FusionStorage是一种与计算高度融合的分布式存储软件，在通用x86服务器上部署该软件后，可以把所有服务器本地硬盘组成一个虚拟存储池，提供块存储功能|
|FusionNetwork|FusionNetwork是提供传统虚拟交换向未来的软件定义网络演进方案，通过VxLAN的二层隧道封装协议的支持，配合由华为SDN controller完成SDN网络自动化配置部署，SLA服务质量控制，多租户隔离和分层，目前由OpenStack中的neutron服务来提供实现|
|FusionSphere OpenStack|FusionSphere OpenStack是华为基于OpenStack Juno版本进行增强，加固后的企业版，对外提供统一的Restful接口；对计算、存储、网络虚拟资源进行集中调度和管理，降低了业务的运行成本，保证系统的安全性和可靠性，协助企业用户构筑安全、绿色、节能的云数据中心|
|FusionSphere OpenStack OM|FusionSphere OpenStack OM作为FusionSphere OpenStack的操作管理界面，主要对FusionSphere的软件和使用的硬件进行全面的监控和管理，实现基础设施运维管理两大核心功能，并向内部运维人员提供运营与管理门户|
|eBackup|eBackup备份软件为所有虚拟化应用提供端到端的备份方案|

角色

- sys-server 控制主机上所有部署的服务的REST接口消息处理
- auth 为DC提供统一的接入鉴权服务
- image 为DC提供镜像管理服务
- controller 在AZ内提供资源管理和调度功能，是OpenStack的管理组件集合
- router 为AZ内的虚拟机提供路由、防火墙服务
- compute 为AZ内虚拟机提供计算资源，并对主机的计算、网络资源进行管理
- blockstorage 为AZ内虚拟机提供块存储资源
- blockstorage-driver 为虚拟机提供逻辑卷分配和管理功能
- swift 提供swift存储管理功能
- database 除celometer(celometer使用mongoDB数据库)的OpenStack服务提供数据库服务

- mongoDB 为Ceilometer提供数据库服务
- zookeeper 提供高可靠、分布式协调功能服务
- rabbitMQ  在AZ范围内提供消息通道服务
- BareMetal 提供裸金属服务器管理功能，包括裸金属服务器的创建和电源管理等功能
- loadBalancer 为AZ内虚拟机提供负载均衡等服务
- sys-client 为主机操作系统提供基本功能服务，该角色不可见

集群模式(最少3个主节点，每个主节点包含全套服务)

精简模式(最少两个主节点，主备，每个主节点包含全套服务，zookeeper和swift至少部署三个节点)

FusionSphere OpenStack网络规划

|类型|VLAN ID|IP地址示例|
|:---|:---|:---|
|external_om|VLAN 4005||
|external_api|VLAN 4004||
|external_base|VLAN 4006|如果RabbitMQ和FNM服务通过external base平面和Fusion Compute平面对接，需要至少规划两个IP地址用于rabbitMQ及FNM|
|internal_base|utage|默认为172.28.0.0/20|