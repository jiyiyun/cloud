kubernetes简介
===
综述
---
kubernetes简写k8s是早期google主导的开源容器编排管理平台，后来谷歌贡献给了开源社区，很多理念来自Google公司私有的Brog，谷歌Brog可以参见论文[《Large-scale cluster management at Google with Borg》](http://dl.acm.org/citation.cfm?id=2741964)
Kubernetes和Borg系出同门，基本是Borg的开源改进版本，引用Google Borg论文里的说法：
it (1) hides the details of resource management and failure handling so its users can focus on a	pplication development instead; (2) operates with very high reliability and availability, and su	pports applica- tions that do the same; and (3) lets us run workloads across tens of thousands o	f machines effectively

个人体验
---
对于k8s个人觉得最大优点就是自动化，也就是设置好参数，系统自动根据负载的轻重自动扩容缩容，目前以做到对pod的自动增删。
截止目前2016年11月部分branch在做对node的自动增删，底层可以兼容docker和rkt还有其他，现在底层已经做到不依赖docker一家，
对于网络，网上说1.5版本打通跟mesos的连接，这样k8s网络中就可以和mesos网络对接。对于大型公司，不止一个容器编排平台的公司来说是件好事。
网上也有同时将mesos和k8s两种编排工具同时用的平台，在吴龙辉编写的《Kubernetes实战 》一书有介绍。可以看看。

参考资料
---
[深入浅出 Kubernetes 架构 ](http://edu.dataguru.cn/article-8492-1.html)
