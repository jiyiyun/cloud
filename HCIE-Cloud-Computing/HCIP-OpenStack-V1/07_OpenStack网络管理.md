1. Linux网络虚拟化基础
2. Neutron简介
3. Neutron概念
4. Neutron架构和组件分析
5. Neutron操作
6. Neutron网络流量分析

Neutron的设计目标是实现“网络即服务”，设计上遵循“软件定义网络(SDN)”,灵活自动化原则。实现上，充分利用Linux各种网络相关技术

物理网络与虚拟化网络

物理网络　NIC <----连接---->Switch

虚拟网络　　vNIC <----连接---->　vSwitch---->NIC物理网络连接

Neutron最核心的工作是对二层网络的抽象与管理，物理服务器虚拟化后，虚拟机网络功能由虚拟网卡vNIC提供，物理交换机Switch也被虚拟化成虚拟交换机vSwitch,各个vNIC连接在vSwitch的端口上，最后这些vSwitch通过物理服务器的物理网卡访问外部的物理网络

Linux网络虚拟化实现技术

|网卡虚拟化|交换机虚拟化|网络隔离|
|:---|:---|:---|
|TAP|Linux Bridge|Network|
|TUN|Open vSwitch|NameSpace|
|VETH|||

OpenStack中使用较多的网络虚拟化技术主要是网卡虚拟化、交换机虚拟化和网络隔离。

Linux网卡虚拟化 - TAP/TUN/VETH

|||Physical NIC|TUN|TAP|Veth Pair|Veth Pair|
|||:---|:---|:---|:---|:---|
|||Socket API|Socket API|Socket API|Socket API|Socket API|
||||APP|APP|||
|User Space|||/dev/tunX|/dev/tapX|||
|Kernel Space||raw ethernet|L3 ethernet|raw packets|raw ethernet| raw ethernet|
|||Network Stack|Network Stack|Network Stack|Network Stack|Network Stack|
|||eth0|tunX|tapX|vethX|vethX|
|||NIC|||||

- TAP设备　模拟一个二层网络设备，可以接收和发送二层网包
- TUN设备：模拟一个三层网络设备，可以接收和发送三层网包
- VETH: 虚拟ethernet接口，通常以pair的方式出现，一端发出的网包，会被另一端接收，可以形成两个网桥之间的通道。

