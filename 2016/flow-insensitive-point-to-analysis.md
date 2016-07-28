## 流非敏感分析 & 指向分析

### [北大课件5](http://sei.pku.edu.cn/~xiongyf04/SA/2015/5%20points-to%20analysis.pdf)

### 流非敏感分析

如果把程序中语句随意交换位置(即:改变控制流),如果分析结果始终不变,则该分析为流非敏感分析。

### 指向分析

每个指针变量可能指向的内存位置

1. Anderson指向分析算法

2. Steensgaard指向分析算法


### 别名分析

给定两个变量a, b,判断这两个变量是否指向相同的内存位置，返回以下结果之一:

- a, b是must aliases:始终指向同样的位置
- a, b是must-not aliases:始终不指向同样的位置
- a, b是may aliases: 可能指向同样的位置,也可能不指向

别名分析结果可以从指向分析导出:

- 如果pts(a)=pts(b)且|pts(a)|=1,则a和b为must aliases
- 如果pts(a)∩pts(b)=∅,则a和b为must-not aliases • 否则a和b为may aliases

### 域非敏感Field-Insensitive分析

```c
Struct Node { 
  int value; 
  Node* next; 
  Node* prev;
};
```
```
a = malloc(); 
a->next = b; 
a->prev = c;
```

把所有struct中的所有fields当 成一个对象
原程序变为
```
a’=malloc();
a’=b;
a’=c;
//其中a’代表a,a->next,a->prev
```
分析结果:
a, a->next, a->prev都有可能指 向malloc(), b和c

### 域敏感Field-sensitive分析

对于Nodel类型的内存位置x, 添加两个指针变量:

- x->next 
- x->prev

对于任何Node类型的内存位置x, 拆分成四个内存位置

- x
- x.value 
- x.next 
- x.prev

a->next = b 转换成 ∀x∈a, x.next⊇b