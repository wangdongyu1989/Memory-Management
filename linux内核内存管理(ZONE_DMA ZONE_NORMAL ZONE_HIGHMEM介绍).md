CPU被划分为多个节点(node)，内存则被分簇，每个CPU对应一个本地物理内存，即一个CPU-node对应一个内存簇bank，即每个内存簇被认为一个节点。系统的物理内存被划分几个节点(node)，一个node对应一个内存簇bank，即每个内存簇被认为一个节点。
*  首先，内存被划分为节点，每个节点关联到系统中的一个处理器，内核中表示为pg_data_t的实例。系统中每个节点被链接到一个以NULL结尾的pgdat_list链表中<而其中的每个节点利用pg_data_tnode_next字段链接到下一节点>。而对PC这种UMA结构的机器来说，只使用了一个称为contig_page_data的静态pg_data_t结构。
*  在具有大容量RAM的现在32位计算机中，CPU不能直接访问所有物理地址，因为线性地址空间太小，内核不可能直接映射所有物理内存到线性地址空间。

## x86的物理地址空间布局：

通常32位linux内核地址空间划分0~3G为用户空间，3-4G为内核空间。注意这里是32位内核地址空间划分，64位内核地址空间划分是不同的。

![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/images/x86%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80%E7%A9%BA%E9%97%B4%E5%88%86%E5%B8%83.jpg)


物理地址空间的顶部以下一段空间，被PCI设备的I/O内存映射占据，它们的大小和布局由PCI规范所决定。640k-1M这段地址空间被BIOS和VGA适配器所占据。

linux系统在初始化时，会根据实际的物理内存的大小，为每个物理页面创建一个page对象，所有的page对象构成一个mem_map数组。

针对不同的用途，linux内核将所有的物理页面划分到3类内存管理区中，如上图，分别为ZONE_DMA，ZONE_NORMAL，ZONE_HIGHMEM。

* ZONE_DMA的范围0~16M，该区域的物理页面专门供I/O设备的DMA使用。之所以需要单独管理DMA的物理页面，是因为DMA使用物理地址访问内存，不经过MMU，并且需要持续的缓冲区，所以为了能提供物理上连续的缓冲区，必须从物理地址空间专门划分一段区域用于DMA。

* ZONE_NORMAL的范围是16M~896M，该区域的物理页面是内核能直接使用的。

* ZONE_HIGHMEM的范围是896M~结束，该区域即为高端内存，内核不能直接使用。

## linux虚拟地址内核空间分布：

![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/images/linux%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%E7%94%A8%E6%88%B7%E7%A9%BA%E9%97%B4%E5%88%86%E5%B8%83.jpg)

在kernel image下面的16M的内核空间用于DMA操作。位于内核空间高端的128M地址主要由3部分组成，分别为vmalloc area,持久化内核映射区，临时内核映射区。

由于ZONE_NORMAL和内核线性存在直接映射关系，所以内核会将频繁使用的数据如kernel代码，GDT，IDT，PGD，mem_map数组等放在ZONE_NORMAL里。而将用户数据，页表(PT)等不常用数据放在ZONE_HIGHMEM里，只在要访问这些数据时才建立映射关系（kmap()）。比如，当内核要访问I/O设备存储空间时，就使用ioremap()将位于物理地址高端的mmio区映射到内核空间的vmalloc area中，在使用完之后便断开映射关系。

## linux虚拟地址用户空间分布：

![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/images/linux%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%E7%94%A8%E6%88%B7%E7%A9%BA%E9%97%B4%E5%88%86%E5%B8%83.jpg)

用户进程的代码区一般从虚拟地址空间的0x08048000开始，这是为了便于检查空指针。代码区之上便是数据区，未初始化数据区，堆区，栈区，以及参数，全局环境变量。

## linux虚拟地址与物理地址映射的关系

![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/images/linux%E8%99%9A%E6%8B%9F%E5%9C%B0%E5%9D%80%E4%B8%8E%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80%E6%98%A0%E5%B0%84%E5%85%B3%E7%B3%BB.jpg)

linux将4G的线性地址空间分为2部分，0~3G为user space，3~4G为kernel space。

由于开启了分页机制，内核想要访问物理地址空间的话，必须先建立映射关系，然后通过虚拟地址来访问。为了能够访问所有物理地址空间，就要将全部物理地址空间映射到1G的内核线性空间中，这显然不可能。于是，内核将0~896M的物理地址空间一对一映射到自己线性空间中，这样它便可以随机访问ZONE_DMA和ZONE_NORMAL里的物理页面；此时内核剩下的128M线性地址空间不足以完全映射所有ZONE_HIGHMEM，Linux采用了动态映射的方法，即按需求的将ZONE_HIGHMEM里的物理页面映射到kernel space的最后128M线性地址空间里，使用完之后释放映射关系，以供其他物理页面映射。虽然这样存在效率的问题，但是内核毕竟可以正常的访问所有的物理地址空间了。




