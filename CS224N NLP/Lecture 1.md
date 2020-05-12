在nlp中，如何表达一个词是一个关键问题。

一个朴素的想法是，用one-hot向量来表达一个词，但是这样存在的问题是，词的数量很多，可能有几万个，那么表达每一个词的one-hot向量维度就会很高；另外的一个关键问题是，在表达成这样的one-hot向量后，在语义上相同的词，他们的embedding并没什么联系。

这是word2vec被提出来了。在w2v的表征中，在某种意义上相近的词(比如同样是动词，或者同样近义词)，在embedding张成的空间中距离就会更近，如下图。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587641440945-1587641440946.png)

word2vec的idea是这样的：
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587641477868-1587641477870.png)

为了建模出词与词之间的关系，需要让现实中某个句子中出现的某个词，其embedding和跟它相近的词的embedding也相近。
因此，对于语料句子中的某个词
$W_t$
设其邻近词出现的情况下，这个词出现的概率为（反过来也可以）
$P(W_{t+j}|W_t)$


![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587641883120-1587641883122.png)

那么对于对于语料中所有位置T的词，当这个词出现之后，其邻居也出现的联合概率分布如下图的公式$L(\theta)$所示。

为了将乘法转换为加法，且不改变单调性，左右两边加log。

同时为了将最大化问题转换为最小化问题，在前边加上负号。

这样我们的目标函数$J(\theta)$就如下图所示。最终的目标是最小化$J(\theta)$。

这里需要说明，现实中的自然语言就满足$J(\theta)$的最优值，即一个合理的语句中，相邻的词是语义高度相关的。而我们的目标是针对我们的语料库，找到这样的$\theta$，使得$J(\theta)$真的满足这样的最优值。而唯一的参数$\theta$就是我们所要求的word embedding。

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587640237597-1587640237600.png)

下面的问题是，怎么计算$P(W_{t+j}|W_t)$呢？

我们使用下图的方式计算，实际上是中心词和他的邻近词embedding点积的softmax，这样如果这两个embedding越接近，其点积越大，从而得到的概率也就越大，符合我们刚才对问题的定义。

另外，对于词W，其作为中心词时使用参数$v_c$，作为语境词(邻居词)时使用参数$u_o$，这两个参数是不同的，因此每个词对应的参数实际上是[$v_c$;$u_o$]拼接起来，长度是2d，其中d是词向量的长度。

这里需要说明的是，为什么要使用softmax：
其一是，softmax中的exp函数，可以保证结果始终为正的；
其二是exp可以将点积的结果缩放的更极端，也就是小的更小，大的更大，而不是一个很平的分布；
其三是使用soft而不是hard，可以保留更多的信息量。
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587645739381-1587645739387.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2020/04/23/1587640206342-1587640206382.png)