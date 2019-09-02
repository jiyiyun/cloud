1. OpenStack故障处理基础
2. OpenStack故障处理工具
3. OpenStack典型故障处理
4. OpenStack动手实验：故障处理
5. OpenStack故障处理相关项目

OpenStack发生故障时，通过以下方法进行故障诊断和处理
- 验证OpenStack服务状态
- 检查OpenStack服务日志记录
- 为OpenStack服务启动调试模式
- 检查OpenStack服务配置文件

验证OpenStack服务状态

确保OpenStack服务已经启动并运行，验证每个控制节点上的服务状态。某些OpenStack服务状态要在非控制节点上进行额外验证

- 方法一：　使用SERVICE_NAME service-list快速验证OpenStack服务状态，例如nova service-list
- 方法二：　如果服务不支持service-list命令，可以使用ps -aux|grep SERVICE  再service SERVICE_NAME status验证服务状态，例如service nova-api status

验证OpenStack服务状态一览表

|服务|控制节点验证|非控制节点验证|
|:---|:---|:---|
|Nova|||Glance|nova service-list|service nova-compute status|
|Neutron|service cinder-api status service|service cinder-volume status|
||Service cinder-scheduler status|service cinder-backup status|
|Glance|service glance-api status||
||service glance-registry status||
|Heat|heat service-list||

- Horizon: service apache2 status 、netstat -nltp|grep '80|443'
- Keystone: service apache2 status 、netstat -ntpl |grep '5000|35357'
- Swift: swift stat
