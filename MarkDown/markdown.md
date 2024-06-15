
---

快捷键查看设置中的 Hotkeys 部分。其中，标记为 `Blank` 的快捷键则是未设置

---

**标题**
# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6

---

**可折叠的注释**

>[!quote]- 注释 > 内容
> 我是注释

---

**工作区内的文件链接**

[[C_Programming_Language]]

---

**为链接添加别名（可适用于所有类型的链接）**

[[C_Programming_Language|我是一个别名]]

---

**当前文件内的任意位置链接**

AAAA ^ec6912

BBBBB

CCCCC

DDDDD

这是链接到一个任意位置，其中后面的英文乱码为系统自动生成
  [[#^ec6912]]   [[#^ec6912|使用别名]] 

这是连接到一个标题
[[#Heading 2]]  [[#Heading 2|使用别名]]

---

**自定义上下对齐边距**
aaaaaaa
<p style="margin-bottom: 44px;"></p>
aaaaaaa

---

**自定义上下对齐边距**

命令：Higlightr

<mark style="background: #FF5582A6;">我是标红色的红色字体</mark>

命令：Higlightr

<span style="color:#ff0000">我是另一种标红的格式</span>

---

**卡片注释**

> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]  
> Crucial information necessary for users to succeed.

> [!WARNING]  
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action

---

**斜体、粗体**

*This text* ***is*** **italicqwdqwdasdasd**

---

**图片附加说明（建议小图片）**

<div style="display: flex; align-items: flex-start;">
  <div style="flex: 1; margin-right: 8px;">
  冒泡排序，有时也称为下沉排序（Sinking sort），是一种<span class="cm-strong">就地的</span>、<span class="cm-strong">稳定的</span>且<span class="cm-strong">自适应</span>（优化后）排序算法。它逐元素重复地遍历输入序列，将当前元素与其后的元素进行比较，如果需要则交换它们的值。这种操作会重复进行，直到在一次遍历期间不需要执行任何交换，这意味着序列已完全排序。这种比较排序的算法是根据较大的元素 “冒” 到序列的顶部/底部的方式而命名。 
  </div> 
  <img width="100" height="100" style="flex-shrink: 0; margin-top: 15px;" src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-12-18-03-54.gif"> 
</div>

---

**预览模式下的换行方案**

&emsp;

&emsp;&emsp;

<br>

---

**任务列表**

* [x] Task 1
* [x] Task 2
* [ ] Task 3

---

**让任意元素居中显示，比如图片**

普通图片

![image.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-13-01-41-03.gif)

居中且带阴影的图片

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2024-06-13-01-41-03.gif">
</center>

---

**分页标签**

~~~tabs
---tab **最佳时间复杂度**

在最好的情况下，此时的数组为完全有序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换

3. 第 $i$ 次遍历：需要 $n - i$ 比较和交换

因此我们可以得出：$O(n^2) = (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=1}^{n-1} i = \frac{(n - 1)n}{2}$

然而，如果能够在单次循环的过程中知道序列已有序，那么复杂度可降低至：$O(n)$

---tab **最坏时间复杂度**

在最坏的情况下，此时的数组为完全逆序，那么：

1. 第 1 次遍历：需要 $n - 1$ 次比较和交换

2. 第 2 次遍历：需要 $n - 2$ 次比较和交换

3. 第 $i$ 次遍历：需要 $n - i$ 比较和交换

因此我们可以得出：$O(n^2) = (n - 1) + (n - 2) + \cdots + 1 + 0 \\ = \sum_{i=1}^{n-1} i = \frac{(n - 1)n}{2}$
~~~

---

