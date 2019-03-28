VXLAN(Virtual Extensible Local Area Network)是VLAN扩展方案，采用MAC in UDP(User Datagram Protocol)封装方式，是NVo3(Network Virtualization over Layer 3)中的一种网络虚拟化技术

作为云计算核心技术之一，服务器虚拟化凭借其大幅降低IT成本、提高业务部署灵活性、降低运维成本等优势，已经取得越来越多的认可和部署

目录
2. VXLAN基本原理
3. VXLAN常见组网
4. BGP-EVPN简介
5. DC内VXLAN based EVPN部署
6. DC间VXLAN based EVPN部署
7. 配置部署优化
8. 应用案例

云数据中心业务对网络有新的诉求
- 虚拟机POD内自由迁移
- 虚拟机POD间自由迁移
- 虚拟机数据中心间迁移

大二层网络  虚拟机摆脱地理位置的限制，自由迁移，构建跨地理区域的大二层网络

传统网络为什么大不起来？
- 传统数据架构VLAN无法跨越三层边界：传统数据中心组网方式通常是网关部署在汇聚交换层，汇聚交换机间通过三层核心打通，虚拟机迁移只能局限于POD内，如果需要跨越POD二层区域迁移，需要更改VM的IP地址，应用会中断
- STP技术在解决网络环路问题时，存在以下主要缺陷：1. 收敛时间长，通常不超过50个网络节点，不适宜云数据中心大规模组网；2. STP构建无环网络时，需要阻断一半链路，带宽利用率低

Fabric技术

- CSS/SVF (中小POD级Fabric) [POD内] 1.通过设备多虚一技术破坏；2.适合单个POD
- TRILL Fabric (大型POD级Fabric) [DC内L2] 1. 采用IS-IS路由控制平面破坏单个POD或整个数据中心
- VXLAN fabric (DC级Fabric)[DC内L3] 1. L2网络Overlay在L3网络上
- EVN Fabric (DCI Fabric)[DCI] 1. vxlan封装，天然无环；2. 数据中心之间DCI互联

VxLAN基本概念
---

基于NVo3的二层FabricZ组网

NVo3(Network Virtualization Over Layer 3),基于三层ip overlay网络构建虚拟网络技术统称为NVo3,目前代表性的有：VXLAN、NVGRE、STT

运行NVo3的设备叫做NVE(Network Virtualization Edge)，它位于overlay网络的边界，实现二三层的虚拟化功能。

VXLAN(Virtual Extensiable LAN虚拟可扩展局域网)是目前NVo3中影响里最广泛的一种，它通过MAC in UDP报文封装的方式，实现基于IP overlay的虚拟局域网
- VXLAN网络中的NVE以VTEP进行标示，VTEP(VXLAN Tunnel Endpoint,VXLAN隧道端点)
- 每一个NVE至少有一个VTEP，VTEP使用NVE的IP地址表示
- 两个VTEP可以确定一条VXLAN隧道

VXLAN逻辑抽象
```txt
NVE                                        VNI 1       VNI 2
                   Overlay Fabric            |           |
VXLAN Fabric  --->               ------->    |           |
                     Underlay                |           |
NVE   
```
VXLAN的简单理解--两次虚拟化
1. 第一次虚拟化：利用隧道技术将边缘设备互连透传二层报文；整网抽象理解成一台端口数目扩展的超大LAN Switch
2. 第二次虚拟化利用VNI将这台超大的交换机虚拟出多个二层的广播域，和VLAN本质是一样，VNI类比VLAN ID通过定义VXLAN header中的VNI字段，将子网范围由4k扩展到16M

VXLAN报文格式

VXLAN是MAC in UDP的网络虚拟化技术，所以其报文封装是在原始以太网报文之前加了一个UDP封装及VXLAN头封装。
```txt

 48 bits  48 bits    16 bits    12 bits      16 bits
---------------------------------------------------------
| MAC DA  | MAC SA | VLAN type | VLAN ID | Ethernet type | 14 bytes
---------------------------------------------------------
    ^
    |     72 bits   8 bits    16 bits   16 bits   16 bits
    |    --------------------------------------------------
    |    | .....  | Protocol | .....  |  IP SA  | IP DA   | 18 bytes
    |    |-------------------------------------------------
    |         ^
    |         |
|<-------------VXLAN封装-----------><-------------原始报文--------->|
| Outer    | outer | Outer | VXLAN | Inner    | Inner  | Payload  |
| Ethernet |  IP   | UDP   | header| Ethernet |  IP    |          |
|  header  | header| header|       | header   | header |          |
-------------------------------------------------------------------
                       |       |
                       |       v
                       |      8 bits       24 bits  24 bits  8 bits
                       |  ----------------------------------------------
                       |  | VXLAN Flags | Reserved |  VNI   | Reserved | 8 bytes
                       |  | (00001000)  |          |        |          |
                       v  ----------------------------------------------
    16 bits    16 bits      16 bits   16 bits 
   ---------------------------------------------
   | Source | DestPort     | UDP    | UDP      |
   | Port   | (VXLAN Port) | length | checksum |8 bytes
   ---------------------------------------------
``` 
VTEP概念
- VXLAN网络中的NVE以VTEP进行标示，VTEP(VXLAN Tunnel EndPoint,VXLAN隧道端点)
- 每一个NVE至少有一个VTEP，VTEP使用NVE的IP地址表示
- 两个VTEP可以确定一条VXLAN隧道，VTEP间的这条VXLAN隧道将被两个NVE间所有VNI共用
```txt
                NVE3   VNI1
                 |     VNI2
    VLAN tunnel  |
NVE2------------NVE1------------NVE4
                 |     VNI1
                 |     VNI2
                NVE5
 VTEP 必须全网唯一
```
隧道和VNI关系
```txt
VTEP 1.1.1.1                                   VTEP2 1.1.1.2
VNI----------------------------------------------VNI
                     vxlan tunnel
VNI----------------------------------------------VNI
```
VNI概念
- 标示VXLAN网络中的二层域
- 两个VTEP可以确定一条VXLAN隧道，VTEP间的这条VXLAN隧道被两个NVE间的所有NVI共用

VXLAN转发数据封装
```txt
A<------>Ingress NVE<------>Transit<------>Egree NVE<------>B
    端到端不变VTEP IP            逐跳改变外层MAC
```
源终端的二层报文能够穿越IP网络到达目的终端，VXLAN网络对于主机来说相当于是Bridge Fabric

VXLAN主要优点

- 网络依赖小 基于IP的overlay，仅需要边界设备间IP可达 
- 环路避免 隧道间水平分割、IP overlay TTL避免环路
- 高效转发 数据流量基于IP路由SPF及ECMP快速转发
- 快速收敛 网络变化实时侦听全网拓扑毫秒收敛
- 虚拟化 overlay+VNI构建虚拟网络，支持多达16M的虚拟网络
- 部署灵活网络 物理设备、vSwitch均能够部署

