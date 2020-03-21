
# BANK
待确认：shared memory中的BANK数量和Thread数量一致
# Scan的应用
# Compact
类似与select，将一个大的集合中的一个小的子集保留下来；
这里scan的作用是:给定一个是否保留某个元素的mask(由1 0组成)，对这个mask数组进行scan sum，就得到了目标数组中的每个元素在原集合中的地址。（为什么要这样做，而不是如果按照朴素的思路，逐一判断mask是否为1，再将满足条件的元素写到结果数组中：这样的问题是，由于判断并写入的过程是并行的，如果想输出到结果数组中，就必须加锁或者用其他复杂的方式来维护结果数组的增长）
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/19/1584556074573-1584556074601.png)
步骤如下
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/03/21/1584728222283-1584728222308.png)

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