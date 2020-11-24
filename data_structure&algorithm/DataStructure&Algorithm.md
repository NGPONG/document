# *Data Structure & Algorithm*

### 目录

  - [前言](#前言)
    - [什么是数据结构](#什么是数据结构)
    - [数据结构的分类](#数据结构的分类)
    - [什么是算法](#什么是算法)
    - [算法的特性](#算法的特性)
    - [算法和数据结构之间的联系](#算法和数据结构之间的联系)
  - [Arrary](#arrary)
    - [Basic Arrary](#basic-arrary)
    - [Dynamic Arrary](#dynamic-arrary)
    - [Fibonacci Sequence](#Fibonacci-sequence)
  - [Linked List](#linked-list)
    - [链表与数组](#链表与数组)
    - [链表中的头节点和尾节点](#链表中的头节点和尾节点)
    - [Singly Linked List](#singly-linked-list)
    - [Doubly linked list(待填坑)](#doubly-linked-list)
    - [Singly Circular linked list(待填坑)](#singly-circular-linked-list)
    - [Doubly Circular linked list(待填坑)](#doubly-circular-linked-list)
  - [Stack & Queue](#stack--queue)
    - [Stack 的存储模型](#stack-的存储模型)
    - [Queue 的存储模型](#queue-的存储模型)
    - [Stack 的具体实现](#stack-的具体实现)
  - [串](#串)
    - [KMP](#kmp)
  - [Tree](#tree)
    - [节点的分类](#节点的分类)
    - [节点间的关系](#节点间的关系)
    - [树的其它相关概念](#树的其它相关概念)
    - [树的表示法](#树的表示法)
  - [Binary Tree](#binary-tree)
    - [二叉树的特点和基本形态](#二叉树的特点和基本形态)
    - [特殊的二叉树](#特殊的二叉树)
    - [二叉树的性质](#二叉树的性质)
    - [二叉树的存储结构](#二叉树的存储结构)
    - [二叉树的遍历](#二叉树的遍历)
    - [二叉树的建立](#二叉树的建立)
    - [二叉树的深度](#二叉树的深度)
    - [线索二叉树](#线索二叉树)
    - [树、森林与二叉树之间的转换](#树森林与二叉树之间的转换)
    - [赫夫曼树(没有代码)](#赫夫曼树)
  - [Disjoint Set](#disjoint-set)
    - [并查集的使用](#并查集的使用)
    - [并查集的优化](#并查集的优化)
  - [Graph](#graph)
    - [各种图的定义](#各种图的定义)
    - [图的顶点与边之间的关系](#图的顶点与边之间的关系)
    - [连通图相关概念](#连通图相关概念)
    - [图的抽象数据类型](#图的抽象数据类型)
    - [图的遍历](#图的遍历)
    - [最小生成树](#最小生成树)
    - [最短路径](#最短路径)
    - [拓补排序](#拓补排序)
    - [关键路径 (待填坑)](#关键路径-待填坑)
  - [Search](#Search)
    - [查找的种类](#查找的种类)
    - [顺序表查找](#顺序表查找)
    - [线性索引查找](#线性索引查找)
    - [二叉排序树](#二叉排序树)
    - [AVL树](#AVL树)


<br/>

### 前言

<span id="前言"></span>

---

#### 什么是数据结构
<span id = "什么是数据结构"></span>

数据是描述客观事物的符号，是计算机中可以操作的对象，是能被计算机识别，并输入给计算机处理的符号集合，而数据结构则为负责描述某一类数据具体的类型的一种结构，更具体来讲，<span style="color:red">数据结构是计算机存储、组织数据的方式，它是相互之间存在一种或多种特定关系的数据元素的集合</span>，通常情况下，精心选择的数据结构可以带来更高的运行或者存储效率

<br/>

#### 数据结构的分类
<span id = "数据结构的分类"></span>

按照视点的不同，我们把数据结构分为 <font color="red">逻辑结构</font> 和 <font color="red">物理结构</font>


- 逻辑结构

    - 集合结构

    集合结构内的各个数据元素是平等的，他们共同属于同一个集合，需要注意的是，<font color = "red">虽然集合内的元素是处在同一个集合，但是它们之间并没有什么实际性的联系</font>，故该模型通常应用于数学模型结构，而在程序中通常不对于它展开额外的探讨
    
    ```c
    +------------------------------+
    | +-----+             +-----+  |
    | |     |   +-----+   |     |  |
    | |  A  |   |     |   |  C  |  |
    | |     |   |  B  |   |     |  |
    | +-----+   |     |   +-----+  |
    |   +-----+ +-----+  +-----+   |
    |   |     |          |     |   |
    |   |  F  |  +-----+ |  D  |   |
    |   |     |  |     | |     |   |
    |   +-----+  |  E  | +-----+   |
    |            |     |           |
    |   +-----+  +-----+ +-----+   |
    |   |     |  +-----+ |     |   |
    |   |  G  |  |     | |  H  |   |
    |   |     |  |  I  | |     |   |
    |   +-----+  |     | +-----+   |
    |            +-----+           |
    +------------------------------+
    ```
    
    - 线性结构

    线性结构中的数据元素之间是 $1 : 1$ 的关系，即除开首元素和尾元素外，每个元素之间都能够找到其唯一的前驱和后继

    ```c
    +---+      +---+      +---+      +---+
    | A +----->+ B +----->+ C +----->+ D |
    +---+      +---+      +---+      +-+-+
                                       |
                                       |
                                       v
                          +---+      +-+-+
                          | F +<-----+ E |
                          +---+      +---+
    ```

    - 树形结构

    树形结构中的数据元素之间是 $1 : n$ 的关系，即除开根元素和叶子元素外，每个元素之间都能够找到唯一的前驱和多个后继

    ```C
                    +---+
                    | A |
                    +-+-+
                      |
                      |
            +-------------------+
            |         |         |
            |         |         |
            v         v         v
          +-+-+     +-+-+     +-+-+
          | B |     | C |     | D |
          +-+-+     +---+     +-+-+
            |                   |
            |                   |
      +-----------+             |
      |     |     |             |
      v     v     v             |
    +-+-+ +-+-+ +-+-+         +-+-+
    | E | | F | | G |         | H |
    +---+ +---+ +---+         +---+
    ```

    - 图形结构

    树形结构中的数据元素之间是 $n : n$ 的关系，即每个元素之间都可能会找到多个全驱和后继

    ```c
    +---------------------------------+
    |                                 v
    |                      +---+    +-+-+
    |       +------------->+ E |    | F |
    |       |              +-+-+    +-+-+
    |     +-+-+              |        |
    +-----+ A +------+       |        |
          +-+-+      |       +---+----+
            ^        |           |
            |        |           |
            |        v           v
          +-+-+    +-+-+       +-+-+
          | B +--->+ C +<------+ G |
          +-+-+    +-+-+       +---+
            |        ^
            |        |
            v        |
          +-+-+      |
          | D +------+
          +---+
    ```

- 物理结构

    - 顺序存储

    顺序存储是把数据元素存放在地址连续的存储单元里，其数据的逻辑关系和物理关系是一致的

    ```c
     +---------------------------->   +---------+------------|
     |                                |0000 0000|            |
     |                0x0000000B <----+---------+ 0x00000008 |
     |                                |0000 0000|            |
     |                0x0000000A <----+---------+     ~      |
     |                                |0000 0000|            |
     |                0x00000009 <----+---------+ 0x0000000B |
     |                                |0000 0011|            |
     |                0x00000008 <----+---------+------------|
     |                                |0000 0000|            |
     |                0x00000007 <----+---------+ 0x00000004 |
     |                                |0000 0000|            |
     |                0x00000006 <----+---------+     ~      |
     |                                |0000 0000|            |
     |                0x00000005 <----+---------+ 0x00000007 |
     |                                |0000 0010|            |
     |                0x00000004 <----+---------+------------|
     |                                |0000 0000|            |
     |                0x00000003 <----+---------+ 0x00000000 |
     |                                |0000 0000|            |
     |                0x00000002 <----+---------+     ~      |
     |                                |0000 0000|            |
     |                0x00000001 <--------------+ 0x00000003 |
     |                                |0000 0001|            |
     +------------>   0x00000000 <----+---------+------------+
    ```

    - 链式存储结构

    链式存储结构是把数据元素存放在任意的存储单元里，这组存储单元可以是连续的，也可以是不连续的，元素的存储关系并不能反映其逻辑关系，因此需要用一个指针存放数据元素的地址，这样通过地址就可以找到相关数据的位置

    ```c
    0xAAAA   +--->0xBBBB   +--->0xCCCC   +--->0xDDDD   +--->0xEEEE   +--->0xFFFF
    +----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
    |Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|
    +----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
    |Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+----> NULL
    +----+        +----+        +----+        +----+        +----+        +----+
    ```

<br/>

#### 什么是算法

<span id = "什么是算法"></span>

算法是特定问题求解步骤的描述，在计算机中表现为指令的有限序列，算法是独立存在的一种解决问题的方法和思想，
对于算法而言，语言并不重要，重要的是思想

<br/>

#### 算法的特性

<span id = "算法的特性"></span>

- 输入输出：算法具有零个或多个输入、至少有一个或多个输出
- 有穷性：指算法在执行有限的步骤之后，自动结束而不会出现无限循环，并且每一个步骤在可接受的时间内完成
- 确定性：算法的每一步骤都有确定的含义，不会出现二义性
- 可行性：算法的每一步都必须是可行的，也就是说，每一步都能通过执行有限次数完成


<br/>

#### 算法和数据结构之间的联系

<span id = "算法和数据结构之间的联系"></span>

算法是为了解决实际问题而设计，而数据结构只是静态的描述了数据元素之间的关系，它是算法需要处理问题的一种载体，即算法和数据结构之间是属于一种 <font color="red">相辅相成</font> 的关系，<font color="red">一个高效的应用程序需要在数据结构的基础上设计和选择算法</font>


<br/>

### Arrary

<span id="Arrary"></span>

---

#### Basic Arrary

<span id = "Basic-Arrary"></span>

作为基础数据类型的一种，其应用对我们来说再熟悉不过，在我们声明了一个数组后，<font color = "red">数组的每个成员的内存地址都是连续且有序的</font>，关于这点特性我们就能够总结出数组的优点和缺点

- 优点
    - 对于元素的随机访问效率更高，即每个元素之间仅通过地址的偏移就能够访问到其他元素的内容
    - 对比链表，数组在预先知道预存数据总量的情况下，数组对比链表对于空间的开销更小
    - 由于数组属于一种基础数据类型，我们对于数组的操作和使用就体验来说对于开发人员更为友好
- 缺点
    - 如果在数组首元素和数组尾元素之间 ( 包括首元素但不包括尾元素 ) ，对于新元素的插入或者旧元素的删除操作，那么效率会非常低。假定声明了一个长度非常长的数组，因为元素与元素之间的内存地址是连续的，如果在这之中进行了插入操作，比如说在第一个元素的位置上插入一个其它的元素，这时候我们需要把第一个元素后面元素的都需要往后偏移一位，可想而知这个工程量带来了多少 <font color = "red">性能损失</font>，除了性能损失之外，在 c 中，如果当前操作在数组的长度已经达到最大阈值的基础下进行，那么我们还需要考虑 <font color="red">内存安全的问题</font>，还有一点的是，由于往后偏移我们无非就是遍历开始偏移位置的元素，逐元素往后复制一份，由于是赋值操作，所以还需要考虑到 <font color = "red">深浅拷贝的问题</font>

    - 数组在声明的过程中就已经给定了一块线性连续的内存空间，如果在不知道预存数据量总量的情况下，那么在给定数据量小于数组所声明的大小的话，会出现对于内存空间使用的浪费的问题，那么在给定数据量大于数组所声明的大小的话，就会出现无法对数组长度在进行动态扩展的窘境

    - 与链表相比，在某种情况下对于数组的操作是不太安全的，因为对于数组的操作我们都需要通过地址的偏移来获取到指定内存的地址再拿到里面的数据，这就有可能会出现偏移过多导致访问到野指针的情况，而相对于链表，由于节点与节点之间是通过地址相连在一起，故我们是可以通过查看当前节点所存储的下一节点的指针是否为空的方式来判断所连接的下一节点是否是一个野指针的问题

关于数组的使用方式，在这篇博客中不进行展开，详情参考 [这一章](https://github.com/NGPONG/document/blob/master/C/2019/12_18/C_Programming_Language.md) 博客中对于数组使用方式的详解

!!! INFO

    对于 [[Basic Arrary]] 来说，虽然其本身存在着优点，但是不可否认的是，它也有许许多多的一些短板，对于这些短板，前辈们都想出了各式各样的数据结构来填补这些缺陷，但，对于数据结构来说永远没有 [[Perfect]] 这一单词，可能某个数据结构解决了一些根本性的问题，但是随之而来的又会产生一些新的问题


<br/>

#### Dynamic Arrary

<span id = " q-Arrary"></span>

动态数组属于对 $Basic Arrary$ 的一项扩展内容，它在 <font color = "red">保持 Basic Arrary 本质不变的情况下，还解决掉了 Basic Arrary 无法进行动态扩展的短板</font>

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct dynamicArrary { 
	void **ptr_arrary;
	int length;
};

/**
 * @description: Checking the available length of an array
 * @param {type} 
 * @return: 
 */
int CheckArraryAvailableLength(struct dynamicArrary *arrary) { 
	if (arrary->length == capacity) { 
		int flag = ReallocArrary(&arrary->ptr_arrary, arrary->length);
		if (!flag) { 
			printf("[Realloc complete]  struct address = %p, arrary adderss = %p\n", arrary, arrary->ptr_arrary);
		}
	}

	return 0;
}

/**
 * @description: Dynamically expand the length of an array
 * @param {type} 
 * @return: 
 */
int ReallocArrary(void ***p_ptr_arrary, int length) { 
	capacity += 10;
	void **arrary_new = malloc(sizeof(void *) * capacity);
	if (arrary_new == NULL) 
		return -1;

	/*move the data from the old array to the new array*/
	memcpy(arrary_new, *p_ptr_arrary, sizeof(void *) * length);

	free(*p_ptr_arrary);
	*p_ptr_arrary = arrary_new;

	return 0;
}

/**
 * @description: Initialize a dynamic array
 * @param {type} 
 * @return: 
 */
struct dynamicArrary *InitDynamicArrary(int m_capacity) { 
	if (m_capacity <= 0) 
		return NULL;

	capacity = m_capacity;

	struct dynamicArrary *arrary = malloc(sizeof(struct dynamicArrary));
	if (arrary == NULL) 
		return NULL;
	memset(arrary, 0, sizeof(struct dynamicArrary));

	arrary->ptr_arrary = malloc(sizeof(void *) * m_capacity);
	if (arrary->ptr_arrary == NULL) 
		return NULL;
	memset(arrary->ptr_arrary, 0, sizeof(void *) * m_capacity);

	return arrary;
}

/**
 * @description: Insert a new element into arrary
 * @param {type} 
 * @return: 
 */
int Insert(struct dynamicArrary *arrary, int index, void *value) { 
	if (index < 0) 
		return -1;
	if (CheckArraryAvailableLength(arrary)) 
		return -1;
	if (index >= arrary->length) 
		index = arrary->length;

	for (int i = arrary->length - 1; i >= index; i--) 
		memcpy(arrary->ptr_arrary + i + 1, arrary->ptr_arrary + i, sizeof(void *));

	/*insert*/
	memcpy(arrary->ptr_arrary + index, &value, sizeof(void *));

	arrary->length++;
	return 0;
}

/**
 * @description: Add a new element into arrary
 * @param {type} 
 * @return: 
 */
int Add(struct dynamicArrary *arrary, void *value) { 
	if (CheckArraryAvailableLength(arrary)) { 
		return -1;
	}

	memcpy(arrary->ptr_arrary + arrary->length, &value, sizeof(void *));
	arrary->length++;
	return 0;
}

/**
 * @description: Remove element by postion
 * @param {type} 
 * @return: 
 */
int RemoveByPostion(struct dynamicArrary *arrary, int index) { 
	if (index < 0) {

		return -1;
	}
	if (index >= arrary->length) { 
		index = arrary->length;
	}

	free(*(arrary->ptr_arrary + index));

	for (int i = index + 1; i < arrary->length; i++) { 
		*(arrary->ptr_arrary + i - 1) = *(arrary->ptr_arrary + i);
		*(arrary->ptr_arrary + i) = NULL;
	}

	arrary->length--;

	return 0;
}

/**
 * @description: Remove element by value
 * @param {type} 
 * @return: 
 */
int RemoveByValue(struct dynamicArrary *arrary, void *value, int (*invoker)(void *data_source, void *data_compare)) { 
	if (arrary == NULL || invoker == NULL) 
		return -1;

	for (int i = 0; i < arrary->length; i++) { 
		if (invoker(*(arrary->ptr_arrary + i), value)) { 
			RemoveByPostion(arrary, i);
			break;
		}
	}

	return 0;
}

/**
 * @description: Dispose dynamic arrary
 * @param {type} 
 * @return: 
 */
int DestoryArrary(struct dynamicArrary *arrary) { 
	if (arrary == NULL) {  
		return -1;
	}

	printf("are you sure destory arrary? [yes/no]\n");

	char str_CustomerInput[8] = { 0 };
	fgets(str_CustomerInput, sizeof(str_CustomerInput), stdin);

	if (strcmp(str_CustomerInput, "yes\n")) 
		return -1;

	for (int i = 0; i < arrary->length; i++) 
		free(*(arrary->ptr_arrary + i));

	free(arrary->ptr_arrary);
	free(arrary);

	capacity = 0;
}
```

<br/>

#### Fibonacci sequence

<span id = "Fibonacci-sequence"></span>

如果一个数列中的数字遵循着 <font color = "red">"前面两项相邻之和用于构成后一项"</font> 这一规律，那么它就是一个斐波那契数列，如下图所示 : 

![2020-11-05-22-03-23](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-05-22-03-23.png)

对于这样一串数列，我们通常会研究该数列中具体的第 $n$ 项的结果究竟是什么，为了获取到该值，使用以下公式来完成 : 

> $
f(n) = \begin{cases}
   0 &\text{if } \:\: n = 0 \\
   1 &\text{if } \:\: n = 1 \\
	 f(n-1) + f(n -2) &\text{if} \:\: n > 1
\end{cases}
$ 

通常，使用递归来实现是一种高效且简洁的编写方案，其代码如下 : 

```c
int fibonacci(unsigned int n) {
  if(n < 2) return n == 0 ? 0 : 1;

  return fibonacci(n - 1) + fibonacci(n - 2);
}

int main(void) {
	/* 打印前斐波那契数列中的前 40 个数 */
	for (size_t i = 0; i < 40; ++i) {
    printf("%d(%d)\n", fibonacci(i), i);
  }

	return EXIT_SUCCESS;
}
```

采用递归实现，在数据量大的时候，$f(n)$ 需要不断地进行往下的分割才能得出算式结果，这就造成了函数需要不断的执行入栈操作，而最上层的栈帧往往需要等到下层栈帧的释放才能释放，这就导致了内存消耗严重，故对于斐波那契数列求值其实我们还可以使用递推的方式来实现

```c
int main(void) {
  int fib_seq[40] = { 0 };

  int i = 0;
  fib_seq[i++] = 0;
  fib_seq[i++] = 1;

  for (i; i < 40; ++i) {
    fib_seq[i] = fib_seq[i - 1] + fib_seq[i - 2];
  }

  /* 打印前斐波那契数列中的前 40 个数 */  
  for(i = 0; i< 40; ++i>) {
    printf("%d\n", fib_seq[i]);
  }

  return EXIT_SUCCESS;
}
```

<br/>

<span id="linkedlist"></span>

### Linked List

---

链表是一种常用的数据结构，它由一系列的 $node$ 通过指针的方式相互连接起来，以此形成一条数据链，而每个节点中主要是维护着两种重要的信息，它们分别是：

- $Data$：数据域 

    存放着当前节点所存储的信息数据
    
- $Pointer$：指针域

    存放着其它节点在内存中的首地址，即每一个节点都能够指向 上一个 / 下一个 节点的信息

<br/>

#### 链表与数组

<span id="链表与数组"></span>

对于链表来说，其通常用于和 $Arrary$ 做对比，由于链表是通过节点与节点之间的连接来完成一条数据链的构造，故该特性能够解决掉 Arrary 本身所存在的许多缺点，但是相对的，更是由于这一特性，链表在解决了 Arrary 本身所存在的缺陷的同时却无法保留掉原有数组本身的 <font color = "red">优点</font>，即 链表 在解决了 Arrary 本身所存在的缺陷的同时又把 Arrary 身上的优点转换为了 链表 所带有的缺陷

- 优点
    - 链表对比数组，在链表头至链表尾中删除或新增某个节点的时候效率会很高，因为我们只用改变当前位置上，前一个节点或者是后一个节点的指针指向即可完成插入或者是删除的操作
    - 链表在建立的时候无需知道数据总量的大小，我们可以随时的对链表进行扩展和分配新的节点空间，这间接的解决了数组对于开辟空间较大而导致的内存浪费的问题
    - 链表的操作对比数组来说是更为安全的，因为对于数组的操作我们都需要通过地址的偏移来获取到指定内存的地址再拿到里面的数据，这就有可能会出现偏移过多导致访问到野指针的情况，而相对于链表，由于节点与节点之间是通过地址相连在一起，故我们是可以通过查看当前节点所存储的下一节点的指针是否为空的方式来判断所连接的下一节点是否是一个野指针的问题

- 缺点
    - 对于元素的随机访问效率较低，因为节点与节点之间是通过指针的方式连接在一起，并且节点与节点之间的内存地址不是连续的，故我们无法做到像数组那样直接通过地址偏移就能访问到某个下标的元素，想访问某个下标的节点的信息我们只能去遍历这张链表，直到达到指定下标的节点
    - 由于链表的本身需要由多种成员来构成 ( 指针域、数据域、…… )，即链表的使用在一定程度上增加了空间复杂度
    - 链表在 c 语言中并不是语言标准的支持，故操作起来没有数组较为方便

<br/>

#### 链表中的头节点和尾节点

<span id="链表中的头节点和尾节点"></span>

什么是链表的头节点和尾节点？

一张链表在初始化完毕后，我们通常会额外添加两个特殊的节点，这两个节点仅仅只是维护了一个指针域，指针域指向这张链表的头节点和尾节点在内存中的首地址，而这两种特殊的节点就为链表的 <font color = "red">头节点</font> 和 <font color = "red">尾节点</font>

为什么需要链表的头节点和尾节点？

先拿头节点来说，如果我们有一个地方能够存储一张链表的头节点的话，就能够对于这张链表起到一个标志位的作用，那么在后面，不管我们对这张链表进行如何操作，我们都可以恒定的知道这张链表起始节点的位置，假设我们对于一张链表遍历到某个节点的时候，这时候其实就已经改变了这张链表中具体某个节点位置的指向了，简而言之就是不在链表的起始位置上，这时候我们还需要对它进行重新遍历的话，那么这个头节点的标志位就起到了标志性的作用，对于一张合格的链表数据结构的定义来说，<font color="red">头节点的存在是必须的</font>

而尾节点的作用在于能够快速的获取到链表尾部节点的位置，以方便对链表进行节点扩展的操作，如果没有尾节点，那么我们想要为一个链表新增节点成员的话，那么我们就需要从头到尾遍历一次整张链表，直至拿到最后一个节点的信息我们才能进行新增节点的操作

<br/>

#### Singly Linked List

<span id="Singly-Linked-List"></span>

单链表的存储模型
```c
  Head                                                                  Tail
+-------+                                                             +-------+
|Pointer|                                                             |Pointer|
+---+---+                                                             +---+---+
    |                                                                     |
    |                                                                     |
    v                                                                     v
 0xAAAA   +--->0xBBBB   +--->0xCCCC   +--->0xDDDD   +--->0xEEEE   +--->0xFFFF
 +----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
 |Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|
 +----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
 |Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+----> NULL
 +----+        +----+        +----+        +----+        +----+        +----+
```

单链表的实现

```c
typedef struct node { 
	void *data;
	struct node *next;
} node_t;

typedef struct list { 
	node_t *head;
	node_t *tail;
	int size;
} list_t;


/**
 * @description: Initialize a singly linked list
 * @param {type} 
 * @return: 
 */
list_t *Create() { 
	list_t *list = malloc(sizeof(list_t));
	if (list == NULL) { 
		perror("heap out");
		return NULL;
	}
	memset(list, 0, sizeof(list_t));

	return list;
}

/**
 * @description: Add a new node into singly linked list
 * @param {type} 
 * @return: 
 */
void AddNode(list_t *list, void *data) { 
	if (list == NULL) { 
		return;
	}

	// create a new node
	node_t *node = malloc(sizeof(node_t));
	if (node == NULL) { 
		perror("heap error");
		return;
	}
	node->data = data;
	node->next = NULL;

	if (list->head == NULL) { 
		list->head = list->tail = node;
	} else { 
		list->tail->next = node;
		list->tail = node;
	}

	list->size++;
}

/**
 * @description: foreach singly linked list
 * @param {type} 
 * @return: 
 */
void ForeachAllNode(list_t *list,void (*invoker)(node_t *)) { 
	if (list == NULL || list->head == NULL) { 
		return;
	}

	node_t *current = list->head;
	while (1) { 
		invoker(current);

		if ((current = current->next) == NULL) { 
			printf("foreach ended******************************************\n");
			printf("head address = %p\n", list->head);
			printf("tail address = %p\n", list->tail);
			printf("list address = %p\n", list);
			printf("list size = %d\n\n", list->size);
			break;
		}
	}
}

/**
 * @description: Insert a new node in singly linked list by postion
 * @param {type} 
 * @return: 
 */
void Insert(list_t *list, int index, void *data) { 
	if (list == NULL) { 
		return;
	}

	node_t *node = malloc(sizeof(node_t));
	if (node == NULL) { 
		return;
	}

	node->data = data;
	node->next = NULL;

	if (index < 0) { 
		return;
	}
	else if (index == 0) { 
		node->next = list->head;
		list->head = node;
	} else if (index > list->size - 1) {
		list->tail->next = node;
		list->tail = node;
	} else { 
		node_t *node_bef = list->head;
		for (size_t i = 0; i < index - 1; i++) {

			node_bef = node_bef->next;
		}

		node_t *node_aft = node_bef->next;
		node_bef->next = node;
		node->next = node_aft;
	}

	list->size++;
}

/**
 * @description: Delete node in singly linked list by postion
 * @param {type} 
 * @return: 
 */
void Delete(list_t *list, int index) {
	if (list == NULL || list->head == NULL || list->tail == NULL) return;

	if (index < 0) {
		return;
	} else if (index == 0) {
		node_t *head = list->head;
		printf("delete and free %p\n", head);

		list->head = list->head->next;
		if (list->size == 1) {

			list->tail = list->head;
		}
		free(head);
	} else if (index >= list->size - 1) {
		node_t *node_current = list->head;
		if (node_current->next == NULL) {
			printf("delete and free %p\n", node_current);
			free(node_current);
			list->head = NULL;
			list->tail = NULL;
			list->size--;
			return;
		}


		while (1) {
			if (node_current->next == list->tail) {
				break;
			}

			node_current = node_current->next;
		}

		node_t *tail = list->tail;
		printf("delete and free %p\n", tail);
		list->tail = node_current;
		list->tail->next = NULL;
		free(tail);
	} else {
		node_t *node_aft = list->head;
		for (size_t i = 0; i < index - 1; i++) 
			node_aft = node_aft->next;
		}

		node_t *node_prev = node_aft->next->next;
		
		printf("delete and free %p\n", node_aft->next);
		free(node_aft->next);

		node_aft->next = node_prev;
	}


	list->size--;
}

/**
 * @description: Reverse singly linked list
 * @param {type} 
 * @return: 
 */
void Reverse_List(list_t *list) {
	if (list == NULL) {
		return;
	}

	node_t *prev = NULL;
	node_t *next = NULL;
	node_t *current = list->head;
	for (size_t i = 0; i < list->size; i++) {

		next = current->next;

		current->next = prev;

		// move
		prev = current;
		current = next;
	}

	node_t *temp = list->head;
	list->head = list->tail;
	list->tail = temp;
}

/**
 * @description: Dispose singly linked list
 * @param {type} 
 * @return: 
 */
void Free(list_t *list) {
	node_t *current = list->head;
	node_t *next_temp;
	while (1) {
		if ((next_temp = current->next) == NULL) {

			printf("free node address = %p, Data address = %p\n", list->tail, list->tail->data);
			free(list->tail->data);
			free(list->tail);
			break;
		}

		printf("free node address = %p, Data address = %p\n", current, current->data);
		free(current->data);
		free(current);

		current = next_temp;
	}

	printf("free list address = %p\n", list);
	free(list);
}
```
<br/>

#### Doubly linked list

<span id="Doubly-Linked-List"></span>

双链表的存储模型

```c
            Head                                                                      Tail
          +-------+                                                                 +-------+
          |Pointer|                                                                 |Pointer|
          +---+---+                                                                 +---+---+
              |                                                                         |
              |                                                                         |
              v                                                                         v
           0xAAAA<--+ +-->0xBBBB<--+ +-->0xCCCC<--+ +-->0xDDDD<--+ +-->0xEEEE<--+ +-->0xFFFF
           +----+   | |   +----+   | |   +----+   | |   +----+   | |   +----+   | |   +----+
NULL <-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|
           +----+     |   +----+     |   +----+     |   +----+     |   +----+     |   +----+
           |Data|     |   |Data|     |   |Data|     |   |Data|     |   |Data|     |   |Data|
           +----+     |   +----+     |   +----+     |   +----+     |   +----+     |   +----+
           |Next+-----+   |Next+-----+   |Next+-----+   |Next+-----+   |Next+-----+   |Next+----> NULL
           +----+         +----+         +----+         +----+         +----+         +----+
```

<br/>

#### Singly Circular linked list

<span id="Singly-Circular-linked-list"></span>

单向循环链表的存储模型

```c
     Head                                                                 Tail
   +-------+                                                          	 +-------+
   |Pointer|                                                          	 |Pointer|
   +---+---+                                                             +---+---+
       |                                                                     |
       |                                                                     |
       v                                                                     v
+-->0xAAAA   +--->0xBBBB   +--->0xCCCC   +--->0xDDDD   +--->0xEEEE   +--->0xFFFF
|	+----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
|	|Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|   |    |Data|
|	+----+   |    +----+   |    +----+   |    +----+   |    +----+   |    +----+
|	|Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+---+    |Next+----+
|	+----+        +----+        +----+        +----+        +----+        +----+    |
|                                                                                   |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

<br/>

#### Doubly Circular linked list

<span id="Doubly-Circular-linked-list"></span>

双向循环链表的存储模型

```c
+--------------------------------------------------------------------------------------------+
|                                                                                            |
|                                                                                            |
| 	   Head                                                                      Tail        |
|	 +-------+                                                                 +-------+     |
|	 |Pointer|                                                                 |Pointer|     |
|	 +---+---+                                                                 +---+---+     |
| 	     |                                                                         |         |
|	     |                                                                         |         |
| 	     v                                                                         v         |
| +-->0xAAAA<--+ +-->0xBBBB<--+ +-->0xCCCC<--+ +-->0xDDDD<--+ +-->0xEEEE<--+ +-->0xFFFF<-----+
| |   +----+   | |   +----+   | |   +----+   | |   +----+   | |   +----+   | |   +----+
+-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|   +-----+Last|
  |	  +----+     |   +----+     |   +----+     |   +----+     |   +----+     |   +----+
  |	  |Data|     |   |Data|     |   |Data|     |   |Data|     |   |Data|     |   |Data|
  |	  +----+     |   +----+     |   +----+     |   +----+     |   +----+     |   +----+
  |	  |Next+-----+   |Next+-----+   |Next+-----+   |Next+-----+   |Next+-----+   |Next+---+
  |	  +----+         +----+         +----+         +----+         +----+         +----+   |
  |                                                                                       |
  |                                                                                       |
  |                                                                                       |
  |                                                                                       |
  +---------------------------------------------------------------------------------------+
```

<br/>

<span id = "stackqueue"></span>

### Stack & Queue

---


把 $Stack$ 和 $Queue$ 放在一起的原因是因为两者的实现的是非常的相似

对于 $Stack$ 来说其实现需要遵循 $FILO$ 原则

对于 $Queue$ 来说其实现需要遵循 $FIFO$ 原则

<br/>

#### Stack 的存储模型

<span id = "Stack的存储模型"></span>

```c
BOTTOM
   ↑
   +
   +------------------+
   |		          |
   |  +------------+  |
   |  |     A      |  |
   |  +------------+  |
   |		          |
   |  +------------+  |
   |  |     B      |  |
   |  +------------+  |
   |		          |
   |   ^           +  |
   |   +           v  |
   | PUSH         POP |
   +                  +
   ↓
  TOP
```

<br/>

#### Queue 的存储模型

<span id = "Queue的存储模型"></span>

```c
   BACK <----------------------------------> FONT

         +-------------------------------+
         +---+   +---+ +---+ +---+   +---+
PUSH +-->+ E |   | D | | C | | B |   | A +---> POP
         +---+   +---+ +---+ +---+   +---+
         +-------------------------------+
```

<br/>

#### Stack 的具体实现

<span id = "Stack的具体实现"></span>

在这篇 Blog 中，我并不打算对 $Queue$ 的存储结构进行实现，因为二者实现起来相似度还是挺高，故本 Blog 中只是针对 $Stack$ 进行了一个实现，下面代码中则为实现了一个 <font color = "red">顺序存储</font> 的字符栈

```c
#define MAX_LENGTH 0x400

struct Stack {
	char *s_data;
	int s_size;
};

/**
 * @description: Initialize an char stack
 * @param {type} 
 * @return: 
 */
struct Stack *Init_Stack() {
	struct Stack *stack = malloc(sizeof(struct Stack));
	if (stack == NULL) {
		return NULL;
	}

	stack->s_data = malloc(sizeof(char) * MAX_LENGTH);
	if (stack->s_data == NULL) {
		return NULL;
	}

	stack->s_size = 0;
	memset(stack->s_data, 0x0, sizeof(struct Stack));

	return stack;
}

/**
 * @description: Push an new element into stack
 * @param {type} 
 * @return: 
 */
int Push(struct Stack *stack, char data) {
	if (stack == NULL || stack->s_size == MAX_LENGTH) {
		return -1;
	}

	*(stack->s_data + stack->s_size) = data;
	stack->s_size++;
	
	return 0;
}

/**
 * @description: Pop an element into stack
 * @param {type} 
 * @return: 
 */
int Pop(struct Stack *stack) {
	if (stack == NULL || stack->s_size == 0) {
	
		return -1;
	}

	char value = *(stack->s_data + (stack->s_size - 1));
	*(stack->s_data + (stack->s_size - 1)) = 0;
	stack->s_size--;
	return value;
}

/**
 * @description: Dispose an char stack
 * @param {type} 
 * @return: 
 */
int Destory(struct Stack *stack) {
	if (stack == NULL || stack->s_data == NULL) {
		return -1;
	}

	free(stack->s_data);
	free(stack);

	return 0;
}
```

!!! INFO PS
	关于 Stack 较为经典的运用就是程序运行时所初始化的线程栈了，关于线程栈的一些概念在 [另一篇](https://github.com/NGPONG/document/blob/master/C/2019/12_18/C_Programming_Language.md) 文章中已提过，这里就不进行额外的探讨了


<br/>

<span id = "串"></span>

### 串

---

关于<font color = "red">串$(String)$</font>的具体定义本文不再细说，因为对于串来说，其抽象数据结构应属我们所见过最多的，这里仅描述串数据结果中所衍生出来的一些算法的应用


#### KMP

KMP 算法的核心在于匹配串 $partten$ 的 $Prefix-table$，即字符前缀表，<font color = "red">它的值为匹配串挨字节拆分出所有的字串的最大公共前后缀的值</font>

在正式匹配字符串步骤时，就可以使用上一步所计算出来的匹配串的 $Prefix-table$ ; 挨字符匹配，当存在一个字符不相同时，指向匹配串的指针则依据当前锁定为到的匹配串的下标更改为 $Prefix-table$ 中找到的值，这时候其实我们就会发现，<font color = "red">原始串的位置和匹配串的位置无非就是跳过了前面一些已经匹配过的内容，并使用匹配串的前缀对齐到了原始串的后缀，是的，KMP算法的核心就是跳过一些没必要匹配的内容来完成的字符串匹配算法</font>

```c
#include <math.h>
#include <time.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

/* @brief 构造前缀表 */
void prefix_table(char *str, int *prefix) {
  prefix[0] = 0; /* 下标为0的prefix始终为0 */

  int len = 0;
  int i = 1;
  int n = strlen(str);
  while (i < n) {
    /**
     * len 作为上一个字符的最大公共前后缀长度，它的值
     * 在字符串 str 串中所指向的值，其实就是作为，如果
     * 下一个字符是该值的话，那么就意味着下一个字符继续
     * 连接着上一个字符所形成的公共前后缀，那么所对应的
     * len 就继续 +1
    */
    if (str[i] == str[len]) {
      ++len;
      prefix[i] = len;
      ++i;
    } else { /* 否则的话，那么就让当前 len 指向已构造前缀表的上一个为止，直至减到 0 为止 */
             /* 事实上，这段代码主要是为了解决串中最后一位无法使用通常手段(进入else条件的话
              * 就意味着当前下标的 prefix 直接为 0 即可)来完成所加的，而所构造的前缀表最终
              * 还要右移一位，并且第 0 为赋值为 -1 以保证在 kvm 算法中能够更方便的计算 */
      if (len > 0) {
        len = prefix[len - 1];
      } else {
        prefix[i] = len;
        i++;
      }
    }
  }

  for (size_t i = n - 1; i > 0; --i) {
    prefix[i] = prefix[i - 1];
  }
  prefix[0] = -1;
}

void kmp(char *str_src, char *partten) {
  int n = strlen(partten);
  int prefix[n];
  memset(prefix, 0x0, sizeof(prefix));

  prefix_table(partten, prefix);
  for (size_t i = 0; i < sizeof(prefix) / sizeof(int); ++i) {
    printf("%d\n", prefix[i]);
  }

  /* start match */
  int j = 0; /* point to prefix table */
  int i = 0; /* point to source string */
  int m = strlen(str_src);
  while (i < m) {
    if(j == n - 1 && str_src[i] == partten[j]) {
      /**
       * 如果 partten 匹配到最后一位也匹配成功，那
       * 么如果想要继续往下匹配字符串的话，则另 j 的
       * 值指向对应 prefix 的值继续匹配即可 
      */
      printf("Found partten %d\n", i - j);
      j = prefix[j];
    } 

    if(str_src[i] == partten[j]) {
      i++;
      j++;
    } else {
      j = prefix[j];
      if(j == -1) {
        j++;
        i++;
      }
    }
  }
}

int main(int argc, char *argv[]) {
  char *partten = "ABABCABAA";
  char *str_src = "ABABABABCABAAB";

  kmp(str_src, partten);
  
  return EXIT_SUCCESS;
}
```


<br/>

<span id = "Tree"></span>

### Tree

---

上面所讨论都是 $1 : 1$ 的线性结构，可现实中，还有很多 $1 : n$ 成员排列方式的情况需要处理，针对这种情况我们采用 <font color = "red">树形结构</font> 去描绘成员 ( 数据 ) 之间的逻辑关系更为合适

<span id = "树的定义"></span>

!!! INFO 树的定义
	树 ( $Tree$ ) 是 $n$ ( $n \geq 0$ ) 个节点的有限集
		
	1. $n = 0$ 时称为 <font color = "red">空树</font>

	2. $n > 0$ 是称为 <font color = "red">非空树</font>，在任意一颗 非空树 中：

	 	2.1 有且只有一个特定的称为根 ( $Root$ ) 的节点；
		 
	 	2.2 当 $n > 1$ 时，其余节点可分为 $m$ ( $m > 0$ ) 个 <font color = "red"> 互不相交</font> 的有限集 $T_1,T_2,\ldots,T_m$，其中每一个集合本身又是一棵树 并且称为子树 ( $SubTree$ )
	
	<br/>

	*Tree*

	![2020-3-4-18-23-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-23-36.png)

	*SubTree T1*

	![2020-3-4-18-24-32.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-24-32.png)

	*SubTree T2*
	
	![2020-3-4-18-24-42.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-24-42.png)


<br/>

#### 节点的分类

<span id = "节点的分类"></span>

树的节点包含一个数据元素及若干指向其子树的分支
 - 节点拥有的子树数称为 <font color = "red">节点的度</font> ( $Degree$ ) ;
 - 度为 0 的节点称为 <font color="red">叶节点</font> ( $Leaf$ ) 或终端节点 ;
 - 度不为 0 的节点称为非终端节点或 <font color = "red">分支节点</font>，除根节点之外，分支节点也称为 <font color="red">内部节点</font> ;
 - 树内各节点的度的最大值则为 <font color="red">树的度</font> ;
 
   ![2020-3-4-17-3-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-3-5.png)


<br/>

#### 节点间的关系

<span id = "节点间的关系"></span>

- 节点的子树的根称为该节点的 <font color="red">孩子</font> ( $Child$ ) ;
- 相应地，该节点称为孩子的 <font color="red">双亲</font> ( $Parent$ ) ;
- 同一个 双亲 的 孩子 之间互称 <font color="red">兄弟</font> ( $Sibling$ ) ;
- 根到该节点所经分支上的所有节点都称为该节点的 <font color="red">祖先</font> ( $ancestor$ ) ;
- 以某节点为根的子树中的任一节点都称为节点的 <font color="red">子孙</font> ( $Offspring$ )

  ![2020-3-4-17-11-50.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-11-50.png)

<br/>

#### 树的其它相关概念

<span id = "树的其它相关概念"></span>

- <font color="red">节点的层次</font> ( $Level$ ) 从根开始定义起，根为第一层，根的孩子为第二层，若某结点在第1层，则其子树的根就在第 *1 + 1* 层 ;

- 双亲在局一层的节点互为 <font color="red">堂兄弟</font> ( $Cousins$ ) ;

- 树中结点的最大层次称为 <font color = "red">树的深度</font> ( $Depth$ ) 或 高度 ;

- 如果将树中结点的各子树看成从左至右是有次序的，不能互换的，则称该树为 <font color= "red">有序树</font> ，否则称为 <font color="red">无序树</font> ; 
- <font color="red">森林</font> ( $Forest$ ) 是 $m$ ( $m > 0$ ) 棵互不相交的树的集合，对树中每个结点而言，其子树的集合即为森林

  ![2020-3-4-17-19-45.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-19-45.png)


<br/>

#### 树的表示法

<span id = "树的表示法"></span>

上面所展示的树形结构是从 <font color="red">人的逻辑概念上所看到的树形结构</font>，而对于开发人员而言，这种逻辑上的数据结构是难以操作和构造的，作为数据结构存储的两大分支无非就是 *顺序存储* 和 *链式存储*，而对于不进行过任何处理的树形结构来说显然是不满足这两种分支的存储模型的，所以我们必须把逻辑上的树形结构转换为一个真实的、具体的数据结构，简而言之，就是 <font color="red">把一颗复杂的树转换为一颗能够处理的树</font>，这种转换方式也称为 <font color = "red">树的表示法</font>

!!! INFO PS : 以下所展示的几种表示法以下图这种原始树形结构作为模型

	![2020-3-4-18-23-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-23-36.png)


<br/>

_*双亲表示法*_

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( $Data$ ) 外，还附设一个指示器去维护该节点的双亲节点在连续空间所处的下标 ( 如果没有则为 $-1$ ) ，我们称它为 <font color = "red">双亲域</font> ( $Parent$ )

这样的存储结构，我们可以根据节点的 $Parent$ 指针很容易找到它的双亲节点，所用的时间复杂度为 $O(1)$ ,直到 $Parent$ 为 -1 时，则表示找到了树节点的根。可如果我们要知道结点的孩子是什么，对不起，请遍历整个结构才行 ; 其实解决这一问题也很简单，我们为何不能维护一个 <font color = "red">长子域</font> ( $FirstChild$ ) 的指示器来记录该节点最左边孩子的下标，抑或是维护一个 <font color = "red">兄弟域</font> ( $RightSib$ ) 来记录当前节点右边兄弟的下标呢？答案当然是可以的，数据结构的具体模型视具体情况而进行定义

下图则为双亲表示法的数据结构模型：

![2020-3-4-18-55-28.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-55-28.png)

<br/>

_*孩子表示法*_

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( $Data$ ) 外，还维护了一个 <font color = "red">孩子域</font> ( $FirstChild$ ) 的指针指向了当前节点下的所有孩子节点，旗下的所有孩子节点以单链表存储结构的形式连接起来，<font color = "red">即 孩子域 指向了存储其下所有孩子节点的单链表在内存中的首地址</font>

为此我们除了需要设计数组中的元素的存储结构以外，还需设置单链表中的节点的数据结构，即附设一个指示器指示当前孩子节点在数组存储结构位置中的下标 ( $Child$ ) 和指向当前孩子的兄弟节点的 指针域 ( $Next$ )

这样的结构对于我们要査找某个节点的某个孩子，或者找某个节点的兄弟，只需要査找这个节点的孩子单链表即可 ; 对于遍而整棵树也是很方便的，对头结点的数组循环即可完成遍历

但是，这也存在着问题，我如何知道某个结点的双亲是谁呢？目前这个阶段是比较麻烦的，需要整棵树遍历才行，但是我们就不可以把双亲表示法和孩子表示法综合一下吗？当然是可以，当综合了之后，我个人认为这种表示法就不能称为 孩子表示法 了，应该称之为 <font color = "red">孩子双亲表示法</font>

下图则为孩子双亲表示法的数据结构模型：

![2020-3-4-19-32-51.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-19-32-51.png)

<br/>

_*孩子兄弟表示法*_

孩子兄弟表示法是一种最为常用的一种表示法，它的好处在于能够把一颗复杂的树转换为一颗 <font color = "red">二叉树</font>，对于二叉树的定义我们在下面的节点会提到，现在先聊下孩子兄弟表示法的具体表示方式

我们对树形结构进行观察后发现，任意一棵树它的节点的第一个孩子如果存在就是唯一的，它的右兄弟如果存在也是唯一的，因此我们大可对该节点所代表的元素结构进行一个构想，首要也是必须的就是 <font color = "red">数据域</font> ( $Data$ )，其次还维护了两个 <font color = "red">指针域</font> 分别指向了当前节点的第一个孩子节点在内存中的首地址 ( $LPointer$ / $FirstChild$ ) 和当前节点右边兄弟节点在内存中的首地址 ( $RPointer$ / $RightSib$ )

下图则为孩子兄弟表示法的数据结构模型：

![2020-3-4-19-49-30.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-19-49-30.png)


<br/>

<span id = "binary_tree"></span>

### Binary Tree

---

对于在某个阶段下都是两种结果的情形，比如：开和关、0和1、真和假、上和下、对与错，正面与反面等，都适合用树状结构来建模，而这种树是一种很特殊的树状结构，叫做二叉树

前面在孩子兄弟表示法中稍微提到了 <font color = "red">二叉树</font> 的概念，对于 二叉树 来说，由于其毎个结点至多只能有左孩子和右孩子，那么在这种情况下，二叉树所面对的变化就少很多了，因此很多相关性质和算法都是以 二叉树 作为模型基准所研究了出来的

!!! INFO 二叉树的定义

	二叉树 ( *Binary Tree* ) 是 $n$ ( $n \geq 0$ ) 个节点的有限集合，该集合或者为空集 ( 称为 <font color = "red">空二叉树</font> )，或者由一个根结点和两裸互不相交的、分别称为根节点的 <font color = "red">左子树</font> 和 <font color = "red">右子树</font> 的二叉树组成

	*Binary-Tree*
	
	![2020-3-8-1-57-31.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-1-57-31.png)


<br/>

#### 二叉树的特点和基本形态

<span id = "二叉树的特点和基本形态"></span>

_*二叉树的特点：*_
- 每个结点 <font color = "red">最多有两棵子树</font>，所以二叉树中不存在度大于 $2$ 的结点，注意，不是只有两棵子树，而是最多有，<font color = "red">没有子树</font> 或者 <font color = "red">有一棵子树</font> 都是可以的

- <font color = "red">左子树和右子树是有顺序的</font>，次序不能任意颠倒，即二叉树是一颗 <font color = "red">有序树</font>

- 即使树中某结点只有一棵子树，也要区分它是 <font color = "red">左子树</font> 还是 <font color = "red">右子树</font>

_*二叉树具有的五种基本形态：*_

- 空二叉树
- 只有一个根节点
- 根节点只有左子树
- 根节点只有右子树
- 根节点既有左子树又有右子树

<br/>

#### 特殊的二叉树

<span id = "特殊的二叉树"></span>

上面所提到的是二叉树的基本形态，但是对于二叉树来说，它是多变的，即可能在某种形态之下有着它一定的研究价值，我们把这些具有特殊形态的树都统称为 特殊的二叉树

1. 斜树

	!!! INFO Description
		所有的结点都只有左子树的二叉树树叫 <font color = "red">左斜树</font>，所有结点都是只有右子树的二叉树叫 <font color = "red">右斜树</font>，这两者统称为斜树

	斜树有很明显的特点，就是 <font color = "red">每一层都只有一个结点</font>，<font color = "red">结点的个数与二叉树的深度相同</font>

	有人会想，斜树的表现形式不是和线性表是一样的吗？对的，其实线性表结构就可以理解为是树的一种极其特殊的表现形式

	![2020-3-8-13-17-55.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-13-17-55.png)

1. 满二叉树
	
	!!! INFO Description
		在一棵二又树中，如果所有分支结点都存在左子树和右子树，并且所有叶子都在同一层上，这样的二叉树树称为满二叉树

	单是每个结点都存在左右子树，不能算是满二叉树，因此，满二叉树的特点有：
	- <font color = "red">所有的叶子都在同一层上</font>，做到整棵树的平衡
	- <font color = "red">叶子只能出现在最下一层</font>，出现在其他层就不可能达成平衡
	- 非叶子结点的度一定是 $2$
	- 在同样深度的二叉树中，满二叉树的结点个数最多，叶子数最多

	 ![2020-3-8-14-0-6.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-14-0-6.png)

3. 完全二叉树

	!!! INFO Description

		对一棵具有 $n$ 个结点的二叉树 <font color = "red">按层序编号</font>，如果编号为 $i$ ( $i \leq 1 \leq n$ ) 的结点与同样深度的 <font color = "red">满二叉树</font> 中编号为 $i$ 的结点在二叉树中位置完全相同，则这棵二叉树称为 <font color = "red">完全二叉树</font>

	完全二叉树是一种有些理解难度的特殊二叉树，首先要从字面上区分，"完全" 和 ＂满＂ 的差异，满二叉树是一颗完全二叉树，但是完全二叉树不一定是满的 ; 其次，完全二叉树的所有结点与同样深度的满二叉树，它们 <font color = "red">按层序编号相同的结点</font> 相对应对应的

	对于完全二叉树的定义，上面所说的话可能过于抽象，但这里也给出一种较为直观的区分完全二叉树的方式：心中默默给二叉树中的毎个结点按照满二叉树的结构逐层顺序编号，如果编号
	出现空档，就说明不是完全二叉树，否则就是

	这里我可以总结一些完全二叉树的特点：
	- 叶子结点只能出现在 <font color = "red">最下两层</font>
	- 最下层的叶子一定 <font color = "red">集中在左部连续位置</font>
	- 倒数二层若有叶子结点，一定 <font color = "red">都在右部连续位置</font>
	- 如果结点度为 $1$，则该结点 <font color = "red">只有左孩子</font>，即不存在只有右子树的情况
	- 同样结点数的二叉树，完全二叉数的 <font color = "red">深度最小</font>

	![2020-3-8-21-53-43.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-21-53-43.png)

	

<br/>

#### 二叉树的性质

<span id = "二叉树的性质"></span>

- 在二叉树的第 $i$ 层上至多有 $2^{i-1}$ 个结点 ( $i\geq$ 1)

- 深度为 $k$ 的二叉树至多有 $2^k-1$ 个结点 ( $k\geq1$ )

- 
  二叉树 $T$ ; 结点总数 $n$ ; 分支线总数 $k$ ; 终端结点数 $n_0$ ; 度为 1 的结点数 $n_1$ ; 度为 2 的结点数 $n_2$ ; 
  - $n_0=n_2+1$
  - $n=n_0+n_1+n_2$
  - $k=n-1=n_1+2n_2$

- 具有 $n$ 个结点的 <font color = "red">完全二叉树</font> 的深度为 $⌊\log(2^n)⌋+1$ ( $⌊x⌋$ 表示为大于 $x$ 的最大整数 )

- 如果对一颗具有 $n$ 个结点的完全二叉树 ( 其深度为 $⌊\log(2^n)⌋+1$ ) 的结点按层序编号 ( 从第 $1$ 层到第 $⌊\log(2^n)⌋+1$ 层，每层从左到右 )，对任意节点 $i$ ( $1 \leq i \leq n$ ) 有：
	- 如果 $i=1$，则结点 $i$ 为二叉树的根，无双亲 ; 如果 $i>1$，则其双亲是节点 $⌊\displaystyle \frac{i}{2}⌋$
	- 如果 $2i>n$，则结点 $i$ 无左孩子 ( 结点 $i$ 为叶子节点 ) ； 否则其左孩子是结点 $2i$
	- 如果 $2i+1>n$，则结点 $i$ 无右孩子 ; 否则其右孩子是结点 $2i+1$
  
	![2020-3-8-23-12-26.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-12-26.png)


<br/>

#### 二叉树的存储结构

<span id = "二叉树的存储结构"></span>

_*二叉树顺序存储结构*_

<font color = "red">顺序存储</font> 对树这种 $1:n$ 的关系结构实现起来是比较困难的，但是二叉树是一种特殊的树，由于它的特殊性，使得用顺序存储结构也可以实现

二叉树的顺序存储结构就是 <font color = "red">用一维数组存储二叉树中的结点</font>，<font color = "red">并且结点的存储位置 ( 数组的下标 ) 要能体现结点之间的逻辑关系</font>，比如双亲与孩子的关系，左右兄弟的关系等，如下图所示，一颗 <font color = "red">完全二叉树</font> 在使用顺序存储后的结构模型

![2020-3-8-23-42-35.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-42-35.png)

需要注意的是，上图特指出使用的是 <font color = "red">完全二叉树</font> 进行顺序存储结构进行存储，由于 完全二叉树 的特性，我们对 完全二叉树 使用顺序存储是很容易能够拿到各结点之间的逻辑关系的，而对于一般的二叉树而言，尽管层序编号并不能够反映这颗二叉树中各结点之间的直接逻辑关系，但是我们同样也可以将其按 完全二叉树 进行编号，只不过把不存在的节点需要 <font color ="red">特别批注出来</font> 而已

针对于刚刚所说到的 <font color = "red">一般的二叉树使用顺序存储结构</font>，我们不妨考利一种特殊情况，一颗深度为 $k$ 的右斜树只有 $k$ 个结点，但是按照刚刚所说的按照 完全二叉树 进行编号的方式却需要分配出 $2^k-1$个存储单元空间，这显然是对于存储空间是一种极度浪费的情况，所以，<font color = "red">顺序存储结构一般只用于完全二叉树</font>

![2020-3-8-23-53-59.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-53-59.png)

<br/>

_*二叉树链表*_

既然顺序存储适用性不强，我们就要考虑 <font color = "red">链式存储结构</font>

二叉树的毎个结点至多只有两个孩子，所以为它设计一个数据域 $Data$ 和 两个指针域 $lchild$ / $rchild$ 是比较自然的想法，同样的，我们称这样的链表叫做二叉链表

通过二叉链表存储结构的方式，我们能够非常方便的得知某个结点的 <font color = "red">左孩子</font> 和 <font color = "red">右孩子</font> 的位置，如有需要，我们同样可以再加多一个指针域 $parent$ 去获取当前结点的双亲位置，这种方式也称之为 <font color = "red">三叉链表</font>

```c
typedef char ElementType_t;

typedef struct binary_node {
	ElementType_t data;
	struct binary_node *parent;	/* Extension */
	struct binary_node *left;  	/* Left sub tree */
	struct binary_node *right; 	/* Right sub tree */
} Binary_Tree;
```

![2020-3-9-0-19-46.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-0-19-46.png)


<br/>

#### 二叉树的遍历

<span id = "二叉树的遍历"></span>

二叉树的遍历次序不同于 <font color = "red">线性结构</font>，最多也就是从头至尾、循环、双向等简单的遍历方式，树的结点之间不存在 <font color = "red">唯一的前驱和后继</font> 关系，在访问一个结点后，下一个被访问的结点面临着不同的选择，由于选择方式的不同，遍历的次序就完全不同了

!!! INFO Description

	二叉树的遍历 ( $Traversing binary-tree$ ) 是指从根结点出发，按照某种 <font color = "red">次序</font> 依次访同二叉树中所有结点，使得每个结点都被访问一次且仅被访问一次

<br/>

_*前序遍历*_

规则是若二又树为空，用空操作返回，否则先访问根结点，然后前序遍历左子树，再前序遍历右子树，如下图所示，其遍历顺序则为：$ABDGHCEIF$

```c
int Preorder_Traverse_BinaryTree(Binary_Tree *tree) {
	if (tree == 0x0)
		return -1;

	printf("[*] Traverse Binary-tree: Value = %c, Address = %p\n", tree->data, tree);
	Preorder_Traverse_BinaryTree(tree->left);
	Preorder_Traverse_BinaryTree(tree->right);

	return 0;
}
```

![2020-3-9-0-39-1.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-0-39-1.png)

<br/>

_*中序遍历*_

规则是若树为空，则空操作返回，否则从根结点开始 ( 注意并不是先访问根結点 )，中序遍历根结点的左子树，然后是访问根结点，最后中序遍历右子树，如下图所示，其遍历顺序则为：$GDHBAEICF$

```C
int Middorder_Traverse_BinaryTree(Binary_Tree *tree) {
	if (tree == 0x0)
		return -1;

	Middorder_Traverse_BinaryTree(tree->left);
	printf("[*] Traverse Binary-tree: Value = %c, Address = %p\n", tree->data, tree);
	Middorder_Traverse_BinaryTree(tree->right);

	return 0;
}
```

![2020-3-9-0-45-37.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-0-45-37.png)

<br/>

_*后序遍历*_

规则是若树为空，则空操作返回，否则从左到右先叶子后结点的方式遍历访问左右子树，最后是访问根结点，如下图所示，其遍历顺序则为：$GHDB1EFCA$

```c
int Postorder_Traverse_BinaryTree(Binary_Tree *tree) {
	if (tree == 0x0) return -1;

	Postorder_Traverse_BinaryTree(tree->left);
	Postorder_Traverse_BinaryTree(tree->right);
	printf("[*] Traverse Binary-tree: Value = %c, Address = %p\n", tree->data, tree);

	return 0;
}
```

![2020-3-9-0-48-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-0-48-36.png)

<br/>

_*遍历结果的推理*_

- 已知前序遍历序列和中序遍历序列，可以唯一确定一棵二叉树
  - 前序：$ABCDEF$
  - 中序：$CBAEDF$
- 已知后序遍历序列和中序遍历序列，可以唯一确定一棵二叉树
  - 前序：$ABCDEFG$
  - 中序：$BDCAFEG$
- 已知前序遍历序列和后序遍历序列，不可以唯一确定一棵二叉树

!!! INFO

	关于推理过程请查看 《大话数据结构》 6.8.6 章节

<br/>

#### 二叉树的建立

<span id = "二叉树的建立"></span>

二叉树的建立同样需要采用遍历的方式去创建，对于存在空子数的结点我们在遍历的过程中需要通过一个 <font color = "red">额外的标识符</font> 来确定，在下面的代码中，提供了两种 <font color = "red">不同的遍历次序</font> 来构造一棵二叉树

<br/>

_*后序遍历*_

```c
/* 
 *                                (A)
 *                                 |
 *                 o---------------+---------------o
 *                 |                               |
 *                (B)                             (C)
 *                 |                               |
 *          o------+-----o                  o------+-----o
 *          |            |                  |            |
 *         (D)          (E)                (F)          (G)
 *          |            |                  |            |
 *      o---+---o    o---+---o          o---+---o    o---+---o
 *      |       |    |       |          |       |    |       |
 *     (H)     (I)  (J)     (K)        (L)     (M)  (N)     (O)
 *
 *
 * Don't direct Postorder create binary-tree, but we can find the last
 * node is root, and previous node is right-child for root, and pre-previous
 * node is left-child for root..... So, we can create binary tree like this.
 */
static size_t index_Postorder = 0;
static ElementType_t Perfect_BinaryTree_data[] = { 'A',																		 /* ROOT */
											'B', 'D', 'H', '#', '#', 'I', '#', '#', 'E', 'J', '#', '#', 'K', '#', '#',		 /* LEFT */
											'C', 'F', 'L', '#', '#', 'M', '#', '#', 'G', 'N', '#', '#', 'O', '#', '#' };	 /* RIGHT */					
Binary_Tree *Postorder_Create_BinaryTree(Binary_Tree *parent) {
	ElementType_t ch = Perfect_BinaryTree_data[index_Postorder++];
	if (ch == '#') {
		return NULL;
	} else {
		Binary_Tree *tree = (Binary_Tree *)malloc(sizeof(struct binary_node));
		if (tree == 0x0) return -1;
		
		memset(tree, 0, sizeof(struct binary_node));

		tree->left = Postorder_Create_BinaryTree(tree);
		tree->right = Postorder_Create_BinaryTree(tree);

		tree->data = ch;
		tree->parent = parent;
		printf("[+] Create Binary-node: Node-Value = %c, Node-Address = %p, Parent-Address = %p\n",
			   tree->data,
			   tree,
			   tree->parent);

		return tree;
	}
}
```

<br/>

_*前序遍历*_

```c
/* 
 *                                (A)
 *                                 |
 *                 o---------------+---------------o
 *                 |                               |
 *                (B)                             (C)
 *                 |                               |
 *          o------+-----o                  o------+-----o
 *          |            |                  |            |
 *         (D)          (E)                (F)          (G)
 *          |                               |            |
 *      o---+                           o---+            +---o
 *      |                               |                    |
 *     (H)                             (I)                  (J)
 *      |
 *      +-----o
 *            |
 *           (K)
 *
 * Inorder create binary-tree can complete the
 * construction of the tree under the model that
 * does not know the tree
 *
 * We use scanf function to complete the data field entry.
 * When the received character is '#', it proves that the current
 * node belongs to an NULL state
 */
static size_t index_Preorder = 0;
static ElementType_t Binary_Tree_Data[] = {
	'A','B','D','H','#','K','#','#','#','E','#','#',
	'C','F','I','#','#','#','G','#','J','#','#' };
int Preorder_Create_BinaryTree(Binary_Tree **tree, Binary_Tree *parent) {
	ElementType_t ch = Binary_Tree_Data[index_Preorder++];
	if (ch == '#') {
		return -1;
	} else {
		*tree = (Binary_Tree *)malloc(sizeof(Binary_Tree));
		if (*tree == 0x0) return -1;

		memset(*tree, 0, sizeof(Binary_Tree));

		(*tree)->data = ch;
		(*tree)->parent = parent;
		printf("[+] Create Binary-node: Node-Value = %c, Node-Address = %p, Parent-value = %c, Parent-Address = %p\n",
			   (*tree)->data,
			   *tree,
			   (*tree)->parent == NULL ? '-' : (*tree)->parent->data,
			   (*tree)->parent);

		Preorder_Create_BinaryTree(&(*tree)->left, *tree);
		Preorder_Create_BinaryTree(&(*tree)->right, *tree);
	}

	return 0;
}
```

<br/>

#### 二叉树的深度

<span id = "二叉树的深度"></span>

```c
/* The deep for Binary-Tree */
int BinaryTree_Deep(struct binary_node *node) {
  int deep = 0;

  if (node != NULL) {
    int leftdeep = BinaryTree_Deep(node->left);
    int rightdeep = BinaryTree_Deep(node->right);

    /*
     * Get a deeper value of the left or right subtrees,
     * plus one to add the depth of the current node
     */
    deep = leftdeep >= rightdeep ? leftdeep + 1 : rightdeep + 1;
  }

  return deep;
}
```

<br/>

#### 线索二叉树

<span id = "线索二叉树"></span>

对于上面的所构造的二叉树中都存在着一个问题，那就是：<font color = "red">空指针域并未得到充分地利用</font>

设：一张有 $n$ 个结点的二叉链表，每个结点有指向左右孩子的指针域，也就是说一张二叉链表中总共有 $2n$ 个指针域，而 $n$ 个结点的二叉树一共有 $n-1$ 条分支线，也就是说，一张二叉链表中总共有 $2n-(n-1)=n+1$ 个空指针域，如下图所示

![2020-3-9-16-43-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-16-43-5.png)

上图中所存在的空指针域不存储任何数据，白白浪费着 $(n+1) \times 4=k(Bytes)$ 的存储空间 ; 另一方面，在已有一张已经构造好的二叉树的前提下，我们是不能够知道每个节点的前驱和后继分别对应着谁的 ( <font color = "red">在某个次序下所遍历的结果作为前驱和后继的判断标准</font> ) ，除非我们对这张二叉链表在进行一次遍历

综上所述，<font color = "red">线索二叉树</font> 就是为了解决以上问题所存在，我们不妨让某个结点的空指针域指向当前结点的前驱或后继，比如说规定结点中的 $lchild$ 成员如果是一个空指针域，则该成员指向当前结点的前驱，反之 $rchild$ 则指向当前结点的后继，但这里还存在着一个问题，如果某个结点是不存在空指针域的，那么我们要如何区分当前结点的 $lchild$ / $rchild$ 到底是指向当前结点下的 左 / 右 孩子，还是指向当前结点的 前驱 / 后继 呢？在这个基础上，我们还可以对于结点的内部构造新增两个成员分别为 $lflag$ / $rflag$ 用于标识当前结点的 $lchild$ / $rchild$ 到底是指向 左 / 右 孩子，还是指向当前结点的 前驱 / 后继 ，比如说当 $lflag$ / $rflag$ 为 $1$ 时，则标志着当前结点的 $lchild$ / $rchild$ 指向的是当前结点的 前驱 / 后继，反之则指向的是当前结点的 左 / 右 孩子

!!! INFO Description

	我们把这种指向前驱和后继的指针称为 <font color = "red">线索</font>，加上线索的二叉链表称为 <font color ="red">线索链表</font>，相应的二叉树就称为 <font color = "red">线索二叉树</font> ( $Threaded$ $Binary-Tree$)，而我们把一颗二叉树以某种次序进行遍历使其转换为线索二叉树的过程又称为 <font color = "red">二叉树的线索化</font>

下图则为一颗进行过线索化的二叉树的结构体现 ( 以中序遍历作为某个结点前驱和后继的判断标准 )：

![2020-3-9-17-12-44.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-12-44.png)

对于上图这颗线索二叉树 ( 以中序遍历作为某个结点前驱和后继的判断标准 ) 的指针域的指向来看，我们会发现二叉树线索化的是指其实就是把一颗二叉树转换为一张 <font color  ="red">双向链表</font>，如下图所示：

![2020-3-9-17-15-41.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-15-41.png)

<br/>

#### 树、森林与二叉树之间的转换

<span id = "树、森林与二叉树之间的转换"></span>

在前面提到过，我们对于树形结构的探究通常都采用 <font color  ="red">孩子兄弟表示法</font> 来将一颗复杂的树转化为二叉树进行存储，而借助二叉链表的特性，树和二叉树之间是可以进行相互转换的

从物理结构来看，树和二叉树是相同的，只是采用了不同的解释方式，因此，只要我们设定一定的规则，用二叉树来表示树，甚至表示森林都是可以的，森林与二叉树也可以互相进行转换

<br/>

_*树转换为二叉树*_

1. 加线：把所有兄弟节点之间加一条连接线
2. 去线：对树中毎个结点，只保留它与第一个孩子结点的连线，删除它与其他孩子结点之间的连线
3. 层次调整：以树的根结点为轴心，将整棵树顺时针旋转一定的角度，使之结构层次分明，注意第一个孩子是二叉树结点的左孩子，兄弟转换过来的孩子是结点的右孩子

	![2020-3-9-17-32-35.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-32-35.png)

<br/>

_*森林转换为二叉树*_

1. 把每颗树转换为二叉树
2. 第一颗二叉树不动，从第二棵二叉树开始，依次把后一棵二叉树的根结点作为前一棵二叉树的根结点的右孩子，用线连接起来，当所有的二叉树连接起来后就得到了由森林转换来的二叉树

	![2020-3-9-17-37-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-37-5.png)

<br/>

_*二叉树转换为树*_

1. 加线：若某结点 $K$ 的左孩子结点存在，则将这个左孩子的 $n$ 个右孩子结点都作结点 $K$ 的孩子，并用线连接起来 
2. 去线：删除原二叉树中所有结点与其右孩子结点的连接线
3. 层次调整：使之结构层次分明

	![2020-3-9-17-49-18.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-49-18.png)

<br/>

_*二叉树转换为森林*_

!!! INFO
	判断一棵二叉树能够转换成一棵树还是森林，标准很简单，那就是只要看这棵二叉树树的根结点有没有右孩子，有就是森林，没有就是一棵树

1. 分离：从根结点开始的 $n$ 个右孩子若存在，则连线刪除，得到多颗分离后的二叉树
2. 拼接：将每棵分离后的二叉树转换为树即可

	![2020-3-9-17-55-48.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-55-48.png)


<br/>

#### 赫夫曼树

<span id = "赫夫曼树"></span>

_*赫夫曼树的定义*_

- 从树中一个结点到另一个结点之间的分支构成两个结点之间的 <font color = "red">路径</font>
- 路径上的分支数目称作 <font color = "red">路径长度</font>
- <font color = "red">树的路径长度</font> 就是从树根到毎一结点的路径长度之和
- 如果考虑到带 <font color = "red">权</font> 的结点
 	- 结点的 <font color = "red">带权的路径长度</font> 为从该结点到树根之间的路径长度与结点上权的乘积
 	- <font color = "red">树的带权路径长度</font> 为树中所有叶子结点的带权路径长度之和
- 假设有 $n$ 个权值 $\{w_1,w_2,\ldots,w_n\}$ 构造一棵有 $n$ 个叶子结点的二叉树，每个叶子结点带权 $w_k$，每个叶子的路径长度为 $lk$，我们通常记作，则其中带权路径长度 $WPL$ 最小的二叉树称作赫夫曼树

<br/>

_*如何构造一颗赫夫曼树*_

1. 根据给定的 $n$ 个权值 $\{w_1,w_2,\ldots,w_n\}$ 构成 $n$ 棵二叉树的集合 $F=\{T_1,T_2,\ldots,T_n\}$，其中每棵二叉树 $T_i$ 中只有一个带权为 $w_i$，其左右树均为空
2. 在 $F$ 中选取两棵根结点的权值最小的树作为左右子树构造一棵新的二叉树树 $K$，且置 $K$ 的权值为其左右子树上根结点的权值之和
3. 在 $F$ 中删除这两棵树，同时将新得到的二叉树 $K$ 加入 $F$ 中
4. 重复 $2$ 和 $3$ 步骤，直到 $F$ 只含一颗树为止，这棵树便为 <font color = "red">赫夫曼树</font>

<br/>

<span id = "Disjoint-Set"></span>

### Disjoint Set

---

给定一组数据集，<span style='color:red'>数据集中的各元素均存在关联</span>，现需要将里面的元素再做一次元素间强关联性的分组，这时候就可以用到<span style='color:red'>并查集</span>这个数据结构来完成

并查集一种简洁而优雅的数据结构，主要用于解决一些元素分组的问题，它管理一系列不相交的集合，我们可以把上面所提到的那一组数据集的内部中再依据元素之间的关联性拆分出多个集合，并查集能够支持对这些所拆分出来的集合执行 :
- 合并( $Union$ ) : 把两个不相交的集合合并为一个集合 
- 查询( $Find$ ) : 查询两个元素是否在同一个集合内

并查集在实际工程项目中的使用通常会和连通图搭配在一起，<span style='color:red'>因为对于一个能够成功合并的集合来说，其内部必定会存在一副连通图</span>，那么所延伸出来的需求诸如 : 判断一张图中是否存在连通图、顶点是否存在于一个连通图内、……，不得不说的是，这并不是并查集最初需要解决的数学问题，但是把它放在连通图中配合使用的确是一个很不错的选择

<br/>

#### 并查集的使用

<span id = "并查集的使用"></span>

既然强调并查集是一种优雅的数据结构，那么就其所提供的合并集合的功能来看，我们就不能单纯的使用暴力解法来解决实际问题了

先思考一个问题，对于两个集合的合并操作，如果屏蔽掉暴力求解法的使用，那么用什么方式才能够更加优雅的对这两个数据集进行合并呢？那就是树，当两棵树之间需要被合并一方的根节点改为合并一方的根节点时，其实这时候就完成了两棵树之间的合并工作

而并查集之所以称之为优雅，是因为其内部同样是依赖于树结构进行合并，<span style='color:red'>在并查集中，对于集合中的每一个元素都看作是一个集合(树)</span>，由这些小的集合根据原数据集中，元素间的关联关系去不断地合并出一个或多个不相交的集合，这个过程就依赖于树的设计理念来实现

其中，对于每颗树中的根节点在并查集中又称呼为<span style='color:red'>代表元</span>，有代表元来最终完成合并的工作。为了保证并查集的简易性，其并不会挪用树中的过多设计结构，一般来说，仅使用一个一维数组来表示一颗树的关系，其中，数组中每个下标其实就是代表着每一个不同的顶点，下标的值就是代表着当前结点所对应的父节点在这个数组中的下标，直至下标的值为其顶点本身，则代表着当前结点属于一个根节点，那么我们就可以抽象出以下的数据结构和使用方式

> 下面代码中还附带了 $Rank$ 的初始化，其主要是应用于并查集的搜寻优化工作，下面会提到

```c
#define VERTICES 6 /* Num of vertex */

static int parent[VERTICES]; /* The idx of arrary is each of vertexs, and value is parent idx of current vertex */ 
static int rank[VERTICES];   /* Record the rank of tree */

/* @breif 初始化并查集 */
void initialise_disjoint_set() {
  /* rank */
  memset(rank, 0x0, sizeof(rank));

  /* parent_set */
  for (size_t i = 0; i < VERTICES; ++i) {
    parent[i] = i;
  }
}

/* @breif 找到当前顶点的最上级根节点 */
int find_root_quick(int f) {
  if (parent[f] == f) { /* 在初始情况下，我们初始化 parent 每个下标所对应的值为顶点本身 */
    return f;
  }

  /**
   * 如果顶点的父节点并不是顶点本身(初始化情况下)，
   * 那么就继续沿着其所对应的父结点继续向上递归搜索，直至找到根
  */
  return find_root_quick(parent[f]);
}

/* @breif 合并两个顶点至一个集合内，有可能会合并失败，这种情况则为两个顶点本身就属于一个集合内 */
bool union_set_quick(int x, int y) {
  int x_root = find_root(x);
  int y_root = find_root(y);

  /* 当根节点相同时，则代表着所输入的 x 和 y 是在同一个集合内 */
  if (x_root == y_root) {
    return false;
  }

  parent[x_root] = y_root;
  return true;
}

void initialise_disjoint_set() {
  /* rank */
  memset(rank, 0x0, sizeof(rank));

  /* parent_set */
  for (size_t i = 0; i < VERTICES; ++i) {
    parent[i] = i;
  }
}

/* @breif 查找一副图是否存在环 */
void find_cycle_in_graph(void) {
  /**
   * 这里本应该拥有一个创建图且初始化图的过程，但是
   * 这里省略掉了，仅模拟出已经查找出图中所有的边
  */
  int edges[VERTICES][2] = {
    { 0, 1 }, { 1, 2 }, { 1, 3 }, 
    { 2, 4 }, { 3, 4 }, { 2, 5 }
  };

  initialise_disjoint_set();

  /**
   * 逐个合并图中的所有的边，也就是两个顶点之间不断进行合并 
   * 对于存在环的图来说(不考虑相同边的情况)，如果合并时存在
   * 两个不同的顶点属于同一个集合内(union_set返回false)，那
   * 么这个集合所构成的图中，任意两点的连线都可以构成一个环
  */
  for (size_t i = 0; i < VERTICES; ++i) {
    int x = edges[i][0];
    int y = edges[i][1];

    if (!union_set_quick(x, y)) {
      printf("Cycle detected\n");
      return;
    }
  }
  printf("None cycle\n");

  return;
}
```

<br/>

#### 并查集的优化

<span id = "并查集的优化"></span>

_*路劲压缩*_

上面的代码展示了并查集的最基本的一些操作，然后实际上是存在着一定的问题的，正如上面最初版的合并集合的代码(union_set_quick)那一部分，由于每次合并仅只是固定把 $x$ 的代表元合并至 $y$ 的代表元，那么考虑每次合并时所输入的结点都是一种线性关系，那么<span style='color:red'>最终所合并出来的树会退化成一个长的单链表，在越往后的合并工作中，每次查找根节点的工作就会呈线性增长</span>，如下图所示

![2020-10-31-01-32-25](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-31-01-32-25.png)

我们可以使用<span style='color:red'>路径压缩</span>的方法来解决这个问题，其实它也是非常好的实现，既然我们只关心一个元素对应的根节点，那我们希望每个元素到根节点的路径尽可能短，简而言之，只要我们在查询的过程中，把沿途的每个节点的父节点都设为根节点即可，那么查找部分的代码(find_root_quick)就可以更改如下

> 路径压缩时直接将节点的父亲修改成最终的祖先节点，在破坏原先的树结构的同时，在有些场景中也会损失信息

```c
int find_root_compress(int f) {
  if (parent[f] == f) { /* 在初始情况下，我们初始化 parent 每个下标所对应的值为顶点本身 */
    return f;
  }

  parent[f] = find_root_compress(parent[f]); /* 将当前结点的父节点设置为根节点 */
  return parent[f];
}
```

<br/>

_*按秩合并*_

为了解决路劲压缩上的一些问题，我们可以按照两颗树的深度的不同来进行合并的次序，我们通过观察可以发现，当把简单的树往复杂的树上合并时，合并出来的树其深度并不会增加，反之则每次合并后，树的深度都会 +1，如下图所示

![2020-10-31-01-46-02](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-31-01-46-02.png)

这里就引入了 $Rank$ 数组的改变，该数组的下标对应着每个结点，而<span style='color:red'>下标所在的值着代表着当前节点所呈现出来的树的深度(以当前结点作为根的树或者子树)</span>，每当我们即将要进行树的合并工作的时候，我们需要增加判断两棵树之间的 $Rank$ 并保证总是由小到大的顺序来合并即可完成优化工作

这里要扩展的是，<span style='color:red'>当树的深度一样的时候，无论由输入值的哪一方的代表元来进行合并其最终结果都一样的</span>，但是我们还要保证的是，每次进行合并后，合并的一方其深度需要 +1

> 按秩合并虽然能够解决时间复杂度上的一些问题，但是其却牺牲了部分空间上的复杂度

> 路径压缩和按秩合并如果一起使用，时间复杂度接近 $O(n)$ ，但是很可能会破坏rank的准确性

```c
bool union_set_rank(int x, int y) {
  int x_root = find_root_quick(x);
  int y_root = find_root_quick(y);

  if (x_root == y_root) {
    return false;
  }

  /** 
   * 为了保证树的生成并不会退化成单向长链表而导致 parent 查找过程的复杂度会线性提升的情景，这
   * 里引入了 Rank 的概念
   * 
   * - 当两颗树(已经合并了的两个暂时不相交的集合)的深度不同时，如果将较深的树合并至较浅的树中
   *   时，整颗树的深度都会比原有较深的树 +1，反之则不变
   * - 当两颗树的深度是相同的时候，这里想象出唯有一个顶点的情景，这时候进行合并，不管是由谁作
   *   为父结点都无所谓，但是要保证作为父结点的那颗树其对应根的深度要 +1
  */
  if (rank[x_root] > rank[y_root]) {
    parent[y_root] = x_root;
  } else if (rank[y_root] < rank[x_root]) {
    parent[x_root] = y_root;
  } else {
    parent[x_root] = y_root;
    ++rank[y_root];
  }

  return true;
}
```



<br/>

<span id = "Graph"></span>

### Graph

---

在树形结构中，一对父母可以拥有多个孩子，但是每个孩子却只能由一对父母，这是一个道理。可现实生活中，人与人之间的关系总是非常复杂的，即我认识我的朋友，而我朋友的朋友又可能认识我，以此往复，这通常映射出一种 $n : n$ 的结构，而为了解决诸如这类 $n : n$ 的问题，**`图`** 就出现了

!!! info 图的简单定义

	图 ( Graph ) 是由顶点的非空集合和顶点之间边的集合组成，通常表示为$: G(V , E)$，其中 $G$ 表示为一个图，$V$ 是图 $G$ 中顶点的集合，$E$ 是图 $G$ 中边的集合

	- 图 $G$ 中的所有元素称之为<span style='color:red'>顶点</span> $(Vertex)$
	- 在图结构中，不允许没有顶点，若 $V$ 是顶点的集合，则强调该集合<span style='color:red'>有穷非空</span>
	- 图 $G$ 中的任意两个顶点之间都可能存在关系，它们之间的逻辑关系用<span style='color:red'>边</span>来表示
	- 在图结构中，不允许没有顶点，若 $E$ 是边的集合，则该集合<span style='color:red'>允许为空</span>，简而言之就是一张图仅有一个顶点的存在

	![2020-10-20-17-29-33](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-17-29-33.png)


<br/>

#### 各种图的定义

<span id = "各种图的定义"></span>

_*简单图与复杂图*_

一张图中，若不存在顶点到其自身的边，且同一条边不重复出现，则称这样的图为<span style='color:red'>简单图</span>(左)，反之则为<span style='color:red'>复杂图/网络</span>(右)，需要强调的是，<span style='color:red'>该文章所讨论的模型均为以简单图为基准所展开</span>

需要补充的是，对于无向图为模型的简单图而言，是不允许两个顶点间出现重复的边的，因为对于该无向边所表示的方向是可互逆的；对于有向图为模型的简单图而言，是允许两点之间出现重复的弧的，但是仅允许这两条重复的弧所表示的有序偶是不同的，即方向不同

![2020-10-20-23-10-00](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-10-00.png)

<br/>

_*无向图*_

若一张图的顶点 $V_i$ 到顶点 $V_j$ 之间的边<span style='color:red'>没有方向</span>，则称这条边为<span style='color:red'>无向边</span> $(Edge)$

无向边所连接的两个顶点 $V_i$ 与 $V_j$，这条无向边表示为<span style='color:red'>无序对</span> $: (V_i ,V_j)$，也正因为它是一条无向边，故我们还可以表示为 $: (V_j ,V_i)$

若一张图中的任意两个顶点之间的边都属于无向边，那么这张图就是一张<span style='color:red'>无向图</span> $(Undirected$ $graphs)$

下图呈现了一张无向图 $G = (V,\{E\})$，其中顶点集合 $: V = \{A, B, C, D\}$；边集合 $: E = \{(A,B),(B,C),(C,D),(D,A),(A,C)\}$；

![2020-10-20-22-41-41](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-22-41-41.png)

<br/>

_*有向图*_

若一张图的顶点 $V_i$ 到顶点 $V_j$ 之间的边<span style='color:red'>有方向</span>，则称这条边为<span style='color:red'>有向边</span>，更多的，我们会称它为<span style='color:red'>弧</span> $(Arc)$

有向边所连接的两个顶点，其中 $V_i$ 指向 $V_j$，那么这条弧表示为<span style='color:red'>有序偶</span> $: (V_i ,V_j)$，$V_i$ 称为<span style='color:red'>弧尾</span> $(Tail)$，$V_j$ 称为<span style='color:red'>弧头</span> $(Head)$

若一张图中的任意两个顶点之间的边都属于有向边，那么这张图就是一张<span style='color:red'>有向图</span> $(Directed$ $graphs)$

下图呈现了一张有向图 $G = (V,\{E\})$，其中顶点集合 $: V = \{A, B, C, D\}$；边集合 $: E = \{<A,D>,<B,A>,<C,A>,<B,C>\}$；

![2020-10-20-23-06-33](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-06-33.png)

<br/>

_*无向完全图*_

在无向图中，如果任意两个顶点之间都存在边，则称这张图为<span style='color:red'>无向完全图</span>

含有 $n$ 个顶点的无向完全图存在边数 $e = \displaystyle \frac{n \times (n-1)}{2}$，其中，$0 \le e \le \displaystyle \frac{n \times (n-1)}{2}n$

下图呈现了一张标准的无向完全图，因为每个顶点都要与除它以外的顶点连线，顶点 $A$ 与 $BCD$ 三个顶点连线，共有四个顶点，自然是 $4 \times 3$ ，但由于顶点 $A$ 与顶点B连线后，计算 $B$ 与 $A$ 连线就是重复，因此要整体除以2，即共有6条边

![2020-10-20-23-18-28](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-18-28.png)

<br/>

_*有向完全图*_

在有向图中，如果任意两个顶点之间都存在<span style='color:red'>方向互为相反</span>的弧，则称这张图为<span style='color:red'>有向完全图</span>

含有 $n$ 个顶点的有向完全图存在边数 $e = n \times (n-1)$，其中，$0 \le e \le n \times (n-1)$

![2020-10-20-23-26-47](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-26-47.png)

<br/>

_*稀疏图与稠密图*_

有很少条边或弧的图称为稀疏图，反之称为稠密图，当然，这里稀疏和稠密是模糊的概念，都是相对而言的，故不进行展开

<br/>

_*网*_

给定图的边或弧中存在与它相关的数据信息，则这种与图的边或弧相关的数据称之为<span style='color:red'>权</span> $(Weight)$；通常，这些权可用来表示一段距离

带权的图通常称为<span style='color:red'>网</span> $(Network)$

![2020-10-20-23-33-49](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-33-49.png)

<br/>

_*子图*_

假设两个图 $G = (V,\{E\})$ 和 $G` = (V`,\{E`\})$，如果 $V` \subseteq V$ 且 $E` \subseteq E$，则称 $G`$ 为 $G$ 的<span style='color:red'>子图</span> $(Subgraph)$

![2020-10-20-23-38-09](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-20-23-38-09.png)


<br/>

#### 图的顶点与边之间的关系

<span id = "图的顶点与边之间的关系"></span>

_*无向图中，顶点与边之间的关系*_

- 给定一张无向图 $G = (V,\{E\})$，如果边 $(V_i,V_j) \in E$，则称顶点 $V_i$ 与 $V_j$ <span style='color:red'>互为邻接点</span> $(Adjacent)$；边 $(V_i,V_j)$ <span style='color:red'>依附</span>$(incident)$ 于顶点 $V_i$ 和 $V_j$；顶点 $V_i$ 和 $V_j$ 与 $(V_i,V_j)$ <span style='color:red'>相关联</span>

- 和某个顶点 $V_i$ 相关联的边的数目称之为顶点 $V_i$ 的<span style='color:red'>度</span> $(Degree)$，记为 $: TD(V_i)$

- <span style='color:red'>一张无向图中的边数其实就是该图中各顶点度的和的一半</span>(多出的一半是因为重复两次计数)，简记之 $: \displaystyle e = \frac{1}{2} \sum_{i = 1}^{n} TD(V_i)$

<br/>

_*有向图中，顶点与边之间的关系*_

- 给定一张有向图 $G = (V,\{E\})$，如果弧 $<V_i,V_j>$ $\in E$，则称顶点 $V_i$ <span style='color:red'>邻接到</span>顶点 $V_j$；弧 $<V_i,V_j>$ 与顶点 $V_i$ 和 $V_j$ <span style='color:red'>相关联</span>

- 以顶点 $V$ 为头的弧的数目称为顶点 $V$ 的<span style='color:red'>入度</span>$(InDegree)$，记为 $: ID(V)$

- 以顶点 $V$ 为尾的弧的数目称为顶点 $V$ 的<span style='color:red'>出度</span>$(InDegree)$，记为 $: OD(V)$

- 顶点 $V$ 的度为 $TD(V) = ID(V) + OD(V)$

- <span style='color:red'>一张有向图中的边数其实各顶点的入度和亦或者出度和</span>，简记之 $: \displaystyle e = \sum_{i = 1}^{n} ID(V_i) = \sum_{i = 1}^{n} OD(V_i)$

<br/>

_*路径*_

在树形结构中，根节点到任意节点的路径是唯一的，但是在<span style='color:red'>一张图中，顶点与顶点之间的路径却是不唯一的</span>

> 顶点 $V_i$ 到 顶点 $V_j$ 中所经过的<span style='color:red'>顶点序列集</span>，称为从顶点 $V_i$ 到 顶点 $V_j$ 的路径<span style='color:red'>路径</span> $(Path)$，这条<span style='color:red'>路径的长度</span>，即为路径上的边或弧的数目

- 无向图中，顶点 $V$ 到 顶点 $V`$ 路径的序列集 $(V = V_{i,0}, V_{i,1} \dots , V_{i,m} = V`)$，其中 $(V_{i,j-1},V_{i,j}) \in E$，$1 \le j \le m$，如下图所示，列举的顶点 $B$ 到顶点 $D$ 的四种不同路径

	![2020-10-21-11-10-57](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-11-10-57.png)

- 有向图中，顶点 $V$ 到 顶点 $V`$ 路径的序列集 $(V = V_{i,0}, V_{i,1} \dots , V_{i,m} = V`)$，其中 $<V_{i,j-1},V_{i,j}>$ $\in E$，$1 \le j \le m$，如下图所示，列举的顶点 $B$ 到顶点 $D$ 的两种不同路径

	![2020-10-21-11-14-18](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-11-14-18.png)

- 第一个顶点到最后一个顶点相同的路径称为<span style='color:red'>回路或环</span> $(Cycle)$；顶点序列集中，顶点不重复出现的路径称为<span style='color:red'>简单路径</span>；除了第一个顶点和最后一个顶点外，其余顶点不重复出现的回路称为<span style='color:red'>简单回路或简单环</span>；如下图所示，左侧的环因第一个顶点和最后一个顶点都是 $B$，且 $C$、$D$、 $A$ 没有重复出现，因此是一个简单环，而右侧的环由于顶点 $C$ 的重复，无法构成简单环

  !!! INFO INFO

      注意有向图的回路的构建顺序，由于有向图中的边是作为有向的弧，故可能会出现形状是环状但是实际上却不是环的情景

	![2020-10-21-11-40-34](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-11-40-34.png)




<br/>

#### 连通图相关概念

<span id = "连通图相关概念"></span>

_*连通图*_

给定一张无向图 $G = (V,\{E\})$，如果顶点 $V_i$ 到顶点 $V_j$ <span style='color:red'>有路径</span>，则称 $V_i$ 和 $V_j$ 是<span style='color:red'>连通的</span>；如果对于一张图中的任意两个顶点 $V_i$ 、$V_j$ $\in E$，$V_i$ 和 $V_j$ 都是连通的，则称 $G$ 为<span style='color:red'>连通图</span> $(Connected-Graph)$；如下图，左图由于顶点 $E$ 、$F$ 无法与外层图进行连通，所以它不是一张连通图，而右图则为一张标准的连通图，其中，任意两顶点间都存在路劲相通

![2020-10-21-13-36-37](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-13-36-37.png)

无向图中的<span style='color:red'>极大连通子图</span>称为<span style='color:red'>连通分量</span>，它要求: 
- 必须作为为一张无向图中的子图而存在
- 子图本身也能够够造出一张连通图
- 连通子图含有最大顶点数，因为一张图中能够拆分出多种不同的子图，故只取顶点数最大的一张
- 具有极大顶点数的连通子图包含依附于这些顶点的所有边

如下图所给出的示范，其中，图一是一个无向非连通图，但是它却有两个极大连通子图，分别为: 图二、图三；而图四尽管页数与图一的子图，但是它却无法满足包含最大顶点数的条件

![2020-10-21-13-44-02](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-13-44-02.png)

![2020-10-21-13-44-14](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-13-44-14.png)

<br/>

_*强连通图*_

给定一张有向图 $G = (V,\{E\})$，如果对于每一对 $V_i,V_j$ $\in E$ 、$V_i \ne V_j$，从 $V_i$ 到 $V_j$ 和从 $V_j$ 到 $V_i$ 都存在路径，则称 $G$ 是<span style='color:red'>强连通图</span>

在有向图中的<span style='color:red'>极大强连通子图</span>称作为有向图的<span style='color:red'>强连通分量</span>，它要求: 
- 必须作为为一张有向图中的子图而存在
- 子图本身也能够够造出一张连通图
- 连通子图含有最大顶点数，因为一张图中能够拆分出多种不同的子图，故只取顶点数最大的一张
- 具有极大顶点数的连通子图包含依附于这些顶点的所有边

如下图，左图由于顶点 $D$ 无法与外层图进行连通，所以它不是一张强连通图；而右图则为一张标准的强连通图，并且它还是属于左图的强连通分量

![2020-10-21-16-24-24](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-16-24-24.png)


<br/>

_*关于连通图的生成树*_

<span id = "关于连通图的生成树"></span>

所谓的<span style='color:red'>连通图生成树</span>是一个<span style='color:red'>极小的连通子图</span>，<span style='color:red'>它含有图中全部的 $n$ 个顶点，但只有足以构成一棵树的 $n-1$ 条边，且图结构中不能存在环</span>

如下图: 图1是一张普通图，但显然它不是生成树，当去掉两条构成环的边后，比如图2或图3，就满足 $n$ 个顶点 $n - 1$ 条边且连通的定义了，它们都是一颗生成树

在这里也能够得到一个结论，即 : <span style='color:red'>如果一个图有 $n$ 个顶点和小于 $n-1$ 的边，则是非连通图，反之，如果它多于 $n-1$ 条边，则必定构成一个环，因为这条边使得它依附的那两个顶点之间有了第二条路径</span>；如图二和图三，我们会发现随便添加两个顶点之间的连线都能够构成一个环(包括子图的构造)，但是，<span style='color:red'>拥有 $n-1$ 条边并不意味着它一定是生成树</span>，如图四的例子

![2020-10-21-16-42-45](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-16-42-45.png)

<span style='color:red'>如果一个有向图恰好有一个顶点的入度为 0，其余顶点入度均为 1，则它同样属于一颗有向树</span>(所谓入度为 0 其实就相当于树中的根节点，其余顶点入度为 1 就相当于这颗树中非根节点的双亲只有一个)

<span style='color:red'>一个有向图的生成森林由若干颗有向树组成，含有图中的所有顶点，但只有足以构成若干颗不相交的有向树的弧</span>

如下图: 图1是一张有向图，去掉一些不必要的弧后，可以分解成图2和图3的两颗有向树，而图2和图3正是由于<span style='color:red'>不相交的特性</span>，它们能够共同的构成图1的生成森林

![2020-10-21-16-50-25](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-21-16-50-25.png)

<br/>

#### 图的抽象数据类型

<span id = "图的抽象数据类型"></span>

_*邻接矩阵*_

图的<span style='color:red'>邻接矩阵</span> $(Adjacency \: Matrix)$ 存储方式是用两个数组来表示图；一个一维数组存储图中顶点信息，一个二维数组 (称为邻接矩阵) 存储图中的边或弧的信息

> 设图 $G$ 有 $n$ 个顶点，则邻接矩阵是一个 $n \times n$ 的方阵，定义为 : <br/><br/>
> $
arc[i][j] = \begin{cases}
   1 &\text{if } \:\: (V_i,V_j) \in E &\text{or } <V_i,V_j> \in E \\
   0 &\text{contrary}
\end{cases}
$

<span style='color:red'>对于无向图而言，其邻接矩阵是一个[对称矩阵](https://baike.baidu.com/item/%E5%AF%B9%E7%A7%B0%E7%9F%A9%E9%98%B5)</span>，如下图所示，展示了一张无向图转换成邻接矩阵后的抽象数据结构

![2020-10-24-20-01-36](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-20-01-36.png)


对于有向图而言，其邻接矩阵和无向图稍有差异 : <span style='color:red'>因为有向图中的边都作为有向边(弧)，故，有向图邻接矩阵的抽象数据结构仅作为一个普通的矩阵而存在</span>，如下图所示

![2020-10-24-20-12-55](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-20-12-55.png)

对于网而言，由于网中的边是带有权的，故我们只需要把最开始的定义替换下即可适用 

> 设图 $G$ 有 $n$ 个顶点，则邻接矩阵是一个 $n \times n$ 的方阵，定义为 : <br/><br/>
> $
arc[i][j] = \begin{cases}
   W_{i,j} &\text{if } \:\: (V_i,V_j) \in E &\text{or } <V_i,V_j> \in E \\
   0 &\text{if } \:\: i = j \\
	 \infty &\text{contrary}
\end{cases}
$

在这里，$W_{i,j}$ 表示为 $(V_i,V_j)$ 或 $<V_i,V_j>$ 上的权值；$infty$ 是一个不可能的极限值，即代表着所对应下标的元素不存在两顶点间的依附边

![2020-10-24-20-25-26](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-20-25-26.png)


当拥有了这副矩阵，我们就可以很容易地知道图中的一些信息
- 我们要排查两顶点是否有边无边，即排查矩阵中对应顶点中的元是否值为 1
- 无向图中的邻接矩阵，我们通常研究的是顶点的度；顶点 $V_i$ 的度为矩阵中第 $V_i$ 行(或列)各元素之和
- 有向图中的邻接矩阵，我们通常研究的是顶点的出度和入度；顶点 $V_i$ 的入度为矩阵中第 $V_i$ 列各元素之和；顶点 $V_i$ 的出度为矩阵中第 $V_i$ 行各元素之和
- 求顶点 $V_i$ 的所有邻接点就是将矩阵中第 $i$ 行中的所有元素扫描一遍，当 $arc[i][j] = 1$ 时，所对应列的顶点就是作为 $V_i$ 的邻接点

下面的代码展示了如何构建一副以邻接矩阵为数据结构的图

```c
/** TODO: 依照《大话数据结构》的 图7-4-4 所构建 */

#define MAXVEX 100          /* Max count of vertex */
#define G_INFINITY 65535    /* Max value of weight */

typedef char vertex_type;   /* Type of vertex, it's might should be defined by users */
typedef int edge_type;      /* Type of weight, it's might should be defined by users */

typedef struct {
  vertex_type vexs[5]; /* Table of vertex */
  edge_type arc[5][5]; /* Adjacency Matrix, can be seen as table of edge */
  int vertexs_num, edges_num;
} n_graph;

void create_edge_node_by_pait(int i, int j, edge_type **arc) {
	arc[i][j] = 1;
}

void init_ort_graph(graph_ort *G) {
  memset(G, 0x0, sizeof(n_graph));

  G->vertexs_num = 5;
  G->edges_num   = 6;

  /* 初始化顶点表 */
  char ascii_start = '0';
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    G->vexs[i] = ascii_start;
    ++ascii_start;
  }

  /* 初始化邻接矩阵中所有元的默认值，默认为正无穷 */
  for (int i = 0; i < G->vertexs_num; ++i) {
    for (int j = 0; j < G->vertexs_num; ++j) {
      int val = 0;
      if (i != j) val = G_INFINITY;

      G->arc[i][j] = val;
	}
  }
}

void main(void) {
  n_graph *G = (n_graph *)malloc(sizeof(n_graph));
  init_graph(G);

  create_edge_node_by_pait(0, 4, G->arc);
  create_edge_node_by_pait(1, 0, G->arc);
  create_edge_node_by_pait(1, 2, G->arc);
  create_edge_node_by_pait(1, 0, G->arc);
  create_edge_node_by_pait(2, 0, G->arc);
  create_edge_node_by_pait(2, 3, G->arc);
  create_edge_node_by_pait(3, 4, G->arc);
}
```

<br/>

_*邻接表*_

邻接矩阵是不错的一种图存储结构，但是，对于边数相对顶点较少的图，这种结构是存在对存储空间的极大浪费的，如下图所示的一副稀疏有向图，邻接矩阵中除了 $arc[1][0]$ 有权值外，没有其它弧，那么其余的存储空间都被浪费掉了

![2020-10-24-21-58-03](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-21-58-03.png)

为此，结合了链式结构与线性结构的特性，<span style='color:red'>邻接表</span>就此诞生了，<span style='color:red'>邻接表 $(Adjacenct \: List)$ 为数组与链表相结合的存储方法</span>，其处理通常是这样的 : 
- 图中顶点 $V$ 用一个一维数组存储，另外，对于顶点数组中，每个数据元素还需要存储指向第一个邻接点的指针，以便于查找该顶点的边信息
- 图中每个顶点 $V$ 的所有邻接点构成一个线性表，由于邻接点的个数不定，所以用单链表存储，所构成的边表中 : 

  - 无向图称为顶点 $V_i$ 的<span style='color:red'>边表</span>
  - 有向图则称为顶点 $V_i$ 作为弧尾的<span style='color:red'>出边表</span> 或者是 以顶点 $V_i$ 作为弧头的 <span style='color:red'>入边表</span>

先从无向图开始，对于无向图而言，其所定义的邻接表如下图所示 : 

![2020-10-24-22-11-22](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-11-22.png)

而对于有向图，其邻接表结构也是类似的，但是需要注意的是，对于有向图而言其通常区分为以弧头作为存储标准还是以弧尾作为存储标准，这取决于我们对于给定一张图的研究方向，是主要针对顶点的入度进行考量呢还是主要针对顶点的出度进行考量呢？而依据这两种目的性的不同，所构造出来的邻接表结构又区分为<span style='color:red'>邻接表(出边表)</span>或者<span style='color:red'>逆邻接表(入边表)</span>

![2020-10-24-22-15-22](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-15-22.png)

对于网而言，仅需要对于链表中的元素结构中新增一个权值的成员即可，这里不做开展

这样的结构，对于我们要获得图的相关信息也是很方便的 : 
- 要想知道某个顶点的度，就去查找这个顶点的边表中结点的个数
- 若要判断顶点 $V_i$ 到 $V_j$ 是否存在边，只需要测试顶点 $V_i$ 的边表中 adjvex 是否存在结点 $V_j$ 的下标 $j$ 就行了
- 若求顶点的所有邻接点，其实就是对此顶点的边表进行遍历，得到的 adjvex 域对应的顶点就是邻接点

以下代码展示了以无向图为基准所构建出来的邻接表，需要补充的是，由于是以无向图为基准，故边的录入是可逆的，所以代码中还添加了对于重复录入边的校验规则；其次，由于所构造的是无向图，所以在每次构造出一个给定边的时候，我们还可以同时构造出该边所对等顶点的边信息(思考无向图中所构造出的对称矩阵)；对于有向图的构建和以下代码的函数展示是一致的，都是依赖于链表的头插法来进行

```c
/** TODO: 依照《大话数据结构》的 图7-4-6 所构建 */

typedef char vertext_type;
typedef int edge_type;

/* 边表节点 */
typedef struct edge_node {
  int adj_vex_idx;        /* 邻接点域，存储该顶点所对应的下标 */
  struct edge_node *next; /* 指向下一个邻接点 */
} edge_node;


/* 顶点表节点 */
typedef struct {
  vertext_type data;
  edge_node *first_edge;
} vertext_node;


/* 图结构 */
typedef struct {
  vertext_node adj_list[4];
  int vertexs_num;
  int edges_num;
} graph_adj;

bool verify_repeat(int idx, vertext_node *head) {
  bool flg = true;

  edge_node *nod = head->first_edge;
  while (true) {
    if (nod == NULL) break;

    if (nod->adj_vex_idx == idx) {
      flg = false;
      break;
    }

    nod = nod->next;
  }

  return flg;
}

void create_edge_node_by_pait(int i, int j, edge_type weight, vertext_node *adj_list) {
  if(adj_list == NULL) return;

  /**
   * 由于是无向图，边的录入是可以反方向的，故这里还要判断一下是否重复录入了，
   * 其实主要是因为下面的代码在构造出一个顺向边的时候又同时构造出其逆向边所导致的，
   * 如果正常录入边的话还不需要判断重复
  */
  if (!verify_repeat(j, &adj_list[i])) {
    return;
  }

  edge_node *e_i = (edge_node *)malloc(sizeof(edge_node));
  e_i->adj_vex_idx = i;
  e_i->next = adj_list[j].first_edge;
  adj_list[j].first_edge = e_i;

  /* 依赖于无向图的对称矩阵的特性，在构造玩顺向下标的边后还可以同时构造出其逆向下标的边 */
  edge_node *e_j = (edge_node *)malloc(sizeof(edge_node));
  e_j->adj_vex_idx = j;
  e_j->next = adj_list[i].first_edge;
  adj_list[i].first_edge = e_j;
}

void init_adjgraph(graph_adj *G) {
  memset(G, 0x0, sizeof(graph_adj));

  G->vertexs_num = 4;
  G->edges_num   = 5;

  /* 初始化顶点表 */
  char ascii_start = '0';
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    G->adj_list[i].data = ascii_start;
	G->adj_list[i].first_edge = NULL;
    ++ascii_start;
  }
}

void main(void) {
  graph_adj grap;
  init_adjgraph(&grap);

  create_edge_node_by_pait(0, 1, 0, G->adj_list);
  create_edge_node_by_pait(0, 2, 0, G->adj_list);
  create_edge_node_by_pait(0, 3, 0, G->adj_list);
  create_edge_node_by_pait(1, 0, 0, G->adj_list);
  create_edge_node_by_pait(1, 2, 0, G->adj_list);
  create_edge_node_by_pait(2, 0, 0, G->adj_list);
  create_edge_node_by_pait(2, 1, 0, G->adj_list);
  create_edge_node_by_pait(2, 3, 0, G->adj_list);
  create_edge_node_by_pait(3, 0, 0, G->adj_list);
  create_edge_node_by_pait(3, 2, 0, G->adj_list);
}
```

<br/>

_*十字邻接表*_

对于有向图来说，邻接表是有缺陷的。关心了出度问题，想了解入度就必须要遍历整个图才能知道，反之，逆邻接表解决了入度却不了解出度的情况，有没有可能把邻接表与逆邻接表结合起来呢？答案是肯定的，就是把它们整合在一起，这就衍生出了有向图的一种存储方法: <span style='color:red'>十字链表 $(OrthogonalList)$</span>

为此，我们需要重新定义下顶点表结点的结构，如下图 : 

![2020-10-24-22-37-58](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-37-58.png)

- firstin : 入边表头指针，指向该顶点的入边表中第一个结点
- firstout : 出边表头指针，指向该顶点的出边表中的第一个结点

而边表结点的结构也需重新定义，如下图 : 

![2020-10-24-22-39-21](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-39-21.png)


- tailvex : 弧起点在顶点表的下标
- headvex : 弧终点在顶点表中的下标
- headlink : 是指入边表指针域，指向终点相同的下一条边
- taillink : 是指出边表指针域，指向起点相同的下一条边
- 如果是网，还可以再增加一个 weight 域来存储权值

那么最终，其整合为下图所示的抽象数据结构

![2020-10-24-22-44-00](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-44-00.png)

我们不必把十字链表想的过于复杂，我们只需要把它理解为是有向图中的邻接表和逆邻接表这两种数据结构给柔和到一个链表中进行管控即可，下面的代码则展示了如何创建一个十字链表

```c
/** TODO: 依照《大话数据结构》的 图7-4-10 所构建 */

typedef char vertext_type;
typedef int edge_type;

/* 边表节点 */
typedef struct edge_node {
  int tail_idx;                /* 出边表的邻接节点域 */
  struct edge_node *next_tail; /* 指向出边表的下一个邻接点 */
  
  int head_idx;                /* 入边表的邻接节点域 */
  struct edge_node *next_head; /* 指向入边表的下一个邻接点 */
} edge_node;


/* 顶点表节点 */
typedef struct {
  vertext_type data;
  edge_node *first_out; /* 对应着出边表 */
  edge_node *first_in;  /* 对应着入边表 */
} vertext_node;


/* 图结构 */
typedef struct {
  vertext_node ort_list[4];
  int vertexs_num;
  int edges_num;
} graph_ort;


void create_edge_node_by_pait(int i, int j, vertext_node *ort_list) {
  if(ort_list == NULL) return;

  /* 初始化入边表的信息 */
  edge_node *e = (edge_node *)malloc(sizeof(edge_node));
  printf("[INFO] create %p\n", e);
  e->head_idx = j;
  e->next_head = ort_list[i].first_in;
  ort_list[i].first_in = e;

  /* 初始化出边表信息 */
  e->tail_idx = i;
  e->next_tail = ort_list[j].first_out;
  ort_list[j].first_out = e;
}

void init_ort_graph(graph_ort *G) {
  memset(G, 0x0, sizeof(graph_ort));

  G->vertexs_num = 4;
  G->edges_num   = 5;

  char ascii_start = '0';
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    G->ort_list[i].data = ascii_start;
    ++ascii_start;
  }
}

void main(void) {
  create_edge_node_by_pait(2, 1, G->ort_list);
  create_edge_node_by_pait(2, 0, G->ort_list);
  create_edge_node_by_pait(1, 2, G->ort_list);
  create_edge_node_by_pait(1, 0, G->ort_list);
  create_edge_node_by_pait(0, 3, G->ort_list);
}
```


<br/>

_*邻接多重表*_

如果我们在无向图的应用中，关注的重点是顶点，那么邻接表是不错的选择，但如果我们更关注边的操作，比如对已访问过的边做标记，删除某一条边等操作，那就意味着，需要找到这条边的两个边表结点进行操作，这其实还是比较麻烦的，如下图所示，若要删除左图的 $(V_0,V_2)$ 这条边，需要对邻接表结构中右边表的阴影两个结点进行删除操作，显然这是比较烦琐的

![2020-10-24-22-50-10](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-50-10.png)

为此，仿造十字链表的设计思想对原有的邻接表进行改造，重新定义了边表结点的结构如下图所示 : 

![2020-10-24-22-51-36](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-51-36.png)

其中 : 
- ivex / jvex : 与某条边依附的两个顶点在顶点表中下标
- link : 指向依附顶点 ivex 的下一条边
- jlink : 指向依附顶点 jvex 的下一条边
- ilink 指向的结点的 jvex 一定要和它本身的 ivex 的值相同

这就是<span style='color:red'>邻接多重表</span>结构

依照上面的构造方式，邻接多重表的抽象数据类型构造如下 : 

![2020-10-24-22-57-26](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-24-22-57-26.png)

<br/>

#### 图的遍历

<span id = "图的遍历"></span>

从图中某一顶点出发访遍图中其余顶点，且使每一个顶点仅被访问一次，这一过程就叫做<span style='color:red'>图的遍历 $(Traversing \: Graph)$</span>

因为图的任一顶点都可能和其余的所有顶点相邻接，极有可能存在沿着某条路径搜索后，又回到原顶点，而有些顶点却还没有遍历到的情况。因此我们需要在遍历过程中把访问过的顶点打上标记，以避免访问多次而不自知，具体办法是<span style='color:red'>设置一个访问数组 `visited[n]`，$n$ 是图中顶点的个数，初值为 0，访问过后设置为 1</span>

<span style='color:red'>通常有两种遍历次序方案，它们分别是深度优先遍历和广度优先遍历</span>

<br/>

_*深度优先遍历*_

<span style='color:red'>深度优先遍历 $(Depth \: First \: Search)$</span>，也有称为深度优先搜索，简称为DFS

深度优先遍历的设计理念和树的前序遍历的过程十分类似，<span style='color:red'>它从图中某个顶点 $V_i$ 出发，访问此顶点，然后从 $V_i$ 的未被访问的邻接点出发深度优先遍历图，直至图中所有和 $V_i$ 有路径相通的顶点都被访问到</span>，简而言之，即先保证从唯一一个方向遍历完所有的顶点，然后再逐个退回挨个沿着当前方向最近的邻接点接着遍历，以此往复

而对于非连通图而言，该算法也是同样能够适用的，<span style='color:red'>只需要对它的连通分量分别进行深度优先遍历，即在先前一个顶点进行一次深度优先遍历后，若图中尚有顶点未被访问，则另选图中一个未曾被访问的顶点作起始点，重复上述过程，直至图中所有顶点都被访问到为止</span>

下图则展示了一幅图在使用深度优先遍历的抽象过程(<span style='color:red'>注意虚线，这是沿路作为往返时会经历的一个判断是否被访问到的过程</span>)，<span style='color:red'>事实上，我们无需理会一张图的开始次序，因为作为图来说，从哪个顶点开始进行遍历都是一个合理的过程</span>

![2020-10-25-01-40-30](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-25-01-40-30.png)

下面的代码展示了以邻接矩阵作为存储结构的遍历方式 : 

```c
bool visted[5] = { false };

void DES(int i, n_graph *G) {
  visted[i] = true;
  printf("[info] search: %c\n", G->vexs[i]);

  for (size_t j = 0; j < G->vertexs_num; ++j) {
    if (G->arc[i][j] && !visted[j]) {
      DES(j, G);
    }
  }
}

void DES_traverse(n_graph *G) {
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    if (visted[i]) continue;

    DES(i, G);
  }
}
```

下面的代码展示了以邻接表作为存储结构的遍历方式 : 

```c
bool visted[5] = { false };

void DES(int i, vertext_node *adj_list) {
  visted[i] = true;
  printf("[info] search: %c\n", adj_list[i].data);

  edge_node *cur = adj_list[i].first_edge;
  while (cur) {
    if (!visted[cur->adj_vex_idx]) 
      DES(cur->adj_vex_idx, adj_list);

    cur = cur->next;
  }
}

void DES_traverse(graph_adj *G) {
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    if (visted[i]) continue;
    DES(i, G->adj_list);
  }
}
```

对比两个不同存储结构的深度优先遍历算法，对于 $n$ 个顶点 $E$ 条边的图来说，邻接矩阵由于是二维数组，要查找每个顶点的邻接点需要访问矩阵中的所有元素，因此其时间复杂度为: $O(n^{2})$；而邻接表做存储结构时，找邻接点所需的时间取决于顶点和边的数量，所以其时间复杂度为 $O(n+E)$。<span style='color:red'>显然对于点多边少的稀疏图来说，邻接表结构使得算法在时间效率上大大提高</span>

<br/>

_*广度优先遍历*_

<span style='color:red'>深度优先遍历 $(Breadth \: First \: Search)$</span>，也有称为广度优先搜索，简称为BFS

如果说图的深度优先遍历类似树的前序遍历，那么图的广度优先遍历就类似于树的<span style='color:red'>层序搜索</span>，注意，理解层序这点很关键，广度优先遍历总会先把一个层级的元素都遍历完，并且依赖于一个数据结构存储当前层级中能够被成功遍历的元素(即还未被访问的)，然后在遍历完当前层级后再挨个取出上一步所存储的元素，以此往复，如下图所示

![2020-10-26-00-52-40](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-10-26-00-52-40.png)

下面的代码展示了以邻接矩阵作为存储结构的遍历方式 : 

```c
bool visted[5] = { false };

void BFS(int _i, n_graph *_G, Queue *_queue) {
  visted[_i] = true;

  AddQ(_queue, _i);

  while (!IsEmptyQ(_queue)) {
    _i = DeleteQ(_queue);
    for (int j = 0; j < _G->vertexs_num; ++j) {
      if(_G->arc[_i][j] == 1 && !visted[j]) {
        visted[j] = true;
        AddQ(_queue, j);
      }
    }
  }
}

void BFS_traverse(n_graph *_G) { 
  memset(visted, false, sizeof(visted));

  Queue *queue = CreateQueue();

  for (size_t i = 0; i < _G->vertexs_num; ++i) {
    if (visted[i]) continue;
    BFS(i, _G, queue);
  }
}
```

下面的代码展示了以邻接表作为存储结构的遍历方式 : 

```c
bool visted[5] = { false };

void BFS(int _i, graph_adj *_G, Queue *_queue) {
  visted[_i] = true;

  AddQ(_queue, _i);

  while (!IsEmptyQ(_queue)) {
    _i = DeleteQ(_queue);
    edge_node *cur = _G->adj_list[_i].first_edge;

    while (cur) {
      if(!visted[cur->adj_vex_idx]) {
        visted[cur->adj_vex_idx] = true;
        AddQ(_queue, cur->adj_vex_idx);
      }
      cur = cur->next;
    }
  }
}

void BFS_traverse(graph_adj *_G) { 
  memset(visted, false, sizeof(visted));

  Queue *queue = CreateQueue();

  for (size_t i = 0; i < _G->vertexs_num; ++i) {
    if (visted[i]) continue;
    BFS(i, _G, queue);
  }
}
```

<br/>

_*关于广度优先算法所构造出的无权图的最短路径*_

<span id = "关于广度优先算法所构造出的无权图的最短路径"></span>

虽然无权图的最短路径没有什么研究价值，但事实上，[Dijksktra](#Dijksktra) 的最短路径的算法也是基于这个所演变过来的

在无权图中，设每条邻接边的权恒为 1，并使用一个 parent 数组记录下最短路径，关于具体原理，参考这两个视频的讲解
- [正月点灯笼 - BFS和DFS算法（第1讲）](#https://www.bilibili.com/video/BV1Ks411579J)
- [正月点灯笼 - BFS和DFS算法（第2讲）](#https://www.bilibili.com/video/BV1Ks411575U)

```c
bool visted[5] = { false };

int *BFS(int _i, graph_adj *_G, Queue *_queue) {
  int parent[_G->vertexs_num];
  memset(parent, 0x0, sizeof(int) ( _G->vertexs_num));

  visted[_i] = true;

  AddQ(_queue, _i);

  while (!IsEmptyQ(_queue)) {
    _i = DeleteQ(_queue);
    edge_node *cur = _G->adj_list[_i].first_edge;

    while (cur) {
      if(!visted[cur->adj_vex_idx]) {
        visted[cur->adj_vex_idx] = true;
        parent[cur->adj_vex_idx] = _i;
        AddQ(_queue, cur->adj_vex_idx);
      }
      cur = cur->next;
    }
  }
}

void BFS_traverse(graph_adj *_G) { 
  memset(visted, false, sizeof(visted));

  Queue *queue = CreateQueue();

  for (size_t i = 0; i < _G->vertexs_num; ++i) {
    if (visted[i]) continue;
    BFS(i, _G, queue);
  }
}
```


<br/>

_*小结*_

对比图的深度优先遍历与广度优先遍历算法，它们在时间复杂度上是一样的，不同之处仅仅在于对顶点访问的顺序不同，可见两者在全图遍历上是没有优劣之分，只是视不同情况采取不同的策略

不过如果图顶点和边非常多，不能在短时间内遍历完成，遍历的目的是为了寻找合适的顶点，那么选择哪种遍历就要仔细斟酌了。<span style='color:red'>深度优先更适合目标比较明确，以找到目标为主要目的的情况</span>；<span style='color:red'>而广度优先更适合在不断扩大遍历范围时找到相对最优解的情况</span>

<br/>

#### 最小生成树

<span id = "最小生成树"></span>

> 最小生成树属于有环图的应用

在前面的[图的定义](#关于连通图的生成树)小节中提到过图的生成树的定义，而<span style='color:red'>构造连通网的最小代价生成的树则称为最小生成树 $(Minimum \:\: Cost \:\: Spaning \:\: Tree)$</span>

找连通网的最小生成树，经典的有两种算法，它们分别为Prim(普里姆)算法和Kruskal(克鲁斯卡尔)算法，对比两个算法，<span style='color:red'>Kruskal算法主要是针对边来展开，边数少时效率会非常高，所以对于稀疏图有很大的优势</span>；<span style='color:red'>而普里姆算法对于稠密图，即边数非常多的情况会更好一些</span>


<br/>

_*Prim*_

假设 $N = (P,\{E\})$ 是连通网，$TE$ 是 $N$ 上最小生成树中边的集合。算法从 $U=\{u_0\} \: (u_0 \in E)$，$TE = \{\}$ 开始。重复执行下述操作 : 在所有 $u \in U$，$v \in V - U$ 的边 $(u，v) \in E$ 中找一条代价最小的边 $(u_0,v_0)$ 并入集合 $TE$，同时 $v_0$ 并入 $U$，直至 $U=V$ 为止。此时 $TE$ 中必有 $n-1$ 条边，则 $T = (V,\{TE\})$ 为 $N$ 的最小生成树

Prim 算法的时间复杂度为 $O(n^2)$

```c
void min_span_tree_prim(n_graph *G) {
  int adj_vexs[G->vertexs_num]; /* 保存顶点的下标 */
  int low_cast[G->vertexs_num]; /* 保存权值，它的下标所对应的值和 adj_vexs 所相呼应
                                    并且该数组所对应的下标为0的时候，也意味着该下标的顶
                                    点已经被纳入至最小生成树当中 */

  low_cast[0] = 0; /* 设最小生成树从0顶点开始，也就是说在最开始的时候，顶点0就被纳入到最小生成树当中去了 */
  adj_vexs[0] = 0;

  /* 依照第一个顶点(0)的那一行中的所有的值都初始化 */
  for (size_t i = 1; i < G->vertexs_num; ++i) {
    adj_vexs[i] = 0;
    low_cast[i] = G->arc[0][i];
  }

  /**
   * 以下的循环至始至终都要过滤掉顶点为 0 的，因为它在最
   * 开始的时候已经被纳入到最小生成树了，故需要跳过进行判断
 */
  for (size_t i = 1; i < G->vertexs_num; ++i) {
    int k = 0;
    int min = G_INFINITY;
    /* 从 low_cast 专门用于保存权值的数组中筛选出一个最小值 */
    for (size_t j = 1; j < G->vertexs_num; ++j) {
      if(low_cast[j] != 0 && low_cast[j] < min) {
        min = low_cast[j];
        k = j;
      }
    }

    /* 最小值的顶点纳入到最小生成树当中 */
    printf("(%d, %d)\n", adj_vexs[k], k);
    low_cast[k] = 0;

    /* 初始化 low_cast，从第 k 行初始化 */
    for (size_t j = 1; j < G->vertexs_num; ++j) {
      if(low_cast[j] != 0 && G->arc[k][j] < low_cast[j]) { /* 这一步: G->arc[k][j] < low_cast[j]，意味着可能会覆
                                                              盖掉，但是仔细想想看，该算法是查找最小的权，故覆盖掉是合理的 */
        low_cast[j] = G->arc[k][j];
        adj_vexs[j] = k;
      }
    }
  }
}
```

<br/>

_*Kruskal*_

Kruskal 算法主要是依赖于[并查集](#Disjoint-Set)来完成，它会提前先找出图中的所有的边，并依照权值进行排序，依次遍历这些边并归并为集合中，通过并查集的特性查找不存在构成环(因为构成环则必定不属于生成树，详情查看[这一小节](#关于连通图的生成树))的边则属于最小生成树的边

假设 $N = (V,\{E\})$ 是连通网，则令最小生成树的初始状态为只有 $n$ 个顶点而无边的非连通图 $T = \{V,\{\})$，图中每个顶点自成一个连通分量。在 $E$ 中选择代价最小的边，若该边依附的顶点落在 $T$ 中不同的连通分量上，则将此边加入到 $T$ 中，否则舍去此边而选择下一条代价最小的边。依次类推，直至 $T$ 中所有顶点都在同一连通分量上为止

此算法在查找树的根节点(find_root) 由边数 $e$ 决定(可以进行优化，详情请查看[并查集](#Disjoint-Set)章节)，时间复杂度为 $O(loge)$，而外面有一个 for 循环 $e$ 次，所以克鲁斯卡尔算法的时间复杂度为 $O(eloge)$

```c
int find_root(int *parent, int x) {
  if(parent[x] == x) {
    return x;
  }

  find(parent, parent[x]);
}

void min_span_tree_kruskal(n_graph *G) {
  Edge edges[G->edges_num];
  int parent[G->vertexs_num];

  /* 这里省略掉对 edges 数组进行初始化的代码，简而言之就是构造出图中的所有的边并依照权值进行排序即可 */

  for (size_t i = 0; i < G->vertexs_num; ++i) {
    parent[i] = 0;
  }

  for (size_t i = 0; i < G->vertexs_num; ++i) {
    int x_root = find_root(parent, edges[i].begin);
    int y_root = find_root(parent, edges[i].end);

    if(n != m) {
      parent[x_root] = y_root;
      printf("(%d, %d)\n", edges[i].begin, edges[i].end);
    }
  }
}
```

<br/>

#### 最短路径

<span id = "最短路径"></span>

> 最短路径属于有环图的应用

在网图和非网图中，最短路径的含义是不同的。由于非网图它没有边上的权值，所谓的最短路径，其实就是指两顶点之间经过的边数最少的路径；<span style='color:red'>而对于网图来说，最短路径，是指两顶点之间经过的边上权值之和最少的路径，并且我们称路径上的第一个顶点是源点，最后一个顶点是终点</span>


![2020-11-02-13-24-31](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-02-13-24-31.png)


<br/>

_*Dijkstra*_

<span id = "Dijksktra"></span>

Dijkstra 算法求得最短路径是基于 [关于广度优先算法所构造出的无权图的最短路径](#关于广度优先算法所构造出的无权图的最短路径) 所演变而来的，两者之间非常的相似，但是，由于需要求得最短的路径，即邻接点间的权值需要为最小值，故原先的队列需要替换为<span style='color:red'>优先队列</span> 来使用，其中，元素的值则作为能够找到当前顶点的唯一标识，而优先度则作为<span style='color:red'>已归纳为最短路径的顶点到当前顶点的权</span>

关于该算法的视频讲解请查看 [正月点灯笼 - BFS和DFS算法（第3讲）](#https://www.bilibili.com/video/BV1ts41157Sy)

```c
void dijkstra(n_graph *G) {
  int parent[G->vertexs_num];  /* 存储最短路径的一维数组 */
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    parent[i] = -1;
  } 

  priority_queue queue;

  /* 初始化原点 */
  node nod_first;
  nod_first.idx = 0;
  nod_first.priority = 0;
  queue.push(&queue, nod_first)
  parent[0] = 0;

  while (!is_empty(&priority_queue)) {
    node pre = pop(&queue);

    /**
     * 如果上一个顶点并没有被访问过，则添加访问，
     * 在这一步中才加入队列和广度优先算法存在一
     * 点区别，因为只有在下一次循环的时候，优先
     * 队列才会对上一次所加入的顶点进行排序，而
     * 我们在下一次则拿到这个排序的结果才加入到
     * 队列中
    */
    if (!visited[pre.idx]) {
      visited[pre.idx] = true;
    }

    for (int j = 0; j < G->vertexs_num; ++j) {
      if(G->arc[pre.idx][j] != 0 &&
         G->arc[pre.idx][j] != G_INFINITY &&
         !visited[j] /* 如果 pre 的邻接点 j 已经被访问过，则跳过当前顶点，否则会出现无限循环 */ ) {
        /* 当前顶点入队 */
        node tmp;
        tmp.idx = j;
        tmp.priority = G->arc[j][pre.idx]
        push(&queue, tmp);

        /* 纳入最短路径的顶点之中 */
        parent[j] = pre.idx; 
      }
    }
  }
}
```

Dijkstra 算法解决了从某个源点到其余各顶点的最短路径问题，从循环嵌套可以很容易得到此算法的时间复杂度为 $O(n^2)$，或许有人会认为，如果我指定了所需要找到的某个源点和终点，那是否就可以缩小该算法的时间复杂度了？事实上，这种做法是并不可取的，这个问题和求源点到其他所有顶点的最短路径一样复杂，时间复杂度依然是 $O(n^2)$，这就好比，你吃了七个包子终于算是吃饱了，就感觉很不划算，前六个包子白吃了，应该直接吃第七个包子，于是你就去寻找可以吃一个就能饱肚子的包子，能够满足你的要求最终结果只能有一个，那就是用七个包子的面粉和馅做的一个大包子，<span style='color:red'>这种只关注结果而忽略过程的思想是非常不可取的</span>

那么当我们想通过该算法来知道任意源点到任意终点的距离，此时简单的办法就是对每个顶点当作源点运行一次 Dijkstra 运算，等于在原有算法的基础上，再来一次循环，此时整个算法的时间复杂度就成了 $O(n^3)$

<br/>

_*Floyd*_

Floyd 算法依赖于两个数组，他们分别为 :
- D : 用于存储权值的数组，在还未开始修改前，它即为邻接矩阵的复制，在算法运行结束后，它对迎着节点 $v$ 到节点 $w$ 的最短路径权值和
- P : 存储最短路径的具体节点，默认情况下它初始化为对应节点的值，在算法运行结束后，它用于存储最短路径节点的前置下标

在算法核心中，依赖于以下公式对上面所说的两个数组进行修改

> 设 $i$ 为循环的每个节点，Floyd 算法需要计算出任意节点到任意节点间的最短路径
> 
> 设 $v$、$w$ 作为邻接矩阵中的索引 ; $v$ 作为行索引 ; $w$ 作为列索引
> 
> 令:
> - $D[v][w] = min\{D[v][w],D[v][i] + D[i][w]\} \:$
> - $P[v][w] = P[w][i]$
> 
> ![2020-11-02-22-40-37](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-02-22-40-37.png)

```c
void shortest_path_floyd(n_graph *G) {
  int P[G->vertexs_num][G->vertexs_num];
  int D[G->vertexs_num][G->vertexs_num];
  for (size_t v = 0; v < G->vertexs_num; ++v) {
    for (size_t w = 0; w < G->vertexs_num; ++w) {
      P[v][w] = w;
      D[v][w] = G->arc[v][w];
    }
  }
  
  for (size_t i = 0; i < G->vertexs_num; ++i) {
    for (size_t v = 0; v < G->vertexs_num; ++v) {
      for (size_t w = 0; w < G->vertexs_num; ++w) {
        if (D[v][w] > D[v][i] + D[i][w]) {
          D[v][w] = D[v][i] + D[i][w];
          P[v][w] = P[v][i];
        }
      }
    }
  }

  /* 打印路径 */
  for (int v = 0; v < G->vertexs_num; ++v) {
    for (int w = v + 1; w < G->vertexs_num; ++w) {
      printf("v%d - v%d weight: %d\n", v, w, D[v][w]);
      int k = P[v][w];
      printf(" path: %d", v);
      while (w != k) {
        printf(" -> %d", k);
        k = P[k][w];
      }
      printf(" -> %d\n", w);
    }
    printf("\n");
  }
}
```

可以发现，Floyd 算法由于 $i$ 的存在，故其默认情况下都会求出任意点到任意点之间的最短路径，这和 Dijkstra 算法稍显不同 ; Dijkstra 针对的是具体的某个源点到终点，如需求出任意点之间的路径，则需要在外层套一个循环 ; 而 Floyd 算法则是针对的整张网

虽然从求任意点的最短路径这一例题来说，Floyd 和 Dijkstra 算法的时间复杂度都需要 $O(n^3)$，但是我们会发现，Floyd 算法再使用的时候会显得更加的优雅


<br/>

#### 拓补排序

<span id = "拓补排序"></span>

我们会把施工过程、生产流程、软件开发、教学安排都当成一整个项目工程来对待，其中，所有的工程都可以分为若干个活动的子工程(<span style='color:red'>也称活动</span>)。在这些活动之间，通常会受到一定的条件约束，如其中的某些活动必须在另一些活动完成后才能开始，就像电影制作不可能在人员到位进驻场地时，导演还没找到，也不可能在拍摄的过程中连场地也没有，这都会将导致荒谬的结果，<span style='color:red'>因此，这样的工程图一定是作为无环有向图而存在</span>

![2020-11-03-10-53-24](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-03-10-53-24.png)

> 在一个表示工程的有向图中，用顶点表示活动，用弧表示活动之间的依赖关系，这样的有向图为顶点表示活动的网，我们称为 <span style='color:red'>AOV 网 $(Activity \: On \: Vertex \: Network)$</span>，其中 :
> - 网中的弧表示两活动之间存在的某种相互的制约依赖关系
> - 网中不能存在环 (让某个活动以自己的开始作为先决条件是不可取的)

通过 AOV 网，我们能够在其中找到多条不同的<span style='color:red'>拓补序列</span> :

> 设 $G = \{V,E\}$ 是一个具有 $n$ 个顶点的有向图，$V$ 中的顶点序列 $\{v_1,\:v_2,\:\dots,\: v_n\}$ 满足若从顶点 $v_i$ 到 $v_j$ 有一条路径，则在顶点序列中顶点 $v_i$ 必在顶点 $v_j$ 之前，则我们称这样的顶点序列为一个<span style='color:red'>拓补序列</span>

<br/>

<span style='color:red'>而所谓的拓补排序，其实就是对一个有向图构造拓补序列的过程</span>，在构造时会存在两种结果
- 如果所构造的序列中，图中的顶点全部输出，则说明当前图是不存在环的 AOV 网
- 如果所构造的序列中，所输出的顶点少于图中的顶点数，则说明这个网存在回路

对 AOV 网进行拓补排序的基本思路是 : 

> 从 AOV 网中选择一个入度为 0 的顶点输出，然后删去此节点，并删除以此顶点为尾的弧(<span style='color:red'>作为弧头的顶点的入度 -1</span>)，继续重复此过程，直至输出全部顶点或是 AOV 网中不存在入度为 0 的顶点为止

```c
#include <stdbool.h>
#include "stack.h"

typedef char vertext_type;
typedef int edge_type;

typedef struct edge_node {
  int adj_vex_idx;       
  struct edge_node *next;
} edge_node;


typedef struct {
  vertext_type data;
  int in;
  edge_node *first_edge;
} vertext_node;


typedef struct {
  vertext_node adj_list[14];
  int vertexs_num;
  int edges_num;
} graph_adj;

/**
 * @breif 拓补排序
 * @param
 *  - graph 网
 * @return 
 *  - true 拓补序列构造成功，此图为 AOV 网，即不存在环
 *  - false 拓补序列构造失败，此图不是 AOV 网，即存在环
*/
bool topo_logic_sort(graph_adj *grap) {
  bool status = true;
  int count = 0; /* 统计纳为拓补序列的顶点个数，当该值小于图中
                    顶点个数的和时，则证明无法生成拓补序列 */
  Stack *stack = init_stack();

  /* 寻找是否存在入度为0的节点，如果存在，则入栈 */
  for (size_t i = 0; i < grap->vertexs_num; ++i) {
    if (grap->adj_list[i].in == 0) {
      push(stack, i);
    }
  }

  while (stack->s_size) {
    int vex = pop(stack);

    printf("%d ", vex);
    ++count;

    for (edge_node *node = grap->adj_list[vex].first_edge; node; node = node->next) {
      int k = node->adj_vex_idx;
      if ((--grap->adj_list[k].in) == 0) {
        push(stack, k);
      }
    }
  }

  if (count < grap->vertexs_num) { /* 存在环 */
    status = false;
  }

  return status;
}

```

分析整个算法，对一个具有 $n$ 个顶点 $e$ 条弧的 $AOV$ 网来说，第一次在顶点表中扫描入度为 0 的顶点入栈的时间复杂度为 $O(n)$，而 while 循环中共执行了 $e$ 次，故整个算法的时间复杂度为 $O(n + e)$

<br/>

#### 关键路径 (待填坑)

<span id = "关键路径-待填坑"></span>

COMING SOON

<br/>

<span id = "Search"></span>

### Search

---

只要你打开电脑，就会涉及到查找技术，如炒股软件中查股票信息、硬盘文件中找照片、在光盘中搜 DVD，甚至玩游戏时在内存中查找攻击力、魅力值等数据修改用来作弊等，都要涉及到查找，当然，在互联网上查找信息就更加是家常便饭，<font color = "red">所有这些需要被查的数据所在的集合，我们给它一个统称叫查找表</font>

- 查找表 $(Search \: \:Table)$ 是由同一类型的数据元素(或记录)构成的集合

- 关键字 $(Key)$ 是数据元素中某个数据项的值，又称为键值，用它可以标识一个数据元素 ; 也可以标识一个记录的某个数据项(字段)，我们称为<font color = "red">关键码</font>

  - 若此关键字可以唯一地标识一个记录，则称此关键字为主关键字 $(Primary \:\: Key)$，这也就意味着，对不同的记录，其主关键字均不相同

  - 若此可以识别多个数据元素(或记录)，我们称为次关键字 $(Secondary \:\: Key)$，次关键字也可以理解为是不以唯一标识一个数据元素(或记录)的关键字，它对应的数据项就是次关键码

<font color = "red">查找 $(Searching)$ 就是根据给定的某个值，在查找表中确定一个其关键字等于给定值的数据元素(或记录)</font>


<br/>

#### 查找的种类

<span id = "查找的种类"></span>

查找表按照操作方式来分有两大种 : 

1. <font color = "red">静态查找表 $(StaticSearch \:\: Table)$</font> ∶ 只作查找操作的查找表，它的主要操作有 :
  
   - 查询某个 **特定的** 数据元素是否在查找表中
  
   - 检索某个 **特定的** 数据元素和各种属性

按照我们大多数人的理解，查找，当然是在已经有的数据中找到我们需要的，静态查找就是在干这样的事情。不过，现实中还有存在这样的应用 : <font color = "red">查找的目的不仅仅只是查找</font>

比如网络时代的新名词，如反应年轻人生活的 "蜗居"、"蚁族"、"孩奴"、"啃老" 等，以及 "X客" 系列如博客、播客、闪客、黑客、威客等，如果需要将它们收录到汉语词典中，显然收录时就需要查找它们是否存在，以及找到如果不存在时应该收录的位置。再比如，如果你需要对某网站上亿的注册用户进行清理工作，注销一些非法用户，你就需查找到它们后进行删除，删除后其实整个查找表也会发生变化，对于这样的应用，我们就需要使用动态查找表

2. <font color = "red">动态查找表 $(Dynamic Search Table)$</font> ∶ 在查找过程中同时插入查找表中不存在的数据元素，或者从查找表中删除已经存在的某个数据元素，动态查找表的主要操作有 : 
  
     - 查找时插入数据元素
  
     - 查找时删除数据元素

<br/>

#### 顺序表查找
<span id = "顺序表查找"></span>

我们可以把散落的图书可以理解为一个集合，而将它们排列整齐，就如同是将此集合构造成一个线性表，<font color = "red">我们要针对这一线性表进行查找操作，因此它就是静态查找表</font>

> 顺序查找 $(Sequential \:\: Search)$ 又叫线性查找，是最基本的查找技术，它的查找过程是 ∶ 从表中第一个(或最后一个)记录开始，逐个进行记录的关键字和给定值比较，若某个记录的关键字和给定值相等，则查找成功，找到所查的记录;如果直到最后一个(或第一个)记录，其关键字和给定值比较都不等时，则表中没有所查的记录，查找不成功

```c
int foo(int *arrary, int key, int n) {
  for (size_t i = 0; i < n; ++i) 
    if(arrary[i] == key) 
      return i;
}

int main(void) {
  int arrary[] = { 1, 6, 3, 5, 9 };
  int idx = foo(arrary, 5, sizeof(arrary) / sizeof(int));

  return EXIT_SUCCESS;
}
```

对于这种顺序查找算法来说，查找成功最好的情况就是在第一个位置就找到了，算法时间复杂度为 $O(1)$，最坏的情况是在最后一位置才找到，需要 $n$ 次比较，时间复杂度为 $O(n)$，当查找不成功时，需要 $n+1$ 次比较，时间复杂度为 $O(n)$，关键字在任何一位置的概率是相同的，所以平均查找次数为 $\displaystyle \frac{(n+1)}{2}$，所以最终时间复杂度还是 $O(n)$


<br/>

#### 有序表查找
<span id = "有序表查找"></span>

我们如果仅仅是把书整理在书架上，要找到一本书还是比较困难的，也就是刚才讲的需要逐个顺序查找。但如果我们在整理书架时，将图书按照书名的拼音排序放置，那么要找到某一本书就相对容易了，说白了，就是对图书做了有序排列，一个线性表有序时，对于查找总是很有帮助的

<br/>

_*折半查找*_

> 折半查找$(Binary \:\: Search)$技术，又称为<font color = "red">二分查找</font>，它的前提是线性表中的记录必须是关键码有序(通常从小到大有序)，线性表必须采用顺序存储，折半查找的基本思想是 ∶ 在有序表中，取中间记录作为比较对象，若给定值与中间记录的关键字相等，则查找成功 ; 若给定值小于中间记录的关键字，则在中间记录的左半区继续查找 ; 若给定值大于中间记录的关键字，则在中间记录的右半区继续查找，不断重复上述过程，直到查找成功，或所有查找区域无记录，查找失败为止

一个符合的例子就是猜 100 数字以内的小游戏，每次依据对比的结果取中间记录，直至猜到数字为止，如下图所示

![2020-11-04-23-58-07](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-04-23-58-07.png)

```c
int binary_search(int *arrary, int key, int size) {
  int low = 0;
  int hig = size - 1;

  while (low <= hig) {
    int mid = (low + hig) / 2;  /* 取中间下标 */
    if(key < arrary[mid])       /* 若 key 值比预估的数小，则最大指针下标需要从 mid 处 -1(跳过重复的判断) */
      hig = mid - 1;
    else if (key > arrary[mid]) /* 若 key 值比预估的数大，则最小指针下标需要从 mid 处 +1(跳过重复的判断) */
      low = mid + 1;
    else                        /* 若最大和最小相等，则意味着 low 和 high 指针指向同一个下标，即找到了位置 */
      return mid;
  }

  return -1;
}

int main(void) {
  int arrary[] = { 0, 1, 16, 24, 35, 47, 59, 62, 73, 88, 99 };
  binary_search(arrary, 62, sizeof(arrary) / sizeof(int));

  return EXIT_SUCCESS;
}
```

在前面二叉树的性质小节中，有过对 "具有 $n$ 个结点的完全二叉树的深度为 $└log2^n┘ + 1$ 性质的推导过程，在这里尽管折半查找判定二叉树并不是完全二叉树，但同样相同的推导可以得出，最坏情况是查找到关键字或查找失败的次数为$└log2^n┘ + 1$，对于最好的情况，，当然是1次了。因此最终我们<font color = "red">折半算法的时间复杂度为 $O(logn)$</font>，它显然远远好于顺序查找的 $O(n)$ 时间复杂度了

由于折半查找的前提条件是需要有序表顺序存储，对于静态查找表，一次排序后不再变化，这样的算法已经比较好了，<font color = "red">但对于需要频繁执行插入或删除操作的数据集来说，维护有序的排序会带来不小的工作量，那就不建议使用</font>

<br/>

_*插值查找*_

<font color = "red">反观二分查找，思考一个问题就是，为何一定是对半分，而不是四分之一亦或者更多呢？</font>因为从结果来看，如果我们所需要的 $key$ 在一个较小亦或者较大的位置的话，那么我们分的越多那么所节省的时间也就越多，举个例子 : 比如要在取值范围0～10000 之间 100 个元素从小到大均匀分布的数组中查找 5，我们自然会考虑从数组下标较小的开始查找

在二分查找中，获取中位数下标的公式经过略微等式变化后可以得到 : $\displaystyle mid = \frac{low + hight}{2} = low + \frac{1}{2}(hight - low)$，也就是 $mid$ 等于最低下标 $low$ 加上最高下标 $high$ 的差的一半，而当我们把上面的公式改进为 : $\displaystyle mid =  low + \frac{key-a[low]}{a[high - a[low]}(hight - low)$，就能够根据我们所录入的 $key$ 去动态的调整所需 <font color = "red">"对折"</font> 的大小了

> 插值查找 $(Interpolation \:\: Search)$ 是根据要查找的关键字 $key$ 与查找表中最大最小记录的关键字比较后的查找方法，其核心就在于插值的计算公式 $\displaystyle \frac{key-a[low]}{a[high - a[low]}$ `

```c
int binary_search(int *arrary, int key, int size) {
  int low = 0;
  int hig = size - 1;

  while (low <= hig) {
    int mid = low + (hig - low) * (key - arrary[low]) / (arrary[hig] - arrary[low]);  /* 插值 */
    if(key < arrary[mid])       /* 若 key 值比预估的数小，则最大指针下标需要从 mid 处 -1(跳过重复的判断) */
      hig = mid - 1;
    else if (key > arrary[mid]) /* 若 key 值比预估的数大，则最小指针下标需要从 mid 处 +1(跳过重复的判断) */
      low = mid + 1;
    else                        /* 若最大和最小相等，则意味着 low 和 high 指针指向同一个下标，即找到了位置 */
      return mid;
  }

  return -1;
}

int main(void) {
  int arrary[] = { 0, 1, 16, 24, 35, 47, 59, 62, 73, 88, 99 };
  binary_search(arrary, 62, sizeof(arrary) / sizeof(int));

  return EXIT_SUCCESS;
}
```

从时间复杂度来看，它也是 $O(n)$，但对于表长较大，而关键字分布又比较均匀的查找表来说，插值查找算法的平均性能比折半查找要好得多 ; 反之，数组中如果分布类似 $\{0,1,2,2000,2001,…,99998,9999\}$ 这种极端不均匀的数据，用插值查找未必是很合适的选择


<br/>

#### 线性索引查找
<span id = "线性索引查找"></span>

事实上，大部分应用场景下，数据集都无法按照有序进行存储，例如 : 服务器上的日志记录、大型论坛的帖子和回复，这些数据的增长方式都是海量级的，那么在这个基础上，要保证记录全部是按照当中的某个关键字有序，无疑其时间代价是非常高昂的，所以这种数据通常都是按<font color = "red">先后顺序存储</font>，那么针对于这种无序的数据集，引入索引的使用就能很大的程度改善查找上的时间复杂度

> <font color = "red">索引就是把一个关键字与它对应的记录相关联的过程</font>
> - 一个索引由若干个索引项构成
> - 每个索引项至少应包含关键字和其对应的记录在存储器中的位置等信息
> 
> 索引技术是组织大型数据库以及磁盘文件的一种重要技术

索引按照结构可以分为<font color = "red">线性索引、树形索引和多级索引</font> ; <font color = "red">所谓线性索引就是将索引项集合组织为线性结构，也称为索引表</font> ; 而线性索引中，又可以划分为<font color = "red">稠密索引、分块索引和倒排索引</font>

<br/>

_*稠密索引*_

<font color = "red">稠密索引是指在线性索引中，将数据集中的每个记录对应一个索引项</font>，一个具体的例子就是在家里找东西，当我使用一个小本子记录下家里一切物品的具体位置后，那么每次我想找某样东西时，我只需根据这个东西的 "关键码" 去笔记本里面查找就能够很快的找到我想要的物品的具体位置所在，而在这里，这个笔记本就是属于前面屡次所提到的 "索引表"

![2020-11-06-00-41-58](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-06-00-41-58.png)

<font color = "red">由于稠密索引要应对的可能是成千上万的数据，因此，为了保证在稠密索引中的筛选效率，对于稠密索引这个索引表来说，索引项一定是按照关键码有序的排列</font>，这也就意味着，我们在检索索引表的时候，就可以使用[有序表查找](#有序表查找)中所介绍的几种算法来快速的定位到要查找的索引项


对于稠密索引能够使用有序查找的方式快速定位到我们要具体查找的数据项，这显然是属于稠密索引的优点，但是如果数据量还是非常巨大乃至上十亿的话，那么也就意味着稠密索引也同样需要一个相同规模的数据集来存储索引表，那么相对的，对于内存有限的计算机来说，稠密索引的使用可能就需要进行考量了

<br/>

_*分块索引*_

图书馆的图书分类摆放是一门非常完整的科学体系，而它最重要的一个特点就是<font color = "red">分块摆放</font>，即同属一个类别(属性)的书都会放在一个区间之内，以让读者去查找自己想要的书籍的时候，能够在茫茫书海中快速的找到自己想要的书籍

稠密索引因为索引项与数据集的记录个数相同，所以空间代价很大，为了减少索引项的个数，我们可以对数据集进行分块，使其<font color = "red">分块有序</font>，然后再对每一块建立一个索引项，从而减少索引项的个数

> <font color = "red">分块有序</font>，是把数据集的记录分成了若干块，并且这些块需要满足两个条件 : 
> - 块内无序，即每一块内的记录不要求有序，当然，你如果能够让块内有序对查找来说更理想，不过这就要付出大量时间和空间的代价，因此通常我们不要求块内有序
> - 块间有序，例如，要求第二块所有记录的关键字均要大于第一块中所有记录的关键字，第三块的所有记录的关键字均要大于第二块的所有记录关键字……因为只有块间有序，才有可能在查找时带来效率
> 
> <font color = "red">对于分块有序的数据集，将每块对应一个索引项，这种索引方法叫做分块索引</font>，我们定义的分块索引的索引项结构分三个数据项∶
> - 最大关键码，它存储每一块中的最大关键字，这样的好处就是可以使得在它之后的下一块中的最小关键字也能比这一块最大的关键字要大
> - 存储了块中的记录个数，以便于循环时使用;
> - 用于指向块首数据元素的指针，便于开始对这一块中记录进行遍历。
> 
> ![2020-11-06-00-53-25](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-06-00-53-25.png)

在分块索引表中查找，就是分两步进行 : 
1. 在分块索引表中查找要查关键字所在的块，由于分块索引表是块间有序的，因此很容易利用折半、插值等算法得到结果
  
2. 根据块首指针找到相应的块，并在块中顺序查找关键码(因为块中可以是无序的，因此只能顺序查找)

最后，来分析一下分块索引的平均查找长度 : 

> 设 $n$ 个记录的数据集被平均分成 $m$ 块，每个块中有 $t$ 条记录，显然 $n=m×t$，或者说 $m=n/t$
> 
> 再假设 $L_{b}$ 为查找索引表的平均查找长度，因最好与最差的等概率原则，所以 $L_{b}$ 的平均长度为 $\displaystyle \frac{(m+1)}{2}$ ; $L_{w}$为块中查找记录的平均查找长度，同理可知它的平均查找长度为 $\displaystyle \frac{(t+1)}{2}$.
> 
> 这样分块索引查找的平均查找长度为 ∶ $ASL_{w} = L_{b} + L_{w} = \frac{(m+1)}{2} + \frac{(t+1)}{2} = \frac{1}{2}(m+t)+1 = \frac{1}{2}(\frac{n}{t} + t) + 1$.
> 
> 注意上面这个式子的推导是为了让整个分块索引查找长度依赖 $n$ 和 $t$ 两个变量，从这里了我们也就得到，平均长度不仅仅取决于数据集的总记录数 $n$，还和每一个块的记录个数 $t$ 相关，最佳的情况就是分的块数 $m$ 与块中的记录数 $t$ 相同，此时意味着
$n=m * t=t^2$，即 $ASL_{w} = \frac{1}{2}(\frac{n}{t} + t) + 1 = t + 1 = \sqrt{n} + 1$
> 
> 可见，分块索引的效率比之顺序查找的 $O(n)$ 是高了不少，不过显然它与折半查找的 $O(logn)$ 相比还有不小的差距，因此在确定所在块的过程中，<font color = "red">由于块间有序，所以可以应用折半、插值等手段来提高效率</font>

<br/>

_*倒排索引*_

在搜索引擎中，通常为了保证查询的效率，需要拥有一份单词表(仅仅只是概括，并不是搜索引擎的真正实现)，其记录了输入的搜索词(<font color = "red">次关键码</font>)和文章编号(指向文章的地址)(<font color = "red">记录号表</font>)之间的映射，那么每当我们输入想要搜索的关键字的时候，搜索引擎就能快速的找到其所匹配的文章，并展现出来

![2020-11-06-01-20-35](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-06-01-20-35.png)

而在这里，上面所简述的单词表其实就是<font color = "red">索引表</font>，其<font color = "red">索引项</font>的通用结构是 : 

- 次关键码，例如上面的 "搜索词"
- 记录号表，例如上面的 "文章编号"

> <font color = "red">其中记录号表存储具有相同次关键字的所有记录的记录号(可以是指向记录的指针或者是该记录的主关键字)，这样的索引方法就是倒排索引$(inverted \:\: index)$</font>
> 
> 倒排索引源于实际应用中需要根据属性(或字段、次关键码)的值来查找记录，这种索引表中的每一项都包括一个属性值和具有该属性值的各记录的地址，由于不是由记录来确定属性值，而是由属性值来确定记录的位置，因而称为倒排索引

倒排索引的优点显然就是查找记录非常快，基本等于生成索引表后，查找时都不用去读取记录，就可以得到结果 ; 但它的<font color = "red">缺点是这个记录号不定长</font>，比如上图有7个单词的文章编号只有一个，而 "book" , "friend" , "good" 有两个文章编号，若是对多篇文章所有单词建立倒排索引，那每个单词都将对应相当多的文章编号，<font color = "red">维护比较困难，插入和删除操作都需要作相应的处理</font>


<br/>

#### 二叉排序树
<span id = "二叉排序树"></span>

对于<font color = "red">顺序存储结构</font>来说，其插入和删除的效率是可以接受的，但这样的表由于无序造成查找的效率很低 ; 那么对于<font color = "red">有序存储结构来说</font>，其虽然可以通过算法来解决查找效率低的问题，但是正因为它的有序，而导致了对其在插入或删除的时候就需要耗费大量的工作来完成

考虑能够完成以上操作的结果实为<font color = "red">动态查找表</font>，那么现在的问题就是，是否拥有一个数据结构能够实现高效的动态查找表呢？那就是<font color = "red">二叉排序树</font>

> 二叉排序树$(Binary \: Sort \: Tree)$，又称为<font color = "red">二叉查找树</font>，它可能是一颗空树，亦或者具有以下性质的二叉树
> - 若它的左子树不空，则左子树上所有结点的值均小于它的根结构的值;
> - 若它的右子树不空，则右子树上所有结点的值均大于它的根结点的值;
> - 它的左、右子树也分别为二叉排序树;

从二叉排序树的定义也可以知道 : 
- 它前提是二叉树
- 然后它采用了递归的定义方法
- 再者，它的结点间满足一定的次序关系，左子树结点一定比其双亲结点小，右子树结点一定比其双亲结点大

<font color = "red">当通过以上定义构造出一颗二叉排序树后，当我们对它进行中序遍历时，就可以得到一个完整的有序的序列</font>

!!! info NOTE
    构造一棵二叉排序树的目的，其实并不是为了排序，而是为了提高查找和插入删除关键字的速度 ; 不管怎么说，在一个有序数据集上的查找，速度总是要快于无序的数据集的，而二叉排序树这种非线性的结构，也有利于插入和删除的实现



<br/>

_*二叉排序树的结构*_

为了避免树结构的复杂性，以下仅定义了一些最基本的树上节点的信息

```c
typedef struct bi_node {
  int data;
  struct bi_node *left;
  struct bi_node *right;
} bi_node, bi_tree;
```

<br/>

_*二叉排序树的查找*_

正因为二叉排序树其存储的特性(左子树上所有结点的值均小于其根结构的值，反之亦然)，我们对二叉排序树进行查找的时候其效率就会快很多，因为我们是可以通过所给定的键(key)去筛选过滤掉一些不必进行查找的结点内容以节省查询的时间

```c
bi_node *search_BST(bi_tree *tree, int key) {
  if (tree == NULL) return tree;

  if(tree->data == key) {
    return tree;
  } else if(tree->data < key) { /* 右子树 */
    return search_BST(tree->right, key);
  } else if(tree->data > key) { /* 左子树 */
    return search_BST(tree->left, key);
  } else { 
    return NULL; /* 事实上，这一结果通常不存在，仅仅只是为了编译不会被警告(函数并未所有段返回) */
  }
}
```

<br/>

_*二叉排序树的插入*_

二叉排序树的插入无非就是需要找到一个适合当前操作key的结点进行插入，这里就可以引用查找的思想，一路向下搜索，直至无法再继续向下搜索的情境之下，那么该节点的位置就是适合当前所操作key的结点，简而言之，<font color = "red">二叉排序树的插入操作必定是发生在叶子节点身上</font>

```c
bool insert_BST(bi_tree *root, int key, bi_tree *prev) {
  if(!root) {
    if(!prev) { /* 函数初始调用的状态下就给定了一颗空树 */
      return false;
    } else {
      root = (bi_node *)malloc(sizeof(bi_node));
      memset(root, 0x0, sizeof(bi_node));
      root->data = key;

      if(prev->data < key) {
        prev->right = root;
      } else {
        prev->left = root;
      }

      return true;
    }
  }

  if(root->data == key) {        /* 防止相同结点的插入 */
    return false; 
  } else if (root->data < key) { /* 从右子树继续往下搜索合适的位置以插入key */
    return insert_BST(root->right, key, root);
  } else {                       /* 从左子树继续往下搜索合适的位置以插入key */
    return insert_BST(root->left, key, root);
  }
}

int main(void) {
  bi_tree *tree = init();

  int val = 42;
  insert_BST(tree, val, NULL));

  system("pause");
  return EXIT_SUCCESS;
} 
```

<br/>

_*二叉排序树的删除*_

删除二叉排序树的结点稍显麻烦，我们需要分情况来考虑

1. 删除的结点是叶子结点，那么我们只需要把叶子结点给删除就好

    ![2020-11-08-00-34-33](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-08-00-34-33.png)

2. 删除的结点仅拥有左子树或右子树的情况，那么我们除了要把目标节点删除外，还要把目标结点所派生的左子树或右子树衔接至删除结点的父节点上即可

    ![2020-11-08-00-38-04](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-08-00-38-04.png)

3. 对于同时拥有左子树和右子树的结点稍显麻烦，比较好的方案就是 : 根据当前树的中序遍历的结果，找到删除结点的直接前驱或后继，那么我们任选其一作为删除结点被删除后的替代结点，把它衔接至删除结点的位置，而衔接的结点所派生的子节点则衔接至衔接结点原先的父节点上即可，<font color = "red">这里要考虑如何查找中序遍历结果上的删除节点的直接前驱和后继的问题，对于这个问题就拿寻找直接前驱举例，直接前驱的寻找只需要找到当前结点左子树上一直往右走，直至没有路后，那么这个结点就属于前驱结点 ; 而对于后继结点反之亦然</font>

    ![2020-11-08-01-11-30](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-08-01-11-30.png)

以下则为二叉排序树删除的代码实现，这里使用到二级指针用于表示当前需要删除的结点，<font color = "red">目的是为了可以直接拿到其父节点上左/右子结点那块内存，以便删除的时候可以直接修改而不需要再返回上一层寻找其父节点，并且还要判断当前结点到底是属于父节点的左子结点还是右子节点</font>

```c
bool delete_BST(bi_node **cur_node, int key) {
  if((*cur_node)->data == key) {
    if((*cur_node)->left == NULL && (*cur_node)->right == NULL) { /* 叶结点 */
      free(*cur_node);
      *cur_node = NULL;
    } else if ((*cur_node)->left == NULL) { /* 仅有右子树的结点 */
      bi_node *tmp = (*cur_node)->right;
      free(*cur_node); /* 释放掉父节点的左/右子结点的内存 */
      *cur_node = tmp; /* 使父节点的左/右子节点内存区指向最开始结点 */
    } else if ((*cur_node)->right == NULL) { /* 仅有左子树的结点 */
      bi_node *tmp = (*cur_node)->left;
      free(*cur_node); /* 释放掉父节点的左/右子结点的内存 */
      *cur_node = tmp; /* 使父节点的左/右子节点内存区指向最开始结点 */
    } else { /* 拥有左右子树的结点 */
      /**
       * 寻找要被删除结点 **cur_node 的直接前驱，即 **cur_node 左子树的最右边的尽头结点
       * 反之，寻找直接后驱就是 **cur_node 右子树的最左边尽头的结点  
      */
      bi_node *par = *cur_node;
      bi_node *new_root = (*cur_node)->left;
      while (new_root->right) {
        par = new_root;
        new_root = new_root->right;
      }

      /**
       * 由于上一步找到了 **cur_node 的直接前驱 new_root，也就是说
       * new_root 是没有右孩子结点的，故可以直接把它的左孩子结点写到它
       * 原先的位置来
      */
      bi_node *child = new_root->left;
      par->right = child;

      /* 替换掉需要删除的结点 */
      new_root->left = (*cur_node)->left;
      new_root->right = (*cur_node)->right;

      free(*cur_node);

      *cur_node = new_root;
    }
    return true;
  } else if ((*cur_node)->data < key) { /* 从右子树继续往下搜索合适的位置以插入key */
    return delete_BST(&(*cur_node)->right, key);
  } else {                              /* 从左子树继续往下搜索合适的位置以插入key */
    return delete_BST(&(*cur_node)->left, key);
  }
}

int main(void) {
  bi_tree *tree = init();

  delete_BST(&tree, 73);

  return EXIT_SUCCESS;
}
```

<br/>

_*小结*_

二叉排序树是以链接的方式存储，保持了链接存储结构在执行插入或删除操作时不用移动元素的优点，只要找到合适的插入和删除位置后，仅需修改链接指针即可，<font color = "red">插入删除的时间性能比较好</font>

而对于二叉排序树的查找，走的就是从根结点到要查找的结点的路径，<font color = "red">其比较次数等于给定值的结点在二叉排序树的层数</font>，极端情况，最少为 1 次，即根结点就是要找的结点，最多也不会超过树的深度，也就是说，<font color = "red">二叉排序树的查找性能取决于二叉排序树的形状</font>

可问题就在于，<font color = "red">二叉排序树的形状是不确定的</font> 例如 
- $\{62，88，58，47，35，73，51，99，37，93\}$ 这样的数组，我们依照左小右大的原则可以构建出下左图这样形状的一个二叉排序树
- $\{35,37,47,51,58,62，73,88,93,99\}$ 如数组元素的次序是从小到大有序，继续依照左小右大的原则去构建，则二叉排序树就成了极端的右斜树，也可以理解为这颗树退化成了一副单链表，注意它依然是一棵二叉排序树，如下右图

此时，同样是查找结点99，左图只需要两次比较，而右图就需要 10 次比较才可以得到结果，二者差异很大

![2020-11-08-23-35-47](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-08-23-35-47.png)

也就是说，我们希望二叉排序树是比较平衡的，即其深度与完全二叉树相同，均为 $└log_{2^n}┘+1$，那么<font color = "red">查找的时间复杂也就为 $O(logn)$</font>，近似于折半查找(事实上，上左图也不够平衡，明显的左重右轻)，而不平衡的最坏情况就是像上右图的斜树，<font color = "red">查找时间复杂度为 $O(n)$</font>，这等同于顺序查找

<br/>

#### AVL树
<span id = "AVL树"></span>

为了解决[二叉排序树](#二叉排序树)所存在的平衡性的问题，有两位俄罗斯数学家 **G.M.Adelson-Velski** 和 **E.M.Landis** 在1962年共同发明一种解决二叉排序树平衡的算法，<font color = "red">这种特殊的树以二叉排序树的基准进行了扩展</font>，它称为<font color = "red">平衡二叉排序树</font>或<font color = "red">AVL树</font>

> 平衡二叉树 $(Self-Balancing \:\: Binary \:\: Search \:\: Tree$ 或 $Height-Balanced \:\: Binary \:\: Search \:\: Tree)$ ，它是一种二叉排序树，<font color = "red">其中每一个节点的左子树和右子树的高度差至多等于 $1$</font>
> 
> 平衡二叉树是一种高度平衡的二叉排序树，意思是说，要么它是一棵空树，要么它的左子树和右子树都是平衡二叉树，且左子树和右子树的深度之差的绝对值不超过 $1$
>   
> 我们将二叉树上结点的左子树深度减去右子树深度的值称为<font color = "red">平衡因子BF$(Balance Factor)$</font>，那么平衡二叉树上所有结点的平衡因子只可能是 $-1、0、1$，只要二叉树上有一个结点的平衡因子的绝对值大于 $1$，则该二叉树就是不平衡的
> 
> ![2020-11-17-01-50-57](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-17-01-50-57.png)

<br/>

_*最小不平衡树*_

距离插入结点最近的，且平衡因子的绝对值大于 $1$ 的结点为根的子树，我们称为<font color = "red">最小不平衡子树</font>，如下图，当新插入结点 $37$ 时，距离它最近的平衡因子绝对值超过 $1$ 的结点是 $58$(即它的左子树高度 $2$ 减去右子树高度 $0$ )，所以从 $58$ 开始以下的子树为最小不平衡子树

![2020-11-17-01-53-17](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-11-17-01-53-17.png)

<br/>

_*平衡二叉树的实现原理*_

平衡二叉树构建的基本思想就是在构建二叉排序树的过程中，每当插入一个结点时，先检查是否因插入而破坏了树的平衡性，若是，<font color = "red">则找出最小不平衡子树</font>，在保持二叉排序树特性的前提下，<font color = "red">调整最小不平衡子树中各结点之间的链接关系，进行相应的旋转，使之成为新的平衡子树</font>

在这里，<font color = "red">如何对这颗最小不平衡树进行旋转成为关键</font>，关于最小不平衡树的旋转情况可以统筹为以下四种情况

> - LL(left-left) / 右单旋转 / 顺时针旋转 : 当 **_新结点_** 在某个最小不平衡树上的 **_左子节点_** 的 **_左子树_** 上进行插入的
> 
>   - 将根节点的左子结点变为新的根节点
> 
>   - 旧的根节点变为变为新的根节点的右结点，在这一步中，如果原先该位置已经存在结点，则直接替换，而被替换掉的结点则归纳为旧根节点的左子节点
> 
> - RR(right-right) / 左单旋转 / 逆时针旋转 : 当 **_新结点_** 在某个最小不平衡树上的 **_右子节点_** 的 **_右子树_** 上进行插入的
> 
>   - 将根节点的右子结点变为新的根节点
> 
>   - 旧的根节点变为变为新的根节点的左结点，在这一步中，如果原先该位置已经存在结点，则直接替换，而被替换掉的结点则归纳为旧根节点的右子节点
> 
> - RL(right-left) / 先右旋后左旋 : 当 **_新结点_** 在某个最小不平衡树上的 **_右子节点_** 的 **_左子树_** 上进行插入的
> 
>   - 先对最小不平衡树根节点右子结点进行右旋
> 
>   - 将上一步旋转的结果的根节点拼接回最小不平衡树的根节点的右子节点处，并对整颗最小不平衡书进行左旋
> 
> 
> - LR(left-right) / 先左旋后右旋 : 当 **_新结点_** 在某个最小不平衡树上的 **_左子节点_** 的 **_右子树_** 上进行插入的
> 
>   - 先对最小不平衡树根节点左子结点进行左旋
> 
>   - 将上一步旋转的结果的根节点拼接回最小不平衡树的根节点的左子节点处，并对整颗最小不平衡书进行右旋

除了要解决旋转到底如何旋的问题外，<font color = "red">还有两个次关键的问题待解决</font>，它们分别是

> - 每次旋转完成后，要对整颗树的 bf 再做一次调整
> 
>   - 其实我们无需记太复杂的推演，我们只需要在自己在本子上自己画一个最小不平衡术，并根据旋转的结果去演练一遍，然后统计结果的 [bf] 即可，最关键的是要掌握旋转的规则，但这<font color = "red">只是解决最小不平衡树的 bf 修改的结果</font>，要知道，大多数情况下，最小不平衡树并不代表整颗 AVL 树本身，即其上级父级节点亦或者其它节点也需要调整 bf 值，关于这点，其实在代码中利用递归来进行了实现了
> 
> - 如何找到不平衡的最小不平衡树
> 
>   - 其实在下面的代码中已经利用递归来实现了，具体参数依赖于一个当前插入节点是否造成树长高的 <font color = "red">taller</font> 标识符来完成

<br/>

_*代码示例*_

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define L_H +1 /* 左高 */
#define E_H +0 /* 等高 */
#define R_H -1 /* 右高 */

/** 
 * @brief LL(left-left)调整，右单旋转，顺时针旋转
 * @param 
 *  - root 最小不平衡树
 * @note 
 *  - LL调整: 新结点在某个最小不平衡树上的左子节点的左子树上进行插入的
 * 
 *  - 将根节点的左子结点变为新的根节点
 *  - 旧的根节点变为变为新的根节点的右结点，在这一步中，如果原先该位置已经存在结点，则直接替换，而被替换掉的结点则归纳为旧根节点的左子节点
*/
void right_rotate(bi_node **root) {
  bi_node *left = (*root)->left;
  (*root)->left = NULL;
  if (left->right != NULL) (*root)->left = left->right;
  left->right = *root;
  *root = left;
}

/** 
 * @brief RR(right-right)调整，左单旋转，逆时针旋转
 * @param 
 *  - root 最小不平衡树
 * @note
 *  - RR调整: 新结点在某个最小不平衡树上的右子节点的右子树上进行插入的
 * 
 *  - 将根节点的右子结点变为新的根节点
 *  - 旧的根节点变为变为新的根节点的左结点，在这一步中，如果原先该位置已经存在结点，则直接替换，而被替换掉的结点则归纳为旧根节点的右子节点
*/
void left_rotate(bi_node **root) {
  bi_node *right = (*root)->right;
  (*root)->right = NULL;
  if (right->left != NULL) (*root)->right = right->left;
  right->left = *root;
  *root = right;
}

/** 
 * @brief 左平衡旋转处理，该函数的主要目的是集成修改最小不平衡树中结点的 bf 的功能
 * @param
 *  - root 最小不平衡树
*/
void left_balance(bi_tree **root) {
  bi_tree *l_r_tree;
  bi_tree *l_tree;

  l_tree = (*root)->left;
  switch (l_tree->bf) {
    /**
     * 判断最小不平衡树 [root] 的左子树的平衡因子来决定到底是需要 LL(右)旋 还是 LR(先左后右)旋.
     * 
     * 由于该函数是左平衡旋转的处理，所以判断是以根节点(最小不平衡树)的左子树为出发点
     * 来进行，并且，判断的标准无非就是两个
     *  - 根节点的左子节点的左子树高: 即符合右单旋转的前提条件，即新结点在某个最小不平衡树上的左子节点的左子树上进行插入的
     *  - 根节点的左子结点的右子树高: 即符合LR旋转的前提条件，即新节点在某个最小不平衡树上的左子结点的右子树上进行插入的
     *  - 这里不存在根节点的左子节点左右树等高的情况，因为必定是因为某个新节点的插入，才导致了这颗树的深度的增加，而正因为树
     *    的深度的增加这个前提条件，才导致了是否需要对该出现不平衡的最小不平衡树进行旋转处理
     * 
     * 对于每种情况所改变的 [bf] 的规则，可以自己在本子上演练一遍，并统计结果的 [bf] 即可，最关键的是要掌握旋转的规则
    */

  case L_H:
    /* 右单旋转，所改变的结点仅为 [根节点] 和 [跟结点的左孩子结点]，故仅需要调整其二者的 bf 即可 */
    (*root)->bf = l_tree->bf = E_H;
    right_rotate(root);
    break;
  case R_H:
    l_r_tree = l_tree->right;

    /* LR旋转，需要考虑 [根节点的左孩子的右子树] 的问题，即这里又涵盖了一层 switch，这种旋转方式需要把 [根节点的左孩子的右子树] 和 [剩下的树] 切分开来考虑 */

    /* 对于 LR 旋转来说，该旋转会影响三种结点，分别是 [根节点], [跟结点的左孩子结点], [根节点的左孩子的右孩子] */

    switch (l_r_tree->bf) {
    case L_H:
      (*root)->bf = R_H;
      l_tree->bf = E_H;
      break;
    case R_H:
      (*root)->bf = E_H;
      l_tree->bf = L_H;
      break;
    case E_H:
      (*root)->bf = l_tree->bf = E_H;
      break;
    }
    l_r_tree->bf = E_H;
    left_rotate(&(*root)->left);
    right_rotate(root);
    break;
  }
}

/** 
 * @brief 右平衡旋转处理，该函数的主要目的是集成修改最小不平衡树中结点的 bf 的功能
 * @param
 *  - root 最小不平衡树
*/
void right_balance(bi_tree **root) {
  bi_tree *r_l_tree;
  bi_tree *r_tree;

  r_tree = (*root)->right;
  switch (r_tree->bf) {
    /**
     * 判断最小不平衡树 [root] 的右子树的平衡因子来决定到底是需要 RR(左)旋 还是 RL(先右后左)旋.
     * 
     * 由于该函数是右平衡旋转的处理，所以判断是以根节点(最小不平衡树)的右子树为出发点
     * 来进行，并且，判断的标准无非就是两个
     *  - 根节点的右子节点的左子树高: 即符合左单旋转的前提条件，即新结点在某个最小不平衡树上的右子节点的右子树上进行插入的
     *  - 根节点的右子结点的右子树高: 即符合RL旋转的前提条件，即新节点在某个最小不平衡树上的右子结点的左子树上进行插入的
     *  - 这里不存在根节点的左子节点左右树等高的情况，因为必定是因为某个新节点的插入，才导致了这颗树的深度的增加，而正因为树
     *    的深度的增加这个前提条件，才导致了是否需要对该出现不平衡的最小不平衡树进行旋转处理
     * 
     * 对于每种情况所改变的 [bf] 的规则，可以自己在本子上演练一遍，并统计结果的 [bf] 即可，最关键的是要掌握旋转的规则
    */

  case R_H:
    /* 左单旋转，所改变的结点仅为 [根节点] 和 [跟结点的右孩子结点]，故仅需要调整其二者的 bf 即可 */
    (*root)->bf = r_tree->bf = E_H;
    left_rotate(root);
    break;
  case L_H:
    r_l_tree = r_tree->left;

    /* RL旋转，需要考虑 [根节点的右孩子的左子树] 的问题，即这里又涵盖了一层 switch，这种旋转方式需要把 [根节点的右孩子的左子树] 和 [剩下的树] 切分开来考虑 */

    /* 对于 RL 旋转来说，该旋转会影响三种结点，分别是 [根节点], [跟结点的右孩子结点], [根节点的右孩子的左孩子] */

    switch (r_l_tree->bf) {
    case L_H:
      (*root)->bf = E_H;
      r_tree->bf = L_H;
      break;
    case R_H:
      (*root)->bf = L_H;
      r_tree->bf = E_H;
      break;
    case E_H:
      (*root)->bf = r_tree->bf = E_H;
      break;
    }
    r_l_tree->bf = E_H;
    right_rotate(&(*root)->right);
    left_rotate(root);
    break;
  }
}

/** 
 * @brief 平衡二叉树结点插入
 * @param 
 *  -   tree 平衡二叉树
 *  -    val 值
 *  - taller 指示当前所插入的结点是否令树的高度增加
*/
bool insert_AVL(bi_tree **tree, int val, bool *taller) {
  if (!*tree) {
    *tree = (bi_tree *)malloc(sizeof(bi_node));
    (*tree)->data = val;
    (*tree)->left = (*tree)->right = NULL;
    (*tree)->bf = E_H;
    *taller = true;
  } else {
    if (val == (*tree)->data) {
      *taller = false;
      return false;
    }

    if (val < (*tree)->data) {
      if (!insert_AVL(&(*tree)->left, val, taller))
        return false;

      if (*taller) {
        switch ((*tree)->bf) {
        case L_H:
          left_balance(tree);
          *taller = false;
          break;
        case E_H:
          (*tree)->bf = L_H;
          *taller = true;
          break;
        case R_H:
          (*tree)->bf = E_H;
          *taller = false;
          break;
        }
      }
    } else {
      if (!insert_AVL(&(*tree)->right, val, taller))
        return false;

      if (*taller) {
        switch((*tree)->bf) {
          case L_H: /* 原先该节点的 bf 是 1，并且此次插入操作是针对该节点的右子结点进行插入的，那不就意味着此次插入并没有改变父节点的 bf 嘛 */
                    /* 并且，既然原先在这个高度下就没有达成最小生成树的条件，那么高度不变的话，上层递归的函数就不需要再进行判断了 */
            (*tree)->bf = E_H;
            *taller = false;
            break;
          case E_H: /* 原先该节点的 bf 是 1，并且此次插入操作是针对该节点的右子结点进行插入的，那不就意味着此次插入使得树的高度增加了 1 嘛，所以父节点的 bf = -1 */
                    /* 那是否证明其根节点的 bf 的绝对值一定 >=0 && <= 1 呢？这是不确定的，故这里还要再上层递归中进行判断 */
            (*tree)->bf = R_H;
            *taller = true;
            break;
          case R_H: /* 既然原先该节点的 bf 是 -1 了，那么此次增加的结点造成了树的高度的增加，所以该结点就属于最小不平衡树的根节点 */
            right_balance(tree);
            *taller = false;
            break;
        }
      }
    }
  }

  return true;
}

int main(void) {
  int a[10] = { 3, 2, 1, 4, 5, 6, 7, 10, 9, 8 };
  bi_tree *tree = NULL;
  bool taller;
  for (size_t i = 0; i < sizeof(a) / sizeof(int); ++i) {
    insert_AVL(&tree, a[i], &taller);
  }

  return EXIT_SUCCESS;
}
```

<br/>

_*小结*_

平衡二叉树的思想并不难理解的，归根结底就是一句话 : <font color = "red">把不平衡消灭在最早时刻</font>(请好好体会)

如果我们需要查找的集合本身没有顺序，在频繁查找的同时也需要经常的插入和删除操作，显然我们需要构建一棵二叉排序树，但是不平衡的二叉排序树，查找效率是非常低的，因此我们需要在构建时，就让这棵二叉排序树是平衡二叉树，此时我们的查找时间复杂度就为 $O(log_n)$，而插入和删除也为 $O(log_n)$，这显然是比较理想的一种<font color = "red">动态查找表算法</font>


