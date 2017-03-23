# Memory-Management
内存地址
======
内存地址(memory address)：
作为访问内存单元的一种方式，但是，当使用80x86微处理器时，必须区分以下三种不同的地址：

逻辑地址
------
  包含在机器语言指令中用来指定一个操作数或一条指令的地址。这种寻址方式在80x86著名的分段结构中表现得尤为具体。它促使MS-DOS或Windows程序员分成若干段。每一个逻辑地址都是由一个段(segment)和偏移量(offset或displacement)组成，偏移量指明了从段开始的地方到实际地址之间的距离。
  
线性地址(又称虚拟地址)
------
物理地址
-------
用于内存芯片级内存单元寻址。他们与从微处理器的地址引脚出发到内存总线上的电信号相对应。物理地址由32位或36位无符号整数表示。

内存控制单元(MMU)通过一种称为分段单元(segmentation unit)的硬件单路把一个逻辑地址转换成线性地址;接着，第二个称为分页单元（paging unit）的硬件电路把线性地址转换成一个物理地址。
![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/%E5%86%85%E5%AD%98%E6%98%A0%E5%B0%8420170322a.jpg)
linux内核采用页式存储管理。虚拟地址空间划分成固定大小的“页面”，由于MMU在运行时将虚拟地址“映射”成某个物理内存页面中的地址。与段式存储管理相比，页式存储有很多好处。首先，页面都是国定大小的，便于管理。更重要的是，当要将一部分物理空间的内存换出到磁盘上的时候，在页式存储管理中是按页进行，效率显然要高得多。页式存储管理和段式存储管理所要求的硬件支持不同，一种CPU既然支持页式存储管理，就无需再支持段式存储管理。但是，i386的情况是特殊的。由于i386系列的历史演变过程，它对页式存储管理的支持是在其段式存储管理已经存在相当长的时间以后才发展起来的。所以，不管程序是怎样写的，i386CPU一律对程序中使用的地址先进行段式映射，然后才能进行页式映射。

段映射阶段
========

段选择符和段寄存器
---------------
一个逻辑地址由两部分组成：一个段标识符和一个指定段内相对地址的偏移量。段标识符是一个16位长的字段，成为段选择符(Segment Selector),而偏移量是一个32位长的字段。
![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/%E5%86%85%E5%AD%98%E6%98%A0%E5%B0%8420170323a.jpg)
                            段选择符 
