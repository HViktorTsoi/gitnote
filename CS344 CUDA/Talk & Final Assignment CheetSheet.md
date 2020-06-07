# Ian Buck访谈
从Brook到CUDA，大佬们已经是在考虑怎么给程序员们设计一种编程模型，让他们更容易编程；而作为程序员，我们还在费力的学习怎么使用这种编程模型。。。。。。
最终CUDA组确定的模型是线程模型；并采用c/c++扩展的方式来引入使用CUDA

# Stephen Jones访谈
Dynamic Parallelism可以用来处理树状问题，比如构建八叉树，快排等。
关于CUDA，以及并行计算的建议：一开始就要以并行的思路取思考问题，而不是先有串行的方案，再去想如何以十倍的速度去加速它。

# 天元
入门 contribute到真实项目；
框架方向分两个，一个是微观的内核优化；另一个是宏观的多机优化；


# Final Assignment

## Question 2
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757879256-1586757879257.png)

## Question 3
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757852742-1586757852744.png)

## Question 4
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757827118-1586757827120.png)

## Question 5
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757796200-1586757796202.png)

## Question 6
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757739813-1586757739814.png)

## Question 7
原题中说的意思是，不做任何tid的映射，1024x1个线程就是载入一整行图像；32x32个线程就是载入一块图像。

这个题的关键点是，由于要做5x5的卷积，那么1024x1的一行图像上的每一个像素还需要自己周围的5x5的像素，就需要载入额外4行的halo(上边2行+下边两行)，总共的GMR(global mem read)是

1024*4+1024 = 5120
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757587000-1586757587001.png)

而对于32x32的线程布局，由于邻居元素可以重用，因此只需载入周围一圈的halo就可以，总共的GMR是

(32+2+2)*4+32*4+32*32 = 1296

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757657688-1586757657688.png)

因此总共的访存带宽比是
5120/1296=~400%

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586757679734-1586757679738.png)

## Question 8

对于升级后64x64的Kernel，其处理64*64大小的图像，加上halo之后一次性访存的带宽为

(64+4)*4+64*4+64*64 = 4624

而原来32x32的Kernel，其处理同样64x64大小的图像需要4个Block，加上Halo之后一次性访存带宽为
4*( (32+2+2)*4+32*4+32*32 ) = 5144

因此升级后访存更少，效率更高，提高的效率为

5144/4624 = 112%, 112% - 100% = 12%

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/13/1586779777824-1586779777827.png)

## Question 9

查表 [CUDA - WIKI](http://en.wikipedia.org/wiki/CUDA)。

用SM的最多reg数量 / SM的最多Threads数量。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/14/1586837249394-1586837249418.png)

## Question 10
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/14/1586841741199-1586841741202.png)

同样查表。

## Question 11
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/14/1586841922231-1586841922234.png)
查表，每个SM最多线程数 / 每个SM最多Blocks数

## Question 12
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/14/1586865603840-1586865603843.png)

## Question 13
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/14/1586874161563-1586874161586.png)

## Question 13

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/15/1586885976330-1586885976332.png)

通过双调排序的原理图可以看出，对于stage和substage而言，每一趟中，升序的部分(蓝色块)的tid满足：

tid % 2^stage+1^ < 2^stage^

而并行的两两元素中，负责交换的那一个线程的tid满足：

tid % 2^substage+2^ < 2^substage+1^

由此可完成Kernel：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/15/1586885961496-1586885961497.png)
