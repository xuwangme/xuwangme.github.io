---
layout:     post
title:      "[背包问题九讲-3] 多重背包问题"
subtitle:   " \"背包问题九讲-3 多重背包问题\""
date:       2019-8-6 17:30:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Algorithm
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## [背包问题九讲 - 3]—— 多重背包问题

> 背包问题九讲来自于崔添翼（ACM亚洲区金牌）的个人整理：<https://github.com/tianyicui/pack>，看完之后能够对背包问题的整体认识上一个台阶。
>
> 这里记录其的思路，并少量加上个人理解，同时增加Java代码的实现。

#### 1. 题目：多重背包问题

> 有`N`种物品和一个容量为`V`的背包，每种物品都有最多有`m[i]`可用。第`i`种物品的费用是`c[i]`，价值是`w[i]`。求解将哪些物品装入背包可使这些物品的费用总和不超过背包容量，且价值总和最大。
>
> 注：此处的费用`c[i]`即指代的物品在背包中占据的容量，可以理解为`cost`。

#### 2. 基本思路

这题目和完全背包问题很类似。基本的方程只需将完全背包问题的方程略微一改即可，因为对于第`i`种物品有`m[i]+1`种策略：取`0`件，取`1`件……取`m[i]`件。令`f[i][v]`表示前`i`种物品恰放入一个容量为`v`的背包的最大权值，则有状态转移方程：1

> `f[i][v] = max(f[i-1][v-k*c[i] + k*w[i]，其中k>=0 && k<=m[i])`

时间复杂度是：O(V*sum(m[1]....m[N]))

#### 3. 转化为01背包问题（实际解题过程中采有这种方法，即二进制优化）

第一种好想好写的基本方法是转化为01背包求解：把第`i`种物品换成`m[i]`件01背包中的物品，则得到了物品数为`Σm[i]`的01背包问题，直接求解，复杂度仍然是`O(V*Σm[i])`。

但是我们期望将它转化为01背包问题之后能够像完全背包一样降低复杂度。仍然考虑二进制的思想，我们考虑把第`i`种物品换成若干件物品，使得原问题中第`i`种物品可取的每种策略——取`0..m[i]`件——均能等价于取若干件代换以后的物品。另外，取超过`m[i]`件的策略必不能出现。

方法是：将第`i`种物品分成若干件物品，其中每件物品有一个系数，这件物品的费用和价值均是原来的费用和价值乘以这个系数。使这些系数分别为`1,2,4,...,2^(k-1),m[i]-2^k+1`，且k是满足`m[i]-2^k+1>0`的最大整数。例如，如果`m[i]`为13，就将这种物品分成系数分别为`1,2,4,6`的四件物品。

> 注意：这里的最后一项是`m[i]-2^k+1`，即要保证拆分后的二进制项求和要等于`m[i]`。与完全背包的二进制拆分不一样。

分成的这几件物品的系数和为`m[i]`，表明不可能取多于`m[i]`件的第`i`种物品。另外这种方法也能保证对于`0..m[i]`间的每一个整数，均可以用若干个系数的和表示，这个证明可以分`0..2^k-1`和`2^k..m[i]`两段来分别讨论得出，并不难，希望你自己思考尝试一下。

这样就将第`i`种物品分成了`O(log m[i])`种物品，将原问题转化为了复杂度为`O(V*Σlog m[i])`的01背包问题，是很大的改进。

下面给出`O(V * log m[i])`时间处理一件多重背包中物品的过程，其中`m`表示物品`i`的数量`m[i]`：

> 注：此处崔添翼给出的一件多重背包中的物品处理过程的时间复杂度为`O(log m[i])`，应为笔误。实际应该为`O(V * log m[i])`。这样经过N个物品的循环之后，总的时间复杂度为`O(V * sum(log(m[i])...log(m[N]))`

```python
def multiplePack(c, w, n)
	if c * n >= V //因为该物品总消费超过了最大消费值，因此可认为数量无限制
		completePack(c, w)
		return
	int k = 1
	while k < n //二进制优化为01背包问题
		zeroOnePack(k * c, k * w)
		n = n - k
		k = k * 2
	zeroOnePack(n * c, n * w)
```



#### 4. O(VN)的算法

多重背包问题同样有O(VN)的算法。这个算法基于基本算法的状态转移方程，但应用单调队列的方法使每个状态的值可以以均摊O(1)的时间求解。由于用单调队列优化的DP已超出了NOIP的范围，故本文不再展开讲解。这个方法在楼天成的“男人八题”幻灯片上。

#### 5. 多重背包的OJ应用：POJ 1014

思路：将钻石的价值同样看做钻石的体积，假设钻石的价值和为V，那么体积和也为V，如果V为奇数直接pass。

如果V为偶数，那么问题就转化为：

在容量为V/2的背包刚好装满的情况下，求最大价值。

> 注意，要求是要V/2的背包刚好装满，初始化时f[]数组，除了f[0]为0，其余均为-∞。

最后，如果f[V]不等于-∞，那么存在V/2的背包的刚好装满的方案，返回true；否则返回false。

AC代码：

```java
import java.util.*;

public class Main
{
    public static void main(String args[])
    {
        Scanner sc = new Scanner(System.in);
        int id = 0;
        while (sc.hasNext()){
            id ++;
            int sum = 0;
            int num = 0;
            List<Integer> input = new ArrayList<Integer>();
            for (int i = 0; i < 6; i++){
                int tmp = sc.nextInt();
                if (tmp == 0) continue;
                sum += tmp * (i+1);
                num += tmp;
                int k = 0;
                while (Math.pow(2, k) <= tmp){
                    k++;
                }
                k--;
                //将输入转化为二进制优化形势
                for (int index = 0; index < k; index++){
                    input.add((int) Math.pow(2, index) * (i+1));
                }
                input.add((tmp - (int)Math.pow(2, k) + 1) * (i+1));
            }
            Integer[] c = new Integer[input.size()];//二进制优化后的输入数组
            input.toArray(c);
            if (num == 0){
                break;
            }else if (sum % 2 == 1) {
                System.out.println("Collection #" + id + ":");
                System.out.println("Can't be divided.");
                System.out.println();
            }else{
                Solution sol = new Solution();
                boolean res = sol.solve(c, c, sum /2);
                if (res){
                    System.out.println("Collection #" + id + ":");
                    System.out.println("Can be divided.");
                    System.out.println();
                }else {
                    System.out.println("Collection #" + id + ":");
                    System.out.println("Can't be divided.");
                    System.out.println();
                }
            }

        }
    }

}
//01背包模板
class Solution{
    public boolean solve(Integer[] w, Integer[] c, int V){
        double[] f = new double[V+1];
        for (int i = 1; i <= V; i++){
            //除0外初始化为负无穷
            f[i] = Double.NEGATIVE_INFINITY;
        }
        for (int i = 0; i < c.length; i++){
            zeroOnePack(f, w[i], c[i]);
        }
        return f[V] != Double.NEGATIVE_INFINITY;
    }


    private void zeroOnePack(double[] f, int w, int c){
        //V = f.length - 1
        for (int v = f.length-1; v >= c; v--){
            f[v] = Math.max(f[v], f[v-c] +w);
        }
    }
}
```



#### 6. 总结

这里我们看到了将一个算法的复杂度由`O(V*Σn[i])`改进到`O(V*Σlog n[i])`的过程，还知道了存在应用超出NOIP范围的知识的O(VN)算法。希望你特别注意“拆分物品”的思想和方法，自己证明一下它的正确性，并将完整的程序代码写出来。


