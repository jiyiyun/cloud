目录
1. 计算虚拟化基础概念
2. CPU虚拟化
3. 内存虚拟化
4. FusionSphere关键特性

虚拟化本质
- 分区 在单一物理服务器上同时运行多个虚拟机
- 隔离 同一服务器上虚拟机之间相互隔离
- 封装 整个虚拟机都保存在文件中，而且通过移动和复制这些文件的方式来移动和复制虚拟机
- 相对于硬件独立 无需修改即可在任何服务器上运行虚拟机

分区： 意味着虚拟化层为多个虚拟机划分服务器资源的能力；每个虚拟机同时运行一个单独操作系统，使服务器能够运行多个程序；每个操作系统只能看到虚拟化层提供的“虚拟硬件”

隔离：虚拟机是相互隔离的
>一个虚拟机崩溃或者故障，不会影响同一服务器上的其它虚拟机

>可以进行资源控制以提供性能隔离

封装：整个虚拟机存储独立于物理硬件的一小组文件中。这样在外部只需复制几个文件就可以随时随地根据需要复制保存和移动虚拟机

||寄宿虚拟化|裸金属虚拟化|混合虚拟化|
|:---|:---|:---|:---|
|优点|简单容易实现|支持多种操作系统和应用性能高|没有冗余，性能高、支持多种操作系统|
|缺点|管理开销大|虚拟层内核开发难度大|需底层硬件支持底层虚拟化扩展功能|
|厂家|VMware Workstation|VMware vSphere、Hyper-V、XenServer、FusionSphere|Redhat KVM|

I/O虚拟化
---

- DomU: 运行在Xen Hypervisor上的普通虚拟机
- Dom0: 运行在Xen Hypervisor上的特权虚拟机，拥有访问物理I/O资源的权限，同时和系统上运行的其它虚拟机进行交互。Dom0需要在其它Domain启动之前启动

|Domain 0|Domain U|
|:---|:---|
|用户态 控制面板|用户态|
|内核 设备驱动 后端驱动|内核 前端驱动|
|虚拟机监控器VMM|
|物理硬件(处理器，内存，IO设备)|

虚拟机复用有限的外设资源
- VMM截获guestOS对设备访问请求，然后通过软件方式来模拟真实设备的效果
- 前端设备驱动将数据通过VMM提供的接口转发到后端驱动
- 后端驱动VM的数据进行分时分通道进行处理

I/O虚拟化需要解决两个问题
- 设备发现 需要控制各虚拟机能够访问设备
- 访问截获 通过I/O端口对设备访问

硬件辅助虚拟化
---

VT-X是Intel运用虚拟化技术中的一个指令集，是CPU虚拟化的技术，AMD处理器类似功能称为AMD-V

主机内存超分配 Host Memory和Guest Memory之间并不是一一对应。可以超分配内存给VM，通过内存复用技术实现超分配功能。

CPU虚拟化
---

PHY CPU     PHY kernel 01       Super thread 020--->vCPU

物理CPU      每个CPU包含的核数　　 每个核里面包含的超线程数

可用资源=(CPU个数×每个CPU包含的核数×每个kernel包含的Thread数－host主机消耗的vCPU)×CPU主频GHz

CPU QoS(Quality of Service)
---

- CPU资源限额 控制虚拟机占用物理资源的上限
- CPU资源份额　CPU份额定义多个虚拟机在竞争物理CPU资源的时候按比例分配资源
- CPU资源预留　多个虚拟机竞争物理CPU资源的时候分配的最低计算资源

示例：单核CPU主频为3GHz，该资源被VM1 和VM2 使用
- 场景1，上限是2GHz,虚拟机可用CPU资源最多为2GHz
- 场景2:按比例分配
- 场景3 VM1预留2GHz,VM1预留0，那么竞争时，VM2获得1GHz(3-2=1)

内存虚拟化
---
- Host physical memory指虚拟机管理程序可用内存
- Guest physical memory 指运行在VM上的Guest OS可用内存
- Guest Virtual memory 指Guest OS向应用程序提交的一个连续的虚拟地址空间，它是在虚拟机中运行的应用程序的内存。

- Application         Guest virtual memory
- Operating System    Guest physical memory
- Hypervisor          Host physical memory

主机内存超分配
---

Host Memory和Guest Memory之间并不是一一对应。可利用内存复用技术实现超分配

内存复用技术
- 内存共享、写时复制 

> 内存共享：虚拟机之间共享同一物理内存空间，此时虚拟机仅对内存只读操作；写时复制 当虚拟机需要对内存进行写操作时，开辟另一内存空间，并修改映射。

- 内存置换

>虚拟机长时间未访问的内存内容被置换到存储中，并建立映射。当虚拟机再次访问内存内容时再置换回来
- 内存气泡

>Hypervisor通过内存气泡将较为空闲的虚拟机内存释放给内存使用率较高的虚拟机，从而提高内存使用率

华为虚拟化平台，通过以上复用技术，可将内存复用比提高到150%

虚拟机内存QoS
---

- 内存预留(下限) 虚拟机预留的最低物理内存,预留的内存会被虚拟机独占。一旦内存被某个虚拟机预留，即使虚拟机实际内存使用量不超过预留量，其它虚拟机也无法抢占该虚拟机空闲的内存资源
- 内存份额 适用于资源复用场景，按比例分配内存资源 
- 内存上限　分配给虚拟机的内存大小就是内存上限

虚拟机热迁移(是DRS和DPM基础)
---

技术特点 基于内存压缩传输技术，虚拟机热迁移效率提升1倍；虚拟机磁盘数据位置不变，只是更改映射关系

适用场景 可容忍短时间中断，但必须要快速恢复业务，比如轻量级数据库业务、桌面云业务

存储热迁移 

技术特点 热迁移可以使客户可以在业务无损的情况下动态调整虚拟机存储资源，以实现设备维护，存储DRS资源调整等操作

无共享热迁移

技术特点 将源物理机上指定的处于运行状态的非共享虚拟机迁移到另一台虚拟机上，以实现不同存储介质上的虚拟机在不同节点之间无缝在线迁移

整机迁移技术，可以让不同存储介质上的虚拟机，在不同的节点之间无缝地进行在线迁移，摆脱共享存储的限制

整机迁移存在以下约束限制：1. 虚拟机整机迁移只支持虚拟化存储 2. 虚拟机整机迁移不支持链接克隆虚拟机 3. 包含共享卷的虚拟机不允许整机迁移

虚拟机热迁移应用场景 服务器下电维护 新老硬件更换 数据迁移 性能优化 服务器补丁升级 本地虚拟机迁移

IMC 设置集群策略，使虚拟机可以在不同CPU类型主机之间迁移

异构热迁移
- 跨代处理器使用不同的处理器架构，CPU特性不兼容，因此不能热迁移
- 新架构CPU调整为旧架构CPU模式，时集群内所有主机运行在相同CPU模式
- 集群内一台主机CPU版本较低，集群需要运行在较低模式，导致整体性能降低，资源浪费
- 支持的最高CPU指令集

建议规划集群时，节点间CPU模式支持能力不要差距太大，导致整体性能较低，资源浪费
```
Intel CPU架构 Merom --->Penryn(2007)--->Nehalem(2008)--->Weatmere(2010)--->Sandy Bridge(2011)-->Ivy Bridge(2012年)--->Haswell(2013年)--->
Broadwell(2014年)--->Skylake(2015年)--->kaby lake(2016年)--->coffee lake(2017年)
Process              Architecture      Optimization         Optimazation
目前环节为 Process、Architecture、Optimization即制程、架构、优化
制程：在架构不变的情况下，缩小晶体管体积，以减少功耗及成本
架构：在制程不变的情况下，更新处理器架构，以提高性能
优化：在制程和架构不变的情况下，对架构进行修复及优化，将BUG减到最低，并提高时钟频率
```
克隆虚拟机

- 在线克隆虚拟机时，一次只能克隆一个虚拟机，不支持批量克隆
- 虚拟机克隆前提包括 虚拟机已安装tools,且tools运行正常；虚拟机的磁盘需创建在虚拟化的数据存储上或FusionStorage上，且不能是共享类磁盘
- 克隆Linux虚拟机时，需配置虚拟机网卡

NUMA技术介绍：
```txt
----------------------|             ------------------|
|Node 0               |            | Node 1           |
|       本地访问       |            |                  |
|                     | Intel QPI  |                  |
|DRAM <-------->CPU0  |<---------->| CPU1<------>DRAM |
|                 |________________________________|  |
----------------------|            |------------------|
```
1. NUMA(Non-uniform Memory Architecture)非一致性内存架构解决了多处理器系统中可扩展性问题
2. NUMA技术将CPU划分成不同组(Node),每个Node由一个或者多个CPU组成，并且有独立的本地内存、I/O等资源
3. CPU访问同Node中内存速度最快，访问其它Node中内存性能较差

操作系统优化 进程级优化，分配内存和CPU时考虑NUMA拓扑，提供初始放置，负载均衡，内存动态迁移等功能(NUMAD,AutoNUMA)

业务层面优化：业务内部进行指定Node内存申请和线程绑定，在业务内部实现各Node业务分发，最大程度实现本地内存访问

虚拟NUMA技术： 虚拟NUMA向guest OS呈现NUMA拓扑并且保证虚拟Node与实际分配情况一致，它的功能包括
- 拓扑呈现 虚拟机内部识别到NUMA,使guest OS及应用NUMA优化功能生效
- 初始放置 根据虚拟机NUMA拓扑，选择物理Node放置vCPU和内存，使vNode中vCPU与内存关系与物理实际一致
- 负载均衡 在调度过程中考虑Node关联性，以及vNode与物理Node对应关系，最大限度保证vCPU访问本地内存
- 动态迁移 当vCPU与物理Node亲和关系发生变化时，触发其对应vNode内存进行迁移

DPM分布式电源管理(Distributed Power Management)

技术特点 系统自动选择合适的物理机上下电，最小迁移VM数量；保证小部分物理机处理休眠态，以快速满足新增业务所需资源

适用场景 夜间低负载，自动迁移虚拟机，下电空闲主机；日间业务需求上升，自动上电主机，迁移虚拟机到新上电主机

FusionCompute计算集群，基于VIMS文件系统的共享存储上；DPM算法能够综合业务量和物理资源情况，利用DRS功能，合并虚拟机腾出空闲主机以达到节能目的，DPM需要集群开启DRS功能

虚拟机规则组
- 聚集虚拟机 列出的虚拟机必须在同一主机上运行
- 互斥虚拟机 列出的虚拟机必须在不同主机上运行
- 虚拟机到主机 虚拟机组的成员是否能在特定主机组的成员上运行

虚拟机的高级策略的配置同样要先打开DRS功能，策略对象可以是虚拟机也可以是虚拟机组





    

