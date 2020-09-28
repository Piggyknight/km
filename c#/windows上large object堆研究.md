

### 一. 大内存块原理概述

大于等于85000bytes的被认为是large object. 大物体在分配时会分配到大物体专属的堆上. windows上.net的垃圾回收使用的多代回收机制(generation collector), 总共使用3代, 0代,1代, 2代. 每一代收集回收时, 都会将所有小的代回收. 因此2代回收, 也称之为全回收(full GC).小内存都分配在gen0中, 然后可能一直存活不被GC回收, 并提升至gen1, 2, 但大物体永远属于gen2. 

3代的设计原因:

- 大多数小内存在gen0时就销毁了, 部分比如动态分配的内存, 会持续到gen1时销毁, gen1作为快速销毁与长期存在数据间的一个缓存, 

- 大物体被认为是长期存在的, 因此永远在gen2中分配和存在. 



当GC被触发时, 为了避免内存碎片会进行压缩(Compact), 对于大物体来说内存移动代价太高, 因此.net使用一个空闲指针表来指向被释放的内存, 相邻的会进行合并. 在.net 4.5.1后, 通过GCSettings.LargetObjectHeapCompactionMode可以控制如何进行压缩的策略.

所有这几代的内存都是靠调用win32 api中的VirtualAlloc接口进行分配的. 在CLR加载完毕后, 会事先生成两大堆:

- 小堆(Small Object Heap): 小于85000的从这个堆分配, 小堆上分配的内存存活与一个GC的话, 会进入后面一代(gen0 -> gen1), 超过2代仍然是2代
- 大堆(Large Object Heap): 大于85000的从这个堆分配



SOH简易gc过程图示

![image-20200925131017188](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/image-20200925131017188.png)



LOH的GC简易流程

![image-20200925132532891](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/image-20200925132532891.png)





Gen2的GC时, 如果内存不够会尝试向系统os申请更多内存段(segment), 如果申请失败才会触发GC进行清理

清理中, 如果有一段segment全空了, 会调用VirtualFree函数交回给系统, 其次会产生三种内存状态

	- 占用(occupied)
	- 空且已经置空(free, committed, reset)
	- 潜在可使用(decommitted space)



![image-20200925133008331](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/image-20200925133008331.png)



分配大块内存的开销影响

- CLR保障了内存分配出来一定是干净的, 应次分配的内存越大, 将会导致clean的开销越大, 2 cycle清理一个byte, 170000 cycles 清理85000bytes,  16MB的内存 在2GHZ cpu上需要近16ms

- GC2触发时, 由于LOH与Gen2在一起, 会导致不一定会能勾释放太多空间, gc2上的内容不多时, 问题不大, 当大内存块分配随机, 同时存在较多小物体内存是时, 则会导致gen2开销过大

- 大块内存往往都是数组, 且很多结构都包含较多的引用(reference-rich), 这时garbage collector不得不遍历所有引用导致更大开销, 因此写代码时应避免使用较引用的方法, 举例如下

  ```c#
  class Node
  {
     Data d;
     Node left;
     Node right;
  };
  
  Node[] binary_tr = new Node [num_nodes];
  ```

如上代码, 一个结构内包含两个引用,  当num_nodes数量较大时会导致开销较大, 可以进行如下优化

```c#
class Node
{
   Data d;
   uint left_index;
   uint right_index;
} ;
```



后续进行测试, 两种方法导致的GC开销区别.



### 二. 监测GC的工具

目前有三种办法来监测LOH的性能, 官方推荐ETW events

- .Net CLR Memory performance counters
- ETW events
- Sos Debugger



#### 1. .Net CLR Memory performance counters

在windows上通过使用Performance Monitor(perfmon.exe)来查看counters. 

![image-20200926133425512](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/image-20200926133425512.png)

如图, 通过配置可以添加相应的counter

	- Gen 1 heap size
	- Gen 1 Provisioned Bytes
	- Gen 2 heap size
	- Large Object heap size

等等, 由于可以导出数据, 经常被用来作为自动监测, 如果数值异常则报警.



#### 2. ETW events

ETW全称Event Tracing for Windows, 是一个kernel层级的事件追踪机制. 允许用户调试时跟踪系统或者应用定义的事件, 并生成相应的log文件. 查看log文件需要用到[PerfView](https://devblogs.microsoft.com/dotnet/improving-your-apps-performance-with-perfview/) 的命令行机制.

> ```console
> perfview /GCCollectOnly /AcceptEULA /nogui collect
> ```

结果如下,

![image-20200926134557644](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/image-20200926134557644.png)

Trigger Reason可以知道分配的原因, 最后倒数低4栏LOH Survival Rate表示存货率, 越低表示都是临时内存, 用完就扔的.



其次通过如下命令来获得谁分配了这些大内存

> perfview /GCOnly /AcceptEULA /nogui collect

会统计AllocationTick事件(每100K触发一次), 结果如下, 可以通过树形结构得知调用栈

![Screenshot that shows a garbage collector heap view.](image/windows%E4%B8%8Alarge%20object%E5%A0%86%E7%A0%94%E7%A9%B6/garbage-collector-heap.png)



#### 3. Sos Debugger











## 参考

1. [large object heap](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/large-object-heap)

2. [event tracing](https://docs.microsoft.com/en-us/windows/win32/etw/about-event-tracing)

3. [sos debugging extension](https://docs.microsoft.com/en-us/dotnet/framework/tools/sos-dll-sos-debugging-extension)

4. [GC-ETW-events-1](https://devblogs.microsoft.com/dotnet/gc-etw-events-1/)
5. [GC-ETW-events-2](https://devblogs.microsoft.com/dotnet/gc-etw-events-2/)
6. [GC-ETW-events-3](https://devblogs.microsoft.com/dotnet/gc-etw-events-3/)
7. [GC-ETW-events-4](https://devblogs.microsoft.com/dotnet/gc-etw-events-4/)



