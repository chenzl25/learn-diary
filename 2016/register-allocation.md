# 寄存器分配

## 局部

### 自顶向下

1. 为每个虚拟寄存器计算优先级(*利用外部性质如：使用频率*)
2. 将虚拟寄存器按优先级次序排列
3. 按优先级指派寄存器
4. 重写代码

### 自底向上

思想：按逐个操作的方式来仔细考察值定义和使用的细节

```c
for each operation, i, in order from 1 to N where i has the form
  op vr[i1], vr[i2] => vr[i3]
  
  r[x] <- Ensure(vr[i1], class(vr[i1]))
  r[y] <- Ensure(vr[i2], class(vr[i2]))
  if vr[i1] is not needed after i
    then Free(r[x], class(vr[i1]))
  if vr[i2] is not needed after i
    then Free(r[y], class(vr[i2]))
  r[z] <- Allocate(vr[i3], class(vr[i3]))
  rewrite i as op r[x], r[y] => r[z]
  if vr[i1] is not needed after i
    then class.Next[r[x]] <- Dist(vr[i1])
  if vr[i2] is not needed after i
    then class.Next[r[y]] <- Dist(vr[i2])
  class.Next[r[z]] <- Dist(vr[i3])

Ensure(vr, class)
  if (vr is already in class)
    then result <- vr‘s physical register
  else
    result <- Allocate(vr, class)
    emit code to move vr into result
  return result

Allocate(vr, class)
  if (class.StackTop >= 0)
    then i <- pop(class)
  else
    i <- j that maximizes class.Next[j]
    store contents of j
  class.Name[i] <- vr
  class.Next[i] <- -1
  class.Free[i] <= false
  return i
```


## 全局

### 自顶向下

1. 通过SSA（静态单赋值形）来寻找所有全局活动范围。之后将虚拟寄存器从写为全局活动范围。
2. 为每个活动范围估算逐出代价。
3. 建立冲突图
4. (依照逐出代价优先级)进行k(*物理寄存器数*)着色，如果遇到某个节点(*活动范围*)无法着色则进行逐出(*简单的做法有：对每个逐出的活动范围的定义后加store，使用前加load，活动范围使用备用寄存器*)，或者拆分活动范围(*不全部逐出，建立新的冲突图*)。


冲突图 I=(N, E) 的建立：

```c
// 利用每个基本块的LiveOut集合(存活变量集合)
// 反向遍历基本块的操作
for each LR[i]
  create a node n[i] ∈ N
for each basic block b
  LiveNow <- LiveOut(b)
  for each operation op[n], op[n-1], ..., op[1] in b
    with form op[i] LR[a], LR[b] => LR[c]
    for each LR[i] ∈ LiveNow
      add(LR[c], LR[i]) to E
    remove LR[c] from LiveNow
    add LR[a] and LR[b] to LiveNow
```

### 自底向上
自底向上和自顶向下的主要区别在于：二者用于对活动范围排序以确定这色顺序的机制不同。自顶向下的分配器使用高层信息来选择一种着色顺序，而自底向上的分配器则根据有关冲突图的详细结构性知识来计算一种顺序。

着色顺序：

```c
initilize stack to empty
while (N not empty)
  if exist n ∈ N with n° < k
    then node <- n
    else node <- n picked from N
  remove node and its edges from I
  push node onto stack
  
// PS：第五行else中的选择需要利用外部知识和启发式策略(*如cost/degree，cost为逐出代价，degree为当前冲突图中的度*)。
```

重建冲突图着色：

```c
while (stack not empty)
  node <- pop(stack)
  insert node and its edges into I
  color node
```

最后当栈为空时，冲突图就重建完毕。如果遇到无法着色的情况则进行逐出或者拆分活动范围。
并循环重复：找到活动范围，建立冲突图并着色的过程。直到算法停止(*栈为空*)。
当然，自底向上的分配器可以保留供逐出代码使用的寄存器，正如自顶向下的分配器所做的那样。这种策略使得它可以在单趟处理之后结束。






