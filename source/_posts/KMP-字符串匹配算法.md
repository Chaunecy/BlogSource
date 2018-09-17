---
title: KMP 字符串匹配算法
date: 2018-09-14 18:55:17
categories:
	- 算法
tags:
	- KMP
	- 数据结构与算法
---

## 什么是 KMP 算法

**Knuth-Morris-Pratt 字符串查找算法**（简称为 **KMP 算法**）可在一个 **主文本字符串** `S` 内查找一个 **词** `W` 的出现位置。此算法通过运用对这个词在不匹配时本身就包含足够的信息来确定下一个匹配将在哪里开始的发现，从而避免重新检查先前匹配的字符。

## 暴力算法

主文本字符串 S：“ABCABD”

待匹配字符串 W：“ABD”。

### 基本思路

第一个字符：匹配。

```
Step 1:
S: ABCABD
W: A
```

第二个字符：匹配。

```
Step 2:
S: ABCABD
W: AB
```

<!-- more -->

第三个字符：**不匹配**。

```
Step 3:
S: ABCABD
W: ABD
```

这时，根据我们的暴力匹配方法，我们对 W 作适当的移动，重新匹配：

```
Step 4:
S: ABCABD
W:  A
```

仍 **不匹配**。

```
Step 5:
S: ABCABD
W:   A
```

找到匹配的字符。

```
Step 6:
S: ABCABD
W:    A
```

```
Step 6:
S: ABCABD
W:    AB
```

**完全匹配**！

```
Step 6:
S: ABCABD
W:    ABD
```

### 代码实现

以下为该暴力算法的 Java 代码实现：

```java
    public int strMatch(String S, String W) {
        char[] s = S.toCharArray();
        char[] w = W.toCharArray();
        int i = 0, j = 0;
        int k = 0;
        while(i < s.length && j < w.length) {
            if (s[i] == w[j]) {
                i++;
                j++;
            } else {
                k++;
                i = k;
                j = 0;
            }
        }
        if (j == w.length) {
            return k;
        } else {
            return -1;
        }
    }
```

### 小结

我们可以看到，Step 3 到 Step 4 的过程其实是没有必要的。原因如下：

```
Step 3:
S: ABCABD
W: ABD
```

待匹配字符串对我们是**已知**的，主文本字符串是**未知**的；在 Step 3 中我们得知，S 的第三个字符与 W 的第三个字符不等，也就是 S 的前两个字符与 W 的前两个字符**相等**；W 的前两个字符是 AB，其中 `A != B`。

根据以上条件，我们知道，Step 4 中必定会造成 **不匹配**，所以我们可以直接跳到 Step 5。

然而暴力算法不会利用这些已知条件（S 中的前两个字符与 W 中的前两个字符相等），所以性能较差，效率不高。

## 如何利用已知条件

主文本字符串 S：待定

待匹配字符串 W：ABCABE

`S[i]` 表示 S 中的第 `i` 个字符，以 0 为开始；`W[j]` 表示 W 中的第 `j` 个字符，以 0 为开始。

假设不匹配情况发生在 W 的第 `j` 位。每一次有不匹配情况发生，我们就知道了 S 中的 `j - 1` 个字符。这些字符可以作为已知条件，来优化算法。

### 不匹配发生在第 0 位

```
S: BBCABE
W: A
j: 0
```

此时 `S[i] != W[0]`，需要同时修改 `i` 和 `j`： `i = i+1, j = 0`。

### 不匹配发生在第 1 位

```
S: AD
W: AB
j: 01
```

W 中第 0 位 A 与第 1 位 B 不相等，但是 W 中的第 0 位 A 可能等于 S 中的第 1 位 D（如果把 D 替换成 A 的话），所以这里只修改 j：`j = 0`。

继续比较，发现第 0 位不匹配：

```
S: AD
W:  A
j:  0
```

### 不匹配发生在第 2 位

```
S: ABDAB
W: ABC
j: 012
```

我们知道 `W[0] != W[1]`，也知道 `S[0..1] == W[0..1]`（`S[m..n]` 表示字符串 S 从 m 到 n 的 `n - m + 1` 个字符，闭区间），同时 `W[0] != W[2]`。

因此，依然只需修改 j：`j = 0`，因为有可能 `W[0] == S[2]`。

```
S: ABDAB
W:   A
j:   0
```

### 不匹配发生在第 3 位

```
S: ABCDDD
W: ABCA
j: 0123
```

注意，现在出现了 `W[0] == W[3]`，而 `S[3] != W[3]`，所以必定有 `W[0] != S[3]`。

也就是说，要同时修改 i 和 j 才能充分利用上述已知条件：`i = i + 1, j = 0`。

```
S: ABCDDD
W:     A
j:     0
```

### 不匹配发生在第 4 位

```
S: ABCAD
W: ABCAB
j: 01234
```

`W[4] != S[4] && W[1] == W[4]` => `W[1] != S[4]`。即无法按下述方法移动：

```
S: ABCAD
W:    ABCAB
j:    01234
```

对 j 的移动应该是：`j = 0`。

```
S: ABCAD
W:     ABCAB
j:     01234
```

### 不匹配发生在第 5 位

```
S: ABCABF
W: ABCABE
j: 012345
```

W 中的字符 `W[5] = E` 与 W 中其他字符都不相同，包括 `W[2] = C`。所以，我们可以推测，有可能 `W[2] = S[5]`。

对 j 的移动应该是：`j = 2`。

```
S: ABCABF
W:    ABCABE
j:    012345
```

### 小结

通过上述分析，我们可以发现一个规律：如果 W 中存在 **重复** 的子字符串，那么我们可以 **复用** 这些重复的子字符串。比如不匹配发生在 3、4、5位时。

## 由表及里： j 移动的规律

### 必要的概念

W 表示一个字符串，`W[i..j]` 表示 W 的子字符串，从 `W[i]` 开始到 `W[j]`，闭区间，其中`i >= 0, j <= W.length - 1`，`W.length` 表示 W 的长度。

**前缀**（prefix）：`W[0, j], j < W.length - 1`。

**后缀**（suffix）：`W[i, W.length - 1], i > 0`。

**最长匹配**：存在最大下整数 k，使得 `W[0, k - 1] = W[len - (k - 1), len]`，其中 `len = W.length - 1`。

### 最长匹配表

给定一个字符串 W，计算它 **每一个前缀** 及 **本身** 的最长匹配，得到的数据可以组成最长匹配表。

以 `W = ABCABE` 为例，求它的最长匹配表：

对于子字符串 `A` 来说，它的前缀与后缀都为空，所以最长匹配为 0。

对于子字符串 `AB` 来说，它的唯一前缀 `A` 与唯一后缀 `B` 不相等，所以最长匹配也为 `0`。

对于子字符串 `ABCA` 来说，它的前缀有 `A`，`AB`，`ABC`；后缀有 `A`，`CA`，`BCA`。最长匹配为 1。

相应地可以求出其他的子字符串的最长匹配，得到下表：

| char  |  A   |  B   |  C   |  A   |  B   |  E   |
| :---: | :--: | :--: | :--: | :--: | :--: | :--: |
| index |  0   |  1   |  2   |  3   |  4   |  5   |
| match |  0   |  0   |  0   |  1   |  2   |  0   |

### 如何使用最长匹配表

如果不匹配发生在第 0 位，则直接将 W 右移一位即可。

如果不匹配发生在第 `i` 位，`i > 0`，则将 W 向右移 `matched_length - table[matched_length - 1]` 位。比如已经匹配了 4 个字符，即 `matched_length = 4`，则将 W 向右移 `4 - table[4 - 1]` 个字符。

当不匹配发生在第 4 位时，也就是 `matched_length = 4`，

```
S: ABCAD
W: ABCAB
j: 01234
```

我们把 W 向右移 `4 - table[3] = 3` 位，

```
S: ABCAD
W: ---ABCAB
j: ---01234
```

噫？不是应该右移 4 位，变成

```
S: ABCAD
W: ----ABCAB
j: ----01234
```

吗？怎么只移动了 3 位，这不是要多一个步骤吗？

别急，这只是基本的实现，之后会有优化版本，实现一步到位。

### 举例说明

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
W: ABCABE
```

第 1 位即不匹配，直接右移。

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
    |
W:  ABCABE
```

不完全匹配字符为 1，右移 `1 - table[0] = 1`位。

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
    X
W:   ABCABE
```

下一次不完全匹配发生在

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
      ||
W:    ABCABE
```

右移 `2 - table[1] = 2` 位。

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
      XX
W:      ABCABE
```

 当不完全匹配数为 5 时，

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
                  |||||
W:                ABCABE
```

右移 ` 5 - table[4] = 5 - 2 = 3`。

```
S: BA AB ABC ABCA ABCAB ABCABD ABCABE
                  XXX||
W:                   ABCABE
```

## 失配指针：next 数组

相比于待匹配字符串 W 向右移动多少位，我们更加关注 j 的值应该怎么变化。而 next 数组就是用来记录 `S[i] != W[j]` 时，如何修改 j 的值的。

当发生 `S[i] != W[j]` 时，我们有已匹配字符数 `matched_length = j`，此时 W 向右移 `shift_right = j - table[j - 1]` 位。此时 j 的值变为 `j - shift_right = table[j - 1]`。

也说是说，next[j] = table[j - 1]，表示当第 j 位失配时，应该把 j 的值修改为 next[j]，即 table[j - 1]，实现“相对右移” W 字符串。

不过这里有一个问题，就是 `next[0] = table[0 - 1] = table[-1]` 要怎么处理？

还记得 **如何使用最长匹配表** 时怎么做的吗？当在 `j = 0` 处失配时，我们直接右移了 W。其实可以这样理解（不一定对）：

| char  |/  |  A   |  B   |  C   |  A   |  B   |  E   |
| :---: |:--: | :--: | :--: | :--: | :--: | :--: | :--: |
| index |-1|  0   |  1   |  2   |  3   |  4   |  5   |
| match |-1|  0   |  0   |  0   |  1   |  2   |  0   |

table[-1] = -1，当在 j = 0 失配时，`matched_length = 0`，右移 `0 - table[-1] = 0 - (-1) = 1` 位。

这样我们就完成了 table 数组 向 next 数组的转换，即

```
next[j] = table[j - 1], j > 0
next[j] = -1,           j = 0
```

## 如何求 next 数组

### 基本思想

求 next 数组的过程，其实就是求最长匹配的过程。

首先是代码的基本思路。

next[0] = -1，可以直接硬编码。如果 `W.length > 1`，也必定有 `next[1] = table[0] =  0`。

对于 next[j]，我们要求出 table[j - 1]。这里有两种情况。

- 假如 next[j] = k，那么如果有 ` W[j] == W[k]`，则有 next[j + 1] = (k + 1)。

  ```
  ABC-__ABC-_
     |     |
     k     j
  ```

- 假如 next[j] = k，但是 `W[j] != W[k]`，那么，需要 **重新确定最长匹配**。

  我们知道 k 表示 `W[0..j-1]` 的最长匹配，即`W[0..k-1] == W[j-k..j-1]` 成立（即下图中的 `ABA == ABA`）。

  这时，我们要使用 `W[0..k-1]` 中的最长匹配，即 `next[k]`，再去比较 `W[j]` 与 `W[k]`。（因为 `ABA/ != ABA*`，但是 **有可能** `AB == A*` 成立）。

  ```
  ABA/__ABA*_
     |     |
     k     j
  ```

简单来讲，即 **已知 `next[j] = k`**：

```pseudocode
next[0] = -1
k = next[0]
j = 0
// k = -1 时，表明有 next[1] = table[0] = 0，
// 或者无法通过 k = next[k] 复用之前的结果
while j < W.length - 1
	if k == -1 || W[j] == W[k]
		next[++j] == ++k
	else
		k = next[k]
```

对应的 Java 代码如下：

```java
public int[] getNext(String W) {
    char[] p = W.toCharArray();
    int[] next = new int[p.length];
    next[0] = -1;
    int j = 0;
    int k = -1;
    
    while (j < p.length - 1) {
       if (k == -1 || p[j] == p[k]) {
           next[++j] = ++k;
       } else {
           k = next[k];
       }
    }
    
    return next;
}
```

### 实例

举个例子来说明一下：

```
W: ABACABADABACABAEA
// 为方便比较查看，添加了连字符
W: AB-AC-ABAD-ABACABAE-A

W: A B A C A B A D A B A  C  A  B  A  E  A
j: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16
```

下面是求 next 数组的步骤：

|  Step  | 分支 |        k         |  j   |     next     |
| :----: | :--: | :--------------: | :--: | :----------: |
| 初始化 |      |        -1        |  0   | next[0] = -1 |
|   1    |  if  |        0         |  1   | next[1] = 0  |
|   2    | else | k = next[0] = -1 |  -   |      -       |
|   3    |  if  |        0         |  2   | next[2] = 0  |
|   4    |  if  |        1         |  3   | next[3] = 1  |
|   5    | else | k = next[1] = 0  |  -   |      -       |
|   6    | else | k = next[0] = -1 |  -   |      -       |
|   7    |  if  |        0         |  4   | next[4] = 0  |
|   8    |  if  |        1         |  5   | next[5] = 1  |
|   9    |  if  |        2         |  6   | next[6] = 2  |
|   10   |  if  |        3         |  7   | next[7] = 3  |
|   11   | else | k = next[3] = 1  |  -   |      -       |
|   12   | else | k = next[1] = 0  |  -   |      -       |
|   13   | else | k = next[0] = -1 |  -   |      -       |
|   14   |  if  |        0         |  8   | next[8] = 0  |
|  ...   | ...  |       ...        | ...  |     ...      |

最后求出的 next 数组为：

```
      j:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16  
next[j]: -1  0  0  1  0  1  2  3  0  1  2  3  4  5  6  7  0
```

## 优化 next 数组

在最长匹配表一节中，我们就知道这种方法求出的 next 数组是不够高效的（见：如何使用最长匹配表）。

如何实现下面这种效果，减少一步操作，就是这一节的主题了。

```
S: ABCAD
W: ----ABCAB
j: ----01234
```

先看一下为什么会导致“多此一举”的情况发生。

对于 `W = ABCABE` 来说，它的 next 数组是 `-1, 0, 0, 0, 1, 2`。

如果当 j = 4 时 发生不匹配，那么对 j 进行修改：

```
S: ABCAD
W: ---ABCAB
j: ---01234
       | 新的 j = 1
```

而很明显地，在新的 `j = 1`处有不匹配发生。这是由于 `W[1] == W[4]`，所以即使更新 j 的值，仍然会发生不匹配。

那么我们就可以对这种相等的情况进行特殊处理，来避免这种额外的开销。

大体思路为：

```pseudocode
while j < W.length - 1
	if k == -1 || W[j] == W[k]
		if W[++j] == W[++k]
			// j = next[next[j]] = next[k]
			next[j] = next[k]
		else
			next[j] = k
	else
		k = next[k]
```

> 额外的开销是指 j = next[j] 操作后立即发生不匹配，需要再一次执行 j = next[j]。
>
> 这里我们直接一步到位，把两次赋值语句合为一句：
>
> `j = next[next[j]] = next[k]`

下面是 Java 版本的代码实现：

```java
public static int[] getNext(String W) {
    char[] p = W.toCharArray();
    int[] next = new int[p.length];
    next[0] = -1;
    int j = 0;
    int k = -1;

    while (j < p.length - 1) {
       if (k == -1 || p[j] == p[k]) {
           if (p[++j] == p[++k]) {
              next[j] = next[k];
           } else {
              next[j] = k;
           }
       } else {
           k = next[k];
       }
    }

    return next;
}
```

## 实现 KMP 算法

```java
public int KMP(String S, String W) {
    char[] s = S.toCharArray();
    char[] w = W.toCharArray();
    int i = 0;
    int j = 0;
    // 只是多了一步求解失配指针
    int[] next = getNext(W);

    while (i < s.length && j < w.length) {
        // j == -1 说明需要： i += 1, j = 0
        if (j == -1 || s[i] == w[j]) {
            i++;
            j++;
        } else {
            j = next[j];
        }
    }
    if (j == w.length) {
        return i - j;
    } else {
        return -1;
    }
}
```

## 时间复杂度分析

> 个人理解，不一定对。

### 求解 next 数组

```pseudocode
while j < W.length - 1
	if k == -1 || W[j] == W[k]
		if W[++j] == W[++k]
			next[j] = next[k]
		else
			next[j] = k
	else
		k = next[k]
```

只有 `k != -1 && W[j] != W[k]` 时，j 的值才 **不会增加**。所以，想要尽可能多地执行，必须保证足够多的不重复元素。但是对于不重复元素，它的回溯是直接回溯到 0 的，因为 next[j] = table[j - 1]，而 W[0..j - 2] 的最大匹配是 0。所以，最坏情况是 **最长匹配表** 中所有的值都为 0，这时的上界约为 2n，n 为待匹配字符串的长度。

所以其时间复杂度为 $O(n)$。

### KMP 算法

因为 i 不回溯，只有 j 会回溯，当某个字符匹配成功时，i++，j++，失败时待匹配字符向前跳过 j - next[j] 个字符。

最坏情况下，在 m - n（n 表示待匹配字符串 W 的长度）处完成匹配，其时间复杂度不超过 2m，加上 next 数组的时间复杂度，KMP 算法的时间复杂度为 $O(m + n)$。

## 总结

通过手动或者单步调试来仔细地观察算法每一步的输出、变量的值的改变，从中总结规律，不失为学习的一个好方法。

## 参考资料

1. [克努斯-莫里斯-普拉特算法](https://zh.wikipedia.org/zh-hans/%E5%85%8B%E5%8A%AA%E6%96%AF-%E8%8E%AB%E9%87%8C%E6%96%AF-%E6%99%AE%E6%8B%89%E7%89%B9%E7%AE%97%E6%B3%95)

2. [The Knuth-Morris-Pratt Algorithm in my own words](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)

3. [字符串匹配的KMP算法](http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)

4. [（原创）详解KMP算法](https://www.cnblogs.com/yjiyjige/p/3263858.html)

