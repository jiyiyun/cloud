命令
行查看ESXI 信息
---
- https://code.vmware.com/web/tool/6.7/vsphere-cli
- https://blog.51cto.com/20161215/1852260

1. 获取ESXI 服务器IP
```txt
~ # esxcli network ip interface ipv4 get
Name  IPv4 Address   IPv4 Netmask   IPv4 Broadcast  Address Type  DHCP DNS
----  -------------  -------------  --------------  ------------  --------
vmk0  192.168.5.116  255.255.255.0  192.168.5.255   STATIC           false
```

2. 默认网关
```txt
~ # esxcli network nic list
Name    PCI Device     Driver  Link  Speed  Duplex  MAC Address         MTU  Description                                          
------  -------------  ------  ----  -----  ------  -----------------  ----  -----------------------------------------------------
vmnic0  0000:001:00.0  e1000e  Up     1000  Full    00:18:71:ea:ca:38  1500  Intel Corporation 82572EI Gigabit Ethernet Controller
~ # esxcli network ip route ipv4 list
Network      Netmask        Gateway      Interface  Source
-----------  -------------  -----------  ---------  ------
default      0.0.0.0        192.168.5.1  vmk0       MANUAL
192.168.5.0  255.255.255.0  0.0.0.0      vmk0       MANUAL
```
3. 查看磁盘列表
```
~ # esxcli storage core device list
mpx.vmhba33:C0:T0:L0
   Display Name: Local PLDS CD-ROM (mpx.vmhba33:C0:T0:L0)
   Has Settable Display Name: false
   Size: 0
   Device Type: CD-ROM 
   Multipath Plugin: NMP
   Devfs Path: /vmfs/devices/cdrom/mpx.vmhba33:C0:T0:L0
   Vendor: PLDS    
   Model: DVD-RW DU8A6SH  
   Revision: DL62
   SCSI Level: 5
   Is Pseudo: false
   Status: on
   Is RDM Capable: false
   Is Local: true
   Is Removable: true
   Is SSD: false
   Is Offline: false
   Is Perennially Reserved: false
   Queue Full Sample Size: 0
   Queue Full Threshold: 0
   Thin Provisioning Status: unknown
   Attached Filters: 
   VAAI Status: unsupported
   Other UIDs: vml.0005000000766d68626133333a303a30
   Is Shared Clusterwide: false
   Is Local SAS Device: false
   Is SAS: false
   Is USB: false
   Is Boot USB Device: false
   Is Boot Device: false
   No of outstanding IOs with competing worlds: 32

t10.ATA_____ST1000DM0032D1SB102__________________________________Z9A3SHJH
   Display Name: Local ATA Disk (t10.ATA_____ST1000DM0032D1SB102__________________________________Z9A3SHJH)
   Has Settable Display Name: true
   Size: 953869
   Device Type: Direct-Access 
   Multipath Plugin: NMP
   Devfs Path: /vmfs/devices/disks/t10.ATA_____ST1000DM0032D1SB102__________________________________Z9A3SHJH
   Vendor: ATA     
   Model: ST1000DM003-1SB1
   Revision: CC62
   SCSI Level: 5
   Is Pseudo: false
   Status: on
   Is RDM Capable: false
   Is Local: true
   Is Removable: false
   Is SSD: false
   Is Offline: false
   Is Perennially Reserved: false
   Queue Full Sample Size: 0
   Queue Full Threshold: 0
   Thin Provisioning Status: unknown
   Attached Filters: 
   VAAI Status: unknown
   Other UIDs: vml.01000000002020202020202020202020205a39413353484a48535431303030
   Is Shared Clusterwide: false
   Is Local SAS Device: false
   Is SAS: false
   Is USB: false
   Is Boot USB Device: false
   Is Boot Device: true
   No of outstanding IOs with competing worlds: 32
```

```txt
~ # esxcli system --help
Usage: esxcli system {cmd} [cmd options]

Available Namespaces:
  boot                  Operations relating to host boot that allow manipulation of VMkernel boot time
                        configuration.
  coredump              Operations pertaining to the VMkernel Core dump configuration.
  module                Operations that allow manipulation of the VMkernel loadable modules and device drivers.
                        Operations include load, list and setting options.
  process               Commands relating to running processes.
  secpolicy             Options related to VMkernel access control subsystem. These options are typically in
                        place for specific workarounds or debugging. These commands should be used at the
                        direction of VMware Support Engineers.
  settings              Operations that allow viewing and manipulation of system settings.
  stats                 Access to various system statistics
  syslog                Operations relating to system logging
  visorfs               Operations pertaining to the visorfs memory filesytem.
  hostname              Operations pertaining the network name of the ESX host.
  maintenanceMode       Command to manage the system's maintenance mode. 这个可以进入维护模式，
  shutdown              Command to shutdown the system.
  snmp                  Commands pertaining to SNMPv1/v2c/v3 Agent configuration.
  time                  Commands to get and set system time.
  uuid                  Get the system UUID
  version               Commands to get version information.
  welcomemsg            Commands to get and set the welcome banner for DCUI.
```

```txt
~ # esxcli system version get
   Product: VMware ESXi
   Version: 5.5.0
   Build: Releasebuild-3248547
   Update: 3

~ # vmware -v
VMware ESXi 5.5.0 build-3248547

~ # esxcli system process stats load get
   Load1Minute: 0.02
   Load15Minutes: 0.03
   Load5Minutes: 0.01
~ # who
root            char/pty/t0     00:00   May  5 09:57:52  192.168.4.145
```

```txt
~ # esxcfg-info --help
Usage: esxcfg-info mode
  -a, --all           Print all information
  -w, --hardware      Print hardware information
  -r, --resource      Print resource information
  -s, --storage       Print storage information
  -n, --network       Print network information
  -y, --system        Print system information
  -o, --advopt        Print advanced options
  -u, --hwuuid        Print hardware uuid
  -b, --bootuuid      Print boot partition uuid
  -e, --boottype      Print boot type (VMVisor Only)
  -c, --cmdline       Print vmkernel command line
  -d, --devicetree    Print the device tree
  -F, --format        Print the information in the given format
                      Valid values are "xml" and "perl"
  -h, --help          Print this message. 
~ # 
```
```txt
~ # uname -a
VMkernel exi116 5.5.0 #1 SMP Release build-3248547 Nov 17 2015 21:38:51 x86_64 GNU/Linux
~ # date
Sun May  5 10:13:33 UTC 2019
~ # date +%D
05/05/19
```

登录EXSI 查看内存
---

```txt
~ # esxcfg-info -w |grep -i memo
            |----Device Class Name..................................Memory controller
   \==+Memory Info : 
      |----Reliable memory..........................................0 bytes
      \==+Aux Source Memory Stats : 
         |----Physical Memory Est...................................12491492 kilobytes
         |----Kernel Memory.........................................12491492 kilobytes
            |----Name...............................................userSharedMemoryHeap-1
            |----Name...............................................userSharedMemoryHeap-1
            |----Name...............................................userSharedMemoryHeap-1
            |----Memory Start Addr..................................4294967296 
            |----Memory Length......................................9999220736 

~ # esxcfg-info -w |grep CPU
   \==+CPU Info : 
         |----Reason................................................incompatible CPU
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
   \==+CPU Power Management Info : 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
                  |----CPU Speed....................................3191999262 
                  \==+CPU ID id0 : 
                  \==+CPU ID id1 : 
                  \==+CPU ID id80 : 
                  \==+CPU ID id81 : 
                  \==+CPU ID id88 : 
                  \==+CPU ID id8a : 
                     |----Number of CPUs sharing L2 Cache...........1 
                     |----Number of CPUs sharing L3 Cache...........1 
~ # 
```