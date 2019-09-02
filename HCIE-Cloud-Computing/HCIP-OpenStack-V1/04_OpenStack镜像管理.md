1. OpenStack镜像服务Glance简介
2. Glance架构
3. Glance工作原理和流程
4. Glance镜像制作
5. OpenStack动手实验

镜像服务Glance
---

- Glance提供发现、注册、和检索虚拟机镜像功能
- 提供的虚拟机镜像可以存放在不同地方，例如本地文件系统、Swift对象存储、Cinder块存储等。

依赖的OpenStack服务Keystone

Glancet提供发现、注册和检索虚拟机镜像功能，编写时遵循高可用、可恢复和高容错的原则。

Glance架构
---

Glance使用C/S架构，提供REST API，用户可以通过API执行对服务器的请求。

```txt
AuthN Client ------>REST API  ------>Glance Domain Controller ---Registry Layer
   |                |                Database Abstraction Layer--->Glance DB
Keystone API <------AuthZ middleware
   |
AuthN<------Glance Store Drivers 
   |
Supported Storages (Swift,S3,Ceph,...,File System,Sheepdog,)
```

Glance组件详解

- Client：Glance-client使用Glance服务器的任何应用程序，接收请求并调用glance-api
- REST API：glance-api通过REST接口对外开放Glance功能，接收请求
- Glance Domain Controller：管理Glance内部服务器，Glance Domain Controller分层实现特定任务，如认证、事件通知、策略控制和数据库连接等。
- Registry Layer：实现Glance-Domain Controller与DAL之间的安全访问 
- Database Abstraction Layer(DAY)-数据库抽象层　提供Glance与数据库之间统一API接口
- Glance　DB Glance DB在所有组件之间共享，存放管理、配置信息等数据。
- Glance　Store 负责与外部存储后端或本地文件系统的交互，持久化存储镜像文件。Glance　Store提供一个统一接口来访问后端存储，遮蔽不同后端存储的差异。

|镜像Image|实例Instance|规格Flavor|
|:---|:---|:---|
|虚拟机包含一个虚拟磁盘，其上包含可以引导的操作系统，为虚拟机提供模板|实例是在OpenStack上运行的虚拟机|规格定义了实例可以有多少个虚拟CPU，多大的RAM以及多大临时磁盘|

镜像、实例、规格之间的关系
- 用户可以从同一个镜像启动任意数量的实例
- 每个启动的实例都基于镜像的一个副本，实例上的任何修改都不会影响到镜像
- 启动实例时，必须指定一个规格，实例按照规格使用资源

创建实例时必须指定镜像和规格

Glance镜像磁盘格式　将镜像添加到Glance时，必须指定虚拟机镜像的磁盘格式

|磁盘格式|描述|
|:---|:---|
|RAW|一种非结构化的磁盘镜像格式|
|VHD|VMware,Xen,MicroSoft,Virtual Box等使用的常见磁盘格式|
|VHDX|VHD格式的增强版，支持更大的磁盘容量和其它功能|
|VMDK|常见的虚拟机磁盘格式|
|VDI|Virtual Box和QEMU支持的磁盘格式|
|ISO|光盘存档格式|
|WCOW2|QEMU 支持的磁盘格式，支持动态扩展和写时复制|
|ploog|Virtuozzo支持使用的磁盘格式，用于运行OS Containers|
|aki|Amazon Kernel Image|
|ari|Amazon RamDisk Image|
|ami|Amazon Machine Image|

其它镜像，可以先转换为OpenStack支持的格式，再导入使用

Glancce状态机

Glance中有两种状态机：镜像状态和任务状态

|镜像状态|描述|
|:---|:---|
|queued|已在glance-registry中保留镜像标识符，但镜像数据未上传，镜像大小未初始化|
|Saving|镜像的原始数据正在上传到Glance中|
|uploading|对镜像调用了import data-put请求|
|importing|导入镜像中，但镜像尚未就绪|
|active|镜像创建完成，可以使用|
|deactivated|禁止任何非管理员用户访问镜像|
|killed|镜像上传出错，镜像不可用|
|deleted|Glance保留了镜像信息，但不能继续使用，镜像在一定时间后自动被清理掉|
|pending_delete|Glance尚未删除镜像数据，处于该镜像可恢复状态|
|pending|任务挂起|
|processing|任务正在处理中|
|sucess|任务执行成功|
|failure|任务执行失败|

Glance状态转化图

```txt
                 Queued<----Stage upload fail <--- Uploading ---- delete-------> Deleted
                 Queued-----Stage upload---------> uploading ---- import-------> Importing
                 Queued<----Import fail----------- Importing----- Import Fail--> Queued
                 Active<----succed---------------- Importing----- delete-------> Deleted
                                             active --->Deactivate deactivated
                                             active<--Reactivate--deactivated--delete-->deleted
Create Image --->Queued-----Add location---->active---------->delete-----------> deleted
                                             active --delayerd delete--->pending delete--delete--deleted
                 Queued ----upload---------------> saving ------- upload fail--> Queued
                 Active <---upload succeed-------- Saving ---upload fail-->killed--delete-->deleted
                                                   saving---------delete-------> deleted
```

Glance镜像缓存　镜像缓存：在API节点本地存放原始镜像一个副本，实质上使多个API服务器能够提供相同的镜像。由于提供镜像的服务器数量的增加，提升了镜像服务器的可伸缩性

控制总cache总量大小

周期清理　周期运行glance-cache-pruner
清理image cache 通过glance-cache-cleaner清理异常的cache文件
预取某些热门镜像到新增的api节点中　glance-cache-manage --host=<HOST>queue-image<IMAGE_ID>

镜像缓存对于用户来说是透明的

镜像与实例交互

