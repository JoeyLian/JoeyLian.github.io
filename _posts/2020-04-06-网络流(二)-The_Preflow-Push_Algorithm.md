---
title: 网络流(二)-The_Preflow-Push_Algorithm
date: 2020-04-06
categories:
- 算法设计
tags:
- Network Flow
- 最大流问题
- Preflow-Push 算法
- Push-Relabel 算法

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


## 基础定义

#### Flow Networks (流网络)

流网络指一个符合以下特征的有向图 G = (V,E):   

+ 每条边关联一个非负数，即边的容量，记为 $c_e$ 
+ 存在单一源点 (source node) $s\in V$：产生流量的点
+ 存在单一汇点 (sink node) $t\in V$：吸收流量的点

一般的，我们假设流网络满足：没有边进入源点，没有边离开汇点。每个结点至少存在一条边与之相连。

#### Flow (流)

s-t 流是一个函数 $f: E\rightarrow \mathbb{R}^+$, 值 $f(e)$ 表示边 $e$ 携带的流量。一个流需要满足两个性质  

+ 容量条件 (Capacity conditions): $$\forall e\in E, 0\le f(e)\le c_e$$
+ 守恒条件 (Conservation conditions): $$\forall v\in V, v\neq s,t, \sum_{e\text{ into }v}f(e) = \sum_{e\text{ out of }v}f(e)$$

为了使记号简洁，我们定义 

$$
f^{\text{out}}(v) = \sum_{e\text{ out of }v}f(e),\  f^{\text{in}}(v) = \sum_{e\text{ into }v}f(e)
$$

自然的，对于点集 $S\subseteq V, f^{\text{out}}(S) = \sum_{e\text{ out of }S}f(e),\  f^{\text{in}}(S) = \sum_{e\text{ into }S}f(e)$。

一个流 $f$ 的值, 记作 $v(f)$，定义为源点产生的流量:

$$
v(f) = \sum_{e\text{ out of } s}f(e) = f^\text{out} (s)
$$

#### The Residual Graph (剩余图)

 给定流网络 $G$ 及 $G$ 上的流 $f$， 定义 $G$ 关于 $f$ 的剩余图 $G_f$ ：  

+ $G_f$ 中的顶点集与 $G$ 中的顶点集相同
+ 对于 $G$ 中的每条边 $e = (u,v)$ ，若 $f(e) < c_e$， 则还有 $c_e-f(e)$ 的剩余流量。$G_f$ 中包含边  $e = (u,v)$ ，其容量为  $c_e-f(e)$。这种边称为**前向边** (forward edges).
+ 对于 $G$ 中的每条边 $e = (u,v)$ ，若 $f(e) > 0$， 则流中有 $f(e)$ 的流可以 "撤销"。$G_f$ 中包含边  $e' = (v,u)$ ，其容量为  $f(e)$。这种边称为**后向边** (backward edges).

直观的理解，剩余图定义了在当前流状态下每条边可以操作的流量：在初始流网络中剩余的流量可以继续使用，以及使用的流量则可以 "撤销" ：使用一条反向的边运输流量。

定义一条边的**剩余流量** (residual capacity) 为这条边在剩余图中的流量。

#### The Maximum-Flow Problem (最大流问题)

定义：给定一个流网络：有向图 $G=(V,E)$ 与每条边的容量 $c_e$，找出一个具有最大值的流 $f$



## 算法设计

### 动机

考虑下面的一个流网络，其中 K 是一个很大的数。如果我们使用 Ford-Fulkerson 方法，我们需要找到图中的每条 s-t 路径，在每条路径上通过 1 的流量，时间复杂度为 $\Omega(k^2)$ (每次寻找路径需要 $O(k)$, 有 $k$ 条路径)。然而，直观上看，我们只需要将 k 的流量从 s 发往 x ，然后将这些流量分流到每条支路上，希望在 $O(k)$ 时间内完成。

图源：[CS261: Lecture #3: The Push-Relabel Algorithm for Maximum Flow](http://timroughgarden.org/w16/l/l3.pdf)

![image-20200406113703037](\images\image-20200406113703037.png)

换句话说，我们希望算法将所有可能的流量从源头发出，再逐渐 "推" 到不同的支路，最后将推不出去的流量返回源头，产生一个合法的最大流。类似于 "泛洪" 的想法。

### Preflow 定义

算法将所有可能的流量从源头发出，此时已经不是一个合法的流。因此，我们定义一种 Preflow：s-t Preflow是一个函数 $f: E\rightarrow \mathbb{R}^+$, 值 $f(e)$ 表示边 $e$ 携带的流量。一个 **Preflow** 需要满足两个性质  

+ 容量条件 (Capacity conditions): $$\forall e\in E, 0\le f(e)\le c_e$$
+ 松弛守恒条件 (Conservation conditions): $$\forall v\in V-\{s\},\sum_{e\text{ into }v}f(e) \ge \sum_{e\text{ out of }v}f(e) $$

一个 Preflow 不需要将流入节点的所有流量流出，而是允许暂时储存在节点中，通过后续操作将这些流量流出。节点流入流量与流出流量的差值称为 **Excess (余量)**:

$$
e_f(v) = \sum_{e\text{ into }v}f(e) - \sum_{e\text{ out of }v}f(e) \ge 0
$$

可以看到，当一个 Preflow 中所有节点的余量均为 0 时，这个 Preflow 就是一个合法的流。

### Push (推) 操作

要把一个 Preflow 调整为一个流，就需要将节点中的余量 "推" 到其他节点，最终推到终点。此时的推操作受到两个方面的限制，一是节点自身的余量，二是边可以通过的流量。因此，推操作为：

>Push(v)  
>{  
>    choose an outgoing edge $(v, w)$ of $v$ in $G_f$ (if any)  
>    // push as much flow as possible   
>	If $e=(v, w)$ is a forward edge then   
>		let $$\Delta=\min\{e_f(v), c_e −f(e)\}$$   
>		increase $f(e)$ by $\Delta$   
>	If $e=(v, w)$ is a backward edge then  
>		let $$\Delta=\min\{e_f(v), f(e)\}$$   
>		decrease $f(e)$ by $\Delta$   
>	Return $(f, h)$  
>}

### Height 定义

只有推操作是不够的。考虑从源点将流量推出到节点之后，剩余图中会生成一条从节点到源点的后向边，算法需要保证在推的过程中不会马上将流量推回源点，造成流量推入推出的无穷循环。

在流量一开始流动时，我们希望尽可能的从源点流向终点。想象流网络在一个平面上，希望流量尽可能的从高出流向低处。因此，为图中的每个节点 $v$ 维持一个高度 $\text{height}(v) $ 。我们定义一个**标签 (labeling)** 是一个从点集到非负整数的函数 $h:V\rightarrow\mathbb{Z}^+ $ 。算法需要维持标签 $h$ 与 preflow $f$是**兼容 (compatible)** 的，即满足：

+ $h(s) = n, n = \vert V\vert$ 
+ $h(t) = 0$
+ Steepness conditions:  $h(v)\le h(w)+1, \forall e = (v,w) \in G_f$ 

可以这么理解，我们将源点放在高处，汇点放在低处，对于剩余图中的边，起点不能比终点高太多,(否则它会自动流入。)接下来我们可以得到两个结论：

**定理 1**：如果 s-t preflow $f$ 与标签 $h$ 兼容，那么在剩余图 $G_f$ 中没有 s-t 路径。

**证明**：反证法，如果在剩余图 $G_f$ 中存在一条 s-t 路径 $P$ 。沿着路径 $P$ 的节点为 $s, v_1, ..., v_{k-1}, t$。由于 $h(s)=n, h(t) = 0, h(s) \le  h(v_{1})+1 \le ...\le h({v_k-1}) + k-1\le h(t) + k$, 又有 $P$中最多有 $n$ 个节点，故 $k<n$, 与 $n\le 0+k$ 矛盾。因此剩余图 $G_f$ 中没有 s-t 路径。

**定理 2**：如果 s-t 流 $f$ 与标签 $h$ 兼容，那么 $f$ 是一个最大流。

**证明**：根据定理 1，我们有流 $f$ 的剩余图 $G_f$ 中没有 s-t 路径。根据 [网络流(一)](https://joeylian.github.io/算法设计/2020/04/05/网络流(一)-最大流与最小割问题/) 中的 定理 4 ，有$$f$$ 是 $$G$$ 上的一个最大流。

因此，如果我们一直维持 preflow $f$ 与标签 $h$ 兼容，逐步将preflow $f$ 修改为合法的流，则可以得到一个最大流。

### 算法流程

#### 初始化

>$h(s) = n$  
>$h(v) = 0, \forall v \neq s$  
>$f_e = c_e ,\forall e = (s,v)$   
>$f_e = 0, else$ 

显然 $h(s) = n,\ h(t) = 0$, 令其他节点的高度为 0 是平凡的。为了满足 preflow 与标签兼容，我们需要消除掉所有从源点 $s$ 出发的边，其他的边取 0.

#### 主循环

>Main Loop  
>{  
>**while** there is a vertex $v \neq s, t$ with $e_f (v) > 0$ **do**   
>	choose such a vertex $v$ with the maximum height $h(v)$  
>	// break ties arbitrarily   
>	**if** there is an edge $e = (v, w)$ of $v$ in $G_f$ with $h(v) = h(w) + 1 (\text{or } h(v)>h(w))$   
>	**then**   
>		Push(v)   
>	**else**   
>		$h(v) = h(v)+1$(relabel 操作)  
>**return** f  
>}

### 正确性

只需证明在算法过程中一直维持 preflow $f$ 与标签 $h$ 兼容，当算法终止时，preflow 转化为合法的流，也是所求的最大流。可以验证初始化后  preflow $f$ 与标签 $h$ 兼容。且$h(s ) = n, h(t) = 0$ 的条件始终满足。只需证明算法不会破坏 Steepness conditions：

+ 对于 push 操作，不会更改节点的高度，可能会向剩余图中加入新的后向边 $e = (w,v)$, 由于 $h(v) = h(w) + 1$, 因此一定满足对  $e = (w,v),\ h(w) \le h(v) + 1$
+ 对于 relabel 操作，它只发生在对于所有从 $v$ 出发的边 $$e = (v, w), h(v)< h(w) +1$$。因此  $h(v) = h(v)+1$ 后，依然满足对所有从 $v$ 出发的边 $$e = (v, w), h(v)\le h(w) +1$$。

## 复杂度分析

### Relabel 操作

**定理 3**： 令 $f$ 是一个 preflow，如果节点 $v$ 有余量，那么在剩余图 $G_f$ 中有从 $v$ 到源点 $s$ 的路径。

**证明**：令 $$A$$ 为 $$G_{f} $$ 中存在一条 v-s 路径的所有点 $$v$$ 的集合，$$B = V-A$$，证明所有有余量的点均在 $A$ 中：

设 $e = (u,v)$ 是 $G$ 中的一条边，满足 $$u\in A, v\in B$$, 则有 $$f(e) = 0$$，否则，剩余图中存在一条流量为 $f(e)$ 的后向边  $e' = (v,u)$ ，由于 $$u\in A$$,  $$G_{f}$$ 中存在一条 u-s 路径，将 e' 接上这条路径，则得到$$G_{f} $$ 中一条 v-s 路径，与  $$v\in B$$ 矛盾。  
考虑集合 $B$ 中的余量之和,由于 $s\notin B$, $B$ 中每个节点都有非负的余量

$$
\begin{align}
0\le \sum_{v\in B} e_f(v) &= \sum_{v\in B}(f^{\text{in}}(v)-f^{\text{out}}(v))\\
&= f^{\text{in}}(B)-f^{\text{out}}(B) \\
&=-f^{\text{out}}(B) \\
&\le0\\
e_f(v) = 0,& \forall v \in B
\end{align}
$$

因此, 所有有余量的点均在 $A$ 中。

**定理 4**: 算法过程中, 对任意的节点有 $h(v) \le 2n+1$。

**证明**: 考虑 $v\neq s,t$ , 算法只在 relabel 操作中修改 $h(v)$ 的值, 此时 $e_f(v) > 0$, 由定理 3, 在剩余图 $G_f$ 中有从 $v$ 到源点 $s$ 的路径 $P$, $\vert P\vert \le n-1$, 节点的高度沿着 P 的每条边至多可以减少 1, 因此有 $h(v) - h(s)\le \vert P\vert, h(v) \le 2n+1$ 。

**定理 5**: 算法过程中, relabel 操作总数少于 $2 n^2$ 。

**证明**: 在算法过程中, $h(v)$ 单调递增, $h(v) \le 2n+1$, 因此 relabel 操作总数不多于 $(n-2)(2n+1)$ 。

#### 饱和 (saturating) Push 操作

将 push 操作分为两类: 在 push 操作中, 当 $e$ 是一条前向边且 $\Delta = c_e-f(e)$ 或  $e$ 是一条后向边且 $\Delta = f(e)$ , 则称该 push 操作是饱和的(saturating), 否则是非饱和(non-saturating)的。换句话说, 饱和 Push 操作使得边 $e = (v,w)$ 不再处于剩余图中, 非饱和 Push 操作使得节点 $v$ 的减小余量为0.

**定理 6**: 算法过程中, 饱和 Push 操作次数至多为 $2 nm$ 。

**证明: **考虑剩余图中的任意一条边 $e = (v,w)$, 如果在 $e$ 上进行饱和 push 操作, 则需要满足 $h(v) = h(w)+1$ , 且操作后 $e$ 不再处于剩余图中。在 $e$ 上进行下一次饱和 push 操作之前,需要将 $e$ 重新加入剩余图中, 则需要有在 $e' = (w,v)$ 上的 push 操作, 此时需要有 $h(w) = h(v)+1$ , 也就是说点 $w$ 至少需要进行 relabel 两次。由于算法过程中, 对任意的节点有 $h(v) \le 2n+1$, 最多有 $n$ 次将 $w$ 的高度提升2。 因此, 对于每条边, 最多有 $n$ 次饱和 push 操作。剩余图中共有 $2m$ 条边, 饱和 Push 操作次数至多为 $2 nm$ 。

**定理 6**: 算法过程中, 非饱和 Push 操作次数至多为 $2n^3$ 。

**证明**: 考虑每相邻的两次 relabel 操作之间的非饱和 Push 操作, 非饱和 Push 操作使顶点的余量变为0, 再对同一个顶点进行下次 push 操作之前, 需要使它的余量变为正数, 即需要有其他顶点 push 流量给它。由于我们的操作是对在有余量的顶点中最大高度的顶点进行,  其他顶点 push 流量给它需要满足高度大于该顶点, 因此至少有一次relabel 操作。也就是说, 每相邻的两次 relabel 操作之间的非饱和 Push 操作在每个顶点上最多有一次, 即总共有 $n$ 次。算法过程中, relabel 操作总数少于 $2 n^2$ , 因此非饱和 Push 操作次数至多为 $2n^3$ 。

### 总结

Push-Relabel 方法的时间复杂度为 $O(n^3)$

## 对偶解释

换个角度看最大流的两种方法, Ford-Fulkerson 方法是在维持解可行的条件下逐步使解更优, 它维持了一个合法流,逐渐增大它的流量。而 Push-Relabel 方法则是在保持解最优的条件下使解逐步可行, 它构造了一个具有最大流量的不可行解 preflow, 逐渐将preflow 转化为一个合法的流。

这个想法似曾相识 emmm 跟对偶单纯形法的思路很像。也就是说, Push-Relabel 方法在维持问题的对偶可行的前提下改进原始可行, 由对偶定理, 当同时满足对偶可行与原始可行时, 可行解也是问题的最优解.



参考资料：

>+ 《Algorithm Design》Jon Kleinberg, Eva Tardos， Chapter 7
>+ Stanford [CS 261](http://timroughgarden.org/w16/w16.html) Lecture 3 (Push-Relabel):  [Video](https://youtu.be/0hI89H39USg)  [Notes](http://timroughgarden.org/w16/l/l3.pdf)

