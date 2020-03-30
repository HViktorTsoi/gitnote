# N体问题
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585390409988-1585390410027.png)
对于N个粒子，计算任意两两粒子之间的力，然后根据这个力移动粒子。

可转换为一个计算矩阵的问题，矩阵的每个元素就是src-dst之间的力。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585390504390-1585390504393.png)

直接用NxN个线程计算，有严重的访存效率问题：在计算的过程中，每个global mem中的元素要被访问2N次(N次是作为src元素被N个线程访问，另外N次是作为dst元素)；
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585391225140-1585391225143.png)

因此要使用tiling方法，在每个pxp的block中，先把global mem存储到大小为2p的shared mem中(其中p为src元素，另外的p为dst元素，因为block可能在原来大矩阵的任何地方，src和dst不等)；
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585392599630-1585392599655.png)

但是这样仍然存在问题：虽然这个时候对于每个元素都只要从global mem里fetch一次，但是在shared mem里，每个元素仍然需要在p个线程里去共享(就是还会有p次shared mem的fetch)，这也会降低效率。另外由于我们要求p个行和，就还要把每一行的p个元素全都存起来，再用p个线程求和，这会浪费存储空间，还会造成额外的shared访存。

解决的办法是，每一行只用一个线程去计算。这样每个线程虽然还要取p次src，但是dst只需要取1次就可以了；代价是并行程度减小了。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585394244619-1585394244621.png)

关于p的大小：如果p太大，则内存的bandwidth会减小，因为有更少的tiling块(这是好事，不会频繁访存了)；同时如果p太大，shared mem就放不下tiling所需的数据了。

同时要保证至少有SM这么多个block，不能让SM有空闲。

另外1个SM中有多个block可能有优势，这时SM上的多个block可能有更好的延迟隐藏特性（同时有多个wrap从不同位置访存）。

## 总结
一个需要注意的trade-off就是： 最大化并行和每个thread更多工作量之间的tradeoff。(考虑更多thread造成的访存、通信的开销)。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585395652541-1585395652544.png)


# SpMv问题
稀疏矩阵x向量问题可以有两种解决方案：

一是为每一个元素分配一个线程，另一个是为每一行分配一个线程。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585396436226-1585396436228.png)

## per row算法：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585396712173-1585396712175.png)
这个算法在每行数量相差还不大时效率很高，因为每个线程内部直接完成求和，不需要线程间通信。但是如果稀疏阵的长短都不一致，在同一个wrap中的线程，其总运行时间取决于最长的那一行，会导致处理完较短的那些行的线程空等。

## per element算法
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585397491974-1585397491976.png)
per element算法不受row算法类似的长短不一致导致的效率问题。但是这个方法需要用segment scan把每一行的结果加起来，这会导致线程之间的通信，造成额外的开销。

## hybrid方法
一个能想到的结合二者优点的方法就是：讲csr表示的稀疏矩阵划分成两部分，对矩阵的前一半用per row方法，因为前一半一般都是比较满的；对后一半使用per element方法。

重点是如何划线。当行满时，per row大约比per ele快3倍，因此一个经验法则是：当超过1/3行在要划线这个位置之前都有非0元素，就在这里划线。这也是cuSparse库中hybrid的内部实现原则。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585405860427-1585405860428.png)

## 总结
- 保证所有thread是busy的，在同一个wrap中尽量避免负载不均衡的情况
- 减少进程通信开销，local reg速度 > shared memory >> global memory
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/28/1585406912217-1585406912218.png)


# Graph遍历
## 一个基本的实现
对于某个特定的深度d的所有顶点，迭代做以下操作：
并行查找所有边,如果某一条边以这些顶点作为其中一端的所有节点如果这些节点里还有没被访问过的(没被标记过深度)，就将其深度标记为d+1。

实际实现中，只要遍历一次所有的边就可以，找那些有一个顶点是当前深度d，且另一个顶点没被访问过的那些边对应的顶点。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585507292091-1585507292113.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585507310948-1585507310954.png)


注意，如果有两个边同时连接到同一个顶点，如下图，那么这两个边对应的线程会同时尝试向这个顶点写入同一个值。但是并不会出现race condition，因此他们写入的是同一个值，谁先谁后 都无所谓。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585507535399-1585507535400.png)

如何判断这个搜索是否完成呢？方法是没设置一个device全局变量done，每一轮bfs开始之前将其设置为true；在搜索开始后，任意一个thread，只要发现有没搜索到的节点，就将done设置为false。同样，这里不必担心race condition的问题，因为只要去写done的线程，写入的都是false，先写后写也无所谓。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585507738063-1585507738067.png)

这个方法的并行度很好，不需要线程之间的同步或者通信；访存还有优化空间；
但是这个方法需要在host上控制bfs的轮次(这个问题可以通过CUDA5之后的dynamic parallel解决)；
另外这个方法的一个非常大的缺陷是work complexity太高，是O(VE)。对于每个节点，至少要扫描一遍所有的边。因为E一般至少要大于，那么这个算法的复杂度至少要O(V^2^)，对于较大的图这样的复杂度过高。

## 高效实现
之前的实现中，每一轮bfs都要遍历所有的边，这里提出一种更高效的实现。

首先要把图表示成以下的格式(其实就是类似图邻接矩阵的稀疏CSR表示，或者一种连续的邻接表)
C: 顺序存储每一个节点的邻接节点；
R: 顺序存储每个节点的第一个邻接节点在C中的位置；另外最后多出一个元素也指向C的最后一节点的起始位置
D: 初始为-1,表示每个节点是否访问过

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585548576513-1585548576516.png)