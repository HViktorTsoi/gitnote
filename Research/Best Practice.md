1. research-oriented code, 一般来说面向两点进行架构设计
   1. 做evaluation，核心module的参数需要暴露，并且需要返回大量中间结果，方便在Experiment部分画图
   2. 同时需要做e2e的demo，要考虑把所有封装的核心模块集成在一起
   3. 所有的data structure需要定义明确，并且随时准备变更
   4. 对于某个code file中的function，即使使用了全局变量来耦合，也最好将所有与全局变量同名的参数通过
   5. 一开始应该面向数据集做实验, temp file 的命名应该是 XX/数据集名称/实验子模块, 这样在code便于保存和新增针对数据集的evaluation
   6. When you are not highly motivated, you should do all the time-wasting/heavy-lifting work first


2. Writing Best Practice
   1. 先把文章中的所有formulation定下来, 再围绕这些formulation展开写作, 否则会导致符号和定义经常变化, 文字也会无法确定