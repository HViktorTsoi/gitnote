$$
\delta_t=r_t+\gamma V_{t+1}-V_{t}
$$

$$
GAE_t=\delta_t+\gamma t GAE_{t+1}
$$

$$
GAE_t=(r_t+\gamma V_{t+1}-V_{t})+\gamma t GAE_{t+1}
$$
（实际上为Bellman方程，是累计衰减的advantage）