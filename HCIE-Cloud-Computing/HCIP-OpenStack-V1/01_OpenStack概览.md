1. 简介
2. 架构
3. OpenStack核心服务简介
4. OpenStack服务间交互示例
5. OpenStack手动实验

OpenStack是开源云操作系统，开源控制整个数据中心的大型计算、存储、网络资源池。用户开源通过Web界面、命令行或API接口配置资源。

OpenStack不是虚拟化　OpenStack只是系统控制面，OpenStack不包括系统的数据面组件，如Hypervisor、存储和网络设备等

|OpenStack|VS|虚拟化|
|:---|:---|:---|
|自身不提供虚拟化技术||环境隔离，资源复用|
|调用多种技术实现多资源池管理||降低隔离损耗，提升运行效率|
|对外提供统一管理接口||提供高级虚拟化特性|

虚拟化是OpenStack底层的技术实现手段之一，但并非核心关注点

OpenStack不是云计算　OpenStack只是构建云计算的关键组件　内核、骨干、框架、总线　为了构建云计算，我们还需要很多东西，例如Cloud Service 、Cloud BSS、Cloud Console,最基础的Hardware and DC Infrastructure

OpenStack的设计思想

|开放|灵活|可扩展|
|:---|:---|:---|
|开源、并尽可能重用已有的开源项目|不使用任何不可替代的私有/商业组件|由多个相互独立的项目组成|
|不要“重复造轮子”|大量使用插件的方式进行架构设计与实现|每个项目包含多个独立服务组件|
|||无中心架构|
|||无状态架构|

OpenStack使用的是Apache 2.0许可证，大约70%代码(核心逻辑)使用python开发，每年两个大版本，一般在4月和10月中旬发布，版本命名从字母A-Z ,2019年4月10日发布版本为stein ，版本信息可参考　https://releases.openstack.org

OpenStack架构
---

OpenStack的服务分为几大类　计算、存储、网络、共用服务、硬件生命周期、排编、工作流、应用程序生命周期、API代理、操作界面

OpenStack服务组件通过消息队列(Message Queue)相互通信

|Deployment Host|Infrastructure Control Plane Host|Compute Host|Storage Host|
|Ansible||Compute Hypervisor|Block Storage Volumes|
|OpenStack-Ansible Repository|MariaDB 、RabbbitMQ 、Memcached|Network L2/L3 Agents||
||Dashboard 、identity、 Image 、Compute Management|||
||Bare Metal Management 、Block Storage Management　、Orchestration|||
||Network Management、Network L2/L3 Agents|||

生产环境一般会有专门的OpenStack部署服务节点、控制节点、计算节点、网络节点、存储节点等。

OpenStack核心服务简介
---

认证服务keystone　提供身份验证、服务发现和分布式多租户授权。keystone支持LDAP、OAuth、OpenID Connect，SAML和SQL，不依赖于其它OpenStack服务，为其它OpenStack服务提供认证支持

- LDAP Lightweight Directory Access Protocol 轻量级目录访问协议
- OAuth Open Authorization 为用户资源的授权提供了一个安全的、开放而又简易的标准。
- OpenID Connect 是OpenID和Oauth2的合集
- SAML Security Assertion Markup Language安全断言标记语言，是一个基于XML的开源标准数据格式，在当事方之间交换身份验证和授权数据，尤其是在身份提供者和服务提供者之间交换。

计算服务Nova 提供大规模、可扩展、按需自助服务的计算资源。nova支持管理裸机、虚拟机和容器。

块存储服务Cinder cinder提供块存储服务，为虚拟机提供持久化存储。调用不同的存储接口驱动，将存储设备转化成块存储池，用户无需了解存储实际部署的位置或设备类型。

对象存储服务Swift 提供高度可用、分布式、最终一致的对象存储服务。Swift可以高效、安全且廉价地存储大量数据，非常适合存储需要弹性扩展的非结构化数据。

网络服务Neutron 负责管理虚拟网络组件，专注为OpenStack提供网络即服务。

编排服务Heat 为云应用程序排编OpenStack基础架构资源，提供OpenStack原生Rest API和CLoudFormation兼容的查询API

OpenStack服务间交互示例
---

创建VM需要计算、存储、网络、镜像等资源

OpenStack核心工作之一是虚拟机生命周期管理，