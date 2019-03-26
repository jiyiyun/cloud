OpenStack是一个云操作系统，用于控制整个数据中心海量计算、存储和网络资源，借助统一视图或OpenStack API进行管理

OpenStack基本设计思想
- 开放 开源，并尽可能重用已有开源项目；不要“重复造轮子”，而是要“站在巨人肩上”
- 灵活 不使用任何不可代替的私有/商业组件；大量使用插件化方式进行架构设计与实现
- 可拓展 由多个独立项目组成；每个项目包含多个独立服务组件；无中心架构；无状态架构

OpenStack不是虚拟化
- OpenStack只是系统的控制面；OpenStack不包括系统的数据面组件，如Hypervisor、存储和网络设备等

云和虚拟化关键区别
- 云计算(IT能力服务化；按需使用，按量计费；多租户隔离)
- 虚拟化(环境隔离，资源复用；降低隔离损耗，提升运行效率；提供高级虚拟化特性)

虚拟化是实现云计算的技术支撑手段之一，并非云计算核心关注点。

OpenStack不是云
- OpenStack只是构建云的关键组件(内核、骨干、框架、总线)，构建云还需要(cloud Console、Cloud BSS、Cloud OSS)

OpenStack服务简介

- 服务                   项目名称    描述
- Block Storage         Cinder     为运行实例而提供的持久性块存储，可插拔驱动架构有助于创建和管理块存储设备
- Identity Service      Keystone   为其它服务提供认证和授权服务，为所有服务提供一个端点目录
- Image Service         Glance     存储和检索虚拟机磁盘镜像，实例部署时使用此服务
- Telemetry Service     Ceilometer 计费、基准、拓展性以及统计等目的提供检测和计量
- Orchestration Service Heat       即可使用本地模板，也可以使用AWS CloudFormation模板格式编排多个综合的云应用

