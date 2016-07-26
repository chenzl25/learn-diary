## 软件分析

### [北大课件1](http://sei.pku.edu.cn/~xiongyf04/SA/2015/1%20intro.pdf)

### 哥德尔不完备定理

蕴含皮亚诺算术公里的一致系统是不完备
皮亚诺算术公理=自然数：

 - 0是自然数
 - 每个自然数都有一个后继
 - 0不是任何自然数的后继
 - 如果b,c的后继都是a,则b=c
 - 自然数仅包含0和其任意多次后继

对任意能表示自然数的系统,一定有定理不能被证明
主流程序语言的语法+语义=能表示自然数的形式系统

### 停机问题证明

假设有算法`bool Halt(p, i)`能够判断(*p为程序，i为输入*)

如果存在，那么对于下面的Evil程序来说必定是矛盾的。
```c
void Evil(p) {
  if (!Halt(p, p)) return; else while(1);
}
```

同理，可证明得到不存在**能够使得其他程序确保无内存泄漏的算法**
`bool LeakFree(Program p)`



如果存在，那么对于下面的Evil程序来说必定是矛盾的。
```c
void Evil(p) {
  int a = malloc();
  if (LeakFree(p)) return;
  else free(a);
}
```

### 莱斯定理(Rice’s Theorem)

关于程序行为的任何非平凡属性,都不存在可以 检查该属性的通用算法

#### 证明(反证法)

给定函数上的性质P,因为P非平凡,所以一定存在程序使得P满足,记为ok_prog。假设检测该性质P的算法为P_holds。


如果P_holds存在,则我们有了检测停机的算法：
```c
Bool halt(Program p, Input i) {
  void evil(Input n) {
    Output v = ok_prog(n); p(i);
    return v;
  }
  return P_holds(evil); 
}
```

### 近似求解判定问题

1. must analysis, lower/under approximation(下近似)，只输出*是*或者*不知道*
2. may analysis, upper/over approximation(上近似)*否*或者*不知道*

### 程序分析

- 对程序代码进行分析,获得关于程序行为的信息
- 程序分析的基本原则就是做抽象,将无法分析的 内容抽象为可分析的内容

### 软件解析学

通过收集和分析数据,为软件开发和使用过程中 的各种任务提供可行的有价值的信息