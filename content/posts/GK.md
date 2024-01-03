---
title: GK 算法简介
date: 2018-01-25　
tags:
- Stream
- quantile problem
- GK

categories:
- paper
description: " Space efficient online computation of quantile summaries"
---
# GK algorithm

## introduction

在这里，我们研究了一种算法，该算法可以持续维护汇总信息，并且只需要使用通常需要的一小部分内存就能在小的有界误差范围内回答查询。有许多算法可以解决这个问题，但是我们要研究的是 Greenwald 和 Khanna 在他们的 2001 年的论文 [“空间高效的在线计算分位数摘要”](https://dl.acm.org/citation.cfm?doid=375663.375670)。比其他许多算法理解起来要复杂得多，但据我所知，它是空间使用最好的。另外，我们稍后将会研究的其他许多论文将用它来解决有趣的问题。

## data structure

Greenwald-Khanna（GK 从现在开始）基于一个简单的数据结构。S(n) 这种数据结构包括了一系列排过序的元组，这些元组对应了我们迄今以来观察到的数据的一个子集，对于每个观察到的在 S 中的ｖ，我们维护一个 implicit bound 对于观察点的最大可能的 rank 值和最小可能的 rank 值．令 rmin(v) 和 rmax(v) 分别表示ｖ可能的 rank 的上届和下届．S 包括了 t0,t1,...ts-1 个元组，每一个元组（v，g，Δ）包含了三个元素．
1. vi 表示数据流中实际出现的一个值．
2. gi ＝　rmin(vi) - rmin(vi-1).
3. Δ = rmax(vi) - rmin(vi).
这样，我们可以确保，在任何时刻，最大可能和最小可能都在 description 中．换句话说，v0 和 vs-1 一直表示我们看到的数据流中的最大值和最小值．rmin(vi) = sum(gj),where j<=i. rmax(vi) = sum(gj+Δj),where j<=i ，因此, gi + Δi - 1 是所有数字的上届，sum(gi) = n, 表示所有出现的数字的数量和．
该算法通过相应的插入和压缩算法，以尽量保持尽可能少的元组，同时始终保持对于任何元组我们有 g +Δ<= 2 *ε* n 的不变量。只要我们能够牢记这一点，算法总会给出正确的ε- 近似的答案。ε的有效值在很大程度上取决于你想要做什么样的查询。例如，如果你想要第 90 百分位数，那么ε= 0.1 就可以了，对于第 99 个百分位数，ε= 0.01 就可以了，依此类推。

## insert

GK Insert（）算法很简单。它扫描我们维护的元组列表，找到应该插入新元组的正确位置。这个元组总是 {值，1，floor（2 *ε* n）}，其中ε是我们所瞄准的精度，n 是到目前为止看到的元素的数量。上述例外是插入新的最大值或最小值。在这种情况下，元组是 {value，1,0}。

```
Insert(double v) {
Tuple newValue = new Tuple(v, 1);
int idx = 0;
foreach (Tuple t in S) {
    if (v < t.value)
break;
idx++;
  }

if (idx == 0 || idx == S.Count) // the new element is the new min or max
newValue.delta = 0;
else
newValue.delta = Floor(2 * epsilon * N);

S.Insert(idx, newValue);
++N;
  Compress();
}

```

## Compress

GK 算法的核心在于 Compress（）函数。这是我们摆脱不必要的元组，同时保持必要的不变量，以确保我们可以给出正确的答案。
```
for (int i = 0; i < S.Count - 1; ++i) {
if (S[i].g + S[i + 1].g + S[i + 1].delta <= Floor(2 * epsilon * N)) {
S[i + 1].g += S[i].g;
S.RemoveAt(i);
}
}

```

## get rank

这个简化的压缩代码遍历列表并合并对，只要新的对的加上Δ的值不违反我们的不变量。最后，我们现在可以轻松地查询我们的数据结构。回想一下，对于任何元组，该值的最小等级是 g 到该元组的总和，最大等级是通过将Δ加到该和上而得到的。因此，只需遍历元组列表就可以了，一旦找到一个元组的值大于要搜索的元素的值，然后元组的值小于该元素的值。下面给出计算分位数的算法：
```
for (int i = 0; i < S.Count - 1; ++i) {
rMin += S[i].g;
rMax = rMin + S[i].delta;

if (r - rMin <= epsilon*N && rMax - r <= epsilon*N) {
return S[i].value;
}
}

```

上面的变量 r 是我们正在寻找的等级。上面的工作是通过按照我们的初始定义连续计算每个元组的最小和最大可能的等级。如果我们想要的等级落在这个范围内，那么我们返回这个元组的值。

## 参考
[Stream Algorithms: Order Statistics](https://papercruncher.wordpress.com/2010/03/02/stream-algorithms-order-statistics/)
[Space-Efficient Online Computation of Quantile
Summaries](http://infolab.stanford.edu/~datar/courses/cs361a/papers/quantiles.pdf)
