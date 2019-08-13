1. FusionSphere虚拟化套件介绍
2. FusionCompute介绍
3. FusionCompute规划部署

FusionSphere虚拟化套件特点
---

|管理至简|性能至优|
|:---|:---|
|多站点统一管理　可支持256个站点统一管理，统一运维|大规格虚拟机　255U/4T|
|异构纳管VMware|高性能虚拟化能力　KVM引擎，SPECvirt测试业界领先|
|最可靠　支持两地三中心，双活，跨地容灾|GPU增强　支持多款显卡直通和虚拟化。适配深度学习，图形渲染，复杂计算等业务场景|
|云演进　支持物理机、第三方平台迁移；支持FusionSphere虚拟化演进到华为云Stack混合云|高性能网络　OVS+DPDK网络调优(>20Gbps)，10GB适配传输仅4s|

|FusionSphere虚拟化套件组成|
|:---|
|FusionCube   FusionAccess  Customer Apps|
|FusionSphere [FusionManager FusionCompute UltraVR eBackup FusionStorage Block]|
|X86 Server  SAN  Firewall  3rd Party Hypervisors|

FusionSphere服务器虚拟化架构

|华为开放API|
|:---|
|FusionManager 分支站点管理　　多资源池管理　　统一运维　　VM生命周期管理|
|纳管第三方虚拟化VMware vSphere 　华为虚拟化FusionCompute(VRM CNA) 备份与容灾(UtraVR eBackup)|
|硬件基础设施　服务器　　存储　网络＆安全|

- FusionCompute:提供对x86物理服务器，SAN设备的虚拟化能力，并提供软件定义网络基础能力
- FusionManager:FusionCompute能力，并集成防火墙，负载均衡器等自动化管理能力，提供企业级和运营级的虚拟数据中心管理方案。
- UltraVR:提供跨站点容灾能力
- eBackup:提供虚拟机的备份能力

FusionSphere架构说明

|部件|说明|
|:---|:---|
|FusionCompute|必选组件，FusionCompute是云操作系统软件，主要负责硬件资源的虚拟化，以及对虚拟资源、业务资源、用户资源的集中管理，它采用虚拟计算、虚拟存储、虚拟网络等技术，完成计算资源、存储资源、网络资源的虚拟化；同时提供统一的接口，对虚拟资源进行集中调度和管理，从而降低业务的运行成本，保证系统的安全性和可靠性，协助运营商和企业构筑安全、绿色、节能的数据中心能力|
|FusionManager|可选部件，FusionManager主要对云计算的软件和硬件进行全面的监控和管理，实现同构，异构VMware虚拟化多资源池管理，软硬件统一告警监控，并向内部运维人员提供管理门户|
|eBackup|可选组件，eBackup是虚拟化备份软件，配合FusionCompute快照功能和CBT(Changed Block Tracking)备份功能实现FusionSphere的虚拟机数据备份方案|
|UltraVR|可选部件，UtrlVR是容灾业务管理软件，利用底层SAN存储系统提供的异步远程复制特性，提供虚拟机关键数据的数据保护和容灾恢复。|

FusionCompute是必选组件，FusionManager、eBackup、UltraVR都是可选组件

FusionSphere应用场景
- 单虚拟化场景　只采用FusionCompute作为统一的操作维护管理平台对整个系统进行操作与维护的应用场景
- 多虚拟化场景　多套虚拟化环境需要进行统一管理　同时接入FusionCompute和VMware虚拟化环境
- 私有云场景　　多租户共享VPC场景　多租户私有VPC场景

FusionCompute介绍
---

FusionCompute 虚拟资源调度　可用性　虚拟计算　　安全性　虚拟网络　　可拓展性　虚拟存储

FusionCompute是云操作系统软件，主要负责硬件资源虚拟化，以及对虚拟资源、业务资源、用户资源的集中管理。它采用虚拟计算、虚拟存储、虚拟网络等技术，完成计算资源、存储资源、网络资源的虚拟化。同时通过统一接口，对这些资源进行集中调度和管理，从而降低业务运行成本，保证系统的安全性和可靠性，协助运营商和企业构筑安全、绿色、节能数据中心能力。

FusionCompute架构

FusionCompute   WebUI Portal

CNA　部署在X86服务器上  VRM VRM可以作为一个虚拟机进行部署，也可以单独部署在X86服务器上

CNA节点可以有一个或者多个

- CNA:Compute Node Agent CNA部署在需要虚拟化的服务器上
- VRM:Virtual Resource ManageMent VRM可部署在虚拟机上或物理机上；VRM对外提供网页操作界面供管理维护

|模块|功能|
|:---|:---|
|CNA|提供虚拟计算功能；管理计算节点上的虚拟机；管理计算节点上的计算、存储、网络资源|
|VRM|管理集群内块存储资源；管理集群内网络资源(IP/VLAN),为虚拟机分配IP地址；管理集群内虚拟机生命周期以及虚拟机在计算节点上的分布和迁移；管理集群内资源的动态调整；通过对虚拟资源、用户数据的统一管理，对外提供弹性计算、存储、IP等服务；提供统一操作维护接口，操作维护人员通过WebUI远程访问整个系统|

FusionCompute带来的价值　　资源共享　　分时共享　节点故障时提供高可靠性　检修时，VM热迁移到其它主机

FusionCompute规划部署
---

CNA服务器　　CPU支持硬件虚拟化技术，如Intel的VT-x 并在BIOS中开启CPU虚拟化功能；内存>8G；硬盘>70G,建议使用RAID 1提高系统可靠性

注意：安装过程不允许修改本地PC的IP地址；关闭本地PC防火墙;文件路径不能含有中文字符；搭建过程中不允许随意重启主机

网络规划
- BMC平面　通过BMC平面可以远程访问服务器BMC系统
- 管理平面　用于管理系统统一管理所有节点，以及节点间内部通信所使用的平面，使用管理平面的IP地址包括　使用主机的管理IP地址，即主机管理网口使用的IP地址；管理节点虚拟机的IP地址；存储设备控制器的IP地址；
- 存储平面　主机与存储设备的存储单元互通所使用的平面，使用存储平面的IP地址包括：所有存储主机的存储IP地址；存储设备的存储IP地址
- 业务平面　用户虚拟业务数据在网络中使用的平面

虚拟化部署VRM逻辑图　小规模部署，可以将VRM部署在虚拟机中，并设置为主备模式，分别部署在两台虚拟机内
物理部署VRM逻辑图　在大规模集群场景中，将VRM部署在真实物理机上，并且设置为主备模式；

