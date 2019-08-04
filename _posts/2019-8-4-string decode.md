---
layout:     post
title:      "字符串解码问题汇总"
subtitle:   " \"字符串解码问题汇总\""
date:       2019-8-4 16:34:00
author:     "Xu"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Algorithm
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"> </script>



## [一天一题]—— 字符串解码问题汇总
> Auther: xuwang
</br>
> Mail: xuwang.me@gmail.com
</br>
> Created Data: 2019/05/06  &emsp; Modified Data: 2019/05/06

### 难度: Medium

#### 1. 题目1

> LeetCode 394. Decode String 解码字符串
>
> 给定一个经过编码的字符串，返回它解码后的字符串。
>
>编码规则为: k[encoded_string]，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
>
>你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
>此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 3a 或 2[4] 的输入。
>
>示例:
>
>s = "3[a]2[bc]", 返回 "aaabcbc".
>
>s = "3[a2[c]]", 返回 "accaccacc".
>
>s = "2[abc]3[cd]ef", 返回 "abcabccdcdcdef".

#### 2. 题目1 解题思路

##### 2.1 解法一：利用递归（函数栈）来实现

```java
class Solution {
    public String decodeString(String s) {
        StringBuilder sb = new StringBuilder(s);
        StringBuilder res = helper(sb);
        return res.toString();

    }

    public StringBuilder helper(StringBuilder sb){
        StringBuilder res = new StringBuilder();
        char c;
        int k = 0;
        while (sb.length() != 0){
            c = sb.charAt(0);
            sb.delete(0,1);
            if (c >= '0' && c <= '9'){
                k = k * 10 + c - '0';
            }else if (c == '['){
                StringBuilder tmp = helper(sb);
                while (k != 0){
                    res.append(tmp);
                    k--;
                }
            }else if (c == ']'){
                return res;
            }else {
                res.append(c);
            }
        }
        return res;
    }
}
```

##### 2.2 解法二：自己利用栈来实现，而不是函数栈

思想：利用两个栈，一个数字栈，一个字符串栈。
```java
class Solution {
  public String decodeString(String s) {
        StringBuilder sb = new StringBuilder(s);
        StringBuilder res = new StringBuilder();
        Stack<Integer> nums = new Stack<>();
        Stack<StringBuilder> strs = new Stack<>();
        int k = 0;
        char c;
        while (sb.length() != 0) {
            c = sb.charAt(0);
            sb.delete(0, 1);
            if (c >= '0' && c <= '9') {
                k = k * 10 + c - '0';
            } else if (c == '[') {
                strs.push(res);
                nums.push(k);
                k = 0;
                res = new StringBuilder();
            } else if (c == ']') {
                String cp = res.toString();
                k = nums.pop();
                while (k > 1) {
                    res.append(cp);
                    k--;
                }
                k = 0;
                StringBuilder pre = strs.pop();
                pre.append(res);
                res = pre;
            } else {
                res.append(c);
            }
        }
        return res.toString();
    }
}
```

#### 3. 题目2：重复的次数放在括弧后面，而不是前面
> 给定一个经过编码的字符串，返回它解码后的字符串。
>
>编码规则为: [encoded_string]k，表示其中方括号内部的 encoded_string 正好重复 k 次。注意 k 保证为正整数。
>
>你可以认为输入字符串总是有效的；输入字符串中没有额外的空格，且输入的方括号总是符合格式要求的。
>
>此外，你可以认为原始数据不包含数字，所有的数字只表示重复的次数 k ，例如不会出现像 a3 或 [4]2 的输入。
>
>示例:
>
>s = "[a]3[bc]2", 返回 "aaabcbc".
>
>s = "[[a]3c]2", 返回 "accaccacc".
>
>s = "[abc]2[cd]3ef", 返回 "abcabccdcdcdef".

#### 4. 题目2解题思路

直接反向求解即可


