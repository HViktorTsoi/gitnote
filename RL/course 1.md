1. 如果想让时间更短 可以为reward定义一个时间负分项 每过一个step就将reward -1
2. 不能使用贪婪的方式
3. Markov性质: 以helicopter为例,什么是满足Markov的状态呢? 状态集合(位置,速度,角位置,角速度)就是一个Markov状态,知道了这个状态之后,就不需要直到之前半小时的历史状态了,因为这个状态就能唯一的生成下一个状态,或者说下一个状态可以只取决于这个状态,而有没有前边的半个小时的状态,都不会对下一个状态产生影响;而如果状态只有(位置),那他就不是Markov状态,因为这个状态无法确定下一个状态,因为不知道速度,这时候前边半小时的状态是能决定下一个状态的.


RL agent 分类:
1. value-based 只有值函数,没有显性策略
2. policy-based 只存储策略,没有值函数
3. Actor-critic 同时结合策略和值函数
或者
1. model-free 不构建模型 不需要知道外部环境是如何变化的 只根据值函数或者策略函数
2. model-based 构建外部环境变化的模型(比如构建直升飞机姿态变化的模型)

RL和Planning的区别
RL是在agent未知环境的运行机理的情况下(也就是说只能看到observation和reward),学习环境的运行机制,
而planning是在agent可以直到环境的运行机制的情况下,已经可以向后看若干步骤(类似一颗搜索树),在这种情况下进行决策

和传统搜索理论一样RL也要解决一个trade-off,即使exploration和exploitation的平衡