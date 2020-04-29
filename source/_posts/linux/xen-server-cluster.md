---
title: xen_server_cluster
categories: linux
tag: hide
date: 2018-12-24 10:04:25
tags:
---

<font color="red" size='10'>安装</font>

6.0.201（目前6.0的最新版本）

6.0.0http://mirror.digitalone.com/xen/XenServer-6.0.201-install-cd.iso
http://downloadns.citrix.com.edgesuite.net/akdlm/6174/XenServer-6.0.0-install-cd.iso
5.6.100 SP2（目前5.0的最新版本）
http://mirror.digitalone.com/xen/XenServer-5.6.100-SP2-install-cd.iso
5.6.1 fp1
http://mirror.digitalone.com/xen/XenServer-5.6.1-fp1-install-cd.iso
5.6.0
http://mirror.digitalone.com/xen/XenServer-5.6.0-install-cd.iso
5.5.0
http://download.alyseo.com:81/contrib/XenServer/ISOs/5.5.0/FREE_XenServer-5.5.0-install-cd.iso
5.0.0
http://download.alyseo.com:81/contrib/XenServer/ISOs/5.0.0/XenServer-5.0.0-install-cd.iso


####安装全过程不再表述


###以下是挂在镜像目录操作步骤，不过有其他比较方便的方法。
1.通过SSH登录Xenserver查看卷组信息
vgdisplay     /记下VGname

2.在VG上创建用于存放ISO的LV(逻辑卷)，并分配大小和命名
lvcreate -L  15G -n iso_image  VG_XenStorage-5e690647-6b9b-c08e-0c6c-48a31d79b69d

3.格式化之前创建的LV
mkfs.ext3 /dev/ VG_XenStorage-5e690647-6b9b-c08e-0c6c-48a31d79b69d/iso_image

1. 创建本地挂在目录
mkdir /iso_image

5.系统中通过编辑/etc/fstab来设置自动挂载之前创建的逻辑卷LV
vi /etc/fstab
添加以下内容
/dev/VG_XenStorage-5e690647-6b9b-c08e-0c6c-48a31d79b69d/iso_image  /iso_image  ext3 defaults 0 0
保存并退出

6.mount此逻辑卷
mount /iso_image

7.然后去/etc/rc.d/rc.sysinit 大概在482行，取消掉下面的注释内容
                if [ -x /sbin/lvm.static ]; then
                       action $"Setting up Logical Volume Management:" /sbin/lvm.static vgchange -a y --ignorelockingfailure
                fi
把去掉即可。

保存退出，然后重启服务器

8.通过命令行创建本地iso SR
xe sr-create name-label=iso_image type=iso device-config:location=/iso_image device-config:legacy_mode=true contente-type=iso

9.Xencenter连接xenserver后会发现多了个iso_image的本地iso存储，将ISO文件丢到此目录中即可安装使用

10.如果没有发现尝试更新一下
```
xe-mount-iso-sr /iso_image
xe-toolstack-restart
```

###Windows 可以利用共享，xen server既可以实现挂载.  
在电脑中将这个ISO文件共享出来，然后再XENCENTER新建CIFS，它会让给写出这个ISO文件的共享位置的 

<font color="red" size='10'>FAQ</font>
```
1.不能删除POOL里面的虚拟机，如何解决？
选中不能删除的虚拟机所在的物理机，在console下输入#xe host-forget uuid=
uuid的信息使用#xe host-list查看
查看虚拟机详细信息 xe vm-list params=all/(name-label,uuid,networks)
关闭虚拟机 xe vm-shutdown uuid=<vm_uuid>ext3-fs error (device xvda2) in start_transaction: journal has aborted
关闭halted/running虚拟机 xe vm-reset-powerstate force=true vm=uuid
删除虚拟机 xe vm-destroy uuid=<vm_uuid>

2.增加LVM根分区容量？
fdisk /dev/xvda
Command (m for help): d
Partition number (1-4): 2
Command (m for help): n
primary partition (1-4) p
Partition number (1-4): 2
First cylinder (14-701, default 14):
Last cylinder or +size or +sizeM or +sizeK (14-701, default 701):
Command (m for help): t
Partition number (1-4): 2
Hex code (type L to list codes): 8e
Command (m for help): w

pvresize -v /dev/xvda2 重新识别卷大小
lvextend -l +100%FREE /dev/VolGroup00/LogVol00 拓展卷利用所有空余空间
resize2fs /dev/mapper/VolGroup00-LogVol00   在线调整文件系统大小
ext2online /dev/mapper/VolGroup00-LogVol00（根分区由于不能卸载用此命令）

3.传统分区增加大小：在分区表扩容、重启、动态扩容分区
yum install e2fsprogs
fdisk /dev/xvda
Command (m for help): d
Partition number (1-4): 3
Command (m for help): n
p
Partition number (1-4): 3
回车
回车
w
reboot
resize2fs /dev/xvda3（RHEL4.7用ext2online）

4.如何添加物理硬盘扩大容量？
添加硬盘扩容(需移出pool处理)
pvcreate /dev/sdb
xe sr-create type=lvm content-type=user device-config:device=/dev/disk/by-id/scsi-SATA_ST31000528AS_5VP5ZV21 name-label='local storage2'

5.虚拟机的系统时间不能修改的问题？
修改 /etc/sysctl.conf 文件,添加
# Set independent wall clock time
xen.independent_wallclock=1

或disable掉Window Time Service。

6.制作NFS ISO library 时，启动portmap,nfs服务器之后，开启端口，NFS服务使用的111和2049端口是固定的，但是mountd是经常会变的。可以指定mountd为一固定端口，这样每次启动NFS后，所有使用的端口就是固定端口了。
找到如下一行
vi /etc/services
# 1001-1009 # Unassigned
插入mountd 1001/tcp mountd 1001/udp
重启 NFS服务
xenserver中的windows vm安装后可以通过xencenter设置从光驱启动，而linux vm则没有这样的选项，可以通过以下命令行解决：
xe vm-param-set uuid=546f896a-ebe6-8071-2c31-b9214dc1d1b5 HVM-boot-policy=BIOS\ order
xe vm-param-set uuid=546f896a-ebe6-8071-2c31-b9214dc1d1b5 HVM-boot-params:order="dc"
uuid为vm的uuid，order中的d表明光驱，c表明启动硬盘。这样设置完后通过xencenter中的虚机属性也可以设置启动顺序了。

7.xenserver下pool中的主结点master崩溃掉之后，xencenter不能连接pool下的所有xenserver主机问题？
执行如下命令：#pool-emergency-transition-to-master
指示 XenServer 成员主机成为池主节点。仅在 XenServer 主机转换到紧急模式后才接受此命令。进入紧急模式意味着该成员主机所在的池中的主节点已从网络中消失，经过若干次重试仍无法连接。
#xe pool-recover-slaves 这些成员此时将指向新主节点
将成员 XenServer 主机转换为主节点后，您还应检查默认池存储库是否设置了适当的值。通过使用 xe pool-param-list 命令
并验证 default-SR 参数是否指向有效存储库，可实现此操作。

8.正常模式下，更改POOL的master
在非主结点master 下，执行如下命令#pool-designate-new-master host-uuid=<要成为新主节点的成员 XenServer 主机的 UUID> 指定该 XenServer 主机成为现有池的主节点，将主节点主机的角色有序移交给资源池中的其他主机。
此命令仅在当前主节点处于联机状态时生效，并且不是下列紧急模式命令的替代项。

9.xencenter中不能显示 CPU，内存，硬盘 信息
在加入POOL时，要保证xenserver的系统时间和master同步或者比它快一点，即能显示硬件信息。

VDI重启或关机已有很长时间，电源管理功能已失效，则执行下列命令重置VDI电源状态：

xe vm-reset-powerstate name-label=[VDI name] --force

必须带--force参数。如果依然无法生效(如下显示host is still live啥的)，




则执行下列操作，在vm所在的host的console上运行命令
--------------------------
xe vm-list name-label= [VDI name]  //找出vm对应的uuid，如下4f6a52ec-f90axxxxxx




xe vm-shutdown uuid=[上条命令找出的uuid] --force //使用uuid作为参数进行关机操作，稍后到XC观察结果






如果上述命令在等待片刻后，还是不生效，则执行下列操作
---------------------------------------
xe vm-list name-label= [VDI name]           //找出vm对应的uuid
list_domains                               //根据上述uuid,找出vm对应的domid
/opt/xensource/debug/destroy_domain -domid [domid,上条命令列出的左边3位数字] //强制关机
----------------------------------------
注：该组命令存在一定的隐患，不到万不得已，不要使用

如果命令执行时一直没有任何回显，处理在等待状态，则通常为xapi协议出现了问题，只要在对应用的XenServer上执行以下命令，就可以得到VDI正常的电源状态（通常为正在运行），然后就可以再次强制关机了。
       service xapi restart


如果上述命令都没有任何效果，且在这台XenServer所有的VDI均停止响应（比如qfarm /load显示所有其上运行的XenApp server都未在工作负载选举列表中），则需要强制重启整台XenServer了。



2.Pool Master更换到另一台XenServer

这分为Pool Master是正常还是异常2种情况，正常情况下可能要对Pool Master做一些停机维护，比如换内存条啥的，此时在Pool Master正常工作的情况下执行以下命令：
-------------------
xe pool-ha-disable
xe host-list
xe pool-designate-new-master host-uuid= [uuid]//这里的uuid是新的Pool Master的uuid
xe pool-ha-enable
---------------------
注:池如果配置了HA,才需要执行头尾2条

如果Pool Master根本就起不来，比如做RAID1的2块盘都坏掉，此时得在池中要替换为新POOL MASTER的XenServer上执行以下命令：
----------------
xe host-list
xe pool-designate-new-master host-uuid= [uuid]//先尝试一下能否变成PM，如果不行，继续执行以下命令
xe pool-emergency-transition-to-master        //强制转换为PM
xe pool-recover-slaves                       //强制更新池成员的PM指向到这个新的PM

然后还需在成员服务器上运行以下命令
xe pool-emergency-reset-master master-address=[新PM的IP地址] //将PM指向到新的PM
----------------



3.XenServer如何进入单用户模式

比如忘记XenServer的root密码，现实中也有因为日志使磁盘空间满无法进入多用户模式，但可以进入单用户模式的情况，然后就可以清理一下磁盘空间了。
-------------------------
。启动XenServer，在看到boot文字提示的时候（也就是XenServer引导前），输入menu.c32，然后回车
。出现启动选项的时候，在5秒内，按TAB选择。（如果默认没有高亮，可以按两下ESC键） //突然想到微信[打飞机]游戏，昨晚发现在屏幕任意一点双击就可以放炸弹，而不需要去点左下角位置的炸弹图标，高兴，哈哈
。然后在现实的启动参数中，在最后的— /boot前面，加上single参数，就可以进入单用户模式了
---------------------------
上述图文操作见：http://xenme.com/386


4.重启XAPI协议栈
有时XenServer跟XenCenter或Pool master通信不正常，可以尝试重启一下xapi协议栈
---------------------------
xe-toolstack-restart
Stopping xapi: ..[ OK ]
Stopping the v6 licensing daemon: [ OK ]
Stopping the memory ballooning daemon: [ OK ]
Stopping perfmon: [FAILED]
Stopping the fork/exec daemon: [ OK ]
Starting the fork/exec daemon: [ OK ]
Starting perfmon: [ OK ]
Starting the memory ballooning daemon: .[ OK ]
Starting the v6 licensing daemon: [ OK ]
Starting xapi: ..start-of-day complete.[ OK ]
done.
-------------------------------
但上述重启的服务比较多，有时会出错，保险起见就先仅重启一下xapi服务
------------
service xapi restart

查看网口的物理链路状态: mii-tool


-w参数是连接监控端口的状态，一旦有down的端口便会显示出来。有link partner状态信息的应当是交换机端设置为trunk模式的端口，否则是默认的access模式

5.Domain-0 CPU利用率高
运行top命令可以查看到是xapi进程非常高，还有一些是qemu-dm进程，用xentop命令可以查看到所有vdi和整个domain-0的cpu利用率，通过以下步骤处理：



1.在xentop -f 界面按S(Sort Order)键直到cpu%被排序，查看哪些vdi的cpu利用率高（通常都超过100%）,则用xe vm-shutdown name-label= --force将它们强制结束掉

2.对于已经被shutdown的vdi，使用xencenter中start on 从其他XenServer上启动vdi，以让用户vdi可用。

3.间或停止xapi进程（过一会儿会自动启动）

###如果因为Pool中Master主机由于某种原因导致失效，会引起整个Pool进入紧急模式，恢复步骤如下：

在成员服务器上输入如下命令

# xe host-emergency-ha-disable              (关闭HA)
       #xe-toolstack-restart        
       这些将会关闭成员服务上运行的虚拟机
       如果确认原Master主机无法恢复，则需要手工指定Master主机，在要将某一台成员服务器设置成新的Master主机，请在该服务器上输入如下命令：
       #xe pool-emergency-transition-to-master
       #xe pool-recover-slaves
      然后在原来的Master主机上运行如下命令，将其设置为Pool的成员服务器。
       #xe pool-emergency-reset-master master-address=<IP ADDRESS of the new master>

如果原Master主机确定崩溃，只能重装，使用原机器名和IP地址重装后无法加入到Pool中，需先清理掉该主机的信息才能添加。使用如下命令删除主机：

#xe pool-eject host uuid=<host_uuid>
      UUID的查找，可以通过如下命令进行：

#xe host-list

正常模式下，要切换Master主机，可以使用下列命令：

# xe pool-designate-new-master host-uuid=
```


