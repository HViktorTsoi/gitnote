#  Planning by Dynamic Programming

注意,这节课讲解的是Planning问题,而不是RL问题.

## Policy Iteration

Policy Iteration 是使用**Bellman期望方程**来求解. 用迭代的方法来求解最优的value fn,以及最优的策略.
这里我们可以使用**synchronous backups**方法,即:

在每一个迭代轮次k中, 对于所有的state, 使用上一个轮次的value fn $v_k(s')$, 根据Bellman公式来更新这一个轮次的value fn $v_{k+1}(s)$ (其中$s'$是s的后继状态,就是s采取action后能转移到的状态), 也即:

![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/28/1558975694364-1558975694365.png)

- 开始的时候可以将所有的$v_0(s)$设置为随机值,并随机选取一个policy  $\pi_0$,然后重复迭代以下过程, 直到收敛:

- 在第k轮时, 首先使用上述的synchronous backups bellman方程, 来评估policy $\pi$(上一轮迭代后得到的), 得到这一轮的value fn $v_k(s)$;

- 然后,使用1 step look ahead的方法, 根据这个新的$v_k(s)$来更新policy, 得到 $\pi'$(这一过程可见ppt中的small grid world例子);

- 重复这两步, 最终能够保证$v(s)$收敛到最优$v^*(s)$, 且$\pi(s)$收敛到最优$\pi^*(s)$, 如下图所示(证明在后面给出).
![title](https://raw.githubusercontent.com/HViktorTsoi/gitnote-image/master/gitnote/2019/05/28/1558976217618-1558976217623.png)

## Value Iteration

value iteration是使用**Bellman最优方程**进行求解.
与policy iteration不同, value iteration没有显式的policy,在迭代过程中,中间的$v(s)$并不代表任何policy(而在policy iteration).
