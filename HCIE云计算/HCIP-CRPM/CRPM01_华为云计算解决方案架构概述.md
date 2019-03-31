开放兼容的基础设施平台-HuaweiFusionSphere
---

```txt

应用   ...|  CRM  |   ERP  |  办公系统  |  数据中心管理平台 MnanagOne|
----------------------------------------------------------－－－－－
基础  Keystone    OpenStack               Heat
设施  Glance   Nova   Cinder  Neutron     Cellometer
服务  Swift    driver driver  plugin      Ironic

虚拟   VM  VM  VM |   存储   |  网络  |   非虚拟化
资源     UVP      |   虚拟化 | 虚拟化 |   资源
------------------------------------------－－－－－－－－－－－－－－
x86服务器  |  存储  |  网络           | 物理设备
```

FusionSphere6.0解决方案定位

- NFV电信云化场景 聚焦运营商 ；交付形态 电信网元集成后面向客户交付，基于OPenStack+KVM
- 云数据中心场景 焦距数据中心 ；交付形态 OpenStack + FC作为独立产品直接向客户交付
- 服务器虚拟化场景  焦距教育金融 ；交付形态 基于FM + FC交付虚拟化平台

```txt
                         ManageOne
         ServiceCenter               OperationCenter
    虚拟化场景             云DC场景                     NFV场景
  FusionManager          FusionSphere OpenStack OM   FusionSphere OpenStack OM
Fusion  Fusion  Fusion   Fusion  Fusion  Fusion      Fusion  Fusion  Fusion
compute Storage Network  compute Storage Network     KVM Storage Network
```
华为云计算虚拟化层

- 虚拟化(Virtulization)是一个广义术语，通常理解为对计算机资源的抽象方法。
- 系统虚拟化的目的是通过使用虚拟化管理器(Virtual Machine Manager简称VMM)，在一台物理机上虚拟和运行一台或多台虚拟机。
- 计算虚拟化包括 CPU虚拟化，内存虚拟化，IO虚拟化

通常我们所说的虚拟化是指具备创建一个抽象的虚拟机的平台，虚拟机可以抽象一台物理机一样运行操作系统和业务。

随着云计算的发展，我们将云计算需求资源分成技术、存储和网络三部分，虚拟化主要功能总结为软硬件的解耦

计算虚拟化趋势

- 物理机：应用独享设备
- 虚拟化：不同应用共享设备
- 容器： 不同应用共享OS

对于虚拟机使用者而言，需要关注的底层技术越来越少
>虚拟化让我们不再关心底层硬件，可以使用差异化的硬件； 容器让我们运行代码不再为了繁多的操作系统版本

越来越少的调用底层资源消耗，更高的利用率和灵活性

FusionCompute

FusionCompute是FusionSphere的虚拟化部件，主要提供资源虚拟化和虚拟化资源池管理功能

存储虚拟化理论
 
数据存储虚拟化
>FusionStorage功能，让虚拟机可以灵活使用存储，实现精简部署、存储热迁移、快照等高级特性

软件定义存储
> 存储不再是特殊硬件设备，使用纯软件的方式定义存储，底层硬件设备为通用商用硬件

存储服务平台
> 业务驱动，统一管理异构存储，提供模板化，自动化的数据服务，提升运营效率，为不同业务提供所需存储服务

存储虚拟化是指利用虚拟化技术让存储更加贴合云计算的需求进行了一系列的软硬件解耦，管理增强和性能提升

数据存储虚拟化
---



