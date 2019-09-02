1. Heat简介
2. Heat架构
3. Heat概念
4. Heat典型排编场景
5. Heat操作

Heat诞生的因素

- 测试环境验证完成，如何将同样的配置，不出差错的部署在生产环境？
- 在其它地方部署一套应用，再重复执行一遍同样的部署动作？
- 在其它地方部署一套类似应用，只有丁点差异，也重复部署一遍？
- 开发测试时，需要频繁部署和删除应用，怎么办？
- 应用扩容需要增加一些虚拟机，怎么办？
- 应用不需要了，如何正确删除应用及周边资源

Heat编排服务，使OpenStack智能化

Heat为云应用程序排编OpenStack基础架构资源，提供OpenStack原生REST API和CloudFormation兼容的查询API

Heat依赖的OpenStack服务　keystone

Heat提供了简便创建和管理一批相关资源的方法，通过有序且可预测的方式对其进行资源配置和更新。

用户预定义一个规定格式的任务模板，任务模板中定义了一连串的相关任务，然后模板由Heat执行，Heat会顺序执行Heat模板中定义的任务

Heat依赖于Nova、Neutron等服务，为上端Harizon提供服务,充当了OpenStack对外接口的角色

Heat架构
