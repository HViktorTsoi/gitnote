# Model-free RL

### First Visit Monte-Carlo Policy Evaluation

在每一个episode中, 当且仅当第一次遇到某个状态s时,就增加s的计数器,并累加s的return,这样在多个episode之后,多次采样到s,求其reward的均值,就会收敛到真实的return.

并且,这种采样的方式打破了对状态空间数目的限制,因为每一次不用把所有的状态空间都计算一遍,要计算的数量只取决于采样的多少.

### Every Visit Monte-Carlo Policy Evaluation

类比与first visit,在每一个episode中, 每当遇到某个状态s时,都增加s的计数器,并累加s的return,这样在多个episode之后,多次采样到s,求其reward的均值,就会收敛到真实的return.

### incremental Mean
对于平常意义下的均值,有一个重要的概念: 可以将某个序列求和均值转化为