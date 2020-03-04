# 并行计算模式

1. Map: 一一映射(color-gray) ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/ro8uchdhvm.png?imageView2/2/w/1620)


2. Gather: 收集元素并计算结果(avg) ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/c5ey0k4vyo.png?imageView2/2/w/1620)


3. Scatter: 分散操作，每个线程向内存输出多个值，也可能多个线程操作同一块内存(注意 排序属于此操作)，且每个线程计算其在哪里写入结果 ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/9n28cfriur.png?imageView2/2/w/1620)


4. stencil: 模板操作，每个线程访问的输入数据为一个固定模式的模板邻居范围内(类似卷积核)的数据，数据存在重复访问 ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/8b71jd6vot.png?imageView2/2/w/1620)


5. Transpose: 转置操作 ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/c3epqiiyjt.png?imageView2/2/w/1620) 注意 对AOS(array of structure)的存储结构进行重构变为SOA(structure of array)的过程也可以成为转置，例如将相同的数据类型的成员组合到同一个数组中，以提高计算效率 ![image](https://ask.qcloudimg.com/http-save/yehe-1215004/wv2p3x9aku.png?imageView2/2/w/1620)
注意，这里map,transpose,stencil的共同特征为1对1操作，其余则不是
![](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/PicGo/Screenshot%20from%202020-02-26%2000-18-36.png)


6. Reduce：reduce必须满足两点性质 
一是二元运算符，
二是满足结合率 
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/04/1583329953204-1583329953229.png)

> reduce或其他的运算模式中，threadID体现的是等价为串行代码中最外层循环的循环变量。并且一个通常的模式为，串行代码中i,i+1这样的访问操作，在kernel中体现为tid，tid+stride，其中stride可以为并行的线程数（这样可以保证相邻线程的访存相邻性）。
reduce时，由于需要频繁访存，较好的办法是使用shared mem

7. Scan：类似与running sum或则cumsum，如果使用串行来处理，则每次第k步的结果都依赖1..k-1步。scan的必须满足的三点要素：
二元运算符，
满足交换律，
单位元I(I op a = a)。

例子 ![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/05/1583343676119-1583343676122.png)
scan结果的第一个元素是单位元，之后每个元素是前边所有元素经过操作符之后得到的结果。（注意这里最后的9不被考虑，这就是这类scan的性质）
8. Histogram

# barrier
__synchronize,类似一道屏障，所有线程到barrier之前都要等待，直到kernel内的所有线程都执行完毕，类似于线程同步

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
