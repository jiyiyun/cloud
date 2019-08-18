1. OpenStack操作界面服务Horizon简介
2. OpenStack操作界面服务Horizon功能
3. OpenStack动手实验：　Horizon操作

Horizon提供基于Web的控制界面，使云管理员和用户能够管理各种OpenStack资源和服务，依赖的OpenStack服务Keystone ,Horizon唯一依赖的服务是keystone认证服务；horizon可以与其它服务jieg结合使用，例如镜像服务，计算服务，网络服务；Horizon可以在有独立服务(如对象存储)的环境中使用。

Horizon的后端是一个Web服务器，默认是Apache Server

Horizon主要提供基于Web的OpenStack控制界面，使云管理员和用户能够管理各种OpenStack资源和服务。

Horizon项目开发思想

|核心支持|可扩展|可管理|一致|稳定|可用|
|:---|:---|:---|:---|:---|:---|
|对所有核心OpenStack项目提供开箱即用的支持|任何人都可用添加新组件|核心代码库简单易用|始终保持视觉和交互范式一致性|API强调向后兼容性|提供人们想要使用的最大界面|

- Horizon界面　Project  Project界面提供租户可以管理的资源，例如计算、存储、网络等。
- Horizon界面　Admin  Admin界面提供管理员可以管理的资源，例如计算、存储、网络和系统设置等。
- Horizon界面　Identity  Identity 界面提供认证管理功能。
- Horizon界面　Settings  Setting界面提供操作界面配置功能。

问题
1. OpenStack操作界面的管理视图和用户shi视图有什么区别？
2. OpenStack使用CLI时，为什么要先Source

- OpenStack管理员视图支持更多底层配置功能，例如配置规格、安全组、外部网络等，用户视图只能使用管理员预定义的规格、外部网络等。
- 将OpenStack相关变量预先定义好，source变量后，后续执行OpenStack命令不需要每次添加变量参数(例如os-url,os-user,admin_url等变量)

 