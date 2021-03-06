[TOC]





<img src="https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219110528955.png" alt="image-20201219110528955" style="zoom:67%;" />

# 垃圾收集器



## 先行概念



上图是7给垃圾收集器，G1是也可以应用新生代，也可以应用老年代，而其他的只能是应用特定的，在以前 除G1的收集器 我们也叫经典收集器



我们需要做的是根据业务采集 ，根据业务需要，根据系统 应用合适的收集器，



**吞吐量：** GC的吞吐量 就是 运行代码和CPU总消耗的时间比，比如说虚拟机总运行了 `100` 分钟，**用户代码** 时间 `99` 分钟，**垃圾回收** 时间 `1` 分钟，那么吞吐量就是 `99%`。



**反馈时间：** 就是GC 回收时，应用程序的暂停时间，对于单线程收集器来说，时间会比较长。对于多线程来说，时间比较少，应用收集器和应用程序交替

> 停顿时间越短就越适合需要与用户交互或需要保证服务响应质量的程序，良好的响应速度能提升用户体验；而高吞吐量则可以最高效率地利用处理器资源，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的分析任务。



**串行：** GC单线程进行工作， 用户线程是在等待状态



**并行：** 描述 的是 同一时刻有多个GC线程在工作



**并发：** 描述的是  GC线程在工作的同时， 用户线程不需要等待，



## 新生代收集器



### serial收集器

![image-20201219112559904](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219112559904.png)

首先这个收集器是单线程的。

只开启一条GC线程来进行垃圾回收。并且在运行时 停止一切用户线程（我们随便创建的一个普通线程就是用户线程，应用进程来调度）（stop the world）

适合所需堆内存和所需内存比较少，如客户端，客户端创建的对象比较少，堆回收时间少，暂停时间没有那么大影响

使用单线程的好处来了，就是避免了线程的上下文切换，

```java
开启选项：-XX:+UseSerialGC

//打开此开关后，使用 Serial + Serial Old 收集器组合来进行内存回收。
```





### Par New收集器



![image-20201219113401815](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219113401815.png)



简单点说 ，就是加强版的  serial 版本，多条GC并行的进行垃圾清除，但是清除过程还是会 stop  the world

ParNew 目的是 减少  暂停时间，适应交互应用

> Serial收集器可用的所有控制参数（例如：-XX：SurvivorRatio、-XX：PretenureSizeThreshold、-XX：HandlePromotionFailure等）、收集算法、StopTheWorld、对象分配规则、回收策略等都与Serial收集器完全一致

已经退出了历史舞台



```
开启选项：-XX:+UseParNewGC
```







### Parallel Scavenge 收集

这个是新生代的垃圾收集器，基于-标记-复制 算法实现， 所以这里我们知道了吧 复制算法 适合 新生代。

Paraler Scavenge 收集器的目的是 尽可能在控制吞吐量。适合在后台做计算

>  通过参数-XX:GCTimeRadio设置垃圾回收时间占总CPU时间的百分比。 
>
> 通过参数-XX:MaxGCPauseMillis设置垃圾处理过程最久停顿时间。
>
> 通过命令-XX: UseAdaptiveSizePolicy**开启自适应策略**。 
>
> 我们只要设置好堆的大小和MaxGCPauseMillis或GCTimeRadio，收集器会自动调整新生代的大小、Eden和Survivor的比例、对象进入老年代的年龄，以最大程度上接近我们设置的MaxGCPauseMillis或GCTimeRadio。





## 老年代收集器



### serial old收集器

![image-20201219140928474](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219140928474.png)

serial old是老年代的 收集器，

他和 serial 的区别也是  一个工作在老年代（标记-整理），一个工作在新生代（复制算法）



###  paraller old收集器

![image-20201219140938453](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219140938453.png)

Parallel Old 收集器是 Parallel Scavenge 的老年代版本，追求 CPU 吞吐量。



```
开启选项：-XX:+UseParallelGC

打开此开关后，使用 Parallel Scavenge + Serial Old 收集器组合来进行内存回收。

开启选项：-XX:+UseParallelOldGC

打开此开关后，使用 Parallel Scavenge + Parallel Old 收集器组合来进行内存回收。
```



###  CMS收集器

![image-20201219141027565](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219141027565.png)



```
开启选项：-XX:+UseConcMarkSweepGC

打开此开关后，使用 CMS + ParNew + Serial Old 收集器组合来进行内存回收。
```

CMS--并发标记清除收集器。并发表明他是在控制用户线程等待时间，那么就会减少吞吐量。

基于标记-清除算法实现

主要过程：

- 1. 初始标记：用户线程等待，使用一条标记线程去标记GC ROOT直接关联到的对象，这里会 stop the world
  2. 并发标记：使用多条标记线程，从GCRoots的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程，可以与垃圾收集线程一起并发运行；
  3. 重新标记：这里会 stop the world，修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录
  4. 并发清除：清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的。



优点缺点：

- 1. 优点是：速度高，没有明显的暂停，并发收集，
  2. 缺点是：吞吐量比较低，可能会让程序变慢，



JDK 9 用户CMS不用了，因为不够完美，还有4核 情况下，如 默认的GC线程是4，如果>4核，并发回收时垃圾收集线程只占用不超过25%的处理器运算资源,当处理器核心数量不足四个时，CMS对用户程序的影响就可能变得很大



## G1收集器

![image-20201219150739322](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219150739322.png)

G1 主要是 面向服务的的收集器， 它没有新生代和老年代的概念，而是将堆划分为一块块独立的 区域Region,。当要进行垃圾收集时，首先估计每个 Region 中垃圾的数量，每次都从垃圾回收价值最大的 区域开始回收，因此可以获得最大的回收效率。

> G1 收集器衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是G1收集器的MixedGC模式。



G1是 标记-整理算法 + 复制算法 实现的。运行期间不会产生内存碎片



对于大对象来说，G1 认为 一个对象只要超过了 Region容量一半的对象 就可以认为它是大对象 了，**每一个区域是可以调整的，取值范围为1MB～32MB，且应为2的N次幂，而对于那些超过了整个Region容量的超级大对象，将会被存放在N个连续的HumongousRegion之中**



一个对象和它内部所引用的对象可能不在同一个 Region 中，那么当垃圾回收时，是否需要扫描整个堆内存才能完整地进行一次可达性分析？

G1 使用 一个记忆set （本质是一个 ==哈希表== key-Region的起始地址,value-存储的是记录索引号，双向的卡表结构）来维护 本区域所以对象引用所在的位置(这些记忆集会记录下别的Region指向自己的指针，并标记这些指针分别在哪些卡页的范围之内)，进行可达性分析时 带着 这个 set去GCROOT 防止 整个堆进行遍历



G1的步骤：

- 1. 初始标记：Stop The World，仅使用一条初始标记线程对所有与 GC Roots 直接关联的对象进行标记，修改TAMS（对象地址快照值）指针的值
  2. 并发标记：从GCRoot开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。重新处理SATB记录下的在并发时有引用变动的对象
  3. 最终标记：stop the world,并发标记上一步留下的对象地址快照记录
  4. 筛选回收：stop the world ，并发GC线程执行。对区域按照一定条件排序，然后把需要回收的区域存活的对象复制到 空区域，再清除个旧Region的全部空间



优点缺点：

- 1. 平衡了吞吐量和暂停时间，
  2. 新生代和老年代变成了多个区域，效率高
  3. 标记-整理 + 复制算法实现，不产生内存碎片，不容易触发下一次GC
  4. 可预测的暂停时间
- 2. 缺点是 内存占的多，并且 用到的记忆set 也是占内存。
  3. 大内存 G1 更好， 小内存 CMS更好



## 低延迟收集器

......





## 收集器的比较



| 收集器                | 串行/并行/并发 | 年轻代/老年代   | 收集算法             | 目标         | 适用场景                                      |
| --------------------- | -------------- | --------------- | -------------------- | ------------ | --------------------------------------------- |
| **Serial**            | 串行           | 年轻代          | 复制                 | 响应速度优先 | 单 CPU 环境下的 Client 模式                   |
| **Serial Old**        | 串行           | 老年代          | 标记-整理            | 响应速度优先 | 单 CPU 环境下的 Client 模式、CMS 的后备预案   |
| **ParNew**            | 串行 + 并行    | 年轻代          | 复制算法             | 响应速度优先 | 多 CPU 环境时在 Server 模式下与 CMS 配合      |
| **Parallel Scavenge** | 串行 + 并行    | 年轻代          | 复制算法             | 吞吐量优先   | 在后台运算而不需要太多交互的任务              |
| **Parallel Old**      | 串行 + 并行    | 老年代          | 标记-整理            | 吞吐量优先   | 在后台运算而不需要太多交互的任务              |
| **CMS**               | 并行 + 并发    | 老年代          | 标记-清除            | 响应速度优先 | 集中在互联网站或 B/S 系统服务端上的 Java 应用 |
| **G1**                | 并行 + 并发    | 年轻代 + 老年代 | 标记-整理 + 复制算法 | 响应速度优先 | 面向服务端应用，将来替换 CMS                  |



## 怎么选择合适垃圾收集器



可以搭配使用

![image-20201219151550958](https://xiaoboblog-bucket.oss-cn-hangzhou.aliyuncs.com/blog/image-20201219151550958.png)

