---
layout:     post
title:      "二叉树形态个数 II"
subtitle:   " \"二叉树的形态个数 II\""
date:       2019-8-1 21:54:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Algorithm
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## [一天一题]—— 二叉树形态个数 II
> Auther: xuwang </br>
> Mail: xuwang.me@gmail.com </br>
> Created Data: 2019/08/01  &emsp; Modified Data: 2019/08/01
---

### 难度：Hard

#### 1. 题目

> 给定二叉树节点个数N，和二叉树高度K，求解可能存在的二叉树形态的个数
>
> 注：只包含一个根节点的二叉树高度视为1



#### 2. 解题思路

这道题如果直接想要对二叉树采用先序/后序/中序遍历构建的方式进行计算几乎是不可能的。

可以换一个思路，计算二叉树的每一层的可能情况进行统计，假设每一层的情况个数是p(i)，那么total = p(1) * p(1) * .... * p(k)

因此解法如下：

```java
public class Solution {
    public static int total = 0;
    public static void main(String[] args){
        solve(7, 4);
    }

    public static void solve(int n, int k){
        int total = helper(n-1, 1, k, 1);
        System.out.println(total);
    }

    public static int helper(int n, int height, int k, int preNuM){
        if (height == k) return 0;
        if (height == k-1){
            if (n > 2 * preNuM || n == 0) return 0;
            return C(2 * preNuM, n);
        }
        int total = 0;
        for (int i = 1; i <= 2 * preNuM; i++){
            total += C(2 * preNuM, i) * helper(n - i, height + 1, k, i);
        }
        return total;
    }

    public static int A(int n, int m)
    {
        int result = 1;
        for (int i = m; i > 0; i--)
        {
            result *= n;
            n--;
        }
        return result;
    }
    public static int C(int n, int m)
    {
        m = m > n/2 ? n-m : m;//因为C(3,1) = C(3,2),计算复杂度更低的
        int numerator = A(n, m);
        int denominator = A(m, m);
        return numerator / denominator;
    }
}
```

