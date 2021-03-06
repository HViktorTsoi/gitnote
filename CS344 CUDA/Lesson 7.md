# Burst Utilization
是将Array of structures(AOS)转换为 structure of arrays(SOA)。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/06/1586159773527-1586159773529.png) 

如下边的例子，这个优化的原理还是访存的连续性。使用SOA之后，可以保证访存的连续性。如果使用AOS，同一个warp中的连续线程访存的间隔是和structure的结构相关的，可能存在很大的间隔。而使用SOA之后，特别是对于大型的array，保证同一个warp内访存都是连续的，整个warp是在间隔较大的几个区域内取几个大块的内存，有效提高内存的吞吐量。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/06/1586159916461-1586159916462.png)


# Scatter to Gather Optimization
scatter通常比gather效率更低，通过下面的例子可以看出来，scatter通常要写入更多的地方，这一方面增加了访存的次数；同时经常存在重叠的写入，会产生race，通常还要额外加入barrier或者atomic operation。相反，gather更多的是读取的操作，GPU可以通通过合并等方式更有效的处理这一操作。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442961120-1586442961124.png)
 

# Tiling
在CPU架构中，通常是隐性地将某些线程用到的数据copy到on-chip-cache中以加速访存；但是在GPU中，线程数量过多，每个线程访问的内存都很分散，直接用透明的cache机制效率很低，因此一般结合业务逻辑，将频繁访问的数据copy到shared memory(on-chip-memory)中，这一技术叫做tiling。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586443360667-1586443360671.png)

在tiling的过程中，可能存在的一个问题是，依赖的数据超出tid对应的范围，比如某个block中tid是128~255，但是依赖的数据是126~257，这时候在载入局部数据时，需要有几个特殊的线程把多出来的数据也加载到shared mem。


# Privatization
私有化指的是每个线程在频繁写入结果时，不是写入到global memory，而是写到线程自己的reg里或者block的shared mem里，然后当所有线程完成写入之后，再将最终的本地结果合并到全局内存中。

一个典型的例子就是histogram，每个线程维护一个小直方图，或者每个block维护一个局部大直方图，最后再把所有的直方图进行合并，写入到globalmem中。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586500966558-1586500966561.png)


# Binning
其实就类似于对点云做voxel，或者对图像划分网格。用某种类似网格的数据结构将输出的位置和输入数据映射起来。

这样做的目的是，这样如果某个元素想找到他的邻居，或者跟他有关的元素，就不用搜索全部的元素了，而只要在binning之后的结构上找到自己的相邻的grid就可以了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586502339683-1586502339685.png)

比如，计算美国每一座城市方圆300公里的邻居城市的人口，就可以先把美国地图划分binning(类似grid)，在找某个城市的邻居时只要简单的将网格索引做加减操作就可以了，不需要将所有城市遍历一遍。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586503307202-1586503307203.png)


# Compaction
如果输入数据中只有很少一部分需要被处理，大部分不需要被处理，此时将所有需要处理的元素压缩(compact)到一个新的数组中再进行处理，会得到更高的效率。这里详情见lesson3的notes。

对于compact加速的倍数，这里需要注意，并不是compact的压缩比越高，加速的倍数就越大，如下例：

- 如果只有idx为8的倍数的元素被处理，由于在GPU上同一个warp只能同时运行32个线程，那么原来一个warp能处理4个元素(32个线程，每8个有一个被处理)，在compact之后能处理32个元素，加速了8倍；
- 如果只有idx为32的倍数的元素被处理，原来一个warp只有1个元素被处理，compact之后32个元素被处理，加速了32倍；
- 如果只有idx为128的倍数的元素被处理，这里需要注意，原来4个warp中只有1个元素被处理，但由于不包含128的warp会直接退出，其运行时间接近0，GPU直接调度其他的warp了，所以可认为原来1个warp(包含128的warp)只能处理1个元素；而caompact之后，由于一个warp中最多也只有32个元素，所以compact之后也只加速了32倍，而无法达到128倍。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586507748708-1586507748710.png)


# Regularzation
当每个线程处理的问题的可能规模不一致时，可能会出现负债不均衡的现象，大多数线程空等一个线程结束。这时候就需要对问题进行Regularzation，将每个线程处理的问题裁剪成大致相等的规模，保证负载均衡，然后对于多出来的未处理数据，要么用cpu去处理，要么用特殊的kernel或算法去处理。

以计算城市人口的问题为例，如果某些城市邻居很多，某些很少，那么如果每个线程的负载就会发生不均衡。这时候进行Regularzation的方法就是，固定每个城市搜索的的邻居数为k(比如5)，对于多出来的城市，用cpu去处理，或者设计特殊的kernel来处理。

Regularzation适用场景：
- 大多数子问题规模都相似，只有少量outlier
- 在运行时可以确定负载情况

对于问题规模相差很大，没有一个确定的均值的情况(比如处理幂率分布的数据)，就不适合Regularzation了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586530419309-1586530419311.png)


# 并行优化模式总结
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586530710185-1586530710188.png)


# Libraries
- cuBLAS：线性代数库
- cuFFT：FFT工具
- cuSparse：稀疏阵代数库
- cuRAND：随机数生成
- NPP：low level的图像处理原语
- Magma：LAPCAK(基本上是一堆优化器的合集)的cuda实现
- CULA：特征值求解器，矩阵分解器等
- ArrayFIre：数组操作库(类似numpy)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586531787258-1586531787260.png)

使用lib的基本姿势：
1. 找到和cpu库对应的cuda api(废话)
2. 有些库的内存管理是透明的，有些需要使用对应库的api来手动管理
3. 注意在编译或者写cmake的时候链接对应的库
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/10/1586532252787-1586532252790.png)

# 编程动力工具

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586588290390-1586588290393.png)

- Thrust
是应用在host端的(他的代码风格host样式的，类似C++ STL)，功能类似STL的通用工具。已经实现了sort，scan，reduce(by key),map等很多通用的原语了，还可以用Transform+用户自定义的functor来实现一些复杂的计算，比如实现向量加法等。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586579179700-1586579179704.png)

- CUB
是应用在kernel里的，在Kernel级别可重用的库，例如对一个kernel的每个block对应的数据进行sort。

CUB提供了一种从global mem到shared mem，到thread的访存和计算模型。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586586056393-1586586056395.png)

- CUDA DMA
用来处理global memory到shared mem之前的数据传输。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586588199943-1586588199945.png)

这种数据传输在tiling或者privatization中经常使用，会遇到例如halo(所需要的数据比tid对应的数据大几圈)，或者数据不规则、有跳跃等等情况，非常繁琐，此时cudaDMA就可以用来处理这些问题。另外还可以更好的解决global mem向shared mem数据传输的效率瓶颈问题。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586588849642-1586588849644.png)

注：但是CUDA DMA发展的不太好，只支持到Kepler一代，并且没有被NVIDIA官方采用。并且GPU本身没有DMA的硬件支持，cudaDMA使用额外的warp来模拟DMA。

另一点需要注意的是，由于CUB的编程模型里，每一个block中的每个线程处理多个元素，因此在总元素一定的情况下，每个线程自己处理的元素大小和线程数量就构成了一对trade-off。

从下图的对角线可以看出一些问题：随着ITEMS_PER_THREAD这个值增加且BLOCK_THREAD减少，性能逐渐提高，因为这样虽然减小了并行度，但是由于每个线程的工作量增大了，因此线程调度的开销和其他一些开销就被冲掉了。但是当继续增大这个值时，并行度的继续降低反而会拉低性能。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586596193678-1586596193694.png)


# 其他Language
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586599131161-1586599131167.png)


# 跨平台解决方案
- Open CL: 类似CUDA
- Open GL Compute: 适用于图形学
- Open ACC: 使用指令式的方式提示编译器那里进行并行，对于改进已有的cpu代码来说开发效率很高。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586599829711-1586599829715.png)



# Dynamic Parallelism

## Bulk Parallelism
批量并行用来解决可以批量并行计算的问题，比如reduce，map等等；与之相反的的是不可以批量并行的算法，比如拼接两个字符串

## Nested Parallelism
如果可以实现嵌套并行，那么在kernel中就可以直接启动其他的kernel(多个线程的global kernel，而不是device kernel)。而不是首先要返回CPU，再由CPU去调用下一个所需的kernel。(这个专业术语叫做Launch-on-the-fly)。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586601439502-1586601439516.png)
这里以音频处理为例：如果想对一串音频信号的幅度进行归一化操作，限制所有音量在一定范围内，传统的模式是划分成两个kernel：
1. 首先使用1个reduce kernel来计算音频的最大值；
2. 再调用1个map kernel将所有的信号的值除以这个最大值。
这个过程会造成调度的开销。如果有嵌套并行，就可以将这两个kernel合并到一起。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586601719119-1586601719141.png)

## Task Parallelism
任务并行同时让GPU运行多个相同的实例(类似batch)，占满GPU，获取尽可能高的吞吐量；

## Recursive Parallelism
递归并行，即可以直接在kernel中递归调用kernel。有了递归之后，就可以将问题划分成更小的子问题，然后使用同样的kernel去并行的处理这样更小的子问题。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/11/1586601731502-1586601731512.png)

## 动态并行要点
1. 每个thread都执行一样的程序，所以如果想要在kernel中启动其他kernel，一定要保证只有一thread启动了这个kernel(比如判断只有tid为0的线程才调用新的kernel)。（否则的话就是所有的线程每一个都启动了一个新的kernel。当然如果需求是多个线程启动多个kernel，可以按照实际情况来）。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586662889815-1586662889818.png)
2. 每个block是独立执行的，包括某个kernel的block和他的子kernel的block也是独立的。syncthreads和stream，event之类同步操作只能同步自己，而不能同步子kernel的任何block。
3. 每个block的shared mem是独立的，block和子block之间也不能共享shared mem。如果想在父子之间共享内存，必须使用global mem。
4. 注意，在动态并行中，在，诶个kernel中启动的实际上是grid，而不是blocks。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586662905244-1586662905245.png)

关于内存共享的访问权限范围，请看下边的例子：
第一项是全局的device变量，所有gpu上的线程都可以访问；
第二项是shared mem，只有当前block可以访问，子线程，包括任何其他block中的线程都不可访问；
第三项是在堆上分配的global memory，任意线程都可以可以通过指针来访问。**(这里是否可以在kernel中使用malloc仍然需要确认)**
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586663246230-1586663246233.png)

## 动态并行-快排
有了动态并行之后，就可以用更优雅的方式递归实现快排了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586673073128-1586673073131.png)

除了实现更简单之外，在原始版本的cuda中，如果用迭代的方式，想把某一层快排结果收集起来，在上一层进行合并，就必须把所有有序块的信息都存下来，有大量的通信开销，这是并行的效率变得非常低。有了动态并行之后，这些信息就直接自顶向下的传递下去了。

另外，由于每一块的大小都不一样，要完成同一层的快排，就需要被分配到小块的线程空等大块的线程，对负载均衡很不利(如下图绿色块所示)。而使用动态并行之后，就可以在划分子问题使，将左右两块划分到不同的stream中，这样左右完全不影响，就可以独立的完成排序了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586673036207-1586673036211.png)

动态并行快排的代码如下图所示，其中partition可以尝试用并行来实现。而下边的左右子问题在不同的stream中，可以完全并行。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586673088776-1586673088780.png)

动态并行实现的快排相比非动态GPU快排的优点：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/12/1586673466505-1586673466507.png)