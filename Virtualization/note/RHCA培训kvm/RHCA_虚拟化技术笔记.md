虚拟化virtualizations 是一种资源管理技术，将计算机各种实体资源(如服务器、网络、存储、内存等)，予以抽象、转换后呈现出来。打破实体结构间的不可切割障碍。使用户可以比原本更好的方式来应用这些资源。

虚拟化分类
---

根据虚拟化层是通过纯软件的方法，还是利用物理资源提供的机制来实现这种“截获并重定向”，分为软件虚拟化和硬件虚拟化，纯软件虚拟化：所有指令都是纯软件模拟的，例如早期QEMU/VMware workstation

根据是否改动操作系统，虚拟化分为全虚拟化和准虚拟化/半虚拟化
全虚拟化：提供完整的操作系统。例如基于硬件辅助的KVM
半虚拟化:改动guest操作系统来与VMM协调，例如XEN(特点:guest知道自己在虚拟化环境中，是通过修改操作系统实现的，需要安装特定的驱动)

Hypervisor类型主要有2类：
1. 宿主型 实现虚拟化的这个软件需要安装在一个操作系统上，它本身没有管理硬件的功能，而操作系统有，因此依赖于操作系统，例如KVM
2. 裸机型 实现虚拟化的这个软件不需要安装在操作系统上，它本身就有管理硬件的功能，例如EXSi

KVM Kernel-base virtual Machine
---

KVM虚拟化技术的前提
1. 基于内核kvm模块，linux 内核2.6.34以后开始有kvm模块，查看lsmod |grep kvm
2. CPU支持,Intel VT,AMD-V
3. 相关软件,例如virt-manager 、libvirtd、virsh

为什么要打开VT,因为KVM是全虚拟化能模拟出的完整操作系统，不需要改动，并且结合硬件辅助，在虚拟机中的速度会更快，操作虚拟机和真机一样

带有虚拟技术的处理器具有额外的指令集，Virtual Machine Extensions 简称VMX，cat /proc/cpuinfo |grep -E 'vmx|svm'

X86服务器有4个特权级别Ring0 ~ Ring3
---

只有运行在Ring 0 ~ 2级时，处理器才可以访问特权资源或执行特权指令，运行在Ring 0级时，处理器可以访问所有的特权状态，x86平台的操作系统一般只使用Ring0和Ring3这两个级别，操作系统运行在Ring0级，用户运行在Ring3级，同时为了避免Guest OS控制系统资源，为了满足第一个充分条件-GuestOS运行在Ring1或者Ring3级(Ring2不使用)

搭建KVM环境
---

libvirt是对KVM虚拟机进程管理的工具和应用程序接口。常用虚拟机管理工具(virsh/virt-install/virt-mananger)和云计算框架平台(OpenStack)都在底层使用libvirt的应用程序接口

```txt
yum install -y libvirt*
启动libvirtd服务，启动了这个服务所有的管理工具才能使用，这个服务调用了一些api接口来管理

systemctl status libvirtd 

brctl show
```
基于virsh-install 安装虚拟机
```txt
yum install -y virt-install

```
基于图形virt-manager图形界面安装虚拟机
```txt
yum install -y virt-manager

virt-manager  #启动virt-manager图形界面
```
应用程序运行在最低级别ring3上，不能做受控操作如果要做(访问磁盘、读写文件)那就通过执行系统调用(函数)，执行系统调用的时候，CPU的运行级别会发生ring3到ring0的切换，并跳到系统调用对应的内核代码位置的执行，这样内核就完成设备访问，完成之后再从ring0返回ring3 。这个过程也称作户态和内核的切换。


```txt
qemu-img create -f qcow2 /var/lib/libvirt/images/vm1.qcow2 10G
chown qemu:qemu  /var/lib/libvirt/images/vm1.qcow2
yum install -y *vnc*
yum install lsof
virt-install --name=vm1 --disk path=/var/lib/libvirt/images/vm1.qcow2 --graphics vnc,listen=192.168.5.120 --vcpus=2 --ram=2048 --location=/mount/rhel-server-7.0.iso

lsof -i:5900

yum install virt-manager
```

KVM只是模块，它可以和硬件打交道，和用户打交道的是qemu-kvm,这个工具将用户操作交给KVM模块

vCPU在KVM中三种模式(客户模式--用户模式--内核模式)

虚拟机的CPU请求，将从guest端到用户端再到内核端
```txt
----Userspace------|--------Kernel---------|------Guest----------------|
 |-----ioctl()----------------|
 |                            |
 |                   |-Swich to Guest mode-----------|
 |                   |                         Native Guest execution
 |                   kernel exit Handler<------------|
 |-Userspace exit Handler<--------|
```

SMP系统(Symmetric Multi-Processor对称多处理器)是指在一个计算机上汇集了一组处理器(多CPU)，各CPU之间共享子系统以及总线架构，在SMP系统中，多个程序(进程)可以做到真正的并行，而且单个进程的多个线程也可以并行执行，提高了性能

```txt
rich@R:~$ uname -a
Linux R 4.15.0-29deepin-generic #31 SMP Fri Jul 27 07:12:08 UTC 2018 x86_64 GNU/Linux
```
内存过载
---

内存过载代表宿主机的所有虚拟内存加起来大于宿主机的内存加宿主机的swap.注意单台虚机的内存不可大于宿主机内存

内存过载的实现方式
- 内存交换swapping 用交换空间swap space来弥补内存不足
- 气球ballooning 通过virio_balloon驱动来实现宿主机hypervisor和客户机之间的协作
- 页共享page sharing 空格KSM(kernel Samepage Merging)合并多个客户机进程使用相同内存页

swap 是将内存中不运行的程序交换到磁盘上

内存气泡
---

我们为什么可以动态调整内存呢？因为有内存气泡，那么什么是内存气泡？它是通过virtio_balloon驱动来实现宿主机Hypervisor和客户机之间的协作

使用的时候，虚拟机安装virt balloon的驱动，内核开启CONFIG_VIRTIO_BALLOON，RHEL7默认已开启，并且默认已经安装virt balloon驱动

气泡里面为: 物理机可使用的内存，也就是说当虚拟机内存被回收后，会到气泡里，然后物理机再分配给其它进程。

在虚拟机中查看virt balloon，可以看到virtio_balloon模块，如果没有这个模块，我们无法动态调整内存
```txt
#lsmod |grep virt
virtio_balloon
```

共享页
---

宿主机内存压缩主要采用KSM(Kernel SamePage Merging)技术，原理和压缩类似，就是将相同的内存页进行合并，如果内存发生变化，则将内存单独复制出来独立运行

KSM原理： KSM作为内核守护进程(ksmd)存在，它定期执行内存页面扫描，识别副本页面并合并副本，释放这些页面供其它用，因此在多个进程中，Linux将内核相似的内存页合并成为一个内存页，提高内存使用效率，由于内存是共享的，所以多个虚拟机使用的内存就少了。这个特性对于虚拟机使用相同镜像和操作系统时，效果更加明显，但是用这个特性增加了内核开销，用时间换空间，如果追求效率则可以将这个特性关闭。


```txt
systemctl status ksm
systemctl status ksmtuned
如果不想使用该服务，关闭该服务即可

root@R:~# ls /sys/kernel/mm/ksm/
full_scans	    pages_to_scan    stable_node_chains
max_page_sharing    pages_unshared   stable_node_chains_prune_millisecs
merge_across_nodes  pages_volatile   stable_node_dups
pages_shared	    run		     use_zero_pages
pages_sharing	    sleep_millisecs
root@R:~# cd /sys/kernel/mm/ksm/
root@R:/sys/kernel/mm/ksm# cat *
0
256
1
0
0
100
0
0
0
20
0
2000

```

在linux系统中，我们可以在一个网卡上配置多个IP地址，以此来实现类似子接口功能，我们称之为IP别名。

Open vSwitch
---

Open vSwitch是一个由软件实现的虚拟交换机，由nicira Networks 主导开发，它的目的是通过编程扩展让大型网络的管理变得自动化。同时仍然支持标准的管理接口协议(如NetFlow,sFlow,SPAN,RSPAN,CLI,LACP,802.aq等)，此外Open vSwitch还可以跨物理服务器，类似VMWare 的vNetwork分布式vSwitch或Cisco Nexus 1000v功能

KVM可以通过Open vSwitch接入网络，相比传统网桥，Open vSwitch更高的灵活性，例如可以在线更高虚拟机的VLAN等

Open vSwitch基本概念
- Bridge Bridge代表一个以太网交换机(Switch),主机可以创建一个或者多个Bridge设备
- Port 相当于物理计算机的端口，每个Port 属于一个Bridge
- Interface 连接到port的网络接口设备。在通常情况下，Port和Interface是一对一的关系，只有在配置Port为boud模式后，port和Interface才是一对一的关系
- Controller OpenFlow控制器，OVS可以同时接受一个或者多个OpenFlow控制器管理
- datapath 在OVS中，datapath负责执行数据交换，也就是把从接收到的数据包在流表中进行匹配，并执行匹配到的动作
- Flow table 每个datapath都和一个Flow table关联，当datapath接收到数据后，OVS会在Flow table中查找可以匹配的flow，执行对应的操作，如转发数据到另外端口

Libvirt概念
---

Libvirt是目前最广泛的对KVM虚拟机进行管理的工具和应用程序接口。而且一些常用的虚拟机管理工具(virsh ,virt-install,virt-manager等)和云计算平台(如OpenStack,OpenNebula等)都在底层使用libvirt的应用程序接口。 创建磁盘，创建虚拟机等都是有libvirt来管理

libvirt支持KVM,Xen, Hyper-V

libvirt中涉及几个重要概念
- 节点(Node) 是一个物理机器，上面可能运行着多个虚拟客户机。Hypervisor和Domain都运行在节点上
- Hypervisor也称为虚拟机监视器VMM，如KVM，XEN,VMware,Hyper-V等，虚拟化中一个底层的软件层，它可以虚拟化一个节点让其运行多个虚拟客户机
- 域Domain 是在Hypervisor上运行的一个客户机操作系统实例。域也被称为实例instance,如AWS中客户机被称为实例

virsh概述
---

virsh是用于普通虚拟化环境中客户机和Hypervisor的命令行工具。与virt-manager等工具类似。它通过调用libvirt API来实现虚拟化的管理。virsh是完全在命令行文本模式下运行的用户态工具，它是系统管理员通过脚本程序实现虚拟化自动部署和管理的理想工具之一

vish命令行在进行虚拟化管理操作时，可以使用两个工作模式，交互模式和非交互模式。交互模式是直接连接到相应的hypervisor上，然后输入一个命令得到一个结果直到用户输入"quit"退出连接。非交互模式直接在命令行中一个建立连接的URL之后添加需要执行的一个或多个命令，执行完将命令输出结果返回到当前终端上，然后断开连接。

半虚拟化驱动

在虚拟机中，CPU优化依赖于硬件辅助VT;内存是经过了2次映射才到虚拟机那的，优化可用ETP，有个叫做影子页的不需要映射，还可使用内存过载与内存限制等来优化内存。而IO是效率最低的，因为所有设备都是软模拟的，也就是qemu，和IO相关的有磁盘IO与网络IO，那么qemu是如何模拟呢？
1. 如果虚拟机发生IO操作，KVM会进行IO拦截，此时KVM陷入拦截虚拟机的IO
2. 然后KVM通知qemu对拦截的IO进行模拟
3. qemu模拟完后返回IO给KVM
4. KVM再返回虚拟机

可以看到1次IO操作需要KVM多次参与，导致虚拟机IO路径太长，相应慢，那么如果解决呢？

于是就有了半虚拟化驱动(Para Virtualized Drivers, PV Drives),使用virtio框架来实现

virtio框架
1. 虚拟机知道自己在虚拟化环境中
2. 分为前端驱动和后端驱动。 前端驱动指虚拟机的驱动(virtio_blk,virtio_net,virtio_ballon等)；后端驱动是指在qemu中实现的也就是宿主主机，默认宿主机已经支持，无需安装
3. 前端驱动与后端驱动通过队列的方式来通信，一个虚拟队列，只需要前端驱动的请求直接附到后端驱动那里，不需要再由KVM陷入，然后再由qemu模拟这么繁琐

优点：减少陷入，提高效率

windows系统半虚拟化驱动 virtio-win.iso

在rpm search网站搜索virtio-win的包
```txt
[root@KVM~]rpm -ivh virtio-win-1.7.2-2-el6.noarch.rpm
[root@KVM~]rpm -ql virtio-win       #查看virtio-win包
[root@KVM~]cd /usr/share/virtio-win

drivers        #这里面放的是驱动文件
guest-agent    #这里面是.msi文件，可以直接拷贝到windows安装
*.vfd          #以软驱的形式存在，可以在安装系统时加载
x86            #32位系统
amd64          #64位系统
```

linux自带半虚拟化驱动，不需要安装 lsmod |grep virt

虚拟机中的硬件设备是如何实现的呢？有3种方式(1.软件模拟qemu 2.通过半虚拟化驱动virtio 3.通过设备直接分配)

迁移
---

实现热迁移的条件 1.宿主机后端存储为共享存储 2 宿主机的上行链路一样，网络配置 3 宿主机CPU模型一样 4. 目标主机的/etc/hosts文件有虚拟机的主机名解析

Xen概述
---

RHEL5原生支持XEN,RHEL6开始不支持xen,开始支持KVM

Xen Hypervisor位于操作系统与硬件之间，为其上运行的操作系统内核提供虚拟化的硬件环境，Xen采用混合模式(Hybrid Model),因此在Xen上的众多Domain中存在一个特权域(Privileged Domain)用来辅助Xen管理其它Domain,提供相应的虚拟资源服务，特别是其它Domain 对I/0 设备的访问，这个特权域称为Domain 0 ,而其它Domain称为Domain U

Xen架构
```txt
Domain 0

用户态       Domain U ......
内核态       Disk Net
         Xen Hypervisor
    CPU,MEM,DISK,NET硬件设备
```
Domain 0是Xen第一个启动的虚拟机，它只有CPU和内存，本身没有网络和磁盘，用来管理DomainU的IO，我们登录不了Domain0也看不到里面的信息






