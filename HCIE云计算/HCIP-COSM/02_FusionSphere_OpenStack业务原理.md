OpenStack为管理虚拟机、容器和裸机提供了一个软件平台
课程目标
1. 了解Neutron组件与网络虚拟化原理
2. 了解Cinder组件与创卷流程
3. 了解Nova服务原理与虚拟机创建流程
4. 了解Ironic服务原理与裸金属实例申请流程

1. Neutron服务与网络虚拟化实现原理
===

Neutron逻辑架构
-            Neutron-server
- service plugin    service plugin ......  <-----> DB
-                        Queue
- Neutron-sriov-nic-agent  Neutron-openvswitch-agent  Neutron-dhcp-agent  Neutron-Metadata-agent Neutron-vc-vswitch-agent Neutron-L3-ac-agent Neutron-huawei-ac-agent LB-agent vcenter
- AC2.0

Neutron组件说明

组件  功能
- Neutron-server           接收REST请求，向keystone鉴权，与数据库交互，提供网络对象的API
- Neutron-dbcp-agent       提供DHCP服务
- Neutron-metadata-agent   为虚拟机访问metadata服务提供网络通道
- Neutron-openvswitch-agent配置openvswitch流表，提供二层转发路径
- Neutron-vc-vswitch-agent 对接vmware,将neutron网络模型转换为vmware的网络模型，形成统一虚拟网络
- Neutron-sriov-nic-agent  支持SRIOV网卡虚拟化
- Neutron-huawei-ac-agent  对接AC2.0控制器，将Neutron中L2网络信息传递给AC2.0控制器
- Neutron-l3-ac-agent      对接AC2.0控制器，将Neutron中L3网络信息传递给AC2.0控制器

网络虚拟化实现原理



2. Nova服务原理与虚拟机创建流程
===


3. Ironic服务原理与裸金属实例申请流程
===