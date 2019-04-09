目录
1. FusionStorage方案介绍
2. FusionStorage架构原理
3. FusionStorage部署配置

认识ServerSAN
- 概念 由多个独立服务器自带的存储组成一个存储资源池，同时融合了计算和存储资源
- 特征 专有设备变通用设备；计算与存储线性扩展；简单管理、低TCP

- 与厂商专用硬件解耦：传统存储系统由软硬件高度集成，Server SAN产品与硬件解耦
- 计算与存储融合：Server SAN构建在通用x86服务器上，计算和存储融合存在
- Server SAN,超融合架构，软件定义存储，业界称呼不同，本质都对应相似产品

传统SAN架构
```txt
    Server           孤立的存储资源 存储通过专用网络
       |             连接到有限数量的服务器
     FC/IP
       |             存储设备通过添加硬盘框增加容量，控制器性能
  Storage Server     成为瓶颈
```
- 机头瓶颈: 通常为GB
- Cache瓶颈 ： 通常为GB
- 网络瓶颈 : 10GE、8G FC
- 传统存储资源缺乏共享：传统存储设备和资源往往由不同厂家提供，之间无法资源共享，形成一个个孤立存储资源
- 传统存储一般采用集中式元数据管理方式，元数据中会记录所有LUN中不同偏移量的数据在硬盘中的分布，例如LUN1+LBA1地址起始地址的4KB长度的数据分布在第32块硬盘的LBA2上。每次IO操作都需要查询元数据服务，随着系统规模逐渐变大，元数据的容量也会越来越大，系统所能提供的并发操作能力将受限于元数据所在服务器的能力，元数据服务将成为系统性能的瓶颈

分布式Server SAN架构
```txt

虚拟化操作系统
                             共享存储资源池
InfiniBand/10GE Network      计算、存储融合部署
                             容量和性能线性增长
Storage Pools
```
- 分布式控制器，可线性扩展至4096节点
- 分布式cache ，拓展至TB级
- p2p无阻塞高速IB网络，56G InfiniBand RDMA
- 数据中心资源共享：一个数据中心内可以构建一个很大的储存资源池，满足数据中心内各类应用对存储容量，性能和可靠性的需求；实现资源共享和统一管理
- 云数据中心的新存储投资选择
- FusionStorage采用的DHT算法有以下特点：

> 均衡性：数据能够尽可能分布到所有节点中，这样可以使得所有节点负载均衡

> 单调性：当有新节点加入大系统中，系统会重新做数据分配，数据迁移仅涉及新增节点，现有节点上数据不需要做很大调整

华为Server SAN产品FusionStorage

分布式块存储软件 将通用x86服务器的本地HDD、SSD等介质通过分布式技术组织成大规模存储资源池；对非虚拟化环境的上层应用和虚拟机提供工业界标准的SCSI和iSCSSI接口，开放的API

- 支持传统块存储典型应用场景 各种业务应用(如SQL,Oracle RAC 、Web 行业应用等)
- 主流云平台集成 可以和各种云平台集成。如华为FusionSphere 、VMWare、开源Openstack等，按需分配存储资源
- 商用支持PB级Server SAN产品

华为FusionStorage两大主要应用场景
```txt
云资源池    数据库及关键应用

虚拟化平台

SCSI /iSCSI

FusionStorage
```
- 开发兼容 兼容主流数据库，兼容虚拟化平台，兼容主流服务器
- 融合部署 支持虚拟化平台和数据库资源池融合部署，即共用一个数据中心FusionStorage存储资源池
- FusionStorage支持使用SSD替代HDD作为高速存储设备，支持使用InfiniBand网络替代 GE/10GE网络提供更高带宽，对性能要求极高的大数据量实时处理场景提供完美的支持千万级IOPS

FusionStorage逻辑架构
```txt
            VM1                    VM2
     FusionStorage(master)  FusionStorage(Standby)
服务器1                 服务器2              服务器3              服务器4
FusionStorage agent    FusionStorage agent FusionStorage agent FusionStorage agent
MDC  VBS               VBS                 VBS
OSD  OSD               OSD  OSD                                OSD
管理计算存储节点         计算存储节点           计算节点             存储节点
```
- FSM(FusionStorage Manager) FusionStorage管理模块，提供告警、监控、日志、配置等操作维护功能。一般情况下FSM主备节点部署
- FSA(FusionStorage Agent) 代理进程，部署在各个节点上，实现各个节点与FSM通信，FSA包含MDC，VBS，和OSD三种不同进程。根据系统不同配置要求，分别在不同节点上启用不同进程组合来完成特定的功能

FusionStorage有分离部署和融合部署模式  分离模式计算节点服务器3和存储节点服务器4 ；融合模式服务器2；两种模式下管理节点都至少需要3台

- MDC(MetaData Controller)元数据控制，实现分布式集群的状态控制，以及控制数据分布式规则、数据重建规则等。MDC默认部署在3个节点的ZK(Zookeeper)盘上，形成MDC集群。
- VBS(Virtual Block System) 虚拟块存储管理软件，负责卷元数据的管理，提供分布式集群接入点服务，使计算资源能够通过VBS访问分布式存储资源。每个节点默认部署一个VBS进程，形成VBS集群。节点上也可以通过部署多个VBS来提升IO性能
- OSD(Object Storage Device) 对象存储设备服务，执行具体的I/O操作，在每个服务器上部署多个OSD进程，一块磁盘默认对应部署一个OSD进程，在SSD卡作为主存储时，为了充分发挥SSD性能，可以在一块SSD卡上部署多个OSD进程进行管理

主存 FusionStorage用服务器的数据存储磁盘

FusionStorage部署方式
- 融合部署 指将VBS和OSD部署在同一台服务器中，虚拟化应用推荐采用融合部署方式部署
- 分离部署 将VBS和OSD分别部署在不同的服务器中 高性能数据库应用推荐采用分离部署方式

基础概念

资源池  FusionStorage中一组硬盘构成的存储池
Volume卷 应用卷，代表了FusionStorage向上层呈现的一个逻辑存储单元






