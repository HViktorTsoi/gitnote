# Model-free RL

### First Visit Monte-Carlo Policy Evaluation

在每一个episode中, 当且仅当第一次遇到某个状态s时,就增加s的计数器,并累加s的return,这样在多个episode之后,多次采样到s,求其reward的均值,就会收敛到真实的return.

并且,这种采样的方式打破了对状态空间数目的限制,因为每一次不用把所有的状态空间都计算一遍,要计算的数量只取决于采样的多少.

### Every Visit Monte-Carlo Policy Evaluation

类比与first visit,在每一个episode中, 每当遇到某个状态s时,都增加s的计数器,并累加s的return,这样在多个episode之后,多次采样到s,求其reward的均值,就会收敛到真实的return.

### Incremental Mean
对于平常意义下的均值,有一个重要的概念: 可以将某个序列的求和均值,转化为增量均值:
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559114588955-1559114588956.png)
这样,就可以在一个动态序列中,每当有一个新的量时,不需要计算整个序列的和再除以n,而是直接就可以计算新的均值了.
而且实际上,增量均值的表示方式揭露了均值的一个重要意义: 看$(x_k-\mu_{k-1})$这一项,这一项称为误差项,这有一种更新的思想,当新的状态比旧的均值高,则误差为正数,需要将老的均值加上这个正数,变得稍微高一点,从而更符合观测;新状态低于旧均值是反之.这是一种重要的思想,在很多其他算法中(如kalman filter),都包含了这种思想.

### Incremental Mente-Carlo Updates
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559116484810-1559116484811.png)
在每个episode末尾进行更新$V(s)$, 查询在这个episode中所有涉及到的state, 看从这个state出发能得到了return(即$G(t)$),再使用Incremental Mean进行更新