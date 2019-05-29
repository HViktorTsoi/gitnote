# Model-free RL

# First Visit Monte-Carlo Policy Evaluation

在每一个episode中, 当且仅当第一次遇到某个状态s时,就增加s的计数器,并累加s的return,这样在多个episode之后,多次采样到s,求其reward的均值,就会收敛到真实的return.