1. keystone简介
2. keystone架构
3. keystone对象模型
4. keystone认证工作原理和流程
5. OpenStack手动实验

OpenStack认证服务keystone是什么？　keysone提供身份认证，服务发现和分布式多租户授权，支持LDAP,OAuth,OpenID Connect,SAML和SQL,不依赖OpenStack其它服务，keystone为其它服务提供认证，外部请求调用OpenStack内部服务时，需要先从keystone获取到相应的token，类似，OpenStack内部不同项目间的调用也需要先从keystone获取到认证才能进行。

keystone提供身份认证时，服务发现和分布式多租户授权，属于共用服务。

Keystone在OpenStack中的作用　身份认证Identity  访问控制Policy  端点注册Endpoint   服务管理Catalog  令牌管理　token

keystone架构

CLI/Dashboard

Keystone API

Identity  Token  Policy   Catalog  Assignment ......

LDAP  SQL  KVS  Rules  Template  Password  Token

keystone我包括以下组件　Keystone API 、Keystone Middle 、Keystone Service 、Keystone Backends 、Keystone Plugins

Keystone 各组件的作用
- keystone API　　　　　接收外部请求
- keystone Middleware　缓存Token，减轻Keystone Service压力
- keystone Service　　　不同的service提供不同的认证或鉴权服务
- keystone Backends　　实现keystone服务，不同service由不同的backend提供
- keystone plugins     提供密码、token等认证方式

keystone对象模型

Service   Policy

|Identity|Resource|Assignment|Token|Catalog|
|:---|:---|:---|:---|:---|
|User|Project|Role||Endpoint|
|Group|Domain|Role Assignment|||

Keystone的管理主要针对Identity,Resource,Assignment,Token,Catalog,Service,这些对象由具体的其它更小的对象实现

OpenStack各种资源和服务的访问策略由Polity定义

keystone对象模型　service
- keystone是在一个或多个端点(Endpoint)上公开的一组内部服务(Service)
- keystone内部服务包括Identity、Resource、Assignment、Token、Catalog等
- keystone许多内部服务以组合方式使用。例如Identity验证用户或项目凭据，并在成功时创建带有返回带有令牌服务token的令牌
- 除内部服务外，keystone还负责与OpenStack其它服务service进行交互，例如计算、存储、镜像，提供一个或多个endpoint,用户可以通过这些endpoint访问资源并执行操作

keystone的service包括内部服务和外部服务

keystone对象模型 Identity
- Identity服务提供身份凭据验证以及用户User和用户组Group的数据
- User是单个OpenStack服务使用者，用户本身必须属于某个特定域，所有用户名不是OpenStack全局唯一的，仅在其所属域唯一
- Groups把多个用户作为一个整体管理，组本身必须属于某个特定域，所有组名不是OpenStack全局唯一的，仅在其所属域唯一

通常情况下，用户和用户组数据由Identity服务管理，允许它处理这些数据关联的所有GRUD操作，复杂情况下，用户和用户组数据由权威后端服务管理。

keystone对象模型　Resource
- Resource服务提供有关项目Project和域Domain的数据
- Project是OpenStack资源拥有者的基本单元，OpenStack中所有资源都属于特定项目组
- Domain把Project、User、Group作为一个整体来管理，每种资源都属于某个特定域，keystone默认域为Default

项目本身必须属于某个特定域，所有项目名不是OpenStack全局唯一，仅是所属域唯一，创建项目时如果未指定域，则将其加入到默认域

keystone对象，模型　Assignment
- Assignment服务提供有关角色Role和角色分配Role Assignment 的数据
- Role规定最终用户可以获得的授权级别。角色可以在项目或项目解绑授予，可以在单个用户或级别分配角色，角色名称在拥有该角色的域中是唯一的
- Role Assignment是一个3元组，Role  Resource Identity

keystone对象模型　Token
- Token服务提供用户访问服务的凭证，代表着用户的账户信息
- Token一般包含User信息，Scope信息(Project、Domain、或者Trust)、Role信息
- Token如果未包含Dcope,即Unscoped Token，这种Token中既不包含服务目录，也不含任何角色，项目范围和域范围。它主要作用是稍后向keystone证明身份，而不重复提交原始数据
- 必须满足一定条件才能接收Unscoped Token

keystone对象模型　Catalog
- Catalog服务提供用于查询端点Endpoint的端点注册表，以便外部访问OpenStack服务
- Endpoint本质上是一个URL。提供服务的入口。有如下几种：　Public 最终用户或其它服务用户使用，通常在公共网上使用。Internal供最终用户使用，通常在未计量的内部网络接口上。 Admin　供管理服务的用户使用，通常是在安全的网络接口上。

keystone对象模型　Policy
- 每个OpenStack服务都在相关策略文件中定义其资源的访问策略Policy.
- 访问策略类似于Linux的权限控制，不同角色的用户或用户组将会拥有不同的操作权限
- 访问策略规则以JSON格式指定，文件名policy.json 策略文件路径/etc/SERVICE_NAME/policy.json
- JSON (JavaScript Object Notation)是一种轻量级的数据交换格式，结构简明，易于阅读和编写，也易于机器解析和生成。

keystone对象模型分配关系

- Region>Domain>Project>Group>User
- 用户提交用户名、密码后，keystone验证生成token，操作OpenStack服务的请求必须携带token，通过端点URL访问服务。