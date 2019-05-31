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
注意,更新$V(s)$ 是在每个episode的末尾进行的,需要查询在这个episode中所有涉及到的states, 看从某个state出发能得到了return(即$G(t)$),再使用Incremental Mean进行更新.

另外,有的时候我们需要遗忘很久之前的记忆,而不是将所有步骤的信息都体现到均值中去,这是就可以将误差项前边原本的$\frac 1 N$替换为一个固定的$\alpha$(这也就相当于在$V(s_t)$前边加了遗忘折扣因子,因为原来的式子里,$\frac 1 N$是不断减小的,从而新的误差信息占比重越来越小,而固定$\alpha$后,新的误差项占的比重不变,就相当于旧均值的权重被不断减小了,类似于SVM的Loss正则项的两种表示形式之间的关系).

### TD Learning
与Monte-Carlo不同, TD直接在每一步更新value fn, 而不是在一个episode结束之后才更新. Monte-Carlo中,是在每个epi结束之后,获得所有有关状态的$G(t)$,并用来计算均值误差更新相关状态,每一个状态的实际return是已知的;而TD是在每一步,使用一个猜测值($R_{t+1}+\gamma V(s_{t+1})$)来计算均值误差,并直接更新value fn.
因此,TD可以适用于没有结束状态的学习过程,或者使用没有还没有结束的,不完整的序列进行学习.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559122915954-1559122915955.png)

### MC,TD和DP之间的关系
首先这里以Driving Home例子说明MC和TD各自的特点:
假如有一下state和对应的时间过程
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559123066061-1559123066062.png)
那么对于MC方法, 需要在真正到家,得到最终的elapsed time 43之后,才能用这个值去计算均值误差,并更新前边多个状态的value;
而对于TD来说,每一步可以用下一步的预测值来计算均值误差并直接更新该状态的value(例如,在raining状态总预测值是40,而下一个状态exit highway后,实际上行驶的时间比之前预期的短,因此总预测值也缩小到了35,那么35这个值就可以和raining这个状态的40一起计算误差,并更新raining状态的value).
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559123079430-1559123079430.png)

另外,MC方法是unbiased的因为其均值是通过真实的过程观测值求得的(每个epi结束之后观测到的真实值),但是其variance高,因为一个状态的return $G(t)$取决于后面多个状态的actions, transitions, rewards;
TD方法是biased的,因为其均值误差是由猜测值计算的到的(下一步的估计value fn),但是variance低,因为一个状态的TD target只由下一个状态action, transition, reward决定.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559124475248-1559124475250.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/29/1559124485136-1559124485137.png)

MC方法最终收敛到与观测值有最小方均根误差(MSE)的解;
TD方法最终收敛到最大似然马尔可夫模型上,即收敛到最能拟合数据的MDP上;
(AB例子见课件)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/30/1559198265839-1559198265840.png)
由此我们可以得出:TD更适用于有马尔可夫性质的问题,而MC更适用于非马尔可夫性质的问题.

MC方法,是将一条路径exploit到底,然后根据return更新value
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/30/1559198912841-1559198912842.png)

TD方法只向前看一步,通过估计来更新value
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/30/1559198967117-1559198967129.png)

DP方法,由于已知系统的dynamic,每次在一个状态可以探索出所有可能的action和后继状态,所以进行的是类似BFS的搜索.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/30/1559199066281-1559199066282.png)

TD和DP方法是有bootstrapping的,包含估计;MC方法没有bootstrapping,只用真实值;
MC和TD都包含采样;DP没有采样,使用的是全体state和action空间.

下面这张图更好的解释了MC,TD和DP之间的关系
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/30/1559200593700-1559200593701.png)
使用全部状态空间的,只看一步的是DP,而看到底的就是搜索(注意这里针对的问题是适用与DP的问题,即最右子结构和重叠子问题,或有MDP性质)

使用采样状态空间的,只看一步的是$TD_0$(还有$TD_\lambda$等算法是看更多步的),而看到底的是MC.

注意,这里说的所有"估计",其实就是指当前时刻的$v(s)$值,例如,求TD的均值误差时,其实就是用当前backup的$V(s_{t+1})$来作为估计值进行计算.

### $TD(\lambda)$算法

由之前的内容我们知道,MC算法和TD算法各有其优缺点,那么有没有办法结合两者呢?
这就是TD($\lambda$)算法.TD($\lambda$)在更新时进行n step look ahead,即比TD(0)要多看若干步,但又不像MC一样要等到episode结束状态之后才更新.
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/31/1559238090827-1559238090828.png)

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/31/1559238099646-1559238099647.png)
在这个过程中,并不是所有return都是同等重要的,因此在每一项中加入衰减因子$\lambda$,构成平均n step return:
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/31/1559238233127-1559238233128.png)
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/31/1559238243215-1559238243216.png)
实际上,当$\lambda=0$时,就是TD(0)算法;当$\lambda=1$,就是MC算法.

以上是TD($\lambda$)算法的forward view, 相当于在一个状态上,用望远镜看未来的状态,并使用

还可以从backward view的角度进行考虑:
