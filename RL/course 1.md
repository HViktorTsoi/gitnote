1. 如果想让时间更短 可以为reward定义一个时间负分项 每过一个step就将reward -1
2. 不能使用贪婪的方式
3. Markov性质: 以helicopter为例,什么是满足Markov的状态呢? 状态集合(位置,速度,角位置,角速度)