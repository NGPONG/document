## 前言

在计算机科学中，排序算法是一种将列表元素按顺序排列的算法。最常用的顺序是数值顺序（[numerical order](https://en.wikipedia.org/wiki/Numerical_order)）和字典顺序（[Lexicographic order](https://en.wikipedia.org/wiki/Lexicographical_order)），可以是升序或降序。一个高效的排序算法使得优化其他算法（如 [search](https://en.wikipedia.org/wiki/Search_algorithm) 和 [merge](https://en.wikipedia.org/wiki/Merge_algorithm)）的效率非常重要。另外一点，排序对于数据规范化和生成人类可读的输出结果也很有用。

从形式上看，任何排序算法的输出都必须满足两个条件：

- 输出按单调顺序排列（根据所需的顺序，每个元素都不比前一个元素小/大）。

- 输出是输入的排列组合（重新排序，但保留所有原始元素）。

为达到最佳效率，输入数据应存储在允许随机存取的数据结构中，而不是只允许顺序存取的数据结构中。

## 一些历史

从计算机诞生之初，排序问题就吸引了大量的研究，这或许是因为尽管排序问题的表述简单而熟悉，但要有效解决它却非常复杂。1951 年前后，Betty Holberton 是早期排序算法的作者之一，她曾参与 ENIAC 和 UNIVAC 的研究。[Bubble sort](https://en.wikipedia.org/wiki/Bubble_sort)早在 1956 年就被分析过。自 20 世纪中叶以来，[渐进最优算法（Asymptotically optimal algorithm）](https://en.wikipedia.org/wiki/Asymptotically_optimal_algorithm)已经被熟知。新算法仍在不断被发明，其中广泛使用的 [Timsort](https://en.wikipedia.org/wiki/Timsort) 可追溯至 2002 年，而 [Library sort](https://en.wikipedia.org/wiki/Library_sort) 首次发表于 2006 年。

排序算法在计算机科学入门课程中很常见，大量用于解决排序问题的算法能够很容易的衍生出其它的概念，诸如：

- [大O表示法（big O notation）](https://en.wikipedia.org/wiki/Big_O_notation "Big O notation")

- [分治算法（divide-and-conquer algorithms）](https://en.wikipedia.org/wiki/Divide-and-conquer_algorithm "Divide-and-conquer algorithm")

- [数据结构（如堆和二叉树）](https://en.wikipedia.org/wiki/Data_structure "Data structure")

- [随机性算法（randomized algorithms）](https://en.wikipedia.org/wiki/Randomized_algorithm "Randomized algorithm")

- [最佳，最差和平均时间复杂度分析](https://en.wikipedia.org/wiki/Best,_worst_and_average_case "Best, worst and average case")

- [时间与空间的权衡（time–space tradeoffs）](https://en.wikipedia.org/wiki/Time%E2%80%93space_tradeoff)

- [上下界（upper and lower bounds）](https://en.wikipedia.org/wiki/Upper_and_lower_bounds "Upper and lower bounds")

对小型序列进行最佳算法的选择（以最少的比较和交换）或快速排序（即考虑到机器的具体细节）仍然是一个开放的研究课题。目前只知道对非常小的序列（至少 < 20 个元素）进行排序的解决方案。同样，并行计算机上的最优（根据不同定义）排序也是一个未完成的研究课题。

## 算法的分类

- [复杂度计算（Computational complexity）](https://en.wikipedia.org/wiki/Computational_complexity_theory "Computational complexity theory")：最好、最坏和平均情况下的表现，取决于列表的大小。对于典型的串行排序算法，好的表现是 $O(n \log n)$，并行排序是 $O(\log n)$，坏的表现是 $O(n^2)$。理想的串行排序表现是 $O(n)$，但在平均情况下不可能。最优的并行排序是 $O(\log n)$。

- 内存使用（以及其他计算机资源的使用）：特别是，一些排序算法是 “就地（[in-place](https://en.wikipedia.org/wiki/In-place_algorithm)）” 的。严格来说，就地排序除了被排序的项之外只需要 $O(1)$ 的内存；有时 $O(log n)$ 的额外内存也被认为是 “就地” 的。

- 递归（Recursion）：一些算法是递归的或非递归的，而另一些可能是两者兼有的（例如归并排序）。

- [[#[排序算法的稳定性](https //en.wikipedia.org/wiki/Sorting_algorithm Stability)|是否稳定（Stability）]]：稳定的排序算法保持具有相同键（即值）的记录的相对顺序。

- [[#[比较排序（Comparison sort）](https //en.wikipedia.org/wiki/Comparison_sort)|是否是比较排序（Comparison sort）]]：比较排序仅通过比较运算符比较两个元素来检查数据。

- 通用方法：插入、交换、选择、合并等。交换排序包括冒泡排序和快速排序。选择排序包括环排序和堆排序。

- 算法是串行（serial）还是并行（parallel）的：本文的讨论几乎完全集中在串行算法上，并假设串行操作。

- 适应性（Adaptability）：输入的序列如果已排序是否会影响算法的运行时间。考虑到这一点的算法被称为自适应算法。

- 在线的（Online）：像插入排序这样的在线算法可以对不断流入的输入进行排序。

## 排序算法之间的差异

|                                         Name                                          |     Best      |     Average     |          Worst          |  Memory  | Stable |          Method          |
| :-----------------------------------------------------------------------------------: | :-----------: | :-------------: | :---------------------: | :------: | :----: | :----------------------: |
|       [In-place merge sort](https://en.wikipedia.org/wiki/In-place_merge_sort)        |       -       |        -        |     $O(n \log^2 n)$     |    1     |  YES   |         Merging          |
|                  [Heapsort](https://en.wikipedia.org/wiki/Heapsort)                   | $O(n \log n)$ |  $O(n \log n)$  |      $O(n \log n)$      |    1     |   NO   |        Selection         |
|                 [Introsort](https://en.wikipedia.org/wiki/Introsort)                  | $O(n \log n)$ |  $O(n \log n)$  |      $O(n \log n)$      | $\log n$ |   NO   | Partitioning & Selection |
|                [Merge sort](https://en.wikipedia.org/wiki/Merge_sort)                 | $O(n \log n)$ |  $O(n \log n)$  |      $O(n \log n)$      |   $n$    |  YES   |         Merging          |
|           [Tournament sort](https://en.wikipedia.org/wiki/Tournament_sort)            | $O(n \log n)$ |  $O(n \log n)$  |      $O(n \log n)$      |   $n$    |   NO   |        Selection         |
|           [Tree sort](https://en.wikipedia.org/wiki/Tree_sort "Tree sort")            | $O(n \log n)$ |  $O(n \log n)$  | $O(n \log n)$(balanced) |   $n$    |  YES   |        Insertion         |
|           [Quicksort](https://en.wikipedia.org/wiki/Quicksort "Quicksort")            | $O(n \log n)$ |  $O(n \log n)$  |        $O(n^2)$         | $\log n$ |   NO   |       Partitioning       |
|          [Block sort](https://en.wikipedia.org/wiki/Block_sort "Block sort")          |    $O(n)$     |  $O(n \log n)$  |      $O(n \log n)$      |    1     |  YES   |   Insertion & Merging    |
|                [Smoothsort](https://en.wikipedia.org/wiki/Smoothsort)                 |    $O(n)$     |  $O(n \log n)$  |      $O(n \log n)$      |    1     |   NO   |        Selection         |
|              [Timsort](https://en.wikipedia.org/wiki/Timsort "Timsort")               |    $O(n)$     |  $O(n \log n)$  |      $O(n \log n)$      |   $n$    |  YES   |   Insertion & Merging    |
| [Patience sorting](https://en.wikipedia.org/wiki/Patience_sorting "Patience sorting") |    $O(n)$     |  $O(n \log n)$  |      $O(n \log n)$      |   $n$    |   NO   |  Insertion & Selection   |
|             [Cubesort](https://en.wikipedia.org/wiki/Cubesort "Cubesort")             |    $O(n)$     |  $O(n \log n)$  |      $O(n \log n)$      |   $n$    |  YES   |        Insertion         |
|       [Library sort](https://en.wikipedia.org/wiki/Library_sort "Library sort")       | $O(n \log n)$ |  $O(n \log n)$  |        $O(n^2)$         |   $n$    |   NO   |        Insertion         |
|           [Shellsort](https://en.wikipedia.org/wiki/Shellsort "Shellsort")            | $O(n \log n)$ | $O(n^\frac 43)$ |     $O(n^\frac 32)$     |    1     |   NO   |        Insertion         |
|           [Comb sort](https://en.wikipedia.org/wiki/Comb_sort "Comb sort")            | $O(n \log n)$ |    $O(n^2)$     |        $O(n^2)$         |    1     |   NO   |        Exchanging        |
|    [Insertion sort](https://en.wikipedia.org/wiki/Insertion_sort "Insertion sort")    |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |    1     |  YES   |        Insertion         |
|               [Bubble sort](https://en.wikipedia.org/wiki/Bubble_sort)                |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |    1     |  YES   |        Exchanging        |
|      [Cocktail shaker sort](https://en.wikipedia.org/wiki/Cocktail_shaker_sort)       |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |    1     |  YES   |        Exchanging        |
|                [Gnome sort](https://en.wikipedia.org/wiki/Gnome_sort)                 |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |    1     |  YES   |        Exchanging        |
| [Odd–even sort](https://en.wikipedia.org/wiki/Odd%E2%80%93even_sort "Odd–even sort")  |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |    1     |  YES   |        Exchanging        |
|               [Strand sort](https://en.wikipedia.org/wiki/Strand_sort)                |    $O(n)$     |    $O(n^2)$     |        $O(n^2)$         |   $n$    |  YES   |        Selection         |
|            [Selection sort](https://en.wikipedia.org/wiki/Selection_sort)             |   $O(n^2)$    |    $O(n^2)$     |        $O(n^2)$         |    1     |   NO   |        Selection         |
|    [Exchange sort](https://en.wikipedia.org/wiki/Sorting_algorithm#Exchange_sort)     |   $O(n^2)$    |    $O(n^2)$     |        $O(n^2)$         |    1     |   NO   |        Exchanging        |
|                [Cycle sort](https://en.wikipedia.org/wiki/Cycle_sort)                 |   $O(n^2)$    |    $O(n^2)$     |        $O(n^2)$         |    1     |   NO   |        Selection         |
<center>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Comparison sorts</div>
</center>

|                                                    Name                                                     | Best |        Average         |                 Worst                 |         Memory         | Stable |
| :---------------------------------------------------------------------------------------------------------: | :--: | :--------------------: | :-----------------------------------: | :--------------------: | :----: |
|                      [Pigeonhole sort](https://en.wikipedia.org/wiki/Pigeonhole_sort)                       |  -   |       $n + 2^k$        |               $n + 2^k$               |         $2^k$          |  YES   |
|            [Bucket sort](https://en.wikipedia.org/wiki/Bucket_sort "Bucket sort") (uniform keys)            |  -   |        $n + k$         |             $n^2 \cdot k$             |      $n \cdot k$       |   NO   |
|                   [Bucket sort](https://en.wikipedia.org/wiki/Bucket_sort) (integer keys)                   |  -   |        $n + r$         |                $n + r$                |        $n + r$         |  YES   |
|                        [Counting sort](https://en.wikipedia.org/wiki/Counting_sort)                         |  -   |        $n + r$         |                $n + r$                |        $n + r$         |  YES   |
| [LSD Radix Sort](https://en.wikipedia.org/wiki/Radix_sort#Least_significant_digit_radix_sorts "Radix sort") | $n$  | $n \cdot \frac {k}{d}$ |        $n \cdot \frac {k}{d}$         |       $n + 2^d$        |  YES   |
| [MSD Radix Sort](https://en.wikipedia.org/wiki/Radix_sort#Most_significant_digit_radix_sorts "Radix sort")  |  -   | $n \cdot \frac {k}{d}$ |        $n \cdot \frac {k}{d}$         |       $n + 2^d$        |  YES   |
| [MSD Radix Sort](https://en.wikipedia.org/wiki/Radix_sort#Most_significant_digit_radix_sorts "Radix sort")  |  -   | $n \cdot \frac {k}{1}$ |        $n \cdot \frac {k}{1}$         |         $2^1$          |   NO   |
|                     [Spreadsort](https://en.wikipedia.org/wiki/Spreadsort "Spreadsort")                     | $n$  | $n \cdot \frac {k}{d}$ | $n \cdot \left( \frac ks + d \right)$ |    $\frac kd + 2^d$    |   No   |
|                            [Burstsort](https://en.wikipedia.org/wiki/Burstsort)                             |  -   | $n \cdot \frac {k}{d}$ |        $n \cdot \frac {k}{d}$         | $n \cdot \frac {k}{d}$ |   NO   |
|                            [Flashsort](https://en.wikipedia.org/wiki/Flashsort)                             | $n$  |        $n + r$         |                 $n^2$                 |          $n$           |   NO   |
|                         [Postman sort](https://en.wikipedia.org/wiki/Postman_sort)                          |  -   | $n \cdot \frac {k}{d}$ |        $n \cdot \frac {k}{d}$         |       $n + 2^d$        |   NO   |
<center>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Non-comparison sorts</div>
</center>

|                                      Name                                       |          Best           |         Average         |          Worst          | Memory | Stable | Comparison |
| :-----------------------------------------------------------------------------: | :---------------------: | :---------------------: | :---------------------: | :----: | :----: | :--------: |
|        [Bead sort](https://en.wikipedia.org/wiki/Bead_sort "Bead sort")         |           $n$           |           $S$           |           $S$           | $n^2$  |   -    |     NO     |
|   [Merge-insertion sort](https://en.wikipedia.org/wiki/Merge-insertion_sort)    |      $O(n \log n)$      |      $O(n \log n)$      |      $O(n \log n)$      |   -    |   NO   |    YES     |
|      [Spaghetti (Poll) sort](https://en.wikipedia.org/wiki/Spaghetti_sort)      |           $n$           |           $n$           |           $n$           | $n^2$  |  YES   |     -      |
|        [Sorting network](https://en.wikipedia.org/wiki/Sorting_network)         |            -            |            -            |            -            |   -    |   -    |    YES     |
| [Bitonic sorter](https://en.wikipedia.org/wiki/Bitonic_sorter "Bitonic sorter") |     $O(n \log^2 n)$     |     $O(n \log^2 n)$     |     $O(n \log^2 n)$     |   1    |   NO   |    YES     |
|          [Bogosort](https://en.wikipedia.org/wiki/Bogosort "Bogosort")          |         $O(n)$          |    $O(n \times n!)$     |            -            |   1    |   NO   |    YES     |
|     [Stooge sort](https://en.wikipedia.org/wiki/Stooge_sort "Stooge sort")      | $n^{\log^3/\log^{1.5}}$ | $n^{\log^3/\log^{1.5}}$ | $n^{\log^3/\log^{1.5}}$ |  $n$   |   NO   |    YES     |
<center>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Others</div>
</center>

## 热门排序解释

### [冒泡排序](https://en.wikipedia.org/wiki/Bubble_sort)（Bubble Sort）

冒泡排序，有时也称为下沉排序（Sinking sort），是一种**就地的**、**稳定的**且**自适应（优化后）** 排序算法。其命名规则与其实现很相似，是根据较大/小的元素 “冒” 到序列的顶部/底部的方式而命名。由于其更易于理解与实现，这种排序算法更倾向于教学的目的（显然目前已经很不提倡这种方式）。

需要注意的是，该排序并不是一种高效的算法，通常来说其复杂度依赖于输入序列的长度呈现二次增长，那么在一些较大数据集上使用上则显得尤为吃力。相反，即使条件相同，在小数据集上也有更多优于冒泡排序的选择，例如插入排序或者选择排序。

#### Examples

设序列的长度为 $𝑛$：

1. 首先，对 $𝑛$ 个元素执行 “冒泡”，**将序列的最大元素交换至正确位置**。

2. 接下来，对剩余 $𝑛−1$ 个元素执行 “冒泡”，**将第二大元素交换至正确位置**。

3. 以此类推，经过 $𝑛−1$ 轮 “冒泡” 后，**前 $𝑛−1$ 大的元素都被交换至正确位置**。

4. 仅剩的一个元素必定是最小元素，无须排序，因此序列排序完成。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-13-01-41-03.gif">
</center>

```cpp
// 默认实现
void solution(Array &datas) {
  for (std::size_t i = 0; i < datas.size(); ++i) {
    for (std::size_t j = 0; j < datas.size() - i - 1; ++j) {
      if (datas[j] > datas[j + 1]) {
        std::swap(datas[j], datas[j + 1]);
      }
    }
  }
}
```

#### Analysis

~~~tabs
---tab **最佳时间复杂度**

在最好的情况下，此时的序列为完全有序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换

3. 第 $i$ 次遍历：需要 $n - i$ 次比较和交换。

因此我们可以得出： 

$$ 
  (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=0}^{n - 1} (n - 1 - i)
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \sum_{i=0}^{n - 1} (n - 1 - i) & = \frac{(n - 1)n}{2} \\
     & = \frac{n^2 - n}{2} \\ 
     & = \frac{1}{2}(n^2 - n) \\
     & = O(n^2)
  \end{align}
$$

然而，如果能够在单次循环的过程中知道序列已有序，那么复杂度可降低至：$O(n)$

---tab **最坏时间复杂度**

在最坏的情况下，此时的序列为完全逆序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换。

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换。

3. 第 $i$ 次遍历：需要 $n - i$ 次比较和交换。

因此我们可以得出： 

$$ 
  (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=0}^{n - 1} (n - 1 - i)
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \sum_{i=0}^{n - 1} (n - 1 - i) & = \frac{(n - 1)n}{2} \\
     & = \frac{n^2 - n}{2} \\ 
     & = \frac{1}{2}(n^2 - n) \\
     & = O(n^2)
  \end{align}
$$

---tab **平均时间复杂度**

在平均情况下，此时的序列为随机序列，那么我们就可以认为平均时间介于最坏与最优复杂度之间。故我们可以得出平均时间复杂度为：$O(n^2)$
~~~

#### Performance

冒泡排序与现代 CPU 硬件的交互性很差。它产生的写入次数至少是插入排序的两倍，缓存缺失次数是插入排序的两倍，分支误预测次数也逐渐增多。在 Java 中对字符串进行排序的实验表明，冒泡排序的速度大约是插入排序的五分之一，是选择排序的 70%。

此外，冒泡排序在处理大型数据集时表现尤为糟糕，因为它的时间复杂度是 $O(n^2)$。这意味着，当数据集规模增加时，排序所需的时间会迅速增加。与其他更高级的算法相比（Quick Sort），冒泡排序的缓存效率较低，因为它频繁地交换元素，导致缓存未命中率高。此外，冒泡排序的分支预测误差也较多，因为每次比较后可能会交换或不交换元素。尽管它在已排序列表上的最佳情况时间复杂度为 $O(n)$，但这种情况较少见。需要注意的是，如果[允许并行处理](https://medium.com/@keshav519sharma/parallel-bubble-sort-7ec75891afff)，这种 $O(n)$ 的复杂度则不受序列状态限制。这比插入排序或选择排序的并行实现要快得多，因为插入排序或选择排序的并行化后的效果并不是很好。

> 在计算机图形学中，冒泡排序因其能以线性复杂度 $2n$ 去检测出几乎已排序的序列中非常小的错误（例如仅两个元素的交换）并修复它而受到欢迎。例如，它被用于多边形填充算法中，在特定扫描线（与 x 轴平行）上按 x 坐标排序边界线，并随着 y 值的增加而交换顺序（仅在两条线的交点处交换两个元素）。

#### Variants

**1. 提前结束**

当持续循环的过程中如果序列已有序，那么此时我们就不需要继续往下进行。要知道，任意的排序算法都可以在开始运行前去检查一个已完成排序的序列以完成 $O(n)$ 的复杂度，然而这却难以在一个相对有序的序列中复刻。冒泡排序能够非常好的实现这一点，我们只需通过一个标志并在单次循环的过程中判断列表是否有序。如果列表已有序，则可以直接退出不用再重复后续的过程，最佳时间复杂度能优化到 $O(n)$；如果没有则退回到冒泡排序的时间复杂度上。

```cpp
void solution(Array &datas) {
  for (size_t i = 0; i < datas.size(); i++) {
    bool is_swapped = false;

    for (size_t j = 0; j < datas.size() - i - 1; j++) {
      if (datas[j] > datas[j + 1]) {
        std::swap(datas[j], datas[j + 1]);
        is_swapped = true;
      }
    }

    if (!is_swapped) {
      break;
    }
  }
}
```

**[2. 鸡尾酒算法](https://en.wikipedia.org/wiki/Cocktail_shaker_sort)**

元素在排序过程中的移动方向和距离决定了该算法的性能。一个需要移到列表末尾的元素可以通过连续交换迅速到达其位置，而需要移到列表开头的元素则移动较慢，因为每次只能移动一步。如果列表末尾是最小元素，则需要 𝑛 − 1 次交换才能将其移到开头。这个现象导致这些元素分别被称为 “兔子” 和 “乌龟”，来源于《龟兔赛跑》的寓言故事。

鸡尾酒算法是一种冒泡排序的变体。为解决前述中所描绘的 “乌龟” 问题，其扩展了双向步进操作元素的顺序以提升了性能的改进。需要意识到，这种性能提升是有限的（同样保留 $O(n^2)$ 最坏情况复杂度），特别是在处理一份逆序序列的情况下，这一点显得更为明显。然而由于双向遍历的特性这使得再处理某些情况下时该算法能有显著的优势，例如：设一个列表为 (2, 3, 4, 5, 1)，那么鸡尾酒排序由于双向排序的原因，其仅需一次操作就可使列表为有序的；而冒泡排序必须经过四次循环遍历。另外一点，更优的写法使得编译器、缓存、流水线拥有更多的优化空间。

```cpp
void solution(Array &datas) {
  std::int32_t left = 0, right = datas.size() - 1;

  while (left < right) {
    for (size_t i = left; i < right; i++) {
      if (datas[i] > datas[i + 1]) {
        std::swap(datas[i], datas[i + 1]);
      }
    }
    right--;

    for (size_t i = right; i > left; i--) {
      if (datas[i - 1] > datas[i]) {
        std::swap(datas[i], datas[i - 1]);
      }
    }
    left++;
  }
}
```

### [选择排序](https://en.wikipedia.org/wiki/Selection_sort)（Selection Sort）

选择排序是一种**就地的**、**不稳定**且<span class="cm-strong">非自适应（原始状态下）</span>排序算法。该算法将输入序列分为两部分：在完成第 $k$ 次迭代后，序列从左侧开始的前 $k$ 项则为已排序项左子序列，和占据序列的其余部分的未排序项右子序列。在这一点上插入排序与此非常相似，而前者的优势在于它只需扫描尽可能多的元素，就能找到 $k + 1$st 元素，反之选择排序必须扫描剩余的所有元素。

#### Examples

设序列的长度为 $n$：

1. 初始状态下，所有元素未排序，即未排序（索引）区间为 $[0,𝑛−1]$ 。

2. 选取区间 $[0,𝑛−1]$ 中的最小元素，将其与索引 $0$ 处的元素交换。完成后，序列前 $1$ 个元素已排序。

3. 选取区间 $[1,𝑛−1]$ 中的最小元素，将其与索引 $1$ 处的元素交换。完成后，序列前 $2$ 个元素已排序。

4. 以此类推。经过 $𝑛−1$ 轮选择与交换后，序列前 $𝑛−1$ 个元素已排序。

5. 仅剩的一个元素必定是最大元素，无须排序，因此序列排序完成。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-14-01-49-12.gif">
</center>

```cpp
  for (std::size_t i = 0; i < datas.size(); ++i) {
    std::int32_t max = i;

    for (std::size_t j = i + 1; j < datas.size(); ++j) {
      if (datas[max] > datas[j]) {
        max = j;
      }
    }

    std::swap(datas[i], datas[max]);
  }

```

#### Analysis

~~~tabs
---tab **最佳时间复杂度**

在最好的情况下，此时的序列为完全有序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换

3. 第 $i$ 次遍历：需要 $n - i$ 次比较和交换。

因此我们可以得出： 

$$ 
  (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=0}^{n - 1} (n - 1 - i)
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \sum_{i=0}^{n - 1} (n - 1 - i) & = \frac{n(n - 1)}{2} \\
     & = \frac{n^2 - n}{2} \\ 
     & = \frac{1}{2}(n^2 - n) \\
     & = O(n^2)
  \end{align}
$$

然而，如果能够在单次循环的过程中知道序列已有序，那么复杂度可降低至：$O(n)$

---tab **最坏时间复杂度**

在最坏的情况下，此时的序列为完全逆序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换。

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换。

3. 第 $i$ 次遍历：需要 $n - i$ 次比较和交换。

因此我们可以得出： 

$$ 
  (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=0}^{n - 1} (n - 1 - i)
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \sum_{i=0}^{n - 1} (n - 1 - i) & = \frac{n(n - 1)}{2} \\
     & = \frac{n^2 - n}{2} \\ 
     & = \frac{1}{2}(n^2 - n) \\
     & = O(n^2)
  \end{align}
$$

---tab **平均时间复杂度**

在平均情况下，此时的序列为随机序列，那么我们就可以认为平均时间介于最坏与最优复杂度之间。故我们可以得出平均时间复杂度为：$O(n^2)$

~~~

#### Performance

在二次排序算法当中，选择排序总是优于冒泡排序。虽然在较大的序列上由于常数因子太大导致选择/插入排序表现不佳；然而更小的内存开销、更小的尝试因子和更简单的缓存友好实现使得对于较小序列（小于10~20个元素）普遍采用选择/插入排序的实现（这是许多递归算法中常用的一种优化手段）。

简单的计算表明，插入排序执行的比较次数通常是选择排序的一半左右，不过根据排序前序列的排序状态，它执行的比较次数可能一样多，也可能少得多。相对而言，对于某些需要依赖于实时的应用程序来说，这能够看作是一种优势；因为无论序列的顺序如何，选择排序的执行次数都是相同的；而插入排序的运行时间可能会有很大差异。

在最坏情况下，虽然选择排序在写操作数量上优于插入排序（选择排序需要 $n - 1$ 次交换，而插入排序最多需要 $\frac{n(n−1)}{2}$ 次交换，每次交换涉及两次写操作），但是选择排序的写操作次数大约是[循环排序](https://en.wikipedia.org/wiki/Cycle_sort)的两倍。循环排序在理论上最多只需 $n$ 次写操作。这种差异在写操作比读取操作昂贵得多的情况下尤为重要，例如在使用 [EEPROM](https://en.wikipedia.org/wiki/EEPROM) 或闪存（[Flash memory](https://en.wikipedia.org/wiki/Flash_memory)）时，每次写操作都会减少存储器的寿命。

#### Variants

**1. 无预测分支实现**

选择排序可以通过无预测的分支实现以使 CPU 分支预测器受益，方法是使用无分支代码找到最小值的位置，然后无条件地执行交换。

```cpp
void solution(Array &datas) {
  for (size_t i = 0; i < datas.size(); i++) {
    std::size_t min = i;

    for (size_t j = i + 1; j < datas.size(); j++) {
      std::int32_t diff = datas[j] - datas[min];
      std::int32_t mask = diff >> 31;
      min               = (min & ~mask) | (j & mask);
    }

    std::swap(datas[i], datas[min]);
  }
}

```

**2. 稳定的选择排序**

如果不执行交换而是插入到最后一个未排序元素的位置上，并将中间值同步上移，那么我们就能得到一个稳定的选择排序。然而这种修改要么需要一个支持高效插入或删除的数据结构（如链表），要么会导致执行退化为 $O(n^2)$ 的写入。

```cpp
void solution(Array &datas) {
    std::int32_t n = datas.size();

    for (std::int32_t i = 0; i < n - 1; ++i) {
        std::int32_t min_idx = i;
        for (std::int32_t j = i + 1; j < n; ++j) {
            if (datas[j] < datas[min_idx]) {
                min_idx = j;
            }
        }

        std::int32_t min_val = datas[min_idx];

        while (min_idx > i) {
            datas[min_idx] = datas[min_idx - 1];
            --min_idx;
        }
        datas[i] = min_val;
    }
}
```

**3. 双向选择排序**

双向选择排序是一种优化方向。在每个步骤中需要同时找到最大值与最小值并执行矫正操作。与普通选择排序的每个元素执行一次比较相比，双向选择排序每两个元素需要进行三次比较（将较大的元素与最大值比较，将较小的元素与最小值比较，然后再比较这一对元素），但所需的次数只有普通选择排序的一半，净节省 25%。

```cpp
void solution_2(Array &datas) {
  std::int32_t l = 0, r = datas.size() - 1;

  while (l < r) {
    std::int32_t min = l;
    for (size_t i = l + 1; i <= r; i++) {
      if (datas[i] < datas[min]) {
        min = i;
      }
    }
    std::swap(datas[min], datas[l]);

    std::int32_t max = r;
    for (std::size_t j = r - 1; j > l; --j) {
      if (datas[j] > datas[max]) {
        max = j;
      }
    }
    std::swap(datas[max], datas[r]);

    ++l;
    --r;
  }
}
```

### [插入排序](https://en.wikipedia.org/wiki/Insertion_sort#:~:text=Insertion%20sort%20is%20a%20simple,%2C%20heapsort%2C%20or%20merge%20sort.)（Insertion Sort）

插入排序是一种实现简单且保有就地性、稳定性、自适应性与在线特性的二次排序算法。在每次迭代中，插入排序从输入数据中移除一个元素，找到它在已排序列表中的位置，并将其插入其中。重复此过程，直到没有剩余输入元素。这种工作方式类似于现实生活中大多数人在整理手握[扑克牌](https://en.wikipedia.org/wiki/Contract_bridge)时使用的方法。

#### Examples

设序列长度为 $n$：

1. 设 $i$ 从 $1$ 开始，将原序列分成有序区和无序区。有序区间为 $[0, i-1]$，无序区为 $[i, n]$

2. 从无序区中取出第 $i$ 个元素，在有序区序列中从后向前扫描。

3. 如果有序元素大于第 $i$ 个元素，将有序元素后移到下一位置。

4. 重复步骤 3，直到找到小于或者等于第 $i$ 个元素的有序元素，将第 $i$ 个元素插入到该有序元素的下一位置中。

5. 重复步骤 2 ~ 4，直到无序区的长度为 $0$。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-14-18-45-47.gif">
</center>

```cpp
void solution(Array &datas) {
  for (size_t i = 1; i < datas.size(); i++) {
    std::int32_t key = datas[i];

    std::int32_t j = i - 1;
    for (; j >= 0 && datas[j] > key; --j) {
      datas[j + 1] = datas[j];
    }
    datas[j + 1] = key;
  }
}
```

#### Analysis

~~~tabs
---tab **最佳时间复杂度**

在最好的情况下，此时的序列为完全有序，那么在每次迭代中，输入的第一个剩余元素只与序列已排序子部分的最右边元素进行比较，即：

$$ O(1) = 1 + 1 + \cdots + 1 $$

---tab **最坏时间复杂度**

在最坏的情况下，此时的序列为完全逆序，那么：

1. 第 1 次遍历：需要 $1$ 次比较和交换。

2. 第 2 次遍历：需要 $2$ 次比较和交换。

3. 倒数第 $2$ 次遍历：需要 $n - 2$ 次比较和交换。

4. 倒数第 $1$ 次遍历：需要 $n - 1$ 次比较和交换。

因此我们可以得出： 

$$ 
  1 + 2 + \cdots + (n - 2) + (n - 1) \\ = \sum_{i=1}^{n-1} i
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \sum_{i=1}^{n-1} i & = \frac{(n - 1)n}{2} \\
     & = \frac{n^2 - n}{2} \\ 
     & = \frac{1}{2}(n^2 - n) \\
     & = O(n^2)
  \end{align}
$$

---tab **平均时间复杂度**

在平均情况下，此时的序列为随机序列。经过实践表明（平均的），插入排序需要比较和移动前 $k$ 个已排序序列的一半，即：

$$ 
  \frac{1}{2}(1 + 2 + \cdots + (n - 2) + (n - 1)) \\ = \frac{1}{2} \cdot \sum_{i=1}^{n-1} i
$$

代入等差数列求和公式，得：

$$
  \begin{align}
    \frac{1}{2} \cdot \sum_{i=1}^{n-1} i & = \frac{1}{2} \cdot \frac{(n - 1)n}{2} \\
     & = \frac{n^2 - n}{4} \\ 
     & = O(n^2)
  \end{align}
$$

~~~
#### Performance

在实践中，插入排序比其他大多数的二次算法（如选择排序或冒泡排序）更高效。然而由于插入排序的平均复杂度下也是二次的，这使得插入排序在处理大序列时的效果相较于分治算法并不太理想。需要知道的是，插入排序是对于非常小的数组来说最快的算法之一，甚至甚至比某些分治算法还要快；因此，在这些算法的实现中，一种有用的优化是采用混合方法，当序列被划分到较小的大小时则退化为插入/选择排序；确切的临界值必须通过实验确定，并且取决于具体的机器，但通常在 $10$ 左右。

事实上，插入排序与选择排序的工作方式非常相似，即在序列中经过 $k$ 次迭代后，前 $k$ 个元素是按顺序排列的。插入排序相对于选择排序的主要优势在于，选择排序必须始终扫描所有剩余元素，以找到未排序部分中的绝对最小元素，而插入排序仅在第 $k + 1$ 个元素大于第 $k$ 个元素时需要进行一次比较；当这种情况经常发生时（例如，如果输入数组已经排序或部分排序），插入排序与选择排序相比效率明显更高。

平均而言（假设 $k + 1$ 个元素的排名是随机的），插入排序将需要比较和移动前 k 个元素的一半，这意味着插入排序平均将执行比选择排序少大约一半的比较。然而在最坏情况下（当输入数组是逆序排序时），插入排序执行的比较次数与选择排序相同或者显著高于。与选择排序相比，插入排序的一个缺点是由于在每次迭代中，将第 $k + 1$ 个元素插入数组的已排序部分时需要许多元素交换来移动所有后续元素，而选择排序每次迭代只需要进行一次交换。一般来说，插入排序会向数组写入 $O(n^2)$ 次，而选择排序只会写入 $O(n)$ 次。因此，在写入内存的成本显著高于读取的情况下，选择排序可能更可取。

#### Variants

**1. 递推改递归**

插入排序的最外层循环可以提供递归的实现。然而这并不会使代码变得更短，也不会减少执行时间。相反，它会将额外的内存消耗从 $O(1)$ 增加到 $O(n)$（在递归的最深层，堆栈包含 $n$ 个对数组的引用，和从 $n$ 到 $1$ 递减的变量值）。

```cpp
void solution(Array &datas) {
  auto insert_sort = make_y_combinator(
    [&](auto insert_sort, std::int32_t i = 1) -> void {
      if (i >= datas.size()) {
        return;
      }

      std::int32_t key = datas[i];

      std::int32_t j = i - 1;
      for (; j >= 0 && key < datas[j]; --j) {
        datas[j + 1] = datas[j];
      }
      datas[j + 1] = key;

      insert_sort(++i);
    });

  insert_sort();
}
```

**2. 二分插入排序**

如果比较的成本超过交换的成本，例如使用引用存储的字符串键，那么使用二分插入排序可能会获得更好的性能。 二分插入排序利用二分搜索来确定插入新元素的正确位置，因此在最坏情况下执行$\lfloor \log n \rfloor$ 次比较。当数组中的每个元素都被搜索并插入时，时间复杂度为 $O(\log n)$。整体算法在平均情况下仍具有 $O(n^2)$ 的运行时间，因为每次插入都涉及到一系列的交换操作。

```cpp
std::int32_t binarySearch(Array &datas, std::int32_t target, std::int32_t low, std::int32_t high) {
  if (high <= low)
    return (target > datas[low]) ? (low + 1) : low;

  int mid = (low + high) / 2;

  if (target == datas[mid])
    return mid + 1;
  if (target > datas[mid])
    return binarySearch(datas, target, mid + 1, high);

  return binarySearch(datas, target, low, mid - 1);
}

void solution(Array &datas) {
  for (std::int32_t i = 1; i < datas.size(); ++i) {
    std::int32_t key = datas[i];

    std::int32_t j = i - 1;

    std::int32_t loc = binarySearch(datas, key, 0, j);
    while (j >= loc) {
      datas[j + 1] = datas[j];
      --j;
    }
    datas[j + 1] = key;
  }
}

```

### 希尔排序（Shell Sort）

希尔排序（缩小增量排序）是一种**就地的**、**不稳定**和**自适应的**比较排序算法。它可以看作是交换排序或插入排序的一般化实现.。基于两点事实，1）当输入序列已排序时，插入排序能够达到 $O(n)$ 的执行效率，2）插入排序的二次平均复杂度对于大序列的处理时间并不理想。故，方法的实现首先会先将整个待排序列划分为若干子序列，然后再分别对齐进行插入排序，待整个序列 “基本有序” 时再整体进行一次进行完整的插入排序。

确定划分的长度通常用一个**通项公式**生成的<span class="cm-strong">间隔序列（Gap Sequence）</span>的序列集合表示。序列中的元素代表了每次新一轮迭代中需要划分长度的基准。希尔排序的运行时间在很大程度上取决于间隔序列的选择。不同的间隔序列会有较大的差异，例如一些性能较差的划分函数会导致排序过程早期退化为 $O(n^2)$ 的直接插入排序，此时整体的执行时间将不会达到预期。

#### Examples

设序列长度为 $n$；

设通项公式为 $\text{gap}_k​=\lfloor \frac{ n}{2^k} \rfloor$ ，那么我们可以得出间隔序列为： $\left\{ \text{gap}_k \right\}_{k=1}^{\left\lfloor \log n \right\rfloor} = \left\{ \left\lfloor \frac{n}{2^k} \right\rfloor \right\}_{k=1}^{\left\lfloor \log n \right\rfloor}$；

1. 取第 $i$ 轮 $\text{gap}$ 将序列从第一个元素开始划分为若干个子序列。即，分别将所有位置相隔为 $\text{gap}$ 的元素视为一个子数组。

2. 在各个子数组中采用某种排序算法（例如插入排序算法）进行排序。

3. 重复步骤 1，直至 $\text{gap} = 1$ 后则执行最后一次整体的插入排序

```cpp
void solution(Array &datas) {
  for (std::int32_t gs = datas.size() / 2; gs > 0; gs /= 2) {
    for (std::int32_t i = gs; i < datas.size(); i++) {
      std::int32_t key = datas[i];

      std::int32_t j = i;
      for (; j >= gs && key < datas[j - gs]; j -= gs) {
        datas[j] = datas[j - gs];
      }
      datas[j] = key;
    }
  }
}
```

#### Analysis

希尔排序的运行速度取决于一系列[间隔函数](https://en.wikipedia.org/wiki/Shellsort#Gap_sequences)（$\text{gap}_k$）的选择。为完成后面分析的工作，在开始前我们需要在这里选择更为 “合适的” 间隔函数；一些更为高效的间隙函数的数学推导过程过于复杂，由于能力有限，我难以在这里给出完整的数学分析。原文所给出的示例代码采用的是  $\text{gap}_k​=\lfloor \frac{ n}{2^k} \rfloor$ 通用项，虽并不是最高效的那一种，但它却足够的简单。

对于具有 $n$ 个元素的数组，如果 $\text{gap}_1​=\lfloor \frac{n}{2^1} \rfloor$ ，则经过 $p = \lfloor \log n \rfloor$ 趟排序后就有 $\text{gap}_p = 1$，因此，我么能够得出排序方法运行的总躺数（即外层循环）为 $\lfloor \log n \rfloor$。

内层循环采用的是插入排序的方案。对于增量为 $\text{gap}_k​$​ 的插入排序，每个子序列的元素数量约为 $\frac{n}{\text{gap}_k​}$。当子序列分得越多时，子序列内的元素就越少（内层循环的次数也就越少）；反之，当所分的子数组个数减少时，子序列内的元素也随之增多。

因为增量序列长度为 $p = \lfloor \log n \rfloor$ ，即需要进行 $\lfloor \log n \rfloor$ 次不同增量的插入排序。

因为插入排序的最差时间复杂度为 $O(n^2)$ 且每一个间隔的增长序列的长度已知，故我们可以得出内层循环插入排序时间复杂度为：$O((n/{\text{gap}_k​})^2) = O((\frac{n}{2^k})^2)$

那么总时间复杂度我们可以表示为：
$$
  \begin{align}
    T(n) & = O\bigg(\sum_{k=1}^{\log n} (\frac{n}{2^k})^2\bigg) \\
     & = O\bigg(\sum_{k=1}^{\log n} \frac{n^2}{(2^k)^2}\bigg) \\ 
     & = O\bigg(\sum_{k=1}^{\log n} \frac{n^2}{2^{2k}}\bigg) \\
     & = O\bigg(n^2 \cdot \sum_{k=1}^{\log n} \frac{1}{2^{2k}}\bigg)
  \end{align}
$$

由于  $\sum_{i=k}^{\infty} \frac{1}{2^{2k}}$ 是一个收敛级数，其和为常数，因此我们可以得出：

$$
  T(n) = O(n^2 \cdot C) = O(n^2)
$$

但由于增量序列的选择及其影响，实际复杂度会介于 $O(n \cdot \log^2 n)$ 和 $O(n^2)$ 之间。

#### Performance

相较于快速排序，希尔排序执行的操作数更多而且缓存未命中率更高。然而，由于它可以使用少量代码实现，并且不依赖于调用堆栈，因此 C 标准库中一些针对嵌入式系统的 [qsort](https://en.wikipedia.org/wiki/Qsort) 函数的实现采用的是希尔排序。例如 [uClibc](https://en.wikipedia.org/wiki/UClibc) 库。此外，希尔排序也可以作为 [Introspective Sort](https://en.wikipedia.org/wiki/Introsort) 的子算法，用于对短子数组进行排序，防止递归深度超过给定限制时速度变慢。例如，[bzip2](https://en.wikipedia.org/wiki/Bzip2) 压缩器就采用了这一原理。

要知道，任何包含数字 1 的隔序列都能在最终产生正确的排序，但是某些间隔序列可能会使排序过程更快，而其他间隔序列可能会导致更多的计算开销。因此，对于许多间隔序列的实际变种，如何选择和确定它们的时间复杂度仍然是一个开放问题。

本文示例中使用的通项公式是由希尔排序的发明者 Donald Shell 最早提出的。该序列按照 $\frac{n}{2^k}$ 的规则递减，每次将数组分割成若干子数组进行排序。虽然这种方法简单易实现，但它的最坏时间复杂度为 $O(n^2)$，即使在一些小规模的数据集上也会表现出较低的效率。在现代应用中更多的是将其作为一个基准点用于与其他间隔序列进行对比。

由 Hibbard 在 1963 年提出来的另一种通项公式为 $2^k - 1$ 的间隔序列。该序列的每个间隔都是前一个间隔的两倍减一，这使得排序过程中比较和交换的次数大大减少，从而提高了效率。序列的最坏时间复杂度为 $O(n^{\frac{3}{2}})$，在实践中的表现比前叙中的 $\frac{n}{2^k}$ 更好，特别是在处理较大规模的数据时。

一种结合了多种间隔生成方式的间隔序列，形如 $4^i + 3 \times 2^{i - 1} + 1$ 的通项公式由 Sedgewick 在 1982 年和 1986 年提出。作者通过多年的研究和实验，发现这种间隔序列在希尔排序中的表现非常优异，其最坏时间复杂度为仅为 $O(n^{\frac4 3})$。这种间隔序列通过平衡间隔的选择，使得排序过程中每一步的效率都能得到最大化提升，因此在实际应用中广泛使用。

### 快速排序（Quick Sort）



#### Examples

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-16-00-11-28.gif">
</center>

```cpp

```



https://www.hello-algo.com/chapter_sorting/quick_sort/#1151

https://segmentfault.com/a/1190000021726667

https://www.cnblogs.com/deng-tao/p/6536302.html

### 堆排序（Heap Sort）


## 相关概念

###  [比较排序（Comparison sort）](https://en.wikipedia.org/wiki/Comparison_sort)

比较排序是一种排序算法，只通过单一的抽象比较操作读取列表元素（通常是“小于或等于”操作符或三向比较）来确定两个元素中哪一个应在最终排序列表中优先出现。唯一的要求是，该操作符在数据上形成一个全预序关系，具有以下特性：

1. 传递性（Transitivity）：如果 $a \leq b$ 且 $b \leq c$，那么 $a \leq c$ 。

2. [完全性（Totality or Connexity）](https://en.wikipedia.org/wiki/Connected_relation)：对于所有的 $a$ 和 $b$， $a \leq b$ 或 $b \leq a$。

有可能 $a \leq b$ 和 $b \leq a$ 同时成立；在这种情况下，两者中的任何一个都可能在排序列表中排在前面。在稳定排序中，输入顺序决定了这种情况下的排序顺序。

文献中研究的比较排序是 “基于比较的”。元素 $a$ 和 $b$ 只有在先前比较结果确立了它们之间的顺序时，才可以被交换或重新排列。当 $a$ 和 $b$ 之间的顺序可以通过这些先前比较结果的传递闭包推导出时，就会出现这种情况。

对于基于比较的排序，执行基本操作的决策（如交换或赋值）基于比较结果。因此，在时间分析中，执行的比较次数用于确定基本操作的上限估计次数。

比较排序的主要优势在于其通用性和灵活性。通过修改比较函数，可以对不同的数据类型进行排序，并且可以对排序顺序进行细致控制。例如，可以通过反转比较函数的结果来实现降序排序，或通过逐个比较元组的各部分来实现字典序排序。

尽管比较排序在许多情况下表现良好，但它也有一些限制。理论上，任何比较排序算法的平均时间复杂度下界为 $O(nlog_{n})$。这是由于比较操作提供的信息有限，导致必须进行足够多的比较来确定正确的排序顺序。

比较排序在计算机科学中有广泛应用，特别是在需要对不同类型的数据进行排序的情况下。例如，在数据库查询中，排序操作通常依赖于比较排序算法。此外，比较排序算法在处理浮点数排序、优先级队列和其他复杂数据结构时也表现出色。

### [排序算法的稳定性（Stability）](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability)

稳定排序算法按照输入中出现的相同顺序对相等的元素进行排序。例如，在下图的纸牌排序示例中，纸牌是按点数排序的，而花色则被忽略。这样，原始列表就有可能出现多个不同的正确排序版本。稳定的排序算法会根据以下规则选择其中一个：如果两个项目比较起来是相等的（比如两张 5 的纸牌），那么它们的相对顺序将被保留，也就是说，如果一个在输入中排在另一个之前，那么它在输出中也会排在另一个之前。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-02-00-55-31.png">
</center>

更正式地说，被排序的数据可以表示为值的记录或元组，用于排序的数据部分称为关键字（*key*）。在纸牌示例中，纸牌被表示为一条记录（点数、花色），关键字是点数。如果有两个具有相同关键字的记录 R 和 S，并且 R 在原始列表中出现在 S 之前，则排序算法是稳定的，则 R 将始终出现在排序列表中的 S 之前。

当相等的元素无法区分时，例如整数，或者更广泛地说，任何以整个元素为键的数据，稳定性都不是问题。如果所有键都不同，稳定性也不是问题。

不稳定的排序算法在关键值相等时不保证元素的相对顺序。为了使其稳定，可以在比较过程中增加一个步骤：如果两个元素的关键值相等，则比较它们在原始列表中的顺序，原先在前的元素仍排在前面。这种做法需要在排序过程中保存原始顺序信息，这可能会增加算法的时间和空间复杂度。

稳定排序算法的一个主要应用是使用主键和次键对列表进行排序。例如，假设我们想对一手牌进行排序，使其花色依次为梅花（♣）、方块（<span style="color:#ff0000">♦</span>）、红心（<span style="color:#ff0000">♥</span>）、黑桃（♠）；在每种花色中，牌按点数排序。这可以通过先按点数排序（使用任何排序算法），然后按花色进行稳定排序来实现。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-02-01-01-03.png">
</center>

在例子中，首先按点数排序，然后按花色进行稳定排序，可以保证在花色相同的情况下，点数的顺序保持不变。这种方法可以扩展到处理多个排序键，例如在基数排序中。在使用不稳定排序算法的情况下，也可以通过使用词典序（lexicographic order）比较的方法来实现稳定排序的效果。即先比较主要键（如花色），再比较次要键（如点数），确保排序结果的正确性和稳定性。

### [序理论（Order theory）](https://en.wikipedia.org/wiki/Order_theory)

序理论是数学的一个分支，研究集合的顺序关系。这些顺序关系包括偏序、全序、弱序和其他更复杂的结构。序理论的应用范围广泛，包括计算机科学、逻辑学和集合论。

#### [偏序（Partially Order）](https://en.wikipedia.org/wiki/Partially_ordered_set)

偏序是一种用于比较和排列集合中元素的关系。与全序不同，偏序中的某些元素可能是不可比较的。偏序关系在数学和计算机科学中有广泛应用，特别是在数据结构和排序理论中。

一个集合 $S$ 上的偏序关系 $\leq$ 是一种二元关系，满足以下三个条件：

1. 自反性（Reflexivity）: 对于集合 $s$ 中的任意元素 $a$，有 $a \leq a$。

2. 传递性（Transitivity）: 对于集合 $s$ 中的任意元素 $a$、$b$ 和 $c$，如果 $a \leq b$ 且 $b \leq c$，那么 $a \leq c$。

3. 反对称性（Antisymmetry）: 对于集合 $s$ 中的任意元素 $a$ 和 $b$，如果 $a \leq b$ 且 $b \leq a$，那么 $a = b$。

在偏序集中，某些元素可能是不可比较的。例如，在集合 $\{1,2,3\}$ 的所有子集上定义的包含关系中，子集 $\{1\}$ 和子集 $\{2\}$ 既不包含于对方也不被对方包含，因此它们是不可比较的。另一个例子是整数集上的整除关系，例如 2 整除 4，但 2 和 3 之间没有整除关系​。

偏序集中有一些特殊的概念，如哈斯图（Hasse Diagram），用于可视化偏序关系。哈斯图通过去除冗余边和自反边，简化了偏序关系的表示。极小元（Minimal Elements）和极大元（Maximal Elements）也是偏序集中的重要概念，极小元是没有其他元素比它小的元素，极大元是没有其他元素比它大的元素。

偏序关系广泛应用于各种数据结构，如树和图。在计算机科学中，偏序关系用于定义优先级队列中的优先级规则，以及数据库查询中的部分排序。理解偏序的定义和特性对于解决复杂的数据排序和组织问题至关重要。

#### [弱序（Weak Order）](https://en.wikipedia.org/wiki/Weak_ordering)

弱序是一种介于偏序和全序之间的关系，用于排列集合中的元素。弱序的定义更加灵活，允许一些元素之间存在相同的排序位置。弱序关系在排序和决策分析中具有重要应用。

一个集合 $S$ 上的弱序关系 $\lesssim$ 满足以下条件：

1. 反身性（Reflexivity）：对于集合 $S$ 中的任意元素 $a$，有 $a \lesssim a$。

2. 传递性（Transitivity）：对于集合 $S$ 中的任意元素 $a$、$b$ 和 $c$，如果 $a \lesssim b$ 且 $b \lesssim c$，那么 $a \lesssim c$。

3. 完全性（Totality or Connexity）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，要么 $a \lesssim b$，要么 $b \lesssim a$，或者两者同时成立。

#### [严格弱序（Strict Weak Order）](https://en.wikipedia.org/wiki/Weak_ordering#Strict_weak_orderings)

严格弱序是弱序的一种特殊形式，它引入了严格比较的概念。严格弱序关系 $\prec$ 满足以下条件：

1. 不反身性（Irreflexivity）：对于集合 $S$ 中的任意元素 $a$，没有 $a \prec a$。

2. 传递性（Transitivity）：对于集合 $S$ 中的任意元素 $a$、$b$ 和 $c$，如果 $a \prec b$ 且 $b \prec c$，那么 $a \prec c$。

3. 不对称性（Asymmetry）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，如果 $a \prec b$，那么不可能 $b \prec a$。

4. 比较性（Comparability）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，要么 $a \prec b$，要么 $b \prec a$，或者两者都不成立。

严格弱序关系的一个显著特性是，它保证了严格比较，即在任何比较中，不存在两个不同的元素互为相等。严格弱序在排序算法中有重要应用，例如用于确保排序过程的稳定性和一致性。一个典型的严格弱序关系例子是集合 $S$ 上的词典顺序。例如，在字符串排序中，"apple" 严格小于 "banana"，并且没有 "banana" 严格小于 "apple" 的情况存在。

在多准则决策分析中，严格弱序关系特别有用，因为它可以帮助区分不同方案的优劣，而不会出现评分相同的情况。例如，在排名系统中，如果两个选项的得分非常接近，但并不完全相同，严格弱序关系可以清楚地指出哪个选项更优。

#### [全预序（Total Preorder）](https://en.wikipedia.org/wiki/Weak_ordering#Total_preorders)

全预序（Total Preorder）是一种用于比较和排列集合中元素的关系，它比全序（Total Order）更为宽松。全预序关系也称为全准序关系（Total Quasi-order），具有以下特性：

一个集合 $S$ 上的全预序关系 $\lesssim$ 是一种二元关系，满足以下三个条件：

1. 反身性（Reflexivity）：对于集合 $S$ 中的任意元素 $a$，有 $a \lesssim a$。

2. 传递性（Transitivity）：对于集合 $S$ 中的任意元素 $a$、$b$ 和 $c$，如果 $a \lesssim b$ 且 $b \lesssim c$，那么 $a \lesssim c$。

3. 完全性（Totality or Connexity）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，要么 $a \lesssim b$ ，要么 $b \lesssim a$ ，或者两者同时成立。

全预序关系的一个显著特性是其可比性，即任意两个元素都可以比较 $a \lesssim b$ 或  $b \lesssim a$ 或两者同时成立。满足总预序定义的关系不一定是全序。全预序允许关系中的一些元素具有相同的 “排序” 或 “等级”，即可能存在 $a \lesssim b$ 且 $b \lesssim a$，但 a ≠ b。在这种情况下我们会认为 a 和 b 是 “等价” 的。这样的性质在评分系统中尤为常见，例如多个选项可能获得相同的评分，从而被认为是等价的。类似地，在字典顺序中，忽略大小写差异时，“Apple” 和 “apple” 被视为等价。

全预序关系在多标准决策分析和数据库排序中也有重要应用。在多标准决策分析中，全预序关系允许将多个选项视为等价，从而简化决策过程。数据库排序也经常需要处理具有等价关系的数据，这时全预序关系可以提供一种有效的排序方法。理解全预序的定义和特性对于解决这些复杂的排序和比较问题至关重要。

#### [全序（Total Order）](https://en.wikipedia.org/wiki/Total_order)

一个集合 $S$ 上的全序是一种二元关系，记作 ≤，满足以下四个条件：

1. 反身性（Reflexivity）：对于集合 $S$ 中的任意元素 $a$，有 $a \leq a$。
    
2. 传递性（Transitivity）：对于集合 $S$ 中的任意元素 $a$、$b$ 和 $c$，如果 $a \leq b$ 且 $b \leq c$，那么 $a \leq c$。
    
3. 完全性（Totality or Connexity）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，要么 $a \leq b$ ，要么 $b \leq a$ 。，或者两者同时成立。
    
4. 反对称性（Antisymmetry）：对于集合 $S$ 中的任意元素 $a$ 和 $b$，如果 $a \leq b$ 且 $b \leq a$，那么 $a = b$。

全序关系的核心特性是所有元素都可以比较，即对于任意 $a$ 和 $b$，要么 $a \leq b$，要么 $b \leq a$。这种完全可比性使得全序关系非常适用于排序算法和数据结构。例如，在自然数集合上，全序关系基于数字的大小顺序。同样，在字典顺序中，字符串可以按照字母表顺序排列。

全序关系在计算机科学中具有广泛应用，特别是在排序和数据管理中。排序算法（如快速排序和归并排序）依赖于全序关系来比较和排列元素。在数据结构中，优先队列和平衡二叉树也依赖全序关系来维护元素的顺序。此外，全序关系在数据库查询中也起着关键作用，如在 SQL 查询中使用 `ORDER BY` 子句进行排序。