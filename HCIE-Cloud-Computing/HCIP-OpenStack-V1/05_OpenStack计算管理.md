1. Nova架构
2. Nova组件详细讲解
3. Nova典型操作
4. Nova典型工作流程
5. Nova操作

Nova提供大规模可扩展，按需自助服务的计算资源。支持管理裸机，虚拟机和容器。

早期Nova版本包含计算、存储、网络，后期分拆出存储和网络。

Nova依赖于Keystone提供认证服务，neutron提供网络服务，glance提供镜像存储服务。(个人添加)nova不依赖与块存储是因为在没有块存储的情况下默认在计算节点使用临时存储，在虚拟机终止后自动删除。

Nova在OpenStack项目中提供计算资源管理服务。

Nova负责　１、虚拟机生命周期管理；　２、其它计算资源生命周期管理

Nova不负责　１、承载虚拟机的物理主机自身的管理。　２、全面的系统状态监控

Nova是OpenStack事实上最核心项目　１、历史最长，是最初两个项目之一　(nova和swift,来历：OpenStack最初由NASA和Rackspace合作开发的项目，2010年7月以apache2.0许可证授权开源，源代码来源于NASA的Nebula云平台和Rackspace的分布式云存储swift项目，后面NASA建立了自己的计算引擎，新平台命名为nova并将其开源，2010年NASA和Rackspace分别将nova和swift开源时，已经获得了25个企业和组织的支持)　２、功能最复杂，代码量最大。　３、大部分集成项目和Nova之间都存在着配合关系　４、贡献者在OpenStack社区影响大

Nova架构
```txt

keystone<--------API----------->DB
Neutron <--------API----------->Glance & Cinder
                  |
               Conductor------->Scheduler---->DB
               Conductor------->DB
                  |
Neutron <------Compute--------->Glance & Cinder
                Hypervisor

External Service : keystone Neutron Glance Cinder
Nova Service : API  Conductor  Scheduler  Compute
```
- Nova-API 接收REST消息
- Nova-scheduler 选择合适的主机
- Nova-conductor 数据库操作和复杂流程控制
- Nova-compute 虚拟机生命周期管理和资源管理

Nova 内部使用REST调用，Nova与外部服务交互时使用消息队列

Nova运行架构

```txt
API Layper               Nova API   Nova-API-Cell      Nova-API-EC2

Conductor Scheduler      AMQP Queue Service <---Nova-scheduler Nova-conductor --->Database

Hypervisor Layper        Nova-Compute

Virtual Infrastructure   VMware VC driver      Xen driver      KVM libvirt driver
```
Nova 通过virtDriver对接不同的虚拟化平台

Nova资源池架构　　Region、Availability Zone、Host Aggregate、Group

Region > Availability Zone > Host Aggregate


