---
layout:     post
title:      "[背包问题九讲-1] 01背包问题"
subtitle:   " \"背包问题九讲-1 01背包问题\""
date:       2019-8-5 9:13:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Algorithm
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## [背包问题九讲 - 1]—— 01 背包问题

> 背包问题九讲来自于崔添翼（ACM亚洲区金牌）的个人整理：<https://github.com/tianyicui/pack>，看完之后能够对背包问题的整体认识上一个台阶。
>
> 这里记录其的思路，并少量加上个人理解，同时增加Java代码的实现。

#### 1. 题目：01背包问题

> 有`N`件物品和一个容量为`V`的背包。第`i`件物品的费用是`c[i]`，价值是`w[i]`。求解将哪些物品装入背包可使价值总和最大。
>
> 注：此处的费用`c[i]`即指代的物品在背包中占据的容量，可以理解为`cost`。

#### 2. 基本思路

这是最基础的背包问题，特点是：每种物品仅有一件，可以选择放或不放。

用子问题定义状态：即`f[i][v]`表示前`i`件物品恰放入一个容量为`v`的背包可以获得的最大价值。则其状态转移方程便是：

`f[i][v]=max{f[i-1][v],f[i-1][v-c[i]]+w[i]}`

这个方程非常重要，基本上所有跟背包相关的问题的方程都是由它衍生出来的。所以有必要将它详细解释一下：“将前`i`件物品放入容量为v的背包中”这个子问题，若只考虑第`i`件物品的策略（放或不放），那么就可以转化为一个只牵扯前`i-1`件物品的问题。如果不放第`i`件物品，那么问题就转化为“前`i-1`件物品放入容量为`v`的背包中”，价值为`f[i-1][v]`；如果放第`i`件物品，那么问题就转化为“前`i-1`件物品放入剩下的容量为`v-c[i]`的背包中”，此时能获得的最大价值就是`f[i-1][v-c[i]]`再加上通过放入第`i`件物品获得的价值`w[i]`。

伪代码如下：

```python
f[0][0...V] = 0
for i = 1 to N
	for v = c[i] to V
		f[i][v] = max(f[i-1][v], f[i-1][v-c[i]] + w[i])
```

> 注意，这里的第二个循环从`c[i]`开始。当`v < c[i]`时，`f[i][v] =f[i-1][v]`， 这是由于此时背包不能装下第`i`件物品，因此不放第`i`物品，最大价值转化为前`i-1`件物品放入容量为`v`的背包中。

#### 3. 优化空间复杂度

以上方法的时间和空间复杂度均为`O(VN)`，其中时间复杂度应该已经不能再优化了，但空间复杂度却可以优化到`O[V]`。

先考虑上面讲的基本思路如何实现，肯定是有一个主循环`i=1..N`，每次算出来二维数组`f[i][0..V]`的所有值。那么，如果只用一个数组`f[0..V]`，能不能保证第`i`次循环结束后`f[v]`中表示的就是我们定义的状态`f[i][v]`呢？`f[i][v]`是由`f[i-1][v]`和`f[i-1][v-c[i]]`两个子问题递推而来，能否保证在推`f[i][v]`时（也即在第`i`次主循环中推`f[v]`时）能够得到`f[i-1][v]`和`f[i-1][v-c[i]]`的值呢？事实上，这要求在每次主循环中我们以`v=V..0`的顺序推`f[v]`，这样才能保证推`f[v]`时`f[v-c[i]]`保存的是状态`f[i-1][v-c[i]]`的值。

伪代码如下：

```python
f[0...V] = 0
for i = 1 to N
	for v = V to c[i]
		f[v] = max(f[v], f[v-c[i]] + w[i])
```

> 注意，此处将`v`进行逆序遍历，仔细观察状态转移方程`f[i][v]=max{f[i-1][v],f[i-1][v-c[i]]+w[i]}`我们可以发现，第`i`件物品的状态确定取决于第`i-1`物品的状态，而且需要的第`i-1`物品的状态中的所剩背包容量`v`的值要小于当前所剩背包容量`v`的。因此，只要逆序更新，就可以保证在推`f[i][v]`时（也即在第`i`次主循环中推`f[v]`时）能够得到`f[i-1][v]`和`f[i-1][v-c[i]]`的值。
>
> 注意：**当采用一维`dp`公式时，`v < c[i]`时，`f[i][v]`会自动继承`f[i-1][v]`的值**。

事实上，使用一维数组解01背包的程序在后面会被多次用到，所以这里抽象出一个处理一件01背包中的物品过程，以后的代码中直接调用不加说明。

过程ZeroOnePack，表示处理一件01背包中的物品，两个参数`c`、`w`分别表明这件物品的费用和价值。

```python
def ZeroOnePack(f,c,w):
	for v = V to c：
		f[v] = max(f[v], f[v-c] + w)
```

有了这个过程之后，01背包问题就可以简化为：

```python
f[0...V] = 0
for i = 1 to N
	ZeroOnePack(f, c[i], w[i])
```

#### 4. 初始化细节问题

我们看到的求最优解的背包问题题目中，事实上有两种不太相同的问法：

1. 有的题目要求“恰好装满背包”时的最优解
2. 有的题目则并没有要求必须把背包装满。

一种区别这两种问法的实现方法是在初始化的时候有所不同。

如果是第一种问法，要求恰好装满背包，那么在初始化时除了`f[0]`为`0`其它`f[1..V]`均设为`-∞`，这样就可以保证最终得到的`f[N]`是一种恰好装满背包的最优解。

如果并没有要求必须把背包装满，而是只希望价格尽量大，初始化时应该将`f[0..V]`全部设为`0`。

为什么呢？可以这样理解：初始化的`f[]`数组事实上就是在没有任何物品可以放入背包时的合法状态。

如果要求背包恰好装满，那么此时只有容量为`0`的背包可能被价值为`0`的nothing“恰好装满”，其它容量的背包均没有合法的解，属于未定义的状态，它们的值就都应该是`-∞`了。

如果背包并非必须被装满，那么任何容量的背包都有一个合法解“什么都不装”，这个解的价值为0，所以初始时状态的值也就全部为0了。

这个小技巧完全可以推广到其它类型的背包问题，后面也就不再对进行状态转移之前的初始化进行讲解。



#### 5. 一个常数优化

上面伪代码中的：

```python
for i = 1 to N
	for v = V to c[i]
```

中的第二重循环的下限可以改进。它可以优化为：

```python
for i = 1 to N
	for v = V to max(V- sum{c[i]..c[N]}, c[i])
```

> 注：此处崔添翼的原文中为`V- sum{w[i]..w[N]}`，个人理解为其笔误所致，应该为`c[i]`

原因解释：

由于只需要最后`f[V]`的值，倒推前一个物品，其实只要知道`f[V-c[N]]`即可。以此类推，对以第`i`个背包，其实只需要知道到`f[v-sum{c[i]...c[N]}]`即可。



#### 6. Java代码实现

二维数组实现

```java
class Solution{
    public boolean solve(Integer[] w, Integer[] c, int V){
        //N = c.length
        double[][] f = new double[c.length+1][V+1];
        for (int v = 1; v <= V; v++){
            //除0外初始化为负无穷
            f[0][v] = Double.NEGATIVE_INFINITY;
        }
        for (int i = 1; i <= c.length; i++){
            zeroOnePack(f, w[i-1], c[i-1], i);
        }
        return f[c.length][V] != Double.NEGATIVE_INFINITY;
    }

    private void zeroOnePack(double[][] f, int w, int c, int i){
        //V = f[0].length - 1
        for (int v = 0; v < f[0].length; v++){
            if (v-c < 0){
                f[i][v] = f[i-1][v];
            }else {
                f[i][v] = Math.max(f[i-1][v], f[i-1][v-c] + w);
            }
        }
    }
}
```



一维数组实现

```java
class Solution {
    public int solve(int[] w, int[] c, int N, int V){
        int[] f = new int[V+1];//初始化为0
        for (int i = 0; i < N; i++){
            zeroOnePack(f, w[i], c[i]);
        }
        return f[V];
    }
    private void zeroOnePack(int[] f, int w, int c){
        //V = f.length-1
        for (int v = f.length-1; v >= c; v--){
            f[v] = Math.max(f[v], f[v-c] + w);
        }
    }
}
```

常数项优化后实现：

```java
class Solution {
    public int solve(int[] w, int[] c, int N, int V){
        int[] sumC = new int[N];//sumC[i] = sum{w[i]..w[N-1]
        sumC[N-1] = c[N-1];
        for (int i = N-2; i>=0; i--){
            sumC[i] = c[i] + sumC[i+1];
        }
        int[] f = new int[V+1];//初始化为0
        for (int i = 0; i < N; i++){
            zeroOnePack(f, w[i], c[i], V, sumC, i);
        }
        return f[V];
    }
    private void zeroOnePack(int[] f, int w, int c, int V, int[] sumC, int i){
        for (int v = V; v >= Math.max(c, V - sumC[i]); v--){
            f[v] = Math.max(f[v], f[v-c] + w);
        }
    }
}
```

