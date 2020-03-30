# Scan的应用
# Compact
类似与select，将一个大的集合中的一个小的子集保留下来；
这里scan的作用是:给定一个是否保留某个元素的mask(由1 0组成)，对这个mask数组进行scan sum，就得到了目标数组中的每个元素在原集合中的地址。（为什么要这样做，而不是如果按照朴素的思路，逐一判断mask是否为1，再将满足条件的元素写到结果数组中：这样的问题是，由于判断并写入的过程是并行的，如果想输出到结果数组中，就必须加锁或者用其他复杂的方式来维护结果数组的增长）
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/19/1584556074573-1584556074601.png)
步骤如下
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584728222283-1584728222308.png)


# Allocate
输入：给定一个向量，存储的是每个位置的元素需要的空间；比如1 2 0 3 就是第1个元素要1空间，第二个要2单位空间，第三个不需要，第四个要3单位空间。
输出：分配空间后的向量，总长度是reduce sum输入；实现方式是对输入进行exclusive scan，就得到每个元素对应空间的起始地址。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585550286095-1585550286098.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585550308560-1585550308564.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/30/1585550325499-1585550325502.png)

# Segmented scan
类似上一个assignment中的长array分block进行scan，Segmented scan是对一个长array分段进行scan，因为单独对某一个小段scan会浪费GPU的资源，因此将这些段合并起来统一用一个kernel来执行，如下图。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584774081894-1584774081897.png)

应用： 稀疏矩阵乘法(SpMV)，例如PageRank就是很大的稀疏矩阵乘法，矩阵的行是世界上所有的网页链接，矩阵的列是所有与某一网页有关联的网页
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584774214139-1584774214143.png)

稀疏矩阵的表示方式：使用CSR方式，其中第一个vec VLAUE存储实际的值；第二个vec COLUMN存储该值属于哪一列；第三个vec ROWPTR存储原矩阵中每一行第一个非零元素在VAlUE中的指针
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584774756476-1584774756479.png)
使用CSR表示之后，首先按照标准矩阵乘法确定在矩阵乘法中哪两个元素是要相乘的(可根据COLUMN和ROWPTR向量来确定)，整理之后发现可以将右矩阵按照与被乘数一一对应的形式也表示为一个向量，再对两个向量做点乘，然后对结果向量进行Segmented scan，得到每一行的结果。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584775856234-1584775856237.png)

# Sort
## 可并行化bubble sort
首先偶数地址的元素标记为红，两两一组，组内按大小进行交换（颜色也交换）；然后分组的开始索引向左移动一个元素，重复进行下一趟的排序；重复N次到所有元素有序。
Work复杂度： O(n^2^)
Step复杂度：O(n)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584786032469-1584786032472.png)

## Merge sort
mergesort很容易实现并行，但是存在这样的问题：在开始阶段需要归并的组非常多，但组内元素很少；中间阶段分组和组内元素都较多；最后阶段只有两组，但是是两个元素非常多的组。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/22/1584816167642-1584816167645.png)
因此按照这个特性，需要将GPU版本的merge sort划分为3个阶段：

1. 多个线程，每个线程归并一个小数组;(问题数>>SM)

2. 有很多中等大小的有序块需要归并，如果串行归并的话效率很低；此时对于两个有K个元素的待归并列表，需要开2K个线程；每个线程首先知道自己对应的元素在自己列表中的位置p1，然后在另一个列表中用二分搜索出元素应该在的位置p2，这样这个元素在归并好的列表中的位置就应该是p1+p2；且这个搜索的复杂度不高，仅为n(logn)；(问题数=SM)
3. 只剩下两组巨大的归并，此时如果还用2的方法，就会有很多SM处于idle的状态(因为2只用于处理单个block)，所以要尽可能将这个两个大数组分为小数组，并占满多个SM。(问题数<<SM)
具体做法：
a) 首先对两个列表做这样的操作：每隔M个元素取一个spliter元素(例如每256个元素取一个元素)标记为A B C D, E F G H等等；
b) 将这些spliter排序，这样构成了一个序列；
c) 如下图所示，以F为例，B<F<C<G，则可以用二分找到F在BC间的位置,以及C在FG间的位置；由于蓝绿数组都是有序的，这时候就可以用一个block直接去merge蓝绿FC分段，然后填充回去spliter序列的FC；（注意，因为BC和FG之间都最多只有256个元素，所以可以保证FC之间不超过256个元素，这样block的内存分配就可以确定下来）。
d) 对spliter序列中的所有分段都做这样的操作，得到最终的有序数组。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/22/1584816381706-1584816381709.png)

## sorting network
排序网络，一种无关排序算法；这里使用双调排序来实现并行，双调序列是一个先单调递增后单调递减（或者先单调递减后单调递增）的序列。双调序列可以应用Batcher定理被划分为两个子双调序列可用于子问题划分。也可以使用单双归并排序网络。

排序网络可以有多种不同的结构。排序网络的一个重要特性就是数据无关，不管数据怎么分布，比较次数都是固定的。

在使用GPU解决排序网络问题时，一般是一个block应用一个排序网络，可有效利用共享内存。每个线程负责一个元素，根据网络上的edge来确定和哪个线程进行比较并且交换；并且每次比较之后需要同步。

以双调网络为例，首先讲两个子数组分别按升降排序并拼接在一起，构成双调序列；然后对这个序列进行并行Batcher划分，构成更小的双调序列；一直划分，直到得到的子序列长度为1为止。这时的输出序列按单调递增顺序排列。此处可参考[双调排序](https://blog.csdn.net/xbinworld/article/details/76408595)。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/22/1584880073645-1584880073651.png)

## Radix sort
基数排序，复杂度为O(kn)。从最低位开始，是0就移动到前边，是1就移动到后边，同为0或者1的保持按之前若干位的排序不变。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/22/1584883515332-1584883515337.png)

重要的是，根据某一位是0或者1来移动的操作，在GPU上可以用compact就可以实现。

compact的predicate可以用位运算来计算；对输入数组的0-predicate的进行一次scan，就可以计算出0元素的地址；保存0元素的总共数量，就是1元素的起始地址，然后再对0-predicate进行scan，得到1元素的地址。

通常GPU版本的radix会分为多路，而不是一位一位的比较。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/22/1584883960374-1584883960375.png)

## quick sort
qsort是递归形式的排序，GPU目前还不支持递归，因此需要将递归形式转换为循环形式，实际上是使用segment讲数组划分为三部分(左边小于pivot,pivot,右边大于pivot)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/23/1584893097660-1584893097662.png)

## key-value sort

### thrust中的vector的operator=是复制，不是传递引用