## 最大的贡献是提高了之前的工作ILCC的性能，但是不知道这个提升是不是由于给出了更多的信息（比如更多棋盘格的信息，如反射率信息）。已知intensity的使用使我很难将其视为自动校准算法，因为intensity将基于激光雷达的激光功率，目标距离等而有所不同，因此可能需要提前手动确定。

这里是reviewer没读懂，或者cost function那里写的不清除，不需要人工指定intensity的值，其值的高低是用来决定点云属于哪一类模式

## 和ILCC相比缺少创新性，在refine和checker localization上做的很好，但是当拿到checker之后，流程几乎和ILCC一致，在方法部分没有显式的引用ILCC

在cost function上，棋盘格投影方式，有区别

## 在实验部分，提出的方法是target-based的，不知道为什么要和Pandey 2015的targetless的对比，如果换成target-based会更fair

修改版本中仅和target-based的进行对比

## 作者说非重复性是SSL的特性，但是实际上有SSL是重复性扫描的(比如)，也有机械雷达是非重复扫描的，这里应该对分类更明确

待解决，

## 作者说SSL的轴向误差很大，但是缺使用了法向分割方法来定位棋盘格，并且work了，这个需要说明。

说明在法向分割的过程中，邻域选择的范围比较大，因此会得到更平滑的法线估计；

## Scaramuzza et al. [20] proposed a geometric model-based… 引用文献错误

## The article is interesting except for issues in language and mathematical expressions. Please help the reader to better understand and appreciate the article by making suitable corrections. 

因为篇幅问题，公式描述的不清晰，对所有的公式增加描述