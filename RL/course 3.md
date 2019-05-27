#  Planning by Dynamic Programming

注意,这节课讲解的是Planning问题,而不是RL问题.

Iterative Policy Evaluation
用迭代的方法来求解最优的value fn,以及最优的策略.

使用**synchronous backups**:
在每一个迭代轮次k中,对于所有的state,使用上一个轮次的value fn $v_k(s')$, 以及Bellman公式来更新这一个轮次的value fn $v_{k+1}(s)$ ()