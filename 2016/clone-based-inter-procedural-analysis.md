##基于克隆的过程间分析

###[北大课件6](http://sei.pku.edu.cn/~xiongyf04/SA/2015/6%20interprocedural%20analysis.pdf)

过程内分析（*Intra-procedural Analysis*）

- 只考虑过程内部语句,不考虑过程调用 • 目前的所有分析都是过程内的

过程间分析（*Inter-procedural Analysis*）

- 考虑过程调用的分析
- 有时又称为全程序分析Whole Program Analysis
- 对于C, C++等语言,有时又称为链接时分析Link-time Analysis

### 基本思路

把Call/Return语句当成goto语句
对本地变量改名，并且调用和返回时添加赋值语句

### 过程间分析的精度问题

如果A调用了B，C也调用了B。那么A的分析结果可能会引入到C中。
ps：*A，B，C均为函数*

过程内分析不精确的条件:存在两个条件互斥
过程间分析不精确的条件:一个过程被调用两次

### 上下文敏感性
Context-sensitivity

上下文非敏感分析（*Context-insensitive analysis *）

-  在过程调用的时候忽略调用的上下文

上下文敏感分析（*Context-sensitive analysis *）

- 在过程调用的时候考虑调用的上下文

基于克隆的上下文敏感分析是在每一处调用创建原函数的一份复制。

### 克隆背后的主要

当直接在控制流图上分析达不到所要求的精度时, 通过复制控制流图的结点来提高分析精度