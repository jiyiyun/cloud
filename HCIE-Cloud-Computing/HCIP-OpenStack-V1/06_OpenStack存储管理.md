1. 存储概述
2. 块存储cinder
3. 对象存储swift

OpenStack存储分为两大类　

|Ephemeral Storage　临时存储|Persistent Storage 持久性存储|
|:---|:---|
|如果只部署了nova服务，则默认分配给虚拟机的磁盘是临时的，当虚拟机终止后，存储空间也会被释放|持久化存储设备的生命周期独立于任何其它系统设备或资源，存储的数据一直可用，无论虚拟机是否运行|
|默认情况下，临时存储以文件的形式放置在计算节点的本地磁盘上|当虚拟机终止后，持久性存储上的数据依然可用|

(个人总结)　在前面学习nova的时候，当时我就发现nova的依赖里没有cinder感觉到奇怪，nova依赖keystone,和neutron，glance这三个服务。原来nova未指定存储的时候，是在计算节点默认创建虚拟机临时文件，虚拟机终止后存储被释放。

目前OpenStack支持三种类型的持久性存储:块存储cinder 、对象存储swift 、文件系统存储manila

|块存储cinder|对象存储swift|文件存储manila|
|:---|:---|:---|
|操作对象是磁盘，直接挂载到主机，一般用于主机的直接存储空间和数据库应用，DAS和SAN都可以提供块存储|操作对象是对象(Object);一个对象名称就是一个域名地址，可以直接通过REST API的方式访问对象|操作对象是文件和文件夹，在存储系统上增加了文件系统，再通过NFS或CIFS协议进行访问|

OpenStack存储类型对比

||用途|访问方式|访问客户端|管理服务|数据生命周期|存储设备容量|典型使用案例|
|:---|:---|:---|:---|:---|:---|:---|:---|
|临时存储||||||||
|块存储||||||||
|对象存储||||||||
|共享文件系统存储||||||||


Swift是什么？

Swift提供高度可用、分布式、最终一致的对象存储服务。可以以高效、安全并且廉价的存储大量数据；swift非常适合存储需要弹性扩展的非结构化数据。

不依赖于其它OpenStack服务

Swift在Openstack中的位置，属于Storage大类Storage(Object:Swift;Block:Cinder;File:Manila)

Swift提供高度可用、分布式、最终一致的对象存储服务。

Swift在OpenStack中的作用

```txt
Cinder -----Provides volumes for----> VM <----Provides images---Glance---Storage images in --->Swift
Nova ------ Provision --------------> VM
Cinder -----Backups volumes in -----> Swift
```
Swift并不是文件系统或者实时的数据存储系统，它称为对象存储，用于永久类型的静态数据的长期存储，这些数据可以检索、调整、必要时进行更新。

最适合存储的数据类型就是虚拟机镜像、图片存储、邮件存储和档案备份。以为没有中心或主控节点，Swift提供了更强的扩展性、冗余和持久性。Swift常用于存储镜像或用于存储虚拟机实例卷的备份副本

Swift特点
- 极高的数据持久性
- 完全对称的系统架构
- 无单节点故障
- 可扩展性

极高的数据持久性　Swift在5个zone，5*10个存储节点环境下，数据复制份是3，数据持久性SLA达到10个9
完全对称的系统架构　各节点完全对等，能极大降低系统维护成本
无限可扩展性　一是数据存储容量无限可扩展；二是swift性能可线性上升。
无单点故障　一般是HA方法只能主从，主一般只有一个，元数据信息一般只能单点存储，单点存储容易成为瓶颈。Swift元数据存储是完全均匀随机分布的，并且对象文件存储也一样，元数据也会存储多份。整个swift集群中，也没有一个角色是单点的，并且在架构设计上保证无单点业务是有效的

swift应用场景
- 镜像存储后端　与glance结合，为其存储镜像文件
- 静态数据存储　由于swift的扩展能力，适合存储日志文件和数据备份仓库

Swift对象存储架构　　完全对称、面向资源的分布式系统架构设计

API(Swift Proxy) <-----> -Account -Container -Object 组成一个闭环

Swift中对象存储URL如下

```
https://swift.example.com/v1/account/container/object
          集群位置           |        存储位置         |
```
/account　　包含账户本身的元数据以及账户中的容器列表
/account/container　用户自定义的存储区域，包含容器本身和容器中的对象列表的元数据
/account/container/object 对象存储数据对象及其元数据的位置

