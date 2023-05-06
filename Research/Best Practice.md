1. research-oriented code, 一般来说面向两点进行架构设计
   1. 做evaluation，核心module的参数需要暴露，并且需要返回大量中间结果，方便在Experiment部分画图
   2. 同时需要做e2e的demo，要考虑把所有封装的核心模块集成在一起
   3. 所有的data structure需要定义明确，并且随时准备变更