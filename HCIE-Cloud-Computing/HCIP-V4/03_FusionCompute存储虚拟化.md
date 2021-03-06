1. 存储虚拟化相关概念及技术
2. 存储虚拟化功能原理

FusionCompute存储虚拟化管理

|兼容存储类型|存储资源能力|存储配置管理|
|:---|:---|:---|
|SAN (Storage Area Network)|精简制备磁盘|存储资源发现与管理|
|NAS (Network Attached Storage)|厚制备\延迟置零磁盘|数据存储创建与管理|
|FusionStorage Block存储池|增量快照|存储资源裸设备映射|
|主机的本地硬盘|存储冷热迁移||
||虚拟磁盘扩容||
||磁盘快照管理||

FusionCompute中存储基本概念
- 存储资源　存储资源表示物理存储设备，如IP-SAN FC-SAN NAS等
- 存储设备　存储设备表示存储资源中的管理单元，类似LUN,FusionStorage存储池，NAS共享目录等
- 数据存储　数据存储表示虚拟化平台中可管理，操作的存储逻辑单元

VIMS 虚拟集群存储文件系统

VIMS是一种高性能的集群文件系统，使虚拟化技术的应用超出了单个存储系统的限制，可让多个虚拟机共同访问一个整合的集群式存储池，从而提高了资源利用率，VIMS是跨越多个存储服务器实现虚拟化的基础。它可以启用存储热迁移、DRS、HA等各种服务。

VIMS分布式锁

一个VIMS卷可以同时被多个CNA节点挂载，并支持多个CNA节点访问，为了保持数据的一致性，VIMS需要实现分布式文件锁，VIMS的DLM(Distribute lock Manager)模块负责实现分布式文件锁，它提供集群概念上的锁服务，调用者通过DLM保证集群间的同步要求

VIMS采用分布式全对称锁机制，在VIMS中有多个资源管理者master，每个master只对应一个锁资源，不同的master并不会集中在同一个节点上，无管理中心节点。

正常情况下成为某个锁资源的master方式有两种: 1. 第一个申请访问某资源的节点。　2. 如果多个节点同时访问某资源，以VIMS节点号较小的节点作为master

当节点发生故障时，此节点负责管理的资源会重新选举出master

VIMS分布式文件锁流量走管理网络平面

VIMS心跳　VIMS存在两种心跳，磁盘心跳用于检测主机是否可以正常读写共享存储，网络心跳用于检测主机间网络通信是否正常，作为集群文件系统，挂载了VIMS卷的CNA节点从来都部署单独个体，作为集群成员之一，通过网络心跳确保与其它节点进行正常通信

    CNA01 <---网络心跳---> CNA02 <---网络心跳---> CNA03

磁盘心跳 |　　　　　　　磁盘心跳 |             磁盘心跳 |

                        共 享 存 储

若节点网络在146s(20+126s)没有恢复，网络分割仲裁任务启动，重启无效分区的节点，以求隔离故障节点

FusionCompute磁盘技术

在存储虚拟化中，所有用户存储都是以文件形式呈现，常见磁盘分为普通磁盘、普通延迟置零磁盘、精简磁盘、差分磁盘；从数据安全性上又划分为持久化、非持久化

- 普通磁盘　创建时大小与磁盘大小相同，并将文件所有位置填0，占用空间大，置备时间长，适用于对IOPS要求较高的场景
- 普通延时置零磁盘　创建时大小和虚拟磁盘大小相同，但是不会填0操作，占用空间大，置备时间短；可以提高存储设备利用率，性能较普通磁盘有所下降，适用于发放速度要求高，但对IOPS要求不高的场景
- 精简磁盘　创建时大小为0,精简磁盘创建时含有少量元数据信息，大小为几十K，创建时间非常短，随着用户数据写入，精简磁盘的大小与实际占用空间逐步增加；精简磁盘可以提高设备使用率，精简磁盘使用动态磁盘技术，可以节省存储空间。该磁盘在创建时不分配磁盘空间，而是在用户IO写入磁盘文件时才进行空间动态分配，性能较普通磁盘下降，适用于用户对需求不明确，或规划的容量比实际使用容量多的场景。
- 差分磁盘　差分磁盘必须基于一个已有的父磁盘来创建，它只记录相对于父磁盘的差异数据，包括数据的增改，差分磁盘不能脱离父磁盘而存在。如果父磁盘进行了修改，则差分磁盘将不再可用。该磁盘用于FusionSphere系统中的快照、非持久化磁盘。链接克隆等功能 
- 持久化与非持久磁盘　持久化磁盘即数据可以永久保存。在创建独立持久磁盘时，快照中不包含该磁盘，更改将立即并永久写入磁盘，回滚快照不会导致数据回滚，类似与U盘，应用于个人独有数据存放。

非持久化磁盘即数据不永久保存。处于保护磁盘数据的目的，在启动虚拟机时，对这种非持久化磁盘先创建差分磁盘，在虚拟机运行过程中，将有更改的数据全部写入差分磁盘，在虚拟机关机后，将差分磁盘数据删除，达到还原磁盘的目的，应用于公共计算机，计算机数据自动还原场景。

存储虚拟化功能原理
---

快照　虚拟机可以将当前状态保存在快照中，包括磁盘内容、内存和寄存器数据。用户可以通过恢复快照多次回到这一状态，虚拟机用户在执行一些重大、高危操作之前，比如系统升级，打补丁，破坏性测试前执行快照，这样故障后可以快速还原。

FusionCompute支持普通快照、一致性快照和内存快照

- 普通快照　快照会保存磁盘当前数据
- 内存快照　快照创建时会保存虚拟机当前内存中的数据
- 一致性快照　快照创建时会将虚拟机当前未保存的缓存数据先保存，再创建快照

未创建快照　　VM <---读写--->　磁盘

创建快照　　　VM <---写操作--->　差分盘　<---读重定向--->　原磁盘文件

回滚快照　　删除最近的差分盘到指定做快照时间点；也可以选择保留现在的差分盘，从指定快照时间点另开一个分支

删除快照　　合并差分盘

链接克隆：链接克隆可以基于同一个模板，快速发放多个类似虚拟机，给每个虚拟机挂载差分盘，适合于大量，对虚拟机性能要求不高，母卷+ 差分卷1＝VM1 、母卷+ 差分卷2＝VM2、母卷+ 差分卷3＝VM3 ...

在链接克隆场景下，将热点数据放在内存中，可以提升虚拟机启动和运行效率

存储热迁移
- 冷迁移是指　虚拟机关机后　将磁盘文件从一个存储迁移到另一个存储
- 热迁移是指　在业务不中断情况下，将虚拟机磁盘从一个存储迁移到另一个存储

存储热迁移原理
1. 在目的存储上面创建差分磁盘文件
2. 源磁盘文件设置为只读，启用写时重定向，将数据写入到目的存储的差分磁盘上
3. 将源磁盘数据依次读取来合并到目标差分磁盘，合并完成后，目标磁盘就是拥有所有最新数据的磁盘
4. 除去目的端对源磁盘的依赖，将差分盘修改称为动态磁盘，这样目的端磁盘文件可以独立运行

存储资源裸设备映射(RDM)

RDM(Raw Device Mapping) 绕过hypervisor,(仅限FC和iSCSI)上的LUN ,RDM能够将虚拟机下发的SCSI命令直接透传，虚拟机可以直接操作物理SCSI设备，避免虚拟化层模拟功能造成的损失

不支持链接克隆、存储瘦分配、磁盘在线/离线扩容、存储增量快照、iCache、存储热迁移、存储QoS、磁盘备份、虚拟机转换为模板等

技术特点　虚拟机直接通过SCSI命令操作裸存储设备；２. 兼容FC存储和IP SAN存储

适用场景　Oracle RAC

存储扩容(虚拟卷扩容和数据存储扩容)

FusionCompute支持在线/离线磁盘扩容，对于普通盘，扩容并磁盘空间置零；对于普通延时置零盘，扩容并预占磁盘空间；对于精简磁盘，仅对数据区进行扩容；

数据存储扩容使得一个数据存储可以管理多个LUN空间

当需要扩容时，先在主节点上将新增的存储空间以线性映射的方式追加至虚拟块设备末尾，完成虚拟块设备扩容后，再将新增加的存储空间分成数段逐渐增加至文件系统(更新文件系统元数据)，主节点完成数据存储扩容。由于虚拟块设备信息都是保存在节点内存中，则当其它节点发现数据存储空间有变化时，则需要更新虚拟块设备信息，完成扩容虚拟块设备。

