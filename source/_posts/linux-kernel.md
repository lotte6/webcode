---
title: linux_kernel
categories: tech
tag: hide
date: 2018-12-18 20:40:39
tags:
---

<font color="red" size='18'>#内核的几个概念</font>

1.     Linux中主要有哪几种内核锁？
Linux的同步机制从2.0到2.6以来不断发展完善。从最初的原子操作，到后来的信号量，从大内核锁到今天的自旋锁。这些同步机制的发展伴随Linux从单处理器到对称多处理器的过渡；
伴随着从非抢占内核到抢占内核的过度。Linux的锁机制越来越有效，也越来越复杂。
Linux的内核锁主要是自旋锁和信号量。
自旋锁最多只能被一个可执行线程持有，如果一个执行线程试图请求一个已被争用（已经被持有）的自旋锁，那么这个线程就会一直进行忙循环——旋转——等待锁重新可用。要是锁未被争用，请求它的执行线程便能立刻得到它并且继续进行。自旋锁可以在任何时刻防止多于一个的执行线程同时进入临界区。
Linux中的信号量是一种睡眠锁。如果有一个任务试图获得一个已被持有的信号量时，信号量会将其推入等待队列，然后让其睡眠。这时处理器获得自由去执行其它代码。当持有信号量的进程将信号量释放后，在等待队列中的一个任务将被唤醒，从而便可以获得这个信号量。
信号量的睡眠特性，使得信号量适用于锁会被长时间持有的情况；只能在进程上下文中使用，因为中断上下文中是不能被调度的；另外当代码持有信号量时，不可以再持有自旋锁。
Linux 内核中的同步机制：原子操作、信号量、读写信号量和自旋锁的API，另外一些同步机制，包括大内核锁、读写锁、大读者锁、RCU (Read-Copy Update，顾名思义就是读-拷贝修改)，和顺序锁。
2.     Linux中的用户模式和内核模式是什么含意？
MS-DOS等操作系统在单一的CPU模式下运行，但是一些类Unix的操作系统则使用了双模式，可以有效地实现时间共享。在Linux机器上，CPU要么处于受信任的内核模式，要么处于受限制的用户模式。除了内核本身处于内核模式以外，所有的用户进程都运行在用户模式之中。
内核模式的代码可以无限制地访问所有处理器指令集以及全部内存和I/O空间。如果用户模式的进程要享有此特权，它必须通过系统调用向设备驱动程序或其他内核模式的代码发出请求。另外，用户模式的代码允许发生缺页，而内核模式的代码则不允许。
在2.4和更早的内核中，仅仅用户模式的进程可以被上下文切换出局，由其他进程抢占。除非发生以下两种情况，否则内核模式代码可以一直独占CPU：
(1) 它自愿放弃CPU；
(2) 发生中断或异常。
2.6内核引入了内核抢占，大多数内核模式的代码也可以被抢占。
3.     怎样申请大块内核内存？
在Linux内核环境下，申请大块内存的成功率随着系统运行时间的增加而减少，虽然可以通过vmalloc系列调用申请物理不连续但虚拟地址连续的内存，但毕竟其使用效率不高且在32位系统上vmalloc的内存地址空间有限。所以，一般的建议是在系统启动阶段申请大块内存，但是其成功的概率也只是比较高而已，而不是100%。如果程序真的比较在意这个申请的成功与否，只能退用“启动内存”（Boot Memory）。下面就是申请并导出启动内存的一段示例代码：
void* x_bootmem = NULL;EXPORT_SYMBOL(x_bootmem);unsigned long x_bootmem_size = 0;EXPORT_SYMBOL(x_bootmem_size);static int __init x_bootmem_setup(char *str){        x_bootmem_size = memparse(str, &str);        x_bootmem = alloc_bootmem(x_bootmem_size);        printk("Reserved %lu bytes from %p for x\n", x_bootmem_size, x_bootmem);        return 1;}__setup("x-bootmem=", x_bootmem_setup);

可见其应用还是比较简单的，不过利弊总是共生的，它不可避免也有其自身的限制：
内存申请代码只能连接进内核，不能在模块中使用。
被申请的内存不会被页分配器和slab分配器所使用和统计，也就是说它处于系统的可见内存之外，即使在将来的某个地方你释放了它。
一般用户只会申请一大块内存，如果需要在其上实现复杂的内存管理则需要自己实现。
在不允许内存分配失败的场合，通过启动内存预留内存空间将是我们唯一的选择。
4.   用户进程间通信主要哪几种方式？
（1）管道（Pipe）：管道可用于具有亲缘关系进程间的通信，允许一个进程和另一个与它有共同祖先的进程之间进行通信。
（2）命名管道（named pipe）：命名管道克服了管道没有名字的限制，因此，除具有管道所具有的功能外，它还允许无亲缘关系进程间的通信。命名管道在文件系统中有对应的文件名。命名管道通过命令mkfifo或系统调用mkfifo来创建。
（3）信号（Signal）：信号是比较复杂的通信方式，用于通知接受进程有某种事件发生，除了用于进程间通信外，进程还可以发送信号给进程本身；linux除了支持Unix早期信号语义函数sigal外，还支持语义符合Posix.1标准的信号函数sigaction（实际上，该函数是基于BSD的，BSD为了实现可靠信号机制，又能够统一对外接口，用sigaction函数重新实现了signal函数）。
（4）消息（Message）队列：消息队列是消息的链接表，包括Posix消息队列system V消息队列。有足够权限的进程可以向队列中添加消息，被赋予读权限的进程则可以读走队列中的消息。消息队列克服了信号承载信息量少，管道只能承载无格式字节流以及缓冲区大小受限等缺
（5）共享内存：使得多个进程可以访问同一块内存空间，是最快的可用IPC形式。是针对其他通信机制运行效率较低而设计的。往往与其它通信机制，如信号量结合使用，来达到进程间的同步及互斥。
（6）信号量（semaphore）：主要作为进程间以及同一进程不同线程之间的同步手段。
（7）套接字（Socket）：更为一般的进程间通信机制，可用于不同机器之间的进程间通信。起初是由Unix系统的BSD分支开发出来的，但现在一般可以移植到其它类Unix系统上：Linux和System V的变种都支持套接字。
5.     通过伙伴系统申请内核内存的函数有哪些？
在物理页面管理上实现了基于区的伙伴系统（zone based buddy system）。对不同区的内存使用单独的伙伴系统(buddy system)管理,而且独立地监控空闲页。相应接口alloc_pages(gfp_mask, order)，_ _get_free_pages(gfp_mask, order)等。
补充知识：
1.原理说明
　　Linux内核中采 用了一种同时适用于32位和64位系统的内 存分页模型，对于32位系统来说，两级页表足够用了，而在x86_64系 统中，用到了四级页表。
　　* 页全局目录(Page Global Directory)
　　* 页上级目录(Page Upper Directory)
　　* 页中间目录(Page Middle Directory)
　　* 页表(Page Table)
　　页全局目录包含若干页上级目录的地址，页上级目录又依次包含若干页中间目录的地址，而页中间目录又包含若干页表的地址，每一个页表项指 向一个页框。Linux中采用4KB大小的 页框作为标准的内存分配单元。
　　多级分页目录结构
　　1.1.伙伴系统算法
　　在实际应用中，经常需要分配一组连续的页框，而频繁地申请和释放不同大小的连续页框，必然导致在已分配页框的内存块中分散了许多小块的 空闲页框。这样，即使这些页框是空闲的，其他需要分配连续页框的应用也很难得到满足。
　　为了避免出现这种情况，Linux内核中引入了伙伴系统算法(buddy system)。把所有的空闲页框分组为11个 块链表，每个块链表分别包含大小为1，2，4，8，16，32，64，128，256，512和1024个连续页框的页框块。最大可以申请1024个连 续页框，对应4MB大小的连续内存。每个页框块的第一个页框的物理地址是该块大小的整数倍。
　　假设要申请一个256个页框的块，先从256个页框的链表中查找空闲块，如果没有，就去512个 页框的链表中找，找到了则将页框块分为2个256个 页框的块，一个分配给应用，另外一个移到256个页框的链表中。如果512个页框的链表中仍没有空闲块，继续向1024个页 框的链表查找，如果仍然没有，则返回错误。
　　页框块在释放时，会主动将两个连续的页框块合并为一个较大的页框块。
　　1.2.slab分配器
　　slab分配器源于 Solaris 2.4 的 分配算法，工作于物理内存页框分配器之上，管理特定大小对象的缓存，进行快速而高效的内存分配。
　　slab分配器为每种使用的内核对象建立单独的缓冲区。Linux 内核已经采用了伙伴系统管理物理内存页框，因此 slab分配器直接工作于伙伴系 统之上。每种缓冲区由多个 slab 组成，每个 slab就是一组连续的物理内存页框，被划分成了固定数目的对象。根据对象大小的不同，缺省情况下一个 slab 最多可以由 1024个页框构成。出于对齐 等其它方面的要求，slab 中分配给对象的内存可能大于用户要求的对象实际大小，这会造成一定的 内存浪费。
　　2.常用内存分配函数
　　2.1.__get_free_pages
　　unsigned long __get_free_pages(gfp_t gfp_mask, unsigned int order)
　　__get_free_pages函数是最原始的内存分配方式，直接从伙伴系统中获取原始页框，返回值为第一个页框的起始地址。__get_free_pages在实现上只是封装了alloc_pages函 数，从代码分析，alloc_pages函数会分配长度为1<
　　2.2.kmem_cache_alloc
　　struct kmem_cache *kmem_cache_create(const char *name, size_t size,
　　size_t align, unsigned long flags,
　　void (*ctor)(void*, struct kmem_cache *, unsigned long),
　　void (*dtor)(void*, struct kmem_cache *, unsigned long))
　　void *kmem_cache_alloc(struct kmem_cache *c, gfp_t flags)
　　kmem_cache_create/ kmem_cache_alloc是基于slab分配器的一种内存分配方式，适用于反复分配释放同一大小内存块的场合。首先用kmem_cache_create创建一个高速缓存区域，然后用kmem_cache_alloc从 该高速缓存区域中获取新的内存块。kmem_cache_alloc一次能分配的最大内存由mm/slab.c文件中的MAX_OBJ_ORDER宏定义，在默认的2.6.18内核版本中，该宏定义为5， 于是一次最多能申请1<<5 * 4KB也就是128KB的 连续物理内存。分析内核源码发现，kmem_cache_create函数的size参数大于128KB时会调用BUG()。测试结果验证了分析结果，用kmem_cache_create分 配超过128KB的内存时使内核崩溃。
　　2.3.kmalloc
　　void *kmalloc(size_t size, gfp_t flags)
　　kmalloc是内核中最常用的一种内存分配方式，它通过调用kmem_cache_alloc函数来实现。kmalloc一次最多能申请的内存大小由include/linux/Kmalloc_size.h的 内容来决定，在默认的2.6.18内核版本中，kmalloc一 次最多能申请大小为131702B也就是128KB字 节的连续物理内存。测试结果表明，如果试图用kmalloc函数分配大于128KB的内存，编译不能通过。
　　2.4.vmalloc
　　void *vmalloc(unsigned long size)
　　前面几种内存分配方式都是物理连续的，能保证较低的平均访问时间。但是在某些场合中，对内存区的请求不是很频繁，较高的内存访问时间也 可以接受，这是就可以分配一段线性连续，物理不连续的地址，带来的好处是一次可以分配较大块的内存。图3-1表 示的是vmalloc分配的内存使用的地址范围。vmalloc对 一次能分配的内存大小没有明确限制。出于性能考虑，应谨慎使用vmalloc函数。在测试过程中， 最大能一次分配1GB的空间。
　　Linux内核部分内存分布
　　2.5.dma_alloc_coherent
　　void *dma_alloc_coherent(struct device *dev, size_t size,
　　ma_addr_t *dma_handle, gfp_t gfp)
　　DMA是一种硬件机制，允许外围设备和主存之间直接传输IO数据，而不需要CPU的参与，使用DMA机制能大幅提高与设备通信的 吞吐量。DMA操作中，涉及到CPU高速缓 存和对应的内存数据一致性的问题，必须保证两者的数据一致，在x86_64体系结构中，硬件已经很 好的解决了这个问题，dma_alloc_coherent和__get_free_pages函数实现差别不大，前者实际是调用__alloc_pages函 数来分配内存，因此一次分配内存的大小限制和后者一样。__get_free_pages分配的内 存同样可以用于DMA操作。测试结果证明，dma_alloc_coherent函 数一次能分配的最大内存也为4M。
　　2.6.ioremap
　　void * ioremap (unsigned long offset, unsigned long size)
　　ioremap是一种更直接的内存“分配”方式，使用时直接指定物理起始地址和需要分配内存的大小，然后将该段 物理地址映射到内核地址空间。ioremap用到的物理地址空间都是事先确定的，和上面的几种内存 分配方式并不太一样，并不是分配一段新的物理内存。ioremap多用于设备驱动，可以让CPU直接访问外部设备的IO空间。ioremap能映射的内存由原有的物理内存空间决定，所以没有进行测试。
 
　　2.7.Boot Memory
　　如果要分配大量的连续物理内存，上述的分配函数都不能满足，就只能用比较特殊的方式，在Linux内 核引导阶段来预留部分内存。
　　2.7.1.在内核引导时分配内存
　　void* alloc_bootmem(unsigned long size)
　　可以在Linux内核引导过程中绕过伙伴系统来分配大块内存。使用方法是在Linux内核引导时，调用mem_init函数之前 用alloc_bootmem函数申请指定大小的内存。如果需要在其他地方调用这块内存，可以将alloc_bootmem返回的内存首地址通过EXPORT_SYMBOL导 出，然后就可以使用这块内存了。这种内存分配方式的缺点是，申请内存的代码必须在链接到内核中的代码里才能使用，因此必须重新编译内核，而且内存管理系统 看不到这部分内存，需要用户自行管理。测试结果表明，重新编译内核后重启，能够访问引导时分配的内存块。
　　2.7.2.通过内核引导参数预留顶部内存
　　在Linux内核引导时，传入参数“mem=size”保留顶部的内存区间。比如系统有256MB内 存，参数“mem=248M”会预留顶部的8MB内存，进入系统后可以调用ioremap(0xF800000，0x800000)来申请这段内存。
　　3.几种分配函数的比较
　　分配原理最大内存其他
　　__get_free_pages直接对页框进行操作4MB适用于分配较大量的连续物理内存
　　kmem_cache_alloc基于slab机制实现128KB适合需要频繁申请释放相同大小内存块时使用
　　kmalloc基于kmem_cache_alloc实现128KB最常见的分配方式，需要小于页框大小的内存时可以使用
　　vmalloc建立非连续物理内存到虚拟地址的映射物理不连续，适合需要大内存，但是对地址连续性没有要求的场合
　　dma_alloc_coherent基于__alloc_pages实现4MB适用于DMA操 作
　　ioremap实现已知物理地址到虚拟地址的映射适用于物理地址已知的场合，如设备驱动
　　alloc_bootmem在启动kernel时，预留一段内存，内核看不见小于物理内存大小，内存管理要求较高


#<font color="red">性能压力下的优化实践</font>
做benchmark测试的过程中，总是会涉及到linux操作系统底层的设置导致无法充分利用机器的性能，在调试的过程中，不少资料没能和linux kernel版本对应上导致一些参数的设置错误。根据现有服务器的硬件条件和软件版本做相关优化，把一些实践的心得分享出来。

Kernel version ： 2.6.32-71.el6.x86_64

Cpu：Intel(R) Xeon(R) CPU E5606 @ 2.13GHz

Memory：8G

Release notes ： v0.1 2012-03-31 句柄数 ， 网络参数

问题1：句柄数的问题

使用webbench在Linux下做varlish访问压力测试的时候，遇到Socket/File: Can’t open so many files的问题，原因是linux下所有的东西都是文件，包括socket接口，对于大量的网络连接，不仅仅消耗socket文件描述符，对于进程本身还打开相当多的文件，Linux的默认句柄数是1024。

解决方式：

修改/etc/security/limits.conf
你的用户名 soft nofile 65535 ##ulimit -Sn
你的用户名 hard nofile 65535 ##ulimit -Hn
unlimt -n 查看
参考资料：

通过ulimit改善系统性能
问题2：网络参数

使用ab或者webbench做压力测试，如果并发数开到1000的时候，无法完成测试。到晚上查看资料发现是linux网络参数设置。

解决方式：

[longhao@longhao etc]vi /etc/sysctl.conf
在kernel2.6之前的添加项：
net.ipv4.netfilter.ip_conntrack_max = 655360
net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 180
 

kernel2.6之后的添加项：
net.nf_conntrack_max = 655360 net.nf_conntrack_max = 655360 也可以
net.netfilter.nf_conntrack_tcp_timeout_established = 1200

[longhao@longhao etc]sysctl -p /etc/sysctl.conf

如果报错：error: "net.nf_conntrack_max" is an unknown key 则需要使用modprobe载入ip_conntrack模块，lsmod查看模块已载入。
[longhao@longhao etc]modprobe ip_conntrack

 

后续说明：
–CONNTRACK_MAX 允许的最大跟踪连接条目，是在内核内存中netfilter可以同时处理的“任务”（连接跟踪条目）
–HASHSIZE 存储跟踪连接条目列表的哈西表的大小

CONNTRACK_MAX和HASHSIZE的默认值
一般来说，CONNTRACK_MAX和HASHSIZE都会设置在“合理”使用的值上，依据可使用的RAM的大小来计算这个值。

CONNTRACK_MAX的默认值
在i386架构上，CONNTRACK_MAX = RAMSIZE (以bytes记) / 16384 =RAMSIZE (以MegaBytes记) * 64，因此，一个32位的带512M内存的PC在默认情况下能够处理512*1024^2/16384 = 512*64 = 32768个并发的netfilter连接。
但是真正的公式是：CONNTRACK_MAX = RAMSIZE (in bytes) / 16384 / (x / 32) 这里x是指针的bit数，（例如，32或者64bit）

请注意：
－默认的CONNTRACK_MAX值不会低于128
－对于带有超过1G内存的系统，CONNTRACK_MAX的默认值会被限制在65536（但是可以手工设置成更大的值）

HASHSIZE的默认值
通常，CONNTRACK_MAX = HASHSIZE * 8。这意味着每个链接的列表平均包含8个conntrack的条目（在优化的情况并且CONNTRACK_MAX达到的情况下），每个链接的列表就是一个哈西表条目（一个桶）。
在i386架构上，HASHSIZE = CONNTRACK_MAX / 8 =RAMSIZE (以bytes记) / 131072 = RAMSIZE (以MegaBytes记) * 8。举例来说，一个32位、带512M内存的PC可以存储512*1024^2/128/1024 =512*8 = 4096 个桶（链接表）
但是真正的公式是：HASHSIZE = CONNTRACK_MAX / 8 = RAMSIZE (以bytes记) / 131072 / (x / 32)这里x是指针的bit数，（例如，32或者64bit）

请注意：
－默认HASHSIZE的值不会小于16
－对于带有超过1G内存的系统，HASHSIZE的默认值会被限制在8192（但是可以手工设置成更大的值）


<font color="red" size='18'>#Sysctl命令及linux内核参数调整</font>

Sysctl命令及linux内核参数调整
 
一、Sysctl命令用来配置与显示在/proc/sys目录中的内核参数．如果想使参数长期保存，可以通过编辑/etc/sysctl.conf文件来实现。
 
 命令格式：
 sysctl [-n] [-e] -w variable=value
 sysctl [-n] [-e] -p (default /etc/sysctl.conf)
 sysctl [-n] [-e] –a
 
常用参数的意义：
 -w  临时改变某个指定参数的值，如
        sysctl -w net.ipv4.ip_forward=1
 -a  显示所有的系统参数
 -p从指定的文件加载系统参数,默认从/etc/sysctl.conf 文件中加载，如：
echo 1 > /proc/sys/net/ipv4/ip_forward
sysctl -w net.ipv4.ip_forward=1
 以上两种方法都可能立即开启路由功能，但如果系统重启，或执行了
     service network restart
命令，所设置的值即会丢失，如果想永久保留配置，可以修改/etc/sysctl.conf文件，将 net.ipv4.ip_forward=0改为net.ipv4.ip_forward=1

 
二、linux内核参数调整：linux 内核参数调整有两种方式
 
方法一：修改/proc下内核参数文件内容，不能使用编辑器来修改内核参数文件，理由是由于内核随时可能更改这些文件中的任意一个，另外，这些内核参数文件都是虚拟文件，实际中不存在，因此不能使用编辑器进行编辑，而是使用echo命令，然后从命令行将输出重定向至 /proc 下所选定的文件中。如：将 timeout_timewait 参数设置为30秒：
echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout
参数修改后立即生效，但是重启系统后，该参数又恢复成默认值。因此，想永久更改内核参数，需要修改/etc/sysctl.conf文件
 
   方法二．修改/etc/sysctl.conf文件。检查sysctl.conf文件，如果已经包含需要修改的参数，则修改该参数的值，如果没有需要修改的参数，在sysctl.conf文件中添加参数。如：
   net.ipv4.tcp_fin_timeout=30
保存退出后，可以重启机器使参数生效，如果想使参数马上生效，也可以执行如下命令：
   sysctl  -p
 
三、sysctl.conf 文件中参数设置及说明
proc/sys/net/core/wmem_max
最大socket写buffer,可参考的优化值:873200
 
/proc/sys/net/core/rmem_max 
最大socket读buffer,可参考的优化值:873200
/proc/sys/net/ipv4/tcp_wmem 
TCP写buffer,可参考的优化值: 8192 436600 873200
 
/proc/sys/net/ipv4/tcp_rmem 
TCP读buffer,可参考的优化值: 32768 436600 873200
 
/proc/sys/net/ipv4/tcp_mem 
同样有3个值,意思是: 
net.ipv4.tcp_mem[0]:低于此值,TCP没有内存压力. 
net.ipv4.tcp_mem[1]:在此值下,进入内存压力阶段. 
net.ipv4.tcp_mem[2]:高于此值,TCP拒绝分配socket. 
上述内存单位是页,而不是字节.可参考的优化值是:786432 1048576 1572864
 
/proc/sys/net/core/netdev_max_backlog 
进入包的最大设备队列.默认是300,对重负载服务器而言,该值太低,可调整到1000
 
/proc/sys/net/core/somaxconn 
listen()的默认参数,挂起请求的最大数量.默认是128.对繁忙的服务器,增加该值有助于网络性能.可调整到256.
 
/proc/sys/net/core/optmem_max 
socket buffer的最大初始化值,默认10K
 
/proc/sys/net/ipv4/tcp_max_syn_backlog 
进入SYN包的最大请求队列.默认1024.对重负载服务器,可调整到2048
 
/proc/sys/net/ipv4/tcp_retries2 
TCP失败重传次数,默认值15,意味着重传15次才彻底放弃.可减少到5,尽早释放内核资源.
 
/proc/sys/net/ipv4/tcp_keepalive_time 
/proc/sys/net/ipv4/tcp_keepalive_intvl 
/proc/sys/net/ipv4/tcp_keepalive_probes 
这3个参数与TCP KeepAlive有关.默认值是: 
tcp_keepalive_time = 7200 seconds (2 hours) 
tcp_keepalive_probes = 9 
tcp_keepalive_intvl = 75 seconds 
意思是如果某个TCP连接在idle 2个小时后,内核才发起probe.如果probe 9次(每次75秒)不成功,内核才彻底放弃,认为该连接已失效.对服务器而言,显然上述值太大. 可调整到: 
/proc/sys/net/ipv4/tcp_keepalive_time 1800 
/proc/sys/net/ipv4/tcp_keepalive_intvl 30 
/proc/sys/net/ipv4/tcp_keepalive_probes 3
 
/proc/sys/net/ipv4/ip_local_port_range 
指定端口范围的一个配置,默认是32768 61000,已够大.
net.ipv4.tcp_syncookies = 1 
表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

net.ipv4.tcp_tw_reuse = 1 
表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；

net.ipv4.tcp_tw_recycle = 1 
表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

net.ipv4.tcp_fin_timeout = 30 
表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。

net.ipv4.tcp_keepalive_time = 1200 
表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。

net.ipv4.ip_local_port_range = 1024 65000 
表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。

net.ipv4.tcp_max_syn_backlog = 8192 
表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。

net.ipv4.tcp_max_tw_buckets = 5000 
表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。默认为 180000，改为 5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。




Linux上的NAT与iptables
谈起Linux上的NAT，大多数人会跟你提到iptables。原因是因为iptables是目前在linux上实现NAT的一个非常好的接口。它通过和内核级直接操作网络包，效率和稳定性都非常高。这里简单列举一些NAT相关的iptables实例命令，可能对于大多数实现有多帮助。
 这里说明一下，为了节省篇幅，这里把准备工作的命令略去了，仅仅列出核心步骤命令，所以如果你单单执行这些没有实现功能的话，很可能由于准备工作没有做好。如果你对整个命令细节感兴趣的话，可以直接访问我的《如何让你的Linux网关更强大》系列文章，其中对于各个脚本有详细的说明和描述。
案例1：实现网关的MASQUERADE
具体功能：内网网卡是eth1，外网eth0，使得内网指定本服务做网关可以访问外网

EXTERNAL="eth0"
INTERNAL="eth1"

这一步开启ip转发支持，这是NAT实现的前提
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -o $EXTERNAL -j MASQUERADE
案例2：实现网关的简单端口映射
具体功能：实现外网通过访问网关的外部ip:80，可以直接达到访问私有网络内的一台主机192.168.1.10:80效果

LOCAL_EX_IP=11.22.33.44 #设定网关的外网卡ip，对于多ip情况，参考《如何让你的Linux网关更强大》系列文章
LOCAL_IN_IP=192.168.1.1  #设定网关的内网卡ip
INTERNAL="eth1" #设定内网卡

这一步开启ip转发支持，这是NAT实现的前提
echo 1 > /proc/sys/net/ipv4/ip_forward

加载需要的ip模块，下面两个是ftp相关的模块，如果有其他特殊需求，也需要加进来
modprobe ip_conntrack_ftp
modprobe ip_nat_ftp

这一步实现目标地址指向网关外部ip:80的访问都吧目标地址改成192.168.1.10:80
iptables -t nat -A PREROUTING -d $LOCAL_EX_IP -p tcp --dport 80 -j DNAT --to 192.168.1.10

这一步实现把目标地址指向192.168.1.10:80的数据包的源地址改成网关自己的本地ip，这里是192.168.1.1
iptables -t nat -A POSTROUTING -d 192.168.1.10 -p tcp --dport 80 -j SNAT --to $LOCAL_IN_IP

在FORWARD链上添加到192.168.1.10:80的允许，否则不能实现转发
iptables -A FORWARD -o $INTERNAL -d 192.168.1.10 -p tcp --dport 80 -j ACCEPT

通过上面重要的三句话之后，实现的效果是，通过网关的外网ip:80访问，全部转发到内网的192.168.1.10:80端口，实现典型的端口映射
特别注意，所有被转发过的数据都是源地址是网关内网ip的数据包，所以192.168.1.10上看到的所有访问都好像是网关发过来的一样，而看不到外部ip
一个重要的思想：数据包根据“从哪里来，回哪里去”的策略来走，所以不必担心回头数据的问题

现在还有一个问题，网关自己访问自己的外网ip:80，是不会被NAT到192.168.1.10的，这不是一个严重的问题，但让人很不爽，解决的方法如下：
iptables -t nat -A OUTPUT -d $LOCAL_EX_IP -p tcp --dport 80 -j DNAT --to 192.168.1.10
获取系统中的NAT信息和诊断错误
了解/proc目录的意义
在Linux系统中，/proc是一个特殊的目录，proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它包含当前系统的一些参数（variables）和状态（status）情况。它以文件系统的方式为访问系统内核数据的操作提供接口
通过/proc可以了解到系统当前的一些重要信息，包括磁盘使用情况，内存使用状况，硬件信息，网络使用情况等等，很多系统监控工具（如HotSaNIC）都通过/proc目录获取系统数据。
另一方面通过直接操作/proc中的参数可以实现系统内核参数的调节，比如是否允许ip转发，syn-cookie是否打开，tcp超时时间等。
获得参数的方式：
第一种：cat /proc/xxx/xxx，如 cat /proc/sys/net/ipv4/conf/all/rp_filter
第二种：sysctl xxx.xxx.xxx，如 sysctl net.ipv4.conf.all.rp_filter
改变参数的方式：
第一种：echo value > /proc/xxx/xxx，如 echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter
第二种：sysctl [-w] variable=value，如 sysctl [-w] net.ipv4.conf.all.rp_filter=1
以上设定系统参数的方式只对当前系统有效，重起系统就没了，想要保存下来，需要写入/etc/sysctl.conf文件中
通过执行 man 5 proc可以获得一些关于proc目录的介绍
查看系统中的NAT情况
和NAT相关的系统变量
/proc/slabinfo：内核缓存使用情况统计信息（Kernel slab allocator statistics）
/proc/sys/net/ipv4/ip_conntrack_max：系统支持的最大ipv4连接数，默认65536（事实上这也是理论最大值）
/proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_established 已建立的tcp连接的超时时间，默认432000，也就是5天
和NAT相关的状态值
/proc/net/ip_conntrack：当前的前被跟踪的连接状况，nat翻译表就在这里体现（对于一个网关为主要功能的Linux主机，里面大部分信息是NAT翻译表）
/proc/sys/net/ipv4/ip_local_port_range：本地开放端口范围，这个范围同样会间接限制NAT表规模
1. 查看当前系统支持的最大连接数
cat /proc/sys/net/ipv4/ip_conntrack_max 
值：默认65536，同时这个值和你的内存大小有关，如果内存128M，这个值最大8192，1G以上内存这个值都是默认65536
影响：这个值决定了你作为NAT网关的工作能力上限，所有局域网内通过这台网关对外的连接都将占用一个连接，如果这个值太低，将会影响吞吐量

2. 查看tcp连接超时时间
cat /proc/sys/net/ipv4/netfilter/ip_conntrack_tcp_timeout_established 
值：默认432000（秒），也就是5天
影响：这个值过大将导致一些可能已经不用的连接常驻于内存中，占用大量链接资源，从而可能导致NAT ip_conntrack: table full的问题
建议：对于NAT负载相对本机的 NAT表大小很紧张的时候，可能需要考虑缩小这个值，以尽早清除连接，保证有可用的连接资源；如果不紧张，不必修改

3. 查看NAT表使用情况（判断NAT表资源是否紧张）
执行下面的命令可以查看你的网关中NAT表情况
cat /proc/net/ip_conntrack

4. 查看本地开放端口的范围
cat /proc/sys/net/ipv4/ip_local_port_range
返回两个值，最小值和最大值

下面的命令帮你明确一下NAT表的规模
wc -l /proc/net/ip_conntrack
或者
grep ip_conntrack /proc/slabinfo | grep -v expect | awk '{print $1 ',' $2;}'

下面的命令帮你明确可用的NAT表项，如果这个值比较大，那就说明NAT表资源不紧张
grep ip_conntrack /proc/slabinfo | grep -v expect | awk '{print $1 ',' $3;}'

下面的命令帮你统计NAT表中占用端口最多的几个ip，很有可能这些家伙再做一些bt的事情，嗯bt的事情:-)
cat /proc/net/ip_conntrack | cut -d ' ' -f 10 | cut -d '=' -f 2 | sort | uniq -c | sort -nr | head -n 10
上面这个命令有点瑕疵cut -d' ' -f10会因为命令输出有些行缺项而造成统计偏差，下面给出一个正确的写法：
cat /proc/net/ip_conntrack | perl -pe s/^\(.*?\)src/src/g | cut -d ' ' -f1 | cut -d '=' -f2 | sort | uniq -c | sort -nr | head -n 10

<font color="red" size='18'>#Linux内核编译安装</font>  

centos6.6 编译升级内核优化网络、添加ceph模块20150907
简介：
        随着业务量的增长，业务服务器网络压力不断增大，查看后端服务器网络连接状态，发现TIMEWAIT状态连接巨多，TIMEWAIT 占用大量的连接端口不释放，影响业务服务响应速度。同时大量的每个TCP连接都各自有个数据结构，叫TCP Control Block.Time_wait的时候这个数据结构没有被释放。所以当有太多的TCP连接时，内存可能会被占用很多。业务需要服务器需要在centos6版本系统上挂在rbd 分区使用ceph分布式存储（这个是顺带解决问题）.
1：需要升级内核主机
84  C0014.5 dell R41    2.6.32-358.el6.x86_64   stream-3-39.ptfuture.com    16核
intel xeon e5520 2.40g
5520/4核/8线程*4   64G 
8g*8    300g （raid 1）
500gsas15000转*2
500gsas7200转*2  172.17.3.39
85  C0010.5 dell R410
16482837885
7KLGF3X 
2.6.32-504.16.2.el6.x86_64
stream-3-60.ptfuture.com    16核
intel xeon e5520 2.40g
5520/4核/8线程*2   64G 
8g*8    1T （raid 1）
1T sas7200转*2
172.17.3.60
86  C0012.5 dell R410
8311459389
3TGFF3X     
2.6.32-504.16.2.el6.x86_64
stream-3-61.ptfuture.com    16核
intel xeon e5520 2.40g
5520/4核/8线程*2   64G 
8g*8    1T （raid 1）
1T sas7200转*2 
1.1：未升级内核之前 modprobe -l|grep ceph #系统内核没有ceph模块
提前踩坑：
安装系统所需要的编译工具：
yum install wget gcc gc bc gd make perl ncurses-devel xz -y 
如果执行上面的安装命令后，在编译过程中提示缺少依赖软件包请执行下面的软件安装命令
yum  install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng  \
libpng-devel freetype freetype-devel libxml2 libxml2-devel \
zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 \
bzip2-devel zip unzip ncurses ncurses-devel curl curl-devel \
e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel \
openssl openssh openssl-devel nss_ldap openldap openldap-devel \
openldap-clients openldap-servers libxslt-devel libevent-devel \
ntp  libtool-ltdl bison libtool vim-enhanced python wget lsof \
iptraf strace lrzsz kernel-devel kernel-headers pam-devel Tcl/Tk  \
cmake  ncurses-devel bison setuptool popt-devel rsynx openssh \
system-config-network-tui gcc gc bc gd make perl ncurses-devel xz -y 
3：升级前查看现系统运行内核版本：
uname -a
Linux stream-3-39.ptfuture.com 2.6.32-573.3.1.el6.x86_64 
4：下载内核文件并解压
cd /usr/src  
wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.38.8.tar.gz
tar -xvf linux-2.6.38.8.tar.gz
cd linux-2.6.38.8 
其它2.6版本内核文件下载地址
cd /usr/src  
wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.39.4.tar.gz
tar -xvf linux-2.6.39.4.tar.gz 
cd linux-2.6.39.4


cd /usr/src  
wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.38.tar.gz
tar -xvf linux-2.6.38.tar.gz
cd linux-2.6.38

cd /usr/src  
wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.36.4.tar.gz
tar -xvf linux-2.6.36.4.tar.gz
cd linux-2.6.36.4 
5：修改 tcp.h 内核文件 （本步骤主要为解决tcp TIME_WAIT过多问题，按需配置）
vim ./include/net/tcp.h  
        内核中一个宏定义，在 $KERNEL/include/net/tcp.h里面，这个宏是真正控制 TCP TIME_WAIT状态的超时时间的。内容如下：
#define TCP_TIMEWAIT_LEN (60*HZ) /* how long to wait to destroy TIME-WAIT
                                      /* state, about 60 seconds 
        修改这个宏定义数值设置，根据我们的测试，常用的值有以下三种：30秒，1分钟，2分钟，可以根据业务实际情况进行实测，我们这边网络压力比较大，通过压测最终确定设置为 10 秒，也就是把上面的修改为：
#define TCP_TIMEWAIT_LEN (10*HZ) /* how long to wait to destroy TIME-WAIT
                                     /* state, about 60 seconds 
copy系统config文件
cp -rp  /boot/config-* /usr/src/kernels/ 
为了方便编译配置，将/boot下的配置文件复制到当前目录下的/usr/src/kernels/文件中
以下是内核进行多次编译后，再次编译需要的操作,如果是第一次编译，可直接跳到步骤6.4
6：编译准备：
6.1:清除环境变量（清除配置文件）确保源代码目录下没有不正确的.o文件和文件依赖关系，执行该命令后，内核选项会回到默认的状态下。(新下载、首次编译内核可以省略)
make mrproper 
6.2:读取配置过程生成的配置文件，来创建对应于配置的依赖关系树，从而决定哪些需要编译而那些不需要。
make dep 
6.3:确保所有东西均保持最新状态(新下载、首次编译内核可以省略)
make clean 
只想在原来内核配置的基础上修改一些小地方,可以考虑此选项
make oldconfig 
如果只想在原来内核配置的基础上修改一些小地方，会省去不少麻烦;
sh -c ‘yes’ "" | make oldconfig 
        自动确定内核选项询问会读取当前目录下的.config文件，在.config文件里没有找到的选项则提示用户填写，然后备份.config文件为.config.old，并生成新的.config文件
6.4:基于文本选单的配置界面，字符终端下推荐使用
make menuconfig

ceph模块路径： （本步骤主要为加载ceph内核模块、按需配置）
1：Device Drivers  ---> [*] Block devices   --->  
   <M>   Rados block device (RBD)
2：-*- Networking support  --->   
   {M}   Ceph core library (EXPERIMENTAL) (NEW) 
3：File systems  --->  [*] Network File Systems  --->   
   <M> Ceph distributed file system (EXPERIMENTAL) 
6.5:生成内核文件 -j后面的数字是线程数，用于加快编译速度，一般的经验是，逻辑CPU，就填写那个数字，例如有8核，则为-j8。
make -j16 bzImage 
6.6:编译模块（注：此处需要很长时间，请耐心等待）
make -j16 modules
6.7: 安装模块
make -j16 modules_install 
6.8:#安装kernel
make install 
报错:一个模块找不到、边角模块不影响使用，略过
System.map "/boot"
ERROR: modinfo: could not find module lpc_ich 
        精密时间协议PTP也将出现在Linux 7里。相对于NTP，PTP适合高频率交易场景，因为它可确保对跨网络分布式时钟精确到亚微秒的同步。PTP已作为技术预览版在RHEL 6.4里出现，并能完全支持6.5.
http://kapok.blog.51cto.com/517862/128805
ERROR: modinfo: could not find module ptp 
如果以上步骤都顺利执行完成，那么恭喜你内核升级已基本完成。
6：修改系统启动菜单并重启服务器，使服务器在下次启动使用新的内核。
[root@stream-3-61 linux-2.6.38.8]# vi /boot/grub/grub.conf 
```
default=0                #编译完成是1，修改为0，启动新编译内核
timeout=5
splashimage=(hd0,0)/grub/splash.xpm.gz
hiddenmenu
title CentOS (2.6.38.8)
        root (hd0,0)
        kernel /vmlinuz-2.6.38.8 ro root=UUID=2cb5fc14-c9ef-4244-
        b99b-95bc0fca7bdf nomodeset rd_NO_LUKS  KEYBOARDTYPE=pc 
        KEYTABLE=us LANG=en_US.UTF-8 rd_NO_MD 
        SYSFONT=latarcyrheb-sun16 crashkernel=auto rd_NO_LV
M rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.38.8.img
title CentOS (2.6.32-573.3.1.el6.x86_64)
        root (hd0,0)
        kernel /vmlinuz-2.6.32-573.3.1.el6.x86_64 ro 
        root=UUID=2cb5fc14-c9ef-4244-b99b-95bc0fca7bdf 
        nomodeset rd_NO_LUKS  KEYBOARDTYPE=pc KEYTABLE=us 
        LANG=en_US.UTF-8 rd_NO_MD SYSFONT=latarcyrheb-sun16 crashker
nel=auto rd_NO_LVM rd_NO_DM rhgb quiet
        initrd /initramfs-2.6.32-573.3.1.el6.x86_64.img
title CentOS (2.6.32-504.16.2.el6.x86_64)
        root (hd0,0)
```
6.1：重启系统是配置生效
init 6 
8：验证：
内核版本验证：
[root@stream-3-61 ~]# uname -a
Linux stream-3-61.ptfuture.com 2.6.38.8 #1 SMP Mon 
Aug 31 21:05:10 JST 2015 x86_64 x86_64 x86_64 GNU/Linux

ceph模块验证：
[root@stream-3-61 ~]# modprobe -l|grep ceph 
kernel/fs/ceph/ceph.ko
kernel/net/ceph/libceph.ko
