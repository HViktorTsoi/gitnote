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

