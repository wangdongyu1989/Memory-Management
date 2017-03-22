# Memory-Management
内存地址
======
内存地址(memory address)作为访问内存单元的一种方式，但是，当使用80x86微处理器时，必须区分以下三种不同的地址：
逻辑地址
------
包含在机器语言指令中用来指定一个操作数或一条指令的地址。这种寻址方式在80x86著名的分段结构中表现得尤为具体。它促使MS-DOS或Windows程序员分成若干段。每一个逻辑地址都是由一个段(segment)和偏移量(offset或displacement)组成，偏移量指明了从段开始的地方到实际地址之间的距离。
![image](https://github.com/wangdongyu1989/Memory-Management/blob/master/%E5%86%85%E5%AD%98%E6%98%A0%E5%B0%8420170322a.jpg)
