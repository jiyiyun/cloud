目录
1. 网络虚拟化背景
2. 虚拟交换机
3. 软件定义网络
4. 典型组网

为什么要网络虚拟化

- 传统数据中心特点 一个网络端口对应一台唯一的计算机；计算机与网络关系固定，很少变动
- 云计算数据中心带来的变化 一个物理网络端口会对应数量不固定的虚拟机；虚拟机会频繁进行跨主机的迁移，与网络的关系不固定；传统南北流向为主的数据中心网络架构不满足未来IT发展的需求；热迁移使得计算能力不再固定在具体位置，传统网络无法灵活满足云计算的诉求

```txt
Application  Application  Application             Workload    Workload   Workload

          x86 Enviroment                            L2 L3 L4-7 network Service
virtual Machine virtual Machine virtual Machine   Virtual network  Virtual network
          Server Hypervisor                         Network Virtualization Platform
          Requirement:x86                              Requirement: IP Transport
     Physical Compute and Memory                              Physical Network
```
- 与服务器虚拟化类似，网络虚拟化可以在很短的时间(秒级)，创建L2、L3到L7的网络服务，如交换，路由，防火墙和负载均衡等
- 虚拟网络独立于底层网络硬件，可以按照业务需求配置、修改、保存、删除，而无需重新配置底层物理硬件或拓扑。这种网络技术的革新为实现软件定义的数据中心奠定了基础

虚拟交换机OVS(Open vSwitch)概述
---

- Open vSwitch(OVS)是一款基于软件实现的开源虚拟交换机(以太网桥)
- OVS能够支持多种标准的网络接口和协议(例如NetFlow,sFow,RSPAN(Remote Switch Port Analyzer远程交换端口分析器、CLI、LACP等)，还可以支持跨多个物理服务器的分布式环境
- OVS提供了对OpenFlow协议的支持，并且能够与众多开源的虚拟化平台相整合
- 主要有两个作用，传递虚拟机VM间流量，实现VM和外界网络通信

OpenFlow是Software Definded Network的一种，由斯坦福大学的Nick McKeown教授2008年4月提出，提出了OpenFlow的控制转发分离结构，将控制逻辑从网络设备盒子中引出来，研究者可以通过一组定义明确的接口对网络设备进行任意的编程从而实现新型的网络协议、拓扑架构而无需修改动网络设备本身

华为分布式虚拟交换机方案

华为分布式虚拟交换支持基于开源Open vSwitch的纯软件的虚拟交换的功能，同时提供支持完整虚拟交换卸载的智能网卡的虚拟交换

开源Open vSwitch与智能网卡的虚拟交换提供的功能完全一致，虚拟交换管理通过不同的插件管理Open vSwitch和智能卡

VxLAN技术(Virtual extensible Local Area Network)虚拟扩展局域网，是一种进行大二层虚拟网络扩展的隧道封装技术；VxLAN引入一个UDP格式的外层隧道，作为数据链路层，而原有数据报文内容作为隧道净荷来传输

VxLAN重要组件
VNI VxLAN Network Identifier 24位虚拟网络标示，可支持16M虚拟网络
VTEP VxLAN Tunnel End Point 完成VxLAN报文和解封装，VTEP与网络相连，分配有物理IP地址，该地址与虚拟网络无关

```txt

server server   ---L2(Ethernet)-----  server  server
 VNI:1 VNI:2                          VNI:2   VNI3
   VxLAN        ------VxLAN---------     VxLAN
    VTEP        --------UDP---------      VTEP
     IP         -----IP/IP-MC-------       IP
                   IP network
```
VxLAN被定义为一种隧道机制，是基于三层物理上的二层叠加网络。隧道是无状态的，每个报文根据一定的规则封装。后续部分的隧道端点(VTEP),位于VM宿主服务器中的Hypervisor,因此，VNI和VxLAN相关的隧道、外层头封装仅有VTEP感知，VM感知不到。


