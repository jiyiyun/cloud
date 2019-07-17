UVP中KVM架构
---

KVM是一个内核模块，用于管理虚拟的CPU和内存，qemu在每个虚拟机中，模拟各种输入输出设备

libvirt是操作qemu-KVM的工具集，libvirt操作Hypervisor Kernel KVM module Kernel

CPU虚拟化
---

CPU虚拟化面临的问题:原生native操作系统对CPU的认识是CPU资源永远就绪；OS对CPU具有最高权限

1. CPU共享　VM使用vCPU,hypervisor将vCPU调度到物理CPU上运行，实现物理CPU资源分时复用
2. 权限管理　定义敏感指令　经典方法是“特权解除(Privilege deprivileging)”“陷入-模拟(Trap-and-Emulation)”,只有执行到特权指令的时候，才会执行“陷入-模拟”
3. VT-x　虚拟化拓展指令集 Intel VT-d AMD-V
4. KVM CPU虚拟化　(用户态模式Qemu user模式，根模式，特权级３、内核态模式KVM Kernel模式，根模式，特权等级0、客户模式guest模式，非根模式)
5. 虚拟化层开销　VMM处理开销
内存虚拟化

---

I/O虚拟化
---