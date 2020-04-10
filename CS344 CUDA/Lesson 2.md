RULE #0：在CUDA中， 先将问题划分到能够满足单个block的规模，并尝试使用shared mem，这样的思路很重要
RULE #1：Kernel Launch参数中的shared meme大小是bytes，不是数量！！！

# 并行计算模式

1. Map: 一一映射(color-gray) ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442165621-1586442165660.png)

2. Gather: 收集元素并计算结果(图像局部区域avg)，线程分配给输出元素，每个线程决定他从什么地方读取。 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442193443-1586442193445.png)


3. Scatter: 分散操作，每个线程向内存输出多个值，也可能多个线程操作同一块内存(注意 排序属于此操作)，且每个线程计算其在哪里写入结果。线程分配给输入元素，每个线程决定他写入什么地方。 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442234809-1586442234812.png)

4. stencil: 模板操作，每个线程访问的输入数据为一个固定模式的模板邻居范围内(类似卷积核)的数据，数据存在重复访问 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442270350-1586442270353.png)


5. Transpose: 转置操作 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442295241-1586442295243.png) 注意 对AOS(array of structure)的存储结构进行重构变为SOA(structure of array)的过程也可以成为转置，例如将相同的数据类型的成员组合到同一个数组中，以提高计算效率 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/09/1586442339379-1586442339381.png)
注意，这里map,transpose,stencil的共同特征为1对1操作，其余则不是
![](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/PicGo/Screenshot%20from%202020-02-26%2000-18-36.png)


6. Reduce：类似求和或者求平均数的操作，reduce必须满足两点性质 
一是二元运算符，
二是满足结合率 
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/04/1583329953204-1583329953229.png)

> reduce或其他的运算模式中，threadID体现的是等价为串行代码中最外层循环的循环变量。并且一个通常的模式为，串行代码中i,i+1这样的访问操作，在kernel中体现为tid，tid+stride，其中stride可以为并行的线程数（这样可以保证相邻线程的访存相邻性）；另外也是因为在线程内使用i,i+1判断和写回相邻的内存对应相邻的线程不方便（因为这样相邻的线程之间就相互依赖了），而使用tid<stride这样的方式，写回i,i+stride时相邻线程不会发生冲突，判断相邻更方便。这个时候只要把tid看做是最外层的循环变量就可以了。

reduce时，由于需要频繁访存，较好的办法是使用shared mem。而想要使用shared memory，就要以block为单位去划分数据，即blockDim最好为2的整数次方。在第一趟reduce中，前边若干个blocks对应的前k个元素是k个blocks的k个最大值；在第二趟reduce中，对第一趟产生的前边若干个blocks对应的元素再进行reduce；反复此步骤，直到最后得到唯一值。

使用shared mem还有一个好处，就是可以在将global拷贝到shared mem时，自动将数据中没有对齐2的整数倍的边角余料数据补0，减少后期处理的复杂性。

7. Scan：类似与running sum或则cumsum，如果使用串行来处理，则每次第k步的结果都依赖1..k-1步。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/05/1583344225270-1583344225272.png)
scan的必须满足的三点要素：
二元运算符，
满足交换律，
单位元I(I op a = a)。

例子 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/05/1583343676119-1583343676122.png)
scan结果的第一个元素是单位元，之后每个元素是前边所有元素经过操作符之后得到的结果。（注意这里最后的9不被考虑，这就是这类scan的性质）

scan可以分为两种，其一是exclusive，不包含当前元素，其二是inclusive，包含当前元素。这两种适用于不同的算法场景。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/05/1583344351006-1583344351009.png)	

**Hill Steele Scan**
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/08/1583669522412-1583669522435.png)
work复杂度 O(n log n)(reduce的复杂度 * 步骤数)
step复杂度 O(log n)

**Blelloch Scan**
首先是reduce，间隔0,2,4,8...做reduce；然后反过来做downswip，详细过程见下图
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/08/1583670374959-1583670374962.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/08/1583673639883-1583673639886.png)
work复杂度 2 O(n)(reduce的复杂度)
step复杂度 2 O(log n)

对比来看，HS算法的step复杂度更低，但是Ble算法的work复杂度更低，具体要根据GPU性能来决定。当算法需要的work大于processor时，要选择work efficient的并行方式；当processor充足，满足要执行的work时，要选择step efficient的算法。

对于元素数量高于一个block，单个block处理不了的情况时，要用以下方式来进行scan：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/18/1584537762178-1584537762205.png)
具体理解用一列数字的实例来计算就可以。

8. Histogram：统计直方图。串行的直方图算法无法直接应用于并行模式，因为在累加这一步骤存在race condition(不同的线程都对同一个bin进行增加)。因此，要将histo算法改为并行，可有以下几种方法:

A. 使用atomic add操作来累加直方图。实现简单，但是实际上最坏的情况多个线程是串行访问的(同一个bin被多次累加)
B. 对于k个线程，为每个线程分配一个局部直方图(局部直方图的bin和全局是一样的)，然后将数据划分为k份，每个线程独立处理维护一个局部直方图；
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/09/1583728950403-1583728950406.png)
然后使用reduce将k个局部直方图加起来（需要使用n binds次atomicadd来合并局部直方图）
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/09/1583729104596-1583729104598.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/09/1583754051205-1583754051208.png)
C. 首先按key排序，然后按key reduce(trust库中有这两个算法的实现)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/09/1583749101668-1583749101672.png)

# barrier
__synchronize,类似一道屏障，所有线程到barrier之前都要等待，直到block内的所有线程都执行完毕，类似于线程同步

**注意__synchronize同步的范围是block，而不是全局！！！！！！**

在GPU编程中，最终要的是尽可能减小内存的读写时间(注意不是次数，而是时间)

# global memory，local memory，shared memory

golbal memory：GPU全局范围的
local memory：thread 内部范围
shared memory：block内部共享，用__shared__关键字标识

# 合并内存访问
由于GPU每次从全局内存读取都是读取一整块，因此相邻的线程读/写连续的GPU存储是最有效率的，这种模式合并(coalesce)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/02/1583158508705-1583158508735.png)

# 原子内存操作
多个thread同时读写同一块内存会产生race，因此需要引入原子操作，cuda支持一系列原子操作，例如atomicadd(a,b)

# 高效并行化建议
1. 尽量减少访存时间，对于global memory，尽量合并访存
2. 防止线程发散，即避免由于条件分支或循环导致的多个线程执行的代码不一致

# Steps and work
Step complexity：把可并行的运算合并为一步，总共需要的步骤数
Work complexity：串行的，一共需要运行的次数
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/04/1583320390102-1583320390109.png)
布伦特定律(brent theorem)：对于p个处理器，处理大小为n的数据，需要的step数量
