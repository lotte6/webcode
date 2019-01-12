---
title: linux_kvm
categories: tech
tag: hide
date: 2018-12-18 23:25:05
tags:
---
yum -y install qemu-kvm python-virtinst libvirt tunctl bridge-utils virt-manager qemu-kvm-tools virt-viewer virt-v2v libguestfs-tools


vim /etc/libvirt/qemu.conf 修改配置
```
user = "root"
group = "root"
clear_emulator_capabilities = 0
```
/etc/init.d/libvirtd start



virsh -c qemu:///system list          #查看虚拟机环境

lsmod |grep kvm          ##查看kvm模块支持

virsh --version          ##查看虚拟工具版本
virt-install --version
ln -s /usr/libexec/qemu-kvm /usr/bin/qemu-kvm

手动配置虚拟网桥 建立em1和em2
locate NetworkManager  没有 这个NetworkManager服务 先不管

cd  /etc/sysconfig/network-scripts/

cp  ifcfg-em1 ifcfg-br1
vim ifcfg-em1 添加 BRIDGE="br1"
```
DEVICE="em1"
BOOTPROTO="static"
HWADDR="B8:CA:3A:FB:A2:C9"
NM_CONTROLLED="yes"
ONBOOT="yes"
IPADDR=42.62.69.8
NETMASK=255.255.255.128
GATEWAY=42.62.69.1
TYPE="Ethernet"
BRIDGE="br1"        ##这一行
UUID="7da53548-ec23-45f1-84cf-f3f14e336438"
```
vim ifcfg-br1     修改
```
DEVICE="br1"     ##修改
BOOTPROTO="static"
HWADDR="B8:CA:3A:FB:A2:C9"
NM_CONTROLLED="yes"
ONBOOT="yes"
IPADDR=42.62.69.8
NETMASK=255.255.255.128
GATEWAY=42.62.69.1
TYPE="Bridge"     ##修改
UUID="7da53548-ec23-45f1-84cf-f3f14e336438"
```
查看网桥
brctl show


建立磁盘格式文件有2种
qcow2格式是kvm支持的标准格式，raw格式为虚拟磁盘文件通用格式。
有测试数据表明raw格式的I/O性能略高于qcow2格式，但是在加密，容量，快照方面qcow2格式有优势。
mkdir  /data/qemu-kvm
qemu-img create -f qcow2 /data/qemu-kvm/centos6.5_100G_1.qcow2 100G          //建立qcow2格式磁盘文件
qemu-img create -f raw /data/qemu-kvm/test_50G.raw 50G               //建立raw格式磁盘文件

查看已经创建的磁盘文件
qemu-img info /data/qemu-kvm/test_50G.raw

centos6.5_100G_test.qcow2


virsh list --all                         ##列表虚拟机
virsh destroy centos6.5_test_50G     ##强制关闭电源
virsh undefine  centos6.5_test_50G      ##删除虚拟机

创建虚拟机
```
virt-install --name=centos6.5_100G_1   --os-variant=rhel6 --ram 1024 --vcpus=2 --disk path=/data/qemu-kvm/centos6.5_100G_1.qcow2,format=raw,size=100,bus=virtio --cdrom=/root/CentOS-6.5-x86_64-bin-DVD1.iso --accelerate --network bridge=br1,model=virtio   --network bridge=br2,model=virtio  --vnc --vncport=5901 --vnclisten=0.0.0.0 --noautoconsole

virt-install --name=sdk-app1   --os-variant=rhel6 --ram 16384 --vcpus=4 --disk path=/data/qemu-kvm/sdk-app1.qcow2,format=qcow2,size=50,bus=virtio  --accelerate --network bridge=br1,model=virtio   --network bridge=br2,model=virtio  --vnc --vncport=5902 --pxe --vnclisten=192.168.1.164 --noautoconsole


/usr/sbin/virt-install --name=centos6.5_100G_test --os-variant=rhel6 --ram 2048 --vcpus=2 --disk path=/data/qemu-kvm/centos6.5_100G_test.qcow2,format=qcow2,size=100,bus=virtio --accelerate --network bridge=br1,model=virtio --network bridge=br2,model=virtio --vnc --vncport=5902 --vnclisten=0.0.0.0 --pxe --noautoconsole
```

KVM VNC密码 anzhi.com@)!$

克隆一个一模一样虚拟机
virt-clone --connect=qemu:///system -o centos1 -n centos2 -f /datata/kvm/centos2.img


--------------------------------------------------------------------------------------------------------------------------------------------
定义kvm虚拟化存储池
mkdir /data1/kvmfs

定义存储池与其目录
virsh pool-define-as vmdisk (名字)--type dir --target /data1/kvmfs/  (目录)

创建已定义的存储池
virsh pool-build vmdisk      

查看已定义的存储池，存储池不激活无法使用
virsh pool-list --all

自动启动已定义的存储池
virsh pool-autostart vmdisk
激活
virsh pool-start vmdisk

在存储池中创建虚拟机存储卷
virsh vol-create-as vmdisk test600G.qcow2 600G --format qcow2

```
virt-install --name=WindowsXp_YLMF_100G   --os-variant=winxp --ram 2048 --vcpus=4 --disk path=/data/qemu-kvm/windows_xp100G.raw,format=raw,size=100,bus=virtio --cdrom=/root/YLMF_201306CJB-XP.iso --accelerate --network bridge=br1,model=virtio --network bridge=br2,model=virtio   --vnc --vncport=5901 --vnclisten=0.0.0.0 --noautoconsole  --disk path=/root/virtio-win-0.1-74.iso,device=cdrom,perms=ro

virt-install --connect qemu:///system --arch=x86_64 \
-n WindowsXp_YLMF_100G -r 2048 --vcpus=4 \
--disk path=/data/qemu-kvm/windows_xp100G.qcow2,format=qcow2,size=100,bus=ide,device=disk,cache=writeback \
--cdrom=/root/YLMF_201306CJB-XP.iso  --autostart \
--disk path=/root/virtio-win-0.1-74.iso,device=cdrom,perms=ro \
--disk path=/root/virtio-win-drivers-20120712-1.vfd,device=floppy \
--vnc --vncport=5901 --vnclisten=0.0.0.0 \
--noautoconsole --os-type windows --os-variant=winxp \
--network bridge=br1,model=virtio --network bridge=br2,model=virtio
```

virsh list --all                         ##列表虚拟机
virsh destroy      WindowsXp_YLMF_100G  ##强制关闭电源
virsh undefine  WindowsXp_YLMF_100G     ##删除虚拟机

anzhi.com@)!$ANKVMihz

qemu-img create -f qcow2 /data/qemu-kvm/windows_xp100G.qcow2  100G


qemu-img create -f qcow2 /data/qemu-kvm/win_xp_drive_1G.qcow2 1G


virsh attach-disk WindowsXp_YLMF_100G /root/virtio-win-0.1-74.iso hdc --type cdrom

virsh attach-disk WindowsXp_YLMF_100G /data/qemu-kvm/win_xp_drive_1G.qcow2 vdc

<target dev='hda' bus='ide'/>
<target dev='vda' bus='virtio'/>

<address type='drive' controller='0' bus='0' target='0' unit='0'/>


克隆虚拟机
virt-clone -o model_300G -n BI_ChunWen300G -f /data1/qemu-kvm/BI_ChunWen300G.qcow2
添加vnc端口号 keymap='en-us' passwd='anzhi.com@)!$ANKVMihz' 密码

vim /etc/udev/rules.d/70-persistent-net.rules  修改网卡名称
ip a  查看MAC地址信息
vim /etc/sysconfig/network-scripts/ifcfg-eth0     修改MAC地址 和IP
vim /etc/sysconfig/network-scripts/ifcfg-eth1     修改MAC地址 和IP
重启网卡。

一、简介：
snapshot（快照）可以把虚拟机某个时间点的内存、磁盘文件等的状态保存为一个镜像文件。通过这个镜像文件，可以在以后的任何时间来恢复虚拟机在当时创建snapshot的状态，这个在使用虚拟机来做测试的时候很有用。

二、创建快照-KVM：

需注意在虚拟机运行时创建快照不会报错，但会出现一些莫名其妙的问题，像恢复快照失败、快照名为空等，所以在创建快照前要先关闭虚拟机。

2.1创建

//raw格式

kvm虚拟机的raw格式磁盘文件不支持快照功能，在创建快照前需要先转换为qcow或qcow2格式。

[root@kvmserver xp_4_test]# qemu-img info disk.raw
```
image: disk.raw

file format: raw

virtual size: 100M (104857600 bytes)

disk size: 6.1M
```
[root@kvmserver xp_4_test]# qemu-img snapshot -c s1 disk.raw //raw格式的转换报错
```
qemu-img: Could not create snapshot'snapshot01': -95 (Operationnot supported)

//qcow2格式

[root@kvmserver xp_4_test]# qemu-img info disk.qcow2

image:disk.qcow2

file format: qcow2

virtual size: 100M (104857600 bytes)

disk size: 4M
```
[root@kvmserver xp_4_test]# qemu-img snapshot -c s1 disk.qcow2

[root@kvmserver xp_4_test]# qemu-img info disk01.qcow2//可以看到刚新建的快照s1
```
image: disk.qcow2

file format: qcow2

virtual size: 100M (104857600 bytes)

disk size: 80M

cluster_size: 65536

Snapshot list:

ID       TAG                VMSIZE               DATE      VM CLOCK

1        s1                       02012-05-10 15:20:40  00:00:00.000
```
[root@kvmserver xp_4_test]# ls -lh

总用量 7G

-rw-r--r--. 1 qemu qemu 108M  5月 10 15:03disk.qcow2

-rw-r--r--. 1 qemu qemu 6.5G  5月 1015:03xp_4_test.img

创建快照后不会有新的镜像文件产生；disk.qcow2镜像文件创建时的大小为100M，这里显示的大小为108M，这是因为快照位于disk.qcow2镜像文件内而没有单独生成一个文件。



2.2列出镜像的所有快照

[root@kvmserver xp_4_test]# qemu-img snapshot  -l  disk.qcow2
```
Snapshot list:

ID       TAG                VMSIZE               DATE      VM CLOCK

1        s1                       02012-05-10 15:20:40  00:00:00.000

2        s2                       02012-05-10 15:32:37  00:00:00.004
```
2.3快照恢复

恢复快照同样也需要在关闭虚拟机的情况下进行，下面的恢复会使虚拟机恢复到2012-05-1015:20:40的状态，在此时间点后对磁盘disk.qcow2的操作将全部失效

[root@kvmserver xp_4_test]# qemu-imgsnapshot -a s1 disk.qcow2

2.4 删除快照

[root@kvmserver xp_4_test]# qemu-img snapshot -d s1 disk.qcow2


三、利用libvirt使用快照
1 同样先确认镜像的格式为qcow2
  [root@nc1 boss]#qemu-img info test.qcow2
  ```
  image: test.qcow2
  file format: qcow2
  virtual size: 10G (10737418240 bytes)
  disk size: 1.1G
  cluster_size: 65536
```
2 创建并启动以test.qcow2作为镜像的虚拟机,假设虚拟机名称为testsnp，如果虚拟机没有启动，也可创建快照，但是意义不大，快照size为0
  开始使用配置文件来创建指定虚拟机的快照
  ```
  <domainsnapshot>
    <name>snapshot02</name> //快照名
    <description>Snapshot of OS install and updates</description>//描述
    <disks>
      <disk name='/home/guodd/boss/test.qcow2'>           //虚拟机镜像的绝对路径
      </disk>
      <disk name='vdb' snapshot='no'/>
    </disks>
  </domainsnapshot>
  ```
  保存为snp.xml,开始创建

  [root@nc1 boss]#virsh snapshot-create testsnp snp.xml  //即以snp.xml作为快照的配置文件为虚拟机testsnp创建快照

   Domain snapshot snapshot02 created from 'snp.xml'
  
3 查看虚拟机testsnp已有的快照

  [root@nc1 boss]# virsh snapshot-list testsnp
  ```
  Name                 Creation Time             State
  ---------------------------------------------------
  1315385065           2011-09-07 16:44:25 +0800 running        //1315385065创建时间比snapshot02早
  snapshot02           2011-09-07 17:32:38 +0800 running
  同样地，也可以通过qemu-img命令来查看快照
  [root@nc1 boss]# qemu-img info test.qcow2
   image: test.qcow2
   file format: qcow2
   virtual size: 10G (10737418240 bytes)
   disk size: 1.2G
   cluster_size: 65536
   Snapshot list:
   ID        TAG                 VM SIZE                DATE       VM CLOCK
   1         1315385065             149M 2011-09-07 16:44:25   00:00:48.575
   2         snapshot02             149M 2011-09-07 17:32:38   00:48:01.341
```
4 可以通过snapshot-dumpxml命令查询该虚拟机某个快照的详细配置
[root@nc1 boss]# virsh snapshot-dumpxml testsnp 1315385065

```

<domainsnapshot>
  <name>1315385065</name>
  <description>Snapshot of OS install and updates</description>
  <state>running</state>     //虚拟机状态  虚拟机关机状态时创建的快照状态为shutoff（虚拟机运行时创建的快照，即使虚拟机状态为shutoff，快照状态依然为running）
  <creationTime>1315385065</creationTime>   //虚拟机的创建时间 Readonly 由此可以看出没有给快照指定名称的话，默认以时间值来命名快照
  <domain>
    <uuid>afbe5fb7-5533-d154-09b6-33c869a05adf</uuid> //此快照所属的虚拟机(uuid)
  </domain>
</domainsnapshot>

查看第二个snapshot
[root@nc1 boss]# virsh snapshot-dumpxml testsnp snapshot02

<domainsnapshot>
   <name>snapshot02</name>
   <description>Snapshot of OS install and updates</description>
   <state>running</state>
   <parent>
     <name>1315385065</name>        //当前快照把前一个快照作为parent
   </parent>
   <creationTime>1315387958</creationTime>
   <domain>
     <uuid>afbe5fb7-5533-d154-09b6-33c869a05adf</uuid>
   </domain>
</domainsnapshot>

5 查看最新的快照信息
  [root@nc1 boss]# virsh snapshot-current testsnp
  <domainsnapshot>
    <name>1315385065</name>
    <description>Snapshot of OS install and updates</description>
    <state>running</state>
    <creationTime>1315385065</creationTime>  
    <domain>
      <uuid>afbe5fb7-5533-d154-09b6-33c869a05adf</uuid>
    </domain>
   </domainsnapshot>
```
6 使用快照，指定使用哪一个快照恢复虚拟机
[root@nc1 boss]# virsh snapshot-revert testsnp snapshot02

7 删除指定快照
  [root@nc1 boss]# virsh snapshot-delete testsnp snapshot02
  Domain snapshot snapshot02 deleted

附：
```
Snapshot (help keyword 'snapshot')
    snapshot-create                Create a snapshot from XML
    snapshot-create-as             Create a snapshot from a set of args
    snapshot-current               Get the current snapshot
    snapshot-delete                Delete a domain snapshot
    snapshot-dumpxml               Dump XML for a domain snapshot
    snapshot-list                  List snapshots for a domain
    snapshot-revert                Revert a domain to a snapshot
```

http://www.ibm.com/developerworks/tivoli/library/t-snaptsm1/index.html#row


给KVM虚拟机增加硬盘
自己搭建VPS系列文章

自己搭建VPS系列文章，介绍了如何利用自己的计算机资源，通过虚拟化技术搭建VPS。

在互联网2.0时代，每个人都有自己的博客，还有很多专属于自己的互联网应用。这些应用大部分都是互联网公司提供的。对于一些有能力的开发人员(geek)来说，他们希望做一些自己的应用，可以用到最新最炫的技术，并且有自己的域名，有自己的服务器。这时就要去租一些互联网上的VPS主机。VPS主机就相当于是一台远程的计算机，可以部署自己的应用程序，然后申请一个域名，就可以正式发布在互联网上了。本站“@晒粉丝” 就使用的Linode主机VPS在美国达拉斯机房。

其实，VPS还可以自己搭建的。只要我们有一台高性能的服务器，一个IP地址，一个路由。可以把一台高性能的服务器，很快的变成5台，10台，20台的虚拟VPS。我们就可以在自己的VPS上面的，发布各种的应用，还可以把剩余的服务器资源租给其他的互联网使用者。 本系列文章将分为以下几个部分介绍：“虚拟化技术选型”，“动态IP解析”，“在Ubuntu上安装KVM并搭建虚拟环境”，“给KVM虚拟机增加硬盘”，“VPS内网的网络架构设计”，“VPS租用云服务”。


前言

虚拟机作为灵活配置的服务器主机，给系统运维和管理带来了巨大的便利。CPU，内存，硬盘，网络等的可配置，给了虚拟机非常强大的优势，是物理机不能比拟的。今天讲一下如何给KVM虚拟机增加新硬盘。

kvm-disk

目录

host增加物理硬盘并分区
通过virsh给guest增加文件硬盘
通过virsh给guest增加分区硬盘
1. host增加物理硬盘并分区

HOST作为KVM的宿主计算机，管理所有GUEST虚拟机。我们通过给HOST增加物理硬盘，然后分给GUEST，从而实现给虚拟机硬盘扩容的效果。

如何给计算机增加物理硬盘并分区，请参考：多硬盘分区管理fdisk 文章

查看HOST机的硬盘


~ sudo fdisk -l
```
Disk /dev/sda: 299.4 GB, 299439751168 bytes
255 heads, 63 sectors/track, 36404 cylinders, total 584843264 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000efd7c

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1            2048    97656831    48827392   82  Linux swap / Solaris
/dev/sda2        97656832   136718335    19530752   83  Linux
/dev/sda3       136718336   214843335    39062500   83  Linux
/dev/sda4   *   214843392   215037951       97280   83  Linux

Disk /dev/sdb: 1999.3 GB, 1999307276288 bytes
255 heads, 63 sectors/track, 243068 cylinders, total 3904897024 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xf919a976

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048  1952448511   976223232    7  HPFS/NTFS/exFAT
/dev/sdb2      1952448512  3904897023   976224256    5  Extended
/dev/sdb5      1952450560  2267023360   157286400+  83  Linux
/dev/sdb6      2267025409  2581596160   157285376   83  Linux
/dev/sdb7      2581598209  2896168960   157285376   83  Linux
/dev/sdb8      2896171009  3210741760   157285376   83  Linux
/dev/sdb9      3210743809  3525314560   157285376   83  Linux
/dev/sdb10     3525316609  3904897023   189790207+  83  Linux

~ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        37G  3.4G   32G  10% /
udev             24G  4.0K   24G   1% /dev
tmpfs           9.5G  1.1M  9.5G   1% /run
none            5.0M  8.0K  5.0M   1% /run/lock
none             24G  152K   24G   1% /run/shm
none            100M   28K  100M   1% /run/user
cgroup           24G     0   24G   0% /sys/fs/cgroup
/dev/sda2        19G  3.6G   14G  21% /home
/dev/sda4        92M   34M   54M  39% /boot
/dev/sdb1       931G  100G  832G  11% /disk/sdb1
/dev/sdb6       148G  188M  140G   1% /disk/sdb6
```
下面将进行两个测试：
通过virsh给guest增加文件硬盘：通过文件硬盘的镜像/disk/sdb6/c1d6.img
通过virsh给guest增加分区硬盘：直接使用分区硬盘/dev/sdb5

2. 通过virsh给guest增加文件硬盘

创建文件硬盘的镜像

```
~ cd /disk/sdb6/
~ sudo qemu-img create -f raw /disk/sdb6/c1d6.img 10G
Formatting '/disk/sdb6/c1d6.img', fmt=raw size=10737418240

~ ls -l
-rw-r--r-- 1 root root 10737418240 Jul  8 16:37 c1d6.img
drwx------ 2 root root       16384 Jul  8 09:03 lost+found/
通过virsh管理工具加载硬盘


~ sudo virsh
Welcome to virsh, the virtualization interactive terminal.
Type:  'help' for help with commands
       'quit' to quit

#查看系统内的虚拟机
virsh # list
Id Name State
----------------------------------------------------
5 server3 running
6 server4 running
7 d2 running
8 r1 running
9 server2 running
12 c1 running

#在这里我们要对c1进行硬盘扩容
virsh # edit c1

#找到硬盘配置(原来的系统硬盘)

<disk type='file' device='disk'>
<driver name='qemu' type='raw'/>
<source file='/disk/sdb1/c1.img'/>
<target dev='vda' bus='virtio'/>
<address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
</disk>

#增加文件硬盘,vdb
<disk type='file' device='disk'>
<driver name='qemu' type='raw' cache='none'/>
<source file='/disk/sdb6/c1d6.img'/>
<target dev='vdb' bus='virtio'/>
<address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
</disk>
```
#保存退出
重启c1虚拟机


#请使用destroy命令，reboot和shutdown不管用。
~ virsh # destroy c1
Domain c1 destroyed

#list找不到c1 
~ virsh # list
```
 Id    Name                           State
----------------------------------------------------
 5     server3                        running
 6     server4                        running
 7     d2                             running
 8     r1                             running
 9     server2                        running
```
#启动虚拟机c1
~ virsh # start c1
Domain c1 started

#进入虚拟机c1
~ console c1
在c1中，进行硬盘查检并分区


~ sudo fdisk -l
```
Disk /dev/vda: 42.9 GB, 42949672960 bytes
16 heads, 63 sectors/track, 83220 cylinders, total 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000516aa

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048      499711      248832   83  Linux
/dev/vda2          501758    83884031    41691137    5  Extended
/dev/vda5          501760    83884031    41691136   8e  Linux LVM

Disk /dev/vdb: 10.7 GB, 10737418240 bytes
16 heads, 63 sectors/track, 20805 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/vdb doesn't contain a valid partition table

Disk /dev/mapper/u1210-root: 38.4 GB, 38394658816 bytes
255 heads, 63 sectors/track, 4667 cylinders, total 74989568 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/mapper/u1210-root doesn't contain a valid partition table

Disk /dev/mapper/u1210-swap_1: 4294 MB, 4294967296 bytes
255 heads, 63 sectors/track, 522 cylinders, total 8388608 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/mapper/u1210-swap_1 doesn't contain a valid partition table
/dev/vdb已经被识别，接下来 分区,格式化,挂载,使用

硬盘分区


~ sudo fdisk /dev/vdb

Command (m for help): p

Disk /dev/vdb: 161.1 GB, 161061274112 bytes
16 heads, 63 sectors/track, 312076 cylinders, total 314572801 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x3b49c6a0

   Device Boot      Start         End      Blocks   Id  System

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
Using default value 1
First sector (2048-314572800, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-314572800, default 314572800):
Using default value 314572800

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

#分区生效
~ sudo partprobe

~ sudo fdisk -l
Disk /dev/vdb: 10.7 GB, 10737418240 bytes
2 heads, 17 sectors/track, 616809 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xf0432cd6

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971519    10484736   83  Linux

格式化


~ sudo mkfs -t ext4 /dev/vdb1
mke2fs 1.42.5 (29-Jul-2012)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
9830400 inodes, 39321344 blocks
1966067 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=4294967296
1200 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
挂载


~ sudo mount /dev/vdb1 /home/cos/hadoopb

~ df -h
Filesystem              Size  Used Avail Use% Mounted on
/dev/mapper/u1210-root   36G  1.1G   33G   4% /
udev                    2.0G  4.0K  2.0G   1% /dev
tmpfs                   791M  232K  791M   1% /run
none                    5.0M     0  5.0M   0% /run/lock
none                    2.0G     0  2.0G   0% /run/shm
none                    100M     0  100M   0% /run/user
/dev/vda1               228M   29M  188M  14% /boot
/dev/vdb1               9.9G  151M  9.2G   2% /home/cos/hadoopb
使用
/home/cos/hadoopb的目录，已经挂载到了/dev/vdb1上面，我可以在hadoopb下载做任何的操作。
```
3. 通过virsh给guest增加分区硬盘

直接使用HOST的分区硬盘/dev/sdb5，做个虚拟机c1的分区


virsh # edit c1
```
#新增新硬盘vbc
<disk type='block' device='disk'>
<driver name='qemu' type='raw' cache='none'/>
<source dev='/dev/sdb5'/>
<target dev='vbc' bus='virtio'/>
</disk>

virsh # destroy c1
Domain c1 destroyed

virsh # start c1
Domain c1 started

virsh # console c1
登陆虚拟c1，查看硬盘信息


sudo fdisk -l
[sudo] password for cos:

Disk /dev/vda: 42.9 GB, 42949672960 bytes
16 heads, 63 sectors/track, 83220 cylinders, total 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000516aa

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048      499711      248832   83  Linux
/dev/vda2          501758    83884031    41691137    5  Extended
/dev/vda5          501760    83884031    41691136   8e  Linux LVM

Disk /dev/vdb: 10.7 GB, 10737418240 bytes
2 heads, 17 sectors/track, 616809 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xf0432cd6

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971519    10484736   83  Linux

Disk /dev/vdc: 161.1 GB, 161061274112 bytes
4 heads, 4 sectors/track, 19660800 cylinders, total 314572801 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x3b49c6a0

   Device Boot      Start         End      Blocks   Id  System
/dev/vdc1            2048   314572800   157285376+  83  Linux

Disk /dev/mapper/u1210-root: 38.4 GB, 38394658816 bytes
255 heads, 63 sectors/track, 4667 cylinders, total 74989568 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/mapper/u1210-root doesn't contain a valid partition table

Disk /dev/mapper/u1210-swap_1: 4294 MB, 4294967296 bytes
255 heads, 63 sectors/track, 522 cylinders, total 8388608 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/mapper/u1210-swap_1 doesn't contain a valid partition table
已经被正确识别
Disk /dev/vdc: 161.1 GB, 161061274112 bytes
```