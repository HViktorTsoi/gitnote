# Burst Utilization
是将Array of structures(AOS)转换为 structure of arrays(SOA)。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/06/1586159773527-1586159773529.png) 

如下边的例子，这个优化的原理还是访存的连续性。使用SOA之后，可以保证访存的连续性。如果使用AOS，同一个warp中的连续线程访存的间隔是和structure的结构相关的，可能存在很大的间隔。而使用SOA之后，特别是对于大型的array，保证同一个warp内访存都是连续的，整个warp是在间隔较大的几个区域内取几个大块的内存，有效提高内存的吞吐量。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/06/1586159916461-1586159916462.png)