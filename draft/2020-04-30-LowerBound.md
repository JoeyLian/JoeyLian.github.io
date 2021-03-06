---
title: LowerBound
date: 2019-11-03
categories:
- DAA
tags:
- Lower_Bound
- 算法下界
typora-copy-images-to: ..\images
---

<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>


Lower Bound 说明了对于相关问题我们无法找到更优的算法。得到一个问题的 Lower Bound主要优有两种方法：

+ Decision tree 
+ Adversary (Oracle) 

需要注意的是， Lower Bound 只说明了一个下界，我们证明了无法找到更优的算法，但不一定存在复杂度为 Lower Bound 的算法。

## Decision tree

Decision tree 的主要想法是将所有可能的结果作为树的叶子，树的高度是需要进行判断的次数。任意行为产生的结果作为一个分支。需要知道的是，在叶结点数确定时，平衡树的高度最低。因此构造一棵平衡树可以得到最少的判断次数，但不可能更少：所有的结果都有可能。

### 有序数列中查找

有一个数列$\{A_n\}$, $A_1 < A_2<...<A_n$, 给定 $x$, 问 $x$ 是否在$\{A_n\}$ 中，要求给出 $i$ 或者说明  $x$ 不在数列中。

每次比较可以得到三种结果 ： < = >

+ 当结果为 = 是，得到一个叶节点：输出问题答案
+ 当结果为 < 或 > 时，得到一个分支，进行下一次比较

![image-20200430174720082](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200430174720082.png)

每个内部节点有两个不是叶子的分支和一个叶节点。因此，在第 $k$ 层最多有 个 $2^k$ 节点。$k$ 层的决策树的叶节点树为
$$
n \le 2^0 + 2^1+...+2^{k-1} = 2^k-1
$$
需要比较的次数即为决策树的高度：
$$
2^k-1 \ge n\\
k \ge \log_2(n+1)
$$
因此，有序数列中查找的一个 Lower Bound 为 $\lceil log_2(n+1) \rceil$ ，这也是二分查找的时间复杂度。

### 排序问题

先找一个 Lower Bound：

每次比较会形成两个分支。对于排序问题，比较的结果只需要分为大于和小于等于（或 大于等于和小于）（相等的两个元素可以随意排列）叶节点都在最后一层，当决策树有 $k$ 层时，有 $2^k$ 个叶节点。对于 $n$  个元素的排序问题， 一共有 $A_n^n= n!$ 种可能的结果。因此，需要比较的次数即为决策树的高度：
$$
2^k \ge n!\\
k \ge \log_2(n!)
$$
因此，排序问题的一个 Lower Bound 为 $\lceil log_2(n!) \rceil$ 

一个常见的排序方法是插入排序，由于每次插入操作都是一次有序数列中的查找操作，因此他的时间复杂度为 $\sum_{k=1}^n \log_2k$ . 在 $n=5$ 时， 插入排序需要 8 次比较，而前面我们得到的下界为 $\lceil log_2(5!) \rceil =7$ 次比较。

**五个元素的排序**

+ 比较 $(a_1, a_2)$ 中的最大值和 $(a_3,a_4)$ 中的最大值 （3次排序）

  ![image-20200511210234796](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200511210234796.png)

+ 将元素 $y$ 插入到其中：先与 $a_2$ 比较，再与 $a_1$ 或 $a_4$ 比较（2次比较）

  ![image-20200511210512589](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200511210512589.png)

+  将元素 $x$ 插入到其中：先与 $\beta_2$ 比较，再与 $\beta_1$ 或 $\beta_3$ 比较（2次比较）



## Adversary strategy

对任意的算法，它的复杂度指的是对任何实例的最坏运行时间。Adversary strategy 使得对任意算法最坏的情况出现，通过根据算法动态调整实例以使算法花费较长时间，从而给出算法的一个 Lower Bound。

### Find x

一个简单的例子，在一个无序的数列中查找一个元素，我们只需要对前 $n-1$ 次询问都回答 *否*，就可以使算法花费时间更多。因此在一个无序的数列中查找元素，至少需要 $n$ 次询问（询问数列中的每个数是否为查找的元素）

### Find Max

在 $n$ 个元素的数组中寻找最大值，至少需要 $n-1$ 次比较：

如果少于 $n-1$ 次比较，则比较图（就是上面那种用箭头表示大小关系的图）是不连通的：$n$ 个顶点，少于 $n-1$ 条边。

比较图不连通，因此我们可以修改图中点的值使得最大值可以改变。如将一个连通分量中的每个数据都增大或减小一个数，连通分量中点的大小关系不变，但数组中的最大值可以改变。

### Find Second Maximum

（我觉得这个 Lower bound 不是很严谨但是不知道为什么（

找到第二大的元素的时候必须知道最大的元素，因此至少需要 $n-1$ 次比较。

第二大的元素一定直接和最大的元素比较过 (是一个direct  loser)：否则，存在另一个元素大于 "第二大的元素" 而小于最大的元素，矛盾。

对于任意的算法，可以通过 Adversary strategy 使得最大的元素的 direct  losers 最多：对于每次 x,y 之间的比较，令 losers (即所有已知的比它小的元素) 更多的元素更大，这样，每次直接比较最多使得它的 losers 变成原来的两倍，$k$ 次直接比较最多有 $2^k$ 个losers。要得到最大元素，至少需要 $k$ 次直接比较：
$$
2^k \ge n\\
k \ge \lceil log_2n\rceil
$$
第二大的元素即为这 k 个 $direct  losers$ 中的最大值，至少需要 $k-1 = \lceil log_2n\rceil-1$ 次比较。

因此，找到第二大的元素的算法的一个 Lower bound 为：$n-1+\lceil log_2n\rceil-1 = n+\lceil log_2n\rceil-2$

达到这个 Lower bound 的一个算法为：

![image-20200511223330368](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200511223330368.png)

97 是最大元素，第二大元素则是绿色圈出来的元素的最大值。

### Find Median

寻找一个数列中的中位数。（Adversary strategy 终于开始奇幻了（x

设有一个n个元素的数列，中位数指第 $(n+1)/2$ 大的数。假设数列中的元素都不相同。

要得到中位数，需要有数列中的每个数与中位数的大小关系：若有一个元素与已知的中位数的大小关系未知，中位数无法同时满足在该元素比它大或比它小时仍是中位数的条件。可以通过直接或间接的比较：

Crucial comparison（下图中的实线部分）：

+ $x > y$ for some $y \ge median$
+ $x < y$ for some $y \le median$

Crucial comparison 得到元素和中位数的大小关系，为了得到中位数，至少需要进行 $n-1$ 次。

但在还不知道中位数时，有一些比较是 Non-crucial 的（图中的虚线部分）

![image-20200514191104532](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200514191104532.png)

Adversary strategy 的主要思想，就是对于任何的比较方法，定义一个实例使得 Non-crucial comparison 的数量较多。

下面证明对任意的算法，存在一个实例使得至少需要 $(n-1)/2$ 次 Non-crucial comparison 。

定义元素的标签：

+ N: no value
+ L: large value，表示比中位数大
+ S: small value，表示小于中位数

初始化时所有元素均为 N，对于最初的比较，按一下规则生成一个实例：

+ N 与 N 比较时，令一个元素为 L，一个元素为 S
+ N 与 L 比较时，令 N 的元素为 S
+ N 与 S 比较时，令 N 的元素为 L

知道数列中有 $(n-1)/2$ 个 L (或 S)，则不再进行上述规则，而是让所有的 N 都为 S (或 L)。直到剩下的一个数为中位数。

可以看到，按照上述规则与 N 的元素进行的比较都是 Non-crucial comparison。至少需要进行  $(n-1)/2$ 次。

因此对任意的算法，存在一个实例使得至少需要 $(n-1)/2$ 次 Non-crucial comparison 。

寻找中位数算法的一个 Lower bound 为 $n-1+(n-1)/2 = 3(n-1)/2$

### Max & Min 

找到数列的最大值和最小值。用分治的方法可以得到时间复杂度为 $3n/2-2$ 的算法。下面用 Adversary strategy 证明这是它的 Lower bound 

定义元素的标签：

+ N: no compare
+ W: at least one win, never lost 
+ L: – at least one lost, never won 
+ WL: at least one win and one lost 

要得到数列的最大值和最小值，就是得到标签为 W 和 L 的元素，而其他的元素都应该是 WL：否则若存在两个元素标签为 W，无法确定谁是最大值.

初始化时所有元素均为 N，为了得到最终结果，至少需要 $2n-2$ 个信息 (WL 算两个)

当两个标记为 N 的元素比较时，一次比较可以得到两个信息，最多只能有 $n/2$ 次，得到 $n$ 个信息

对于其他情况，都可以通过 Adversary strategy 得到一个实例使得只增加一个信息：

+ N 与 W 比较时，令 N < W，则 N 变为 L，W不变
+ W 与 W 比较时，一个变为 WL，另一个不变
+ W 与 L 比较时，令 W > L, 不会增加信息：这是可以做到的，W的信息限制了它需要大于某个数，而 L 的信息限制它需要小于某个数，总是可以取得两个值使得 W > L

因此，至少还需要进行 $2n-2-n$ 次比较。

因此，找到 max&min 的一个 Lower bound 为 $n/2+2n-2-n = 3n/2-2$

#### Find X in a Sorted Matrix

有序的矩阵满足：

![image-20200514201640276](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200514201640276.png)

需要查找元素 X 是否在矩阵中：

一个简单的算法是从右上角开始查找，当矩阵中的元素大于 X 时，往左移动；当矩阵中的元素小于 X 时，往下移动。可以验证算法是正确的，且时间复杂度为 $2n-1$

下面说明 $2n-1$ 是这个问题的一个 Lower bound 。在下面的矩阵中，任何一个等于6或8的元素均可以改为 7，而不改变矩阵有序。

![image-20200514201917612](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200514201917612.png)

因此如果查找 7 这个元素，至少需要把所有等于 6 和 8 的元素访问过，需要 $2n-1$ 次比较。

### Merge Sorted Lists

将两个有序的数列合并为一个有序数列。

这也是一个简单问题，将数列遍历一遍依次比较即可。需要 $2n-1$ 次比较。

下面说明 $2n-1$ 是这个问题的一个 Lower bound 。

假设存在一个算法的时间复杂度为 $2n-2$ .

假设有两个数列 $X: X_i = 2_i-1$,  $Y:Y_i=2i$, $i = 1,...,n$。则合并生成的数列为 $X_1, Y_1, X_2, Y_2 ,..., X_n, Y_n$ 

由于进行了 $2n-2$ 次比较，至少存在一个 $X_i$，没有同时和 $Y_i, Y_{i+1}$ 比较过 (对于 $X_1$，只需要和 $Y_1$ 比较)。不妨假设 $X_i, Y_i$ 没有比较过

令 $X_i=2i, Y_i=2i-1$，则算法会给出和原来一直的结果，但不是正确结果

因此， $2n-1$ 是这个问题的一个 Lower bound 。

### Graph Connectivity

给出图的邻接矩阵，判断图是否连通。问是否需要访问邻接矩阵中的每个元素才能确定是否连通。显然在一些实例中不需要访问每个元素，但我们可以证明，对任意的算法，存在实例使得必须访问每个元素才能确定是否连通。即该问题的 Lower bound 为 $n(n-1)/2$

Adversary strategy 构造出一个连通图。在算法的运行中维持两张图 Y(Yes) 和 M(Maybe): Y 表示确定在实例中的边，M包含了在实例中的边和还没有被访问过的边。

初始化 Y 为空，M 为完全图。

对任意对边 e = (i,j) 的访问，如果将 e 移出图 M 后 M 仍连通，则返回 0，并将其移出 M；否则返回 1 并加入到 Y 中。

可以验证需要访问到最后一条边才能确定图连通。

假设 M 中还有唯一一条边 e =(i,j) 未访问，证明若访问结果为 0，则构造的实例不连通；若访问结果为 1，则构造的实例连通，即最后一条边确定了问题答案。由于其他边都已访问过，若存在另一条 i 到 j 的路径，对于路径上的其中一条边 e' = (i',j'), 访问时将其移出图 M 后 M 仍连通，因此访问的结果是 0，即需要从图 M 中移除这条边。因此，不存在另一条 i 到 j 的路径。（有一点点绕 QAQ

### Find Patterns in Bit Strings

查找一个有 n 个字符的 bit 串中是否含有子串 01。求问任意的算法最坏情况是否需要对每个字符进行查询。

当 n 为奇数时，可以通过一些小 tips 减少至少一次的访问：先访问偶数位上的字符 B(2), B(4),...,B(n-1)

+ 如果存在 $j > i, B(2i) = 0, B(2j) = 1 $, 无需继续访问，一定存在字符 01 
+  如果任意的 $i, B(2i) = 0$ ，无需访问 B(1)
+  如果对任意的 $i, B(2i) = 1$ ，无需访问 B(n)
+ 如果存在 $k, B(2i) = 1, i < k, B(2j) = 0, j\ge k$ ，无需访问 B(2k-1)

当 n 是偶数时，可以证明存在实例使得需要访问到最后一个字符才能确定是否存在子串 01。

Adversary strategy 构造一个不含有子串01的序列 111,...,10,...,000。在算法中维持两个编号 s<t

![image-20200515093131729](C:\Users\13353\OneDrive\note\JoeyLian.github.io\images\image-20200515093131729.png)

+ 所有小于 s 的字符需要被查询，因为他们可能为 0 
+ 所有大于 t 的字符需要被查询，因为他们可能为 1
+ 当 s 和 t 直接有偶数个字符时，中间的每个数都需要被查询

初始化 s 为0，t 为 n+1。Adversary strategy 维持 s 和 t 之间始终有偶数个字符 (即 t-s-1为偶数)：对任意对第 i 个字符的查询，构造实例：

+ i  < s, B(i) = 1
+ i > t, B(i) = 0
+ s < i < t, i-s 是偶数， B(i) = 1, s = i
+ s < i < t, i-s 是奇数， B(i) = 0, t = i

因此，序列中的任何一个元素需要被访问才能确定字符串中不包含子列 01。该问题的 Lower bound 为 n。



