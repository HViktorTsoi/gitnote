# BANK
待确认：shared memory中的BANK数量和Thread数量一致



# 优化等级
优化一本分为4个等级，其中一二是最重要的，三会获得较少的速度提升
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585065864828-1585065864833.png)
一般的原则有：
- 合并global mem访问
- 尽量使用shared mem
- 减少bank conflict
- 优化寄存器
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585066073657-1585066073659.png)



# APOD
一个重要的原则是不要在真空中优化，一定要根据deploy之后的实际数据效果来优化，不要想当然的在parallelize和optimize之间循环。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585072731180-1585072731181.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585072982891-1585072982895.png)

最终的并行化效率收到Amdahl定律影响。

例如，如果IO占了50%的时间，那就不可能获高于2x的加速比



# 内存带宽

首先，绝大部分GPU程序都是收到内存限制的(瓶颈是访存)，所以要尽可能接近内存峰值带宽。

在优化的时候，要根据内存的理论峰值带宽来评判，如果只达到40~60%的带宽，说明有优化空间；一般要优化使带宽高于60 ~ 75%
在计算实际带宽的时候，就是看读取/写入了多少bits的内存，并且如果是读写的话，要x2，然后除以总共的耗时。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585113159968-1585113159972.png)



# 合并内存访问
首先要清楚，在物理上，GPU调度线程是以wrap为单位；（*一个wrap中有32个线程？*）
以下三种内存访问模式：
1 是较好的，因为在同一个wrap多个线程是访问相邻的内存，这样一个wrap取一整块连续的内存就可以；
2 的效率很低，因为在一个wrap运行时，wrap内不同的线程要随机访问不相邻的内存，可能要取很多块内存；
3 可能导致最差情况，如果访存有间隔同一个wrap可能最差要取size(wrap)这么多块的内存(如果间隔足够大)，而最好的情况1只需要取1块。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585113591689-1585113591694.png)

还是以矩阵转置为例，这里对于i来说，读取in的操作是连续的[i+j * N]，相邻的线程i之增加了1，有很大的可能是合并访存的，即1个wrap只需要取一块内存；而写入out的操作则是非常坏的，因为[j+i * N]，连续线程的访存间隔达到了N，就是说很有可能一个wrap要取N块内存；
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585117046174-1585117046176.png)

解决方法： Tiling



# Tiling
一个经常使用的原则，由于我们的写入数据时存在非连续访存，那么需要现将数据copy到shared mem，在shared mem中处理(局部转置)之后，再以连续访存的形式写回到output中。

这里的连续访存实际上就是对于threadIdx.x连续的线程，其访问的内存地址也要连续。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/25/1585130943127-1585130943128.png)



# Little's Law

GPU速度快除了有成百上千的processor，还因为有非常大的内存带宽。因此一定要尽可能理应到所有的带宽。

传送的字节数=每个事务的平均延迟时间×带宽

这里的事务是指内存事务（包括有用的和没用的）

通常内存读写要耗费几百个时钟周期，可以将内存看做一个很宽且很深的管道，此时如果各个线程取的内存较小，则管道不能填满，带宽利用率就会降低；

一个有用的解决方案是让单个线程取更多的内存，比如原来处理一个float的线程，现在一次处理4个float(float4,128bit)，以提高带宽，但是这样会引入额外的逻辑，让每个线程执行的算法不那么自然。

在矩阵转置的例子中，虽然在使用每个线程处理一个元素和tiling之后，带宽有上升，但是整个内存的利用率还是不够高，其原因就是有某些因素在影响延迟(latency)，就是barrier，即在读取到sharedmem后，由于需要sync，结果导致好多已经完成访存的线程在空等那些仍在在访存过程中的线程(这个空等时间非常长)，最终导致延迟升高。

解决的办法可能是，减少每个block中的线程数，这样在每个block中就有更少的线程在sync处等待，减少sync造成空等的概率(sync的作用域就是block)；或者增加每个SM执行的block数量，这样一个block没有sync完的时候另一个block还能继续向下走。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/26/1585214715007-1585214715037.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/26/1585215053736-1585215053739.png)



# Occupancy
Occupancy是指SM上实际运行的线程数和最多能运行的线程数之比。一般来说要尽量增加occupancy的比例，占满SM，但同时也要根据实际的计算任务，注意多者之间的trade-off。

在硬件层面上，每个SM的shared mem，线程数，block数是固定的，并且这几个值会相互约束。

比如，如果一个SM有48Kshared mem，支持1536个线程，那只能同时处理3个需要16K shared mem的blocks；而如果这是我的一个block需要1024个线程，则此时一个SM只能处理1个block了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/26/1585216673092-1585216673094.png)



# 最小化Thread Divergence
GPU的架构一般为SIMT(单指令多线程)，这样的架构可能存在Thread Divergence的现象。

如果在kernel的代码段中存在分支，则如下图，在同一个wrap中，不同的线程就可能取不同的指令，而一个指令周期只能取一次指令，因此走另一个分支的那些线程就必须要等走当前分支的线程的指令都执行完，才能继续执行，否则就是空等，这就产生了Thread Divergence。

一个block中无论有多少个线程，最多产生32路的Thread Divergence(因为同时最多只能执行一个wrap，即32个线程)。

Thread DIvergence都要在wrap的层级来考虑。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585246256353-1585246256358.png)

## 分支语句
这里举2个例子，
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585247343057-1585247343061.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585247780344-1585247780346.png)

## 循环语句
在循环语句的divergence情况中，首先要考虑不同分支循环体的开销。由于一个kernel负责整个循环的执行，因此一个wrap中的Divergence的严重程度会被放大循环次数这么多倍。

举例：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585249808893-1585249808896.png)

以真实例子，对于kernel中对图像边界的判断代码，最多可以产生2-way的divergence
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585289082060-1585289082064.png)

## 避免Divergence的方法
- 尽量减少大量的分支代码
- 让相邻的thread尽量执行相同的分支
- 注意线程要有尽量相近的workload



# 数学优化
- 如果能用float的话，尽可能不要用double。这里需要注意，字面量会被编译为double，因此计算a=b+2.5时如果想再快一点，就要写成a=b+2.5f。
- 使用intrinsic 函数，内建的数学函数，如__sin(),__cos(),__exp()等。注意这些函数比math.h精度地2~3bit，但是比它快多得多。
- 使用优化了数学操作的编译选项(参考developer guide)。



# Streams

## Host-device交互
在host上申请的内存，如果想要传送到gpu上，需要先copy到一块固定的pinned memory上，再通过PCIe copy到gpu，这会浪费很多时间。（称之为staging）

一个解决办法是不使用原生C的malloc，使用cudaHostMalloc，直接申请pinned的内存。

或者使用cudaHostRegister，把用原生malloc申请的内存直接pin了，然后在后续也可以直接传到GPU。

也可以使用cudaMemcpyAsync，这样在开始host-device内存拷贝时，cpu就不会阻塞了，memcpy一边进行，cpu一边继续工作。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585290645115-1585290645119.png)

## Streams
stream是一系列顺序cuda操作。可以将不同的cuda操作分配到不同的stream上，这样就可以将某些操作并行起来了，特别是内存操作和内核操作就可以并行了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585293905701-1585293905702.png)

使用stream后代码的执行顺序就不一定和代码顺序一致了，只有同一个stream中的才会顺序执行，不同stream中的根据GPU的资源，有可能并行。

在非默认stream中，host-device传输要采用cudaMemcopyAsync。

见以下的例子。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585294445771-1585294445774.png)

有一点需要注意，如果某一个kernel A需要内存arr1，但是arr1的copy和kenel A的调用不在一个stream中，这时虽然两个操作也可以并行起来(cuda并不会分析kernel对内存的依赖)，但是就会产生错误的结果，因为内存arr1在同时被copy写入和被kernel A操作写入，见下图2例子。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585294706277-1585294706279.png)

## 流水线
stream的一个重要用途就是流水线，当处理一个非常大的数据时，没法放进GPU，这时候需要分块处理，但是在 复制块到GPU->计算->从GPU复制回来 这个过程中，大量时间被消耗到来回复制上了，没有利用好GPU的计算效率。

这个时候就可以讲数据划分成更小的块，使用不同的stream，将上述过程组成流水线。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585296779125-1585296779127.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585296833221-1585296833222.png)

## 使用stream的好处：
- 重叠访存和计算；
- 用更小的kernels填满GPU（比如reduce到后期，计算量越来越小，此时GPU就有一部分空闲了，这个时候就可以启动其他的kernels，来填满GPU）；
- 参考CUDA文档的Stream和Events。



# 总结
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585300049928-1585300049929.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585300104933-1585300104936.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585300146944-1585300146946.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585300360437-1585300360437.png)

# Hint
- 可以使用nvvp来profile，而不需要去计算实际的内存带宽应用

- shared mem可以是任何维度的，不像kernel input只能一维的

- 在Block中，线程的物理id(在wrap中)是按照x,y,z维度的顺序递增的，也就是说，以dim(32,16,8)这样一个线程配置的block为例子，0...31号线程都是是挨着的(y,z相同)，0...512(32*16)号线程都是挨着的(z相同)。以下面的二维矩阵为例，同一行内相邻的32个线程就是属于一个wrap的。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/27/1585289303250-1585289303252.png)