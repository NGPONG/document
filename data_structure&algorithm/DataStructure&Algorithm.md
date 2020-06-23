# *Data Structure & Algorithm*

### 目录

- [开始](#前言)
- [Arrary](#arrary)
- [Linked List](#linkedlist)
- [Stack & Queue](#stackqueue)
- [Tree](#tree)
- [Binary Tree](#binary_tree)

<br/>

### 前言

<span id="前言"></span>

---

#### 什么是数据结构

数据是描述客观事物的符号，是计算机中可以操作的对象，是能被计算机识别，并输入给计算机处理的符号集合，而数据结构则为负责描述某一类数据具体的类型的一种结构，更具体来讲，<span style="color:red">数据结构是计算机存储、组织数据的方式，它是相互之间存在一种或多种特定关系的数据元素的集合</span>，通常情况下，精心选择的数据结构可以带来更高的运行或者存储效率

<br/>

#### 数据结构的分类

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

算法是特定问题求解步骤的描述，在计算机中表现为指令的有限序列，算法是独立存在的一种解决问题的方法和思想，
对于算法而言，语言并不重要，重要的是思想

<br/>

#### 算法的特性

- 输入输出：算法具有零个或多个输入、至少有一个或多个输出
- 有穷性：指算法在执行有限的步骤之后，自动结束而不会出现无限循环，并且每一个步骤在可接受的时间内完成
- 确定性：算法的每一步骤都有确定的含义，不会出现二义性
- 可行性：算法的每一步都必须是可行的，也就是说，每一步都能通过执行有限次数完成


<br/>

#### 算法和数据结构之间的联系

算法是为了解决实际问题而设计，而数据结构只是静态的描述了数据元素之间的关系，它是算法需要处理问题的一种载体，即算法和数据结构之间是属于一种 <font color="red">相辅相成</font> 的关系，<font color="red">一个高效的应用程序需要在数据结构的基础上设计和选择算法</font>


<br/>

### Arrary

<span id="Arrary"></span>

---

#### Basic Arrary

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
	if (arrary_new == NULL) {

		return -1;
	}

	printf("[Realloc]  address old = %p, address new = %p\n", *p_ptr_arrary, arrary_new);
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

	if (m_capacity <= 0) {

		return NULL;
	}

	capacity = m_capacity;

	struct dynamicArrary *arrary = malloc(sizeof(struct dynamicArrary));
	if (arrary == NULL) {

		return NULL;
	}
	memset(arrary, 0, sizeof(struct dynamicArrary));

	arrary->ptr_arrary = malloc(sizeof(void *) * m_capacity);
	if (arrary->ptr_arrary == NULL) {

		return NULL;
	}
	memset(arrary->ptr_arrary, 0, sizeof(void *) * m_capacity);

	printf("[Init dynamic arrary]  struct address = %p, arrary adderss = %p\n", arrary, arrary->ptr_arrary);

	return arrary;
}

/**
 * @description: Insert a new element into arrary
 * @param {type} 
 * @return: 
 */
int Insert(struct dynamicArrary *arrary, int index, void *value) {

	if (index < 0) {

		return -1;
	}
	if (CheckArraryAvailableLength(arrary)) {

		return -1;
	}
	if (index >= arrary->length) {

		index = arrary->length;
	}

	for (int i = arrary->length - 1; i >= index; i--) {

		memcpy(arrary->ptr_arrary + i + 1, arrary->ptr_arrary + i, sizeof(void *));
	}

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

	printf("[Add]  arrary address = %p, value address = %p, arrary length = %d\n", arrary->ptr_arrary + arrary->length, value, arrary->length + 1);
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

	printf("[Remove]  address = %p\n", *(arrary->ptr_arrary + index));
	free(*(arrary->ptr_arrary + index));

	for (int i = index + 1; i < arrary->length; i++) {

		printf("[Exchange]  address_exchanged = %p, address_exchanger = %p\n", arrary->ptr_arrary + i - 1, arrary->ptr_arrary + i);
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

	if (arrary == NULL || invoker == NULL) {

		return -1;
	}

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

	if (strcmp(str_CustomerInput, "yes\n")) {
	
		return -1;
	}

	for (int i = 0; i < arrary->length; i++) {

		printf("[free]  element address = %p\n", *(arrary->ptr_arrary + i));
		free(*(arrary->ptr_arrary + i));
	}

	printf("[free]  arrary address = %p\n", arrary->ptr_arrary);
	free(arrary->ptr_arrary);

	printf("[free]  struct address = %p\n", arrary);
	free(arrary);

	capacity = 0;
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

什么是链表的头节点和尾节点？

一张链表在初始化完毕后，我们通常会额外添加两个特殊的节点，这两个节点仅仅只是维护了一个指针域，指针域指向这张链表的头节点和尾节点在内存中的首地址，而这两种特殊的节点就为链表的 <font color = "red">头节点</font> 和 <font color = "red">尾节点</font>

为什么需要链表的头节点和尾节点？

先拿头节点来说，如果我们有一个地方能够存储一张链表的头节点的话，就能够对于这张链表起到一个标志位的作用，那么在后面，不管我们对这张链表进行如何操作，我们都可以恒定的知道这张链表起始节点的位置，假设我们对于一张链表遍历到某个节点的时候，这时候其实就已经改变了这张链表中具体某个节点位置的指向了，简而言之就是不在链表的起始位置上，这时候我们还需要对它进行重新遍历的话，那么这个头节点的标志位就起到了标志性的作用，对于一张合格的链表数据结构的定义来说，<font color="red">头节点的存在是必须的</font>

而尾节点的作用在于能够快速的获取到链表尾部节点的位置，以方便对链表进行节点扩展的操作，如果没有尾节点，那么我们想要为一个链表新增节点成员的话，那么我们就需要从头到尾遍历一次整张链表，直至拿到最后一个节点的信息我们才能进行新增节点的操作

<br/>

#### Singly Linked List

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
	}
	else {
	
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
	}
	else if (index > list->size - 1) {
	
		list->tail->next = node;
		list->tail = node;
	}
	else {
	
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

	if (list == NULL || list->head == NULL || list->tail == NULL) {

		return;
	}

	if (index < 0) {

		return;
	}
	else if (index == 0) {
	
		node_t *head = list->head;
		printf("delete and free %p\n", head);

		list->head = list->head->next;
		if (list->size == 1) {

			list->tail = list->head;
		}
		free(head);
	}
	else if (index >= list->size - 1) {
	
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
	}
	else {
	
		node_t *node_aft = list->head;
		for (size_t i = 0; i < index - 1; i++) {

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

<span id = "Tree"></span>

### Tree

---

上面所讨论都是 $1 : 1$ 的线性结构，可现实中，还有很多 $1 : n$ 成员排列方式的情况需要处理，针对这种情况我们采用 <font color = "red">树形结构</font> 去描绘成员 ( 数据 ) 之间的逻辑关系更为合适

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

树的节点包含一个数据元素及若干指向其子树的分支
 - 节点拥有的子树数称为 <font color = "red">节点的度</font> ( $Degree$ ) ;
 - 度为 0 的节点称为 <font color="red">叶节点</font> ( $Leaf$ ) 或终端节点 ;
 - 度不为 0 的节点 称为非终端节点或 <font color = "red">分支节点</font>，除根节点之外，分支节点也称为 <font color="red">内部节点</font> ;
 - 树 内各节点的度的最大值则为 <font color="red">树的度</font> ;
 
   ![2020-3-4-17-3-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-3-5.png)


<br/>

#### 节点间的关系

- 节点的子树的根称为该节点的 <font color="red">孩子</font> ( $Child$ ) ;
- 相应地，该节点称为孩子的 <font color="red">双亲 </font> ( $Parent$ ) ;
- 同一个 双亲 的 孩子 之间互称 <font color="red">兄弟</font> ( $Sibling$ ) ;
- 根到该节点所经分支上的所有节点都称为该节点的 <font color="red">祖先</font> ( $ancestor$ ) ;
- 以某节点为根的子树中的任一节点都称为节点的 <font color="red">子孙</font> ( $Offspring$ )

  ![2020-3-4-17-11-50.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-11-50.png)

<br/>

#### 树的其它相关概念

- <font color="red">节点的层次</font> ( $Level$ ) 从根开始定义起，根为第一层，根的孩子为第二层，若某结 点在第1层，则其子树的根就在第 *1 + 1* 层 ;

- 双亲在局一层的节点互为 <font color="red">堂兄弟</font> ( $Cousins$ ) ;

- 树中结点的最大层次称为 <font color = "red">树 的深度</font> ( $Depth$ ) 或 高度 ;

- 如果将树中结点的各子树看成从左至右是有次序的，不能互换的，则称该树为 <font color= "red">有 序树</font> ，否则称为 <font color="red">无序树</font> ; 
- <font color="red">森林</font> ( $Forest$ ) 是 $m$ ( $m > 0$ ) 棵互不相交的树的集合，对树中每个结点而言，其 子树的集合即为森林

  ![2020-3-4-17-19-45.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-19-45.png)


<br/>

#### 树的表示法

上面所展示的树形结构是从 <font color="red">人的逻辑概念上所看到的树形结构</font>，而对于开发人员而言，这种逻辑上的数据结构是难以操作和构造的，作为数据结构存储的两大分支无非就是 *顺序存储* 和 *链式存储*，而对于不进行过任何处理的树形结构来说显然是不满足这两种分支的存储模型的，所以我们必须把逻辑上的树形结构转换为一个真实的、具体的数据结构，简而言之，就是 <font color="red">把一颗复杂的树转换为一颗能够处理的树</font>，这种转换方式也称为 <font color = "red">树的表示法</font>

!!! INFO PS : 以下所展示的几种表示法以下图这种原始树形结构作为模型

	![2020-3-4-18-23-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-23-36.png)


<br/>

*双亲表示法*

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( $Data$ ) 外，还附设一个指示器去维护该节点的双亲节点在连续空间所处的下标 ( 如果没有则为 $-1$ ) ，我们称它为 <font color = "red">双亲域</font> ( $Parent$ )

这样的存储结构，我们可以根据节点的 $Parent$ 指针很容易找到它的双亲节点，所用的时间复杂度为 $O(1)$ ,直到 $Parent$ 为 -1 时，则表示找到了树节点的根。可如果我们要知道结点的孩子是什么，对不起，请遍历整个结构才行 ; 其实解决这一问题也很简单，我们为何不能维护一个 <font color = "red">长子域</font> ( $FirstChild$ ) 的指示器来记录该节点最左边孩子的下标，抑或是维护一个 <font color = "red">兄弟域</font> ( $RightSib$ ) 来记录当前节点右边兄弟的下标呢？答案当然是可以的，数据结构的具体模型视具体情况而进行定义

下图则为双亲表示法的数据结构模型：

![2020-3-4-18-55-28.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-55-28.png)

<br/>

*孩子表示法*

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( $Data$ ) 外，还维护了一个 <font color = "red">孩子域</font> ( $FirstChild$ ) 的指针指向了当前节点下的所有孩子节点，旗下的所有孩子节点以单链表存储结构的形式连接起来，<font color = "red">即 孩子域 指向了存储其下所有孩子节点的单链表在内存中的首地址</font>

为此我们除了需要设计数组中的元素的存储结构以外，还需设置单链表中的节点的数据结构，即附设一个指示器指示当前孩子节点在数组存储结构位置中的下标 ( $Child$ ) 和指向当前孩子的兄弟节点的 指针域 ( $Next$ )

这样的结构对于我们要査找某个节点的某个孩子，或者找某个节点的兄弟，只需要査找这个节点的孩子单链表即可 ; 对于遍而整棵树也是很方便的，对头结点的数组循环即可完成遍历

但是，这也存在着问题，我如何知道某个结点的双亲是谁呢？目前这个阶段是比较麻烦的，需要整棵树遍历才行，但是我们就不可以把双亲表示法和孩子表示法综合一下吗？当然是可以，当综合了之后，我个人认为这种表示法就不能称为 孩子表示法 了，应该称之为 <font color = "red">孩子双亲表示法</font>

下图则为孩子双亲表示法的数据结构模型：

![2020-3-4-19-32-51.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-19-32-51.png)

<br/>

*孩子兄弟表示法*

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

*二叉树的特点：*
- 每个结点 <font color = "red">最多有两棵子树</font>，所以二叉树中不存在度大于 $2$ 的结点，注意，不是只有两棵子树，而是最多有，<font color = "red">没有子树</font> 或者 <font color = "red">有一棵子树</font> 都是可以的

- <font color = "red">左子树和右子树是有顺序的</font>，次序不能任意颠倒，即二叉树是一颗 <font color = "red">有序树</font>

- 即使树中某结点只有一棵子树，也要区分它是 <font color = "red">左子树</font> 还是 <font color = "red">右子树</font>

*二叉树具有的五种基本形态：*

- 空二叉树
- 只有一个根节点
- 根节点只有左子树
- 根节点只有右子树
- 根节点既有左子树又有右子树

<br/>

#### 特殊的二叉树

上面所提到的是二叉树的基本形态，但是对于二叉树来说，它是多变的，即可能在某种形态之下有着它一定的研究价值，我们把这些具有特殊形态的树都统称为 特殊的二叉树

1. 斜树

	!!! INFO Description
		所有的结点都只有左子树的二叉树树叫 <font color = "red">左斜树</font>，所有结点都是只有右子树的二叉树叫 <font color = "red">右斜树</font>，这两者统称为斜树

	斜树有很明显的特点，就是 <font color = "red">每一层都只有一个结点</font>，<font color = "red">结点的个数与二叉树的深度相同</font>

	有人会想，斜树的表现形式不是和线性表是一样的吗？对的，其实线性表结构就可以理解为是树的一种极其特殊的表现形式

	![2020-3-8-13-17-55.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-13-17-55.png)

2. 满二叉树
	
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

- 在二叉树的第 $i$ 层上至多有 $2^{i-1}$ 个结点 ( $i\geq$ 1)

- 深度为 $k$ 的二叉树至多有 $2^k-1$ 个结点 ( $k\geq1$ )

- 
  二叉树 $T$ ; 结点总数 $n$ ; 分支线总数 $k$ ; 终端结点数 $n_0$ ; 度为 1 的结点数 $n_1$ ; 度为 2 的结点数 $n_2$ ; 
  - $n_0=n_2+1$
  - $n=n_0+n_1+n_2$
  - $k=n-1=n_1+2n_2$

- 具有 $n$ 个结点的 <font color = "red">完全二叉树</font> 的深度为 $⌊\log(2^n)⌋+1$ ( $⌊x⌋$ 表示为大于 $x$ 的最大整数 )

- 如果对一颗具有 $n$ 个结点的完全二叉树 ( 其深度为 $⌊\log(2^n)⌋+1$ ) 的结点按层序编号 ( 从第 $1$ 层到第 $⌊\log(2^n)⌋+1$ 层，每层从左到右 )，对任意节点 $i$ ( $1 \leq i \leq n$ ) 有：
	- 如果 $i=1$，则结点 $i$ 为二叉树的根，无双亲 ; 如果 $i>1$，则其双亲是节点 $⌊i/2⌋$
	- 如果 $2i>n$，则结点 $i$ 无左孩子 ( 结点 $i$ 为叶子节点 ) ； 否则其左孩子是结点 $2i$
	- 如果 $2i+1>n$，则结点 $i$ 无右孩子 ; 否则其右孩子是结点 $2i+1$
  
	![2020-3-8-23-12-26.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-12-26.png)


<br/>

#### 二叉树的存储结构

*二叉树顺序存储结构*

<font color = "red">顺序存储</font> 对树这种 $1:n$ 的关系结构实现起来是比较困难的，但是二叉树是一种特殊的树，由于它的特殊性，使得用顺序存储结构也可以实现

二叉树的顺序存储结构就是 <font color = "red">用一维数组存储二叉树中的结点</font>，<font color = "red">并且结点的存储位置 ( 数组的下标 ) 要能体现结点之间的逻辑关系</font>，比如双亲与孩子的关系，左右兄弟的关系等，如下图所示，一颗 <font color = "red">完全二叉树</font> 在使用顺序存储后的结构模型

![2020-3-8-23-42-35.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-42-35.png)

需要注意的是，上图特指出使用的是 <font color = "red">完全二叉树</font> 进行顺序存储结构进行存储，由于 完全二叉树 的特性，我们对 完全二叉树 使用顺序存储是很容易能够拿到各结点之间的逻辑关系的，而对于一般的二叉树而言，尽管层序编号并不能够反映这颗二叉树中各结点之间的直接逻辑关系，但是我们同样也可以将其按 完全二叉树 进行编号，只不过把不存在的节点需要 <font color ="red">特别批注出来</font> 而已

针对于刚刚所说到的 <font color = "red">一般的二叉树使用顺序存储结构</font>，我们不妨考利一种特殊情况，一颗深度为 $k$ 的右斜树只有 $k$ 个结点，但是按照刚刚所说的按照 完全二叉树 进行编号的方式却需要分配出 $2^k-1$个存储单元空间，这显然是对于存储空间是一种极度浪费的情况，所以，<font color = "red">顺序存储结构一般只用于完全二叉树</font>

![2020-3-8-23-53-59.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-23-53-59.png)

<br/>

*二叉树链表*

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

二叉树的遍历次序不同于 <font color = "red">线性结构</font>，最多也就是从头至尾、循环、双向等简单的遍历方式，树的结点之间不存在 <font color = "red">唯一的前驱和后继</font> 关系，在访问一个结点后，下一个被访问的结点面临着不同的选择，由于选择方式的不同，遍历的次序就完全不同了

!!! INFO Description

	二叉树的遍历 ( $Traversing binary-tree$ ) 是指从根结点出发，按照某种 <font color = "red">次序</font> 依次访同二叉树中所有结点，使得每个结点都被访问一次且仅被访问一次

<br/>

*前序遍历*

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

*中序遍历*

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

*后序遍历*

规则是若树为空，则空操作返回，否则从左到右先叶子后结点的方式遍历访问左右子树，最后是访问根结点，如下图所示，其遍历顺序则为：$GHDB1EFCA$

```c
int Postorder_Traverse_BinaryTree(Binary_Tree *tree) {

	if (tree == 0x0)
		return -1;

	Postorder_Traverse_BinaryTree(tree->left);
	Postorder_Traverse_BinaryTree(tree->right);
	printf("[*] Traverse Binary-tree: Value = %c, Address = %p\n", tree->data, tree);

	return 0;
}
```

![2020-3-9-0-48-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-0-48-36.png)

<br/>

*遍历结果的推理*

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

二叉树的建立同样需要采用遍历的方式去创建，对于存在空子数的结点我们在遍历的过程中需要通过一个 <font color = "red">额外的标识符</font> 来确定，在下面的代码中，提供了两种 <font color = "red">不同的遍历次序</font> 来构造一棵二叉树

<br/>

*后序遍历*

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
	}
	else {

		Binary_Tree *tree = (Binary_Tree *)malloc(sizeof(struct binary_node));
		if (tree == 0x0) {

			return -1;
		}
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

*前序遍历*

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
	}
	else {

		*tree = (Binary_Tree *)malloc(sizeof(Binary_Tree));
		if (*tree == 0x0) {

			return -1;
		}
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

#### 线索二叉树

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

在前面提到过，我们对于树形结构的探究通常都采用 <font color  ="red">孩子兄弟表示法</font> 来将一颗复杂的树转化为二叉树进行存储，而借助二叉链表的特性，树和二叉树之间是可以进行相互转换的

从物理结构来看，树和二叉树是相同的，只是采用了不同的解释方式，因此，只要我们设定一定的规则，用二叉树来表示树，甚至表示森林都是可以的，森林与二叉树也可以互相进行转换

<br/>

*树转换为二叉树*

1. 加线：把所有兄弟节点之间加一条连接线
2. 去线：对树中毎个结点，只保留它与第一个孩子结点的连线，删除它与其他孩子结点之间的连线
3. 层次调整：以树的根结点为轴心，将整棵树顺时针旋转一定的角度，使之结构层次分明，注意第一个孩子是二叉树结点的左孩子，兄弟转换过来的孩子是结点的右孩子

	![2020-3-9-17-32-35.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-32-35.png)

<br/>

*森林转换为二叉树*

1. 把每颗树转换为二叉树
2. 第一颗二叉树不动，从第二棵二叉树开始，依次把后一棵二叉树的根结点作为前一棵二叉树的根结点的右孩子，用线连接起来，当所有的二又树连接起来后就得到了由森林转换来的二叉树

	![2020-3-9-17-37-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-37-5.png)

<br/>

*二叉树转换为树*

1. 加线：若某结点 $K$ 的左孩子结点存在，则将这个左孩子的 $n$ 个右孩子结点都作结点 $K$ 的孩子，并用线连接起来 
2. 去线：删除原二叉树中所有结点与其右孩子结点的连接线
3. 层次调整：使之结构层次分明

	![2020-3-9-17-49-18.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-49-18.png)

<br/>

*二叉树转换为森林*

!!! INFO
	判断一棵二叉树能够转换成一棵树还是森林，标准很简单，那就是只要看这棵二叉树树的根结点有没有右孩子，有就是森林，没有就是一棵树

1. 分离：从根结点开始的 $n$ 个右孩子若存在，则连线刪除，得到多颗分离后的二叉树
2. 拼接：将每棵分离后的二叉树转换为树即可

	![2020-3-9-17-55-48.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-9-17-55-48.png)


<br/>

#### 赫夫曼树

*赫夫曼树的定义*

- 从树中一个结点到另一个结点之间的分支构成两个结点之间的 <font color = "red">路径</font>
- 路径上的分支数目称作 <font color = "red">路径长度</font>
- <font color = "red">树的路径长度</font> 就是从树根到毎一结点的路径长度之和
- 如果考虑到带 <font color = "red">权</font> 的结点
 	- 结点的 <font color = "red">带权的路径长度</font> 为从该结点到树根之间的路径长度与结点上权的乘积
 	- <font color = "red">树的带权路径长度</font> 为树中所有叶子结点的带权路径长度之和
- 假设有 $n$ 个权值 $\{w_1,w_2,\ldots,w_n\}$ 构造一棵有 $n$ 个叶子结点的二叉树，每个叶子结点带权 $w_k$，每个叶子的路径长度为 $lk$，我们通常记作，则其中带权路径长度 $WPL$ 最小的二叉树称作赫夫曼树

<br/>

*如何构造一颗赫夫曼树*

1. 根据给定的 $n$ 个权值 $\{w_1,w_2,\ldots,w_n\}$ 构成 $n$ 棵二叉树的集合 $F=\{T_1,T_2,\ldots,T_n\}$，其中每棵二叉树 $T_i$ 中只有一个带权为 $w_i$，其左右树均为空
2. 在 $F$ 中选取两棵根结点的权值最小的树作为左右子树构造一棵新的二叉树树 $K$，且置 $K$ 的权值为其左右子树上根结点的权值之和
3. 在 $F$ 中删除这两棵树，同时将新得到的二叉树 $K$ 加入 $F$ 中
4. 重复 $2$ 和 $3$ 步骤，直到 $F$ 只含一颗树为止，这棵树便为 <font color = "red">赫夫曼树</font>