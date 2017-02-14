k8s主要功能模块
===
   kubernetes 最重要，最直接的第一手资料就是官网http://kubernetes.io 具体文档http://kubernetes.io/docs/whatisk8s/

操作对象
---
Kubernetes以RESTFul形式开放接口，用户可操作的REST对象有三个：
*   pod：是Kubernetes最基本的部署调度单元，可以包含container，逻辑上表示某种应用的一个实例。比如一个web站点应用由前端、后端及数据库构建而成，这三个组件将运行在各自的容器中，那么我们可以创建包含三个container的pod。
*   service：是pod的路由代理抽象，用于解决pod之间的服务发现问题。因为pod的运行状态可动态变化(比如切换机器了、缩容过程中被终止了等)，所以访问端不能以写死IP的方式去访问该pod提供的服务。service的引入旨在保证pod的动态变化对访问端透明，访问端只需要知道service的地址，由service来提供代理。
*   replicationController：控制pod副本数量，用于解决pod的扩容缩容问题。高可用就靠它了。通常，分布式应用为了性能或高可用性的考虑，需要复制多份资源，并且根据负载情况动态伸缩。通过replicationController，我们可以指定一个应用需要几份复制，Kubernetes将为每份复制创建一个pod，并且保证实际运行pod数量总是与该复制数量相等(例如，当前某个pod宕机时，自动创建新的pod来替换)。

server端
---
*   apiserver：作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。
*   kube-scheduler 调度器，主要干一件事情：监听etcd中的pod目录变更，然后通过调度算法分配node，最后调用apiserver的bind接口将分配的node和pod进行关联（修改pod节点中的nodeName属性）。scheduler在Kubernetes中是一个plugin，可以用其他的实现替换（比如mesos）。
*   kube-controller-manager 承担了master的主要功能，比如和CloudProvider(IaaS)交互，管理node，pod，replication，service，namespace等。基本机制是监听etcd /registry/events下对应的事件，进行处理。
    controller-manager目前有两类：
    endpoint-controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
    replication-controller：缩写rc，定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。

node端
---
*   kubelet 主要包含容器管理，镜像管理，Volume管理等。同时kubelet也是一个rest服务，和pod相关的命令操作都是通过调用接口实现的。比如：查看pod日志，在pod上执行命令等。pod的启动以及销毁操作依然是通过监听etcd的变更进行操作的。但kubelet不直接和etcd交互，而是通过apiserver提供的watch机制，应该是出于安全的考虑。kubelet提供插件机制，用于支持Volume和Network的扩展。
*   kube-proxy 主要用于实现Kubernetes的service机制。提供一部分SDN功能以及集群内部的智能LoadBalancer。应用实例在多个服务器节点之间迁移的一个难题是网络和端口冲突问题。Kubernetes为每个service分配一个clusterIP（虚拟ip）。不同的service用不同的ip，所以端口也不会冲突。Kubernetes的虚拟ip是通过iptables机制实现的。每个service定义的端口，kube-proxy都会监听一个随机端口对应，然后通过iptables nat规则做转发。
*   Pods 
    Kubernetes将应用的具体实例抽象为pod。每个pod首先会启动一个google_containers/pause docker容器，然后再启动应用真正的docker容器。这样做的目的是为了可以将多个docker容器封装到一个pod中，共享网络地址。
    pod是容器的集合, 每个pod可以包含一个或者多个容器; 为了便于管理一般情况下同一个pod里运行相同业务的容器,同一个pod的容器共享相同的系统栈(网络,存储),同一个pod只能运行在同一个机器上
etcd
---
    etcd是一个分布式, 高性能, 高可用的键值存储系统,由CoreOS开发并维护的，灵感来自于 ZooKeeper 和 Doozer，它使用Go语言编写，并通过Raft一致性算法处理日志复制以保证强一致性。
    etcd 作为配置中心和存储服务（架构图中的Distributed Watchable Storage），保存了所有组件的定义以及状态，Kubernetes的多个组件之间的互相交互也主要通过etcd。 lable的存储在etcd(一个分布式的高性能,持久化缓存)中; kubernetes用etcd一下子解决了传统服务中的服务之间通信(消息服务)与数据存储(数据库)的问题

Label
---
key-value格式的标签，主要用于筛选，比如service和后端的pod是通过label进行筛选的，是弱关联的。

Namespace 命名空间、多租户用户隔离
---
Kubernetes中的namespace主要用来避免pod，service的名称冲突。多租户，用户隔离，可以通过namespace配置文件对特定部门CPU、内存、带宽等资源进行限制，这样就不会出现一个部门的应用占用了所有服务器资源，同一个namespace内的pod，service的名称必须是唯一的。

Kubectl 
---
Kubernetes的命令行工具，主要是通过调用apiserver来实现管理。例如kubectl get node 获取所有node

Kube-dns 
---
dns是Kubernetes之上的应用，通过设置Pod的dns searchDomain（由kubelet启动pod时进行操作），可以实现同一个namespace中的service直接通过名称解析（这样带来的好处是开发测试正式环境可以共用同一套配置）。
主要包含以下组件，这几个组件是打包到同一个pod中的。
*   etcd skydns依赖，用于存储dns数据
*   skydns 开源的dns服务
*   kube2sky 通过apiserver的接口监听kube内部变更，然后调用skydns的接口操作dns
Network 
---
Kubernetes的理念里，pod之间是可以直接通讯的,但实际上并没有内置解决方案，需要用户自己选择解决方案: Flannel,Calico，OpenVSwitch,Weave 等。

配置文件 
---
Kubernetes 支持yaml和json格式的配置文件，主要用来定义pod,replication controller,service,namespace等。

参考资料
---
01、[Kubernetes初探：原理及实践应用](http://www.csdn.net/article/2014-10-31/2822393)
02、[深入浅出 Kubernetes 架构](http://edu.dataguru.cn/article-8492-1.html)
