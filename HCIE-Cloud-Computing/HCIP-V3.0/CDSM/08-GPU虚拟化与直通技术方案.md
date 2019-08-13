高清制图特性分类
- GPU直通
- GPU硬件虚拟化

GPU Graphic Processing Unit

GPU直通图形桌面方案技术原理
- 每个图形加速虚拟机绑定一个GPU满足3D应用渲染要求
- 剩余CPU资源创建不带GPU的普通虚拟机

GPU直通技术是将每个物理GPU绑定一个虚拟机，该虚拟机独享GPU卡，通过驱动直接访问GPU,本特性兼容性好，支持符号最新的DirctX、OpenGL规范的3D应用

```txt
        UVP Domin U VM(Graphics Desktop)
app            HDP Server     <-------------->HDP Client
         Windows GDI/DirctX/OpenGL Interface
         Windows Graphics Subsystem
         Windows Graphics dirver Interface

             NVIDIA       Driver
             UVP      Hypervisor
                GPU
```
HDP Plus会对GPU渲染后的画面进行实时全屏抓取、识别、和压缩等处理。

GPU直通考虑要素　服务器　CPU 桌面OS 配套TC 桌面规格　网络带宽　GPU

由于服务器上支持的显卡数不可能太多，这导致GPU直通比较适合高性能图形需求比例低的场景

- GPU卡和虚拟机是一对一关系
- 绑定了GPU的虚拟机不能进行迁移(包括磁盘迁移)以及HA
- GPU直通虚拟机不支持内存复用功能
- GPU直通不支持休眠和唤醒功能
- GPU直通不支持创建和恢复内存快照
- GPU直通不支持通过VNC方式登录
- 非HDP Plus模式不支持Aero效果
- HDP Plus模式只支持windows客户端接入

GPU硬件虚拟化图形桌面方案
---

GPU硬件虚拟化使用单个显卡为多个图形桌面提供显卡能力

虚拟化平台将一块物理显卡虚拟化成多个虚拟显卡，每个GPU虚拟机可以绑定一个vGPU，满足3D应用渲染要求

GPU硬件虚拟化虚拟机和GPU直通虚拟机一样，可以通过NVDIA显卡驱动可以直接访问GPU硬件资源，具有与PC一样的兼容性

> 一块支持GPU硬件虚拟化的显卡(NVIDIA GRID K1/K2)可以虚拟成不同类型的多个vGPU

> 每个图形加速虚拟机绑定一个vGPU满足３D应用渲染要求。剩余CPU资源可以创建不带vGPU的虚拟机

GPU硬件虚拟化图形方案关键技术

硬件虚拟vGPU(Virtual GPU)、高密度并发、低延迟显示(GRID SDK、HDP Server)、VNC通道显示(bGPU FB)

硬件虚拟GPU(Virtual GPU) 使用虚拟化技术将一个物理GPU虚拟成多个供虚拟机使用，与GPU共享的软件虚拟化相比，GPU硬件虚拟化是在硬件显卡上实现的虚拟化，性能更高。采用高性能SMX流处理器

高密度并发　k1显卡有４个GPU和16G显存，最多支持32中低端用户;K2显卡有2个GPU和８G显存支持16高端用户

高速图形指令处理　NVIDIA vGPU Driver 、vGPU Support 每个vGPU可以通过NVIDIA的显卡驱动直接访问GPU硬件资源，具有与PC一样的性能和兼容性

GPU硬件虚拟化图形桌面应用约束
---

- 绑定了vGPU的虚拟机不能进行迁移(包括磁盘迁移)以及HA
- 同一物理GPU同时只支持一种类型的vGPU
- GPU硬件虚拟化的虚拟机不支持内存复用功能

应用场景
---

Designer: 3D图形总装设计师(计算、渲染密集型)
Power User:3D图形部件设计人员(计算、渲染中载)
Knowlwdge worker:轻量级3D图形用户

||GPU直通-windows|GPU硬件虚拟化-windows|
|:---:|:---:|:---|
|方案特点|用户独享GPU,相当于独立显卡|将硬件GPU虚拟化成多个vGPU，每个用户独占一个vGPU|
|用户类型|Designer,Power User|Designer,Power User,Knowledge worker|
|适用场景|复杂图纸编辑，图形组装；视频编辑，中高端游戏|中等复杂图纸编辑；中端游戏|
|操作系统|windows 7/10|windows 7/10|
|OpenGL/DirctX|OpenGL 4.4/DirectX9/10/11|OpenGL 4.4/DirectX9/10/11|