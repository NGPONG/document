# *Data Structure & Algorithm*

<br/>

### 目录

- [开始](#前言)
- [Arrary](#Arrary)
- [Linked List](#LinkedList)
- [Stack & Queue](#Stack&Queue)
- [Tree](#Tree)

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

    线性结构中的数据元素之间是 [[1 : 1]] 的关系，即除开首元素和尾元素外，每个元素之间都能够找到其唯一的前驱和后继

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

    树形结构中的数据元素之间是 [[1 : n]] 的关系，即除开根元素和叶子元素外，每个元素之间都能够找到唯一的前驱和多个后继

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

    树形结构中的数据元素之间是 [[n : n]] 的关系，即每个元素之间都可能会找到多个全驱和后继

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

动态数组属于对 [[Basic Arrary]] 的一项扩展内容，它在 <font color = "red">保持 [[Basic Arrary]] 本质不变的情况下，还解决掉了 [[Basic Arrary]] 无法进行动态扩展的短板</font>

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

### Linked List

---

<span id="LinkedList"></span>

链表是一种常用的数据结构，它由一系列的 [[node]] 通过指针的方式相互连接起来，以此形成一条数据链，而每个节点中主要是维护着两种重要的信息，它们分别是：

- [[Data：数据域]] 

    存放着当前节点所存储的信息数据
    
- [[Pointer：指针域]]

    存放着其它节点在内存中的首地址，即每一个节点都能够指向 上一个 / 下一个 节点的信息

<br/>

#### 链表与数组

对于链表来说，其通常用于和 [[Arrary]] 做对比，由于链表是通过节点与节点之间的连接来完成一条数据链的构造，故该特性能够解决掉 Arrary 本身所存在的许多缺点，但是相对的，更是由于这一特性，链表在解决了 Arrary 本身所存在的缺陷的同时却无法保留掉原有数组本身的 [[优点]]，即 链表 在解决了 Arrary 本身所存在的缺陷的同时又把 Arrary 身上的优点转换为了 链表 所带有的缺陷

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

一张链表在初始化完毕后，我们通常会额外添加两个特殊的节点，这两个节点仅仅只是维护了一个指针域，指针域指向这张链表的头节点和尾节点在内存中的首地址，而这两种特殊的节点就为链表的 [[头节点]] 和 [[尾节点]]

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

### Stack & Queue

<span id = "Stack&Queue"></span>

---

把 [[Stack]] 和 [[Queue]] 放在一起的原因是因为两者的实现的是非常的相似

对于 Stack 来说其实现需要遵循 [[FILO]] 原则

对于 Queue 来说其实现需要遵循 [[FIFO]] 原则

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

在这篇 Blog 中，我并不打算对 [[Queue]] 的存储结构进行实现，因为二者实现起来相似度还是挺高，故本 Blog 中只是针对 Stack 进行了一个实现，下面代码中则为实现了一个 <font color = "red">顺序存储</font> 的字符栈

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

### Tree

---

<span id = "Tree"></span>

上面所讨论都是 [[1 : 1]] 的线性结构，可现实中，还有很多 [[1 : n]] 成员排列方式的情况需要处理，针对这种情况我们采用 [[树形结构]] 去描绘成员 ( 数据 ) 之间的逻辑关系更为合适

!!! INFO 树的定义
	树 ( *Tree* ) 是 [[n]] ( *n >= 0* ) 个节点的有限集
		
	1. *n = 0* 时称为 <font color = "red">空树</font>

	2. *n > 0* 是称为 <font color = "red">非空树</font>，在任意一颗 非空树 中：

	 	2.1 有且只有一个特定的称为根 ( *Root* ) 的节点；
		 
	 	2.2 当 n > 1 时，其余节点可分为 [[m]] ( *m > 0* ) 个 <font color = "red"> 互不相交</font> 的有限集 *T1、T2、……、Tm*，其中每一个集合本身又是一棵树 并且称为子树 ( *SubTree* )
	
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
 - 节点拥有的子树数称为 <font color = "red">节点的度</font> ( *Degree* ) ;
 - 度为 0 的节点称为 <font color="red">叶节点</font> ( *Leaf* ) 或终端节点 ;
 - 度不为 0 的节点 称为非终端节点或 <font color = "red">分支节点</font>，除根节点之外，分支节点也称为 <font color="red">内部节点</font> ;
 - 树 内各节点的度的最大值则为 <font color="red">树的度</font> ;
 
   ![2020-3-4-17-3-5.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-3-5.png)


<br/>

#### 节点间的关系

- 节点的子树的根称为该节点的 <font color="red">孩子</font> ( *Child* ) ;
- 相应地，该节点称为孩子的 <font color="red">双亲 </font> ( *Parent* ) ;
- 同一个 双亲 的 孩子 之间互称 <font color="red">兄弟</font> ( *Sibling* ) ;
- 根到该节点所经分支上的所有节点都称为该节点的 <font color="red">祖先</font> ( *ancestor* ) ;
- 以某节点为根的子树中的任一节点都称为节点的 <font color="red">子孙</font> ( *Offspring* )

  ![2020-3-4-17-11-50.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-11-50.png)

<br/>

#### 树的其它相关概念

- <font color="red">节点的层次</font> ( *Level* ) 从根开始定义起，根为第一层，根的孩子为第二层，若某结 点在第1层，则其子树的根就在第 *1 + 1* 层 ;

- 双亲在局一层的节点互为 <font color="red">堂兄弟</font> ( *Cousins* ) ;

- 树中结点的最大层次称为 <font color = "red">树 的深度</font> ( *Depth* ) 或 高度 ;

- 如果将树中结点的各子树看成从左至右是有次序的，不能互换的，则称该树为 <font color= "red">有 序树</font> ，否则称为 <font color="red">无序树</font> ; 
- <font color="red">森林</font> ( *Forest* ) 是 [[m]] ( *m > 0* ) 棵互不相交的树的集合，对树中每个结点而言，其 子树的集合即为森林

  ![2020-3-4-17-19-45.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-17-19-45.png)


<br/>

#### 树的表示法

上面所展示的树形结构是从 <font color="red">人的逻辑概念上所看到的树形结构</font>，而对于开发人员而言，这种逻辑上的数据结构是难以操作和构造的，作为数据结构存储的两大分支无非就是 **顺序存储** 和 **链式存储**，而对于不进行过任何处理的树形结构来说显然是不满足这两种分支的存储模型的，所以我们必须把逻辑上的树形结构转换为一个真实的、具体的数据结构，简而言之，就是 <font color="red">把一颗复杂的树转换为一颗能够处理的树</font>，这种转换方式也称为 <font color = "red">树的表示法</font>

!!! INFO PS : 以下所展示的几种表示法以下图这种原始树形结构作为模型

	![2020-3-4-18-23-36.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-23-36.png)


<br/>

*双亲表示法*

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( *Data* ) 外，还附设一个指示器去维护该节点的双亲节点在连续空间所处的下标 ( 如果没有则为 -1 ) ，我们称它为 <font color = "red">双亲域</font> ( *Parent* )

这样的存储结构，我们可以根据节点的 *Parent* 指针很容易找到它的双亲节点，所用的时间复杂度为 [[O(1)]] ,直到 *Parent* 为 -1 时，则表示找到了树节点的根。可如果我们要知道结点的孩子是什么，对不起，请遍历整个结构才行 ; 其实解决这一问题也很简单，我们为何不能维护一个 <font color = "red">长子域</font> ( *FirstChild* ) 的指示器来记录该节点最左边孩子的下标，抑或是维护一个 <font color = "red">兄弟域</font> ( *RightSib* ) 来记录当前节点右边兄弟的下标呢？答案当然是可以的，数据结构的具体模型视具体情况而进行定义

下图则为双亲表示法的数据结构模型：

![2020-3-4-18-55-28.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-18-55-28.png)

<br/>

*孩子表示法*

我们以一组连续的空间去存储一棵树中的所有节点，每个节点除了维护了一个该节点所指示的 <font color="red">数据域</font> ( *Data* ) 外，还维护了一个 <font color = "red">孩子域</font> ( *FirstChild* ) 的指针指向了当前节点下的所有孩子节点，旗下的所有孩子节点以单链表存储结构的形式连接起来，<font color = "red">即 孩子域 指向了存储其下所有孩子节点的单链表在内存中的首地址</font>

为此我们除了需要设计数组中的元素的存储结构以外，还需设置单链表中的节点的数据结构，即附设一个指示器指示当前孩子节点在数组存储结构位置中的下标 ( *Child* ) 和指向当前孩子的兄弟节点的 指针域 ( *Next* )

这样的结构对于我们要査找某个节点的某个孩子，或者找某个节点的兄弟，只需要査找这个节点的孩子单链表即可 ; 对于遍而整棵树也是很方便的，对头结点的数组循环即可完成遍历

但是，这也存在着问题，我如何知道某个结点的双亲是谁呢？目前这个阶段是比较麻烦的，需要整棵树遍历才行，但是我们就不可以把双亲表示法和孩子表示法综合一下吗？当然是可以，当综合了之后，我个人认为这种表示法就不能称为 孩子表示法 了，应该称之为 <font color = "red">孩子双亲表示法</font>

下图则为孩子双亲表示法的数据结构模型：

![2020-3-4-19-32-51.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-19-32-51.png)

<br/>

*孩子兄弟表示法*

孩子兄弟表示法是一种最为常用的一种表示法，它的好处在于能够把一颗复杂的树转换为一颗 <font color = "red">二叉树</font>，对于二叉树的定义我们在下面的节点会提到，现在先聊下孩子兄弟表示法的具体表示方式

我们对树形结构进行观察后发现，任意一棵树它的节点的第一个孩子如果存在就是唯一的，它的右兄弟如果存在也是唯一的，因此我们大可对该节点所代表的元素结构进行一个构想，首要也是必须的就是 <font color = "red">数据域</font> ( *Data* )，其次还维护了两个 <font color = "red">指针域</font> 分别指向了当前节点的第一个孩子节点在内存中的首地址 ( *LPointer / FirstChild* ) 和当前节点右边兄弟节点在内存中的首地址 ( *RPointer / RightSib* )

下图则为孩子兄弟表示法的数据结构模型：

![2020-3-4-19-49-30.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-4-19-49-30.png)


<br/>

### Binary Tree

---

<span id = "Binary_Tree"></span>

对于在某个阶段下都是两种结果的情形，比如：开和关、0和1、真和假、上和下、对与错，正面与反面等，都适合用树状结构来建模，而这种树是一种很特殊的树状结构，叫做二叉树

前面在孩子兄弟表示法中稍微提到了 <font color = "red">二叉树</font> 的概念，对于 二叉树 来说，由于其毎个结点至多只能有左孩子和右孩子，那么在这种情况下，二叉树所面对的变化就少很多了，因此很多相关性质和算法都是以 二叉树 作为模型基准所研究了出来的

!!! INFO 二叉树的定义

	二叉树 ( *Binary Tree* ) 是 [[n]] ( *n >= 0* ) 个节点的有限集合，该集合或者为空集 ( 称为 <font color = "red">空二叉树</font> )，或者由一个根结点和两裸互不相交的、分别称为根节点的 <font color = "red">左子树</font> 和 <font color = "red">右子树</font> 的二叉树组成

	*Binary-Tree*
	
	![2020-3-8-1-57-31.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-1-57-31.png)


<br/>

#### 二叉树的特点和基本形态

*二叉树的特点：*
- 每个结点 <font color = "red">最多有两棵子树</font>，所以二叉树中不存在度大于 [[2]] 的结点，注意，不是只有两棵子树，而是最多有，<font color = "red">没有子树</font> 或者 <font color = "red">有一棵子树</font> 都是可以的

- <font color = "red">左子树和右子树是有顺序的</font>，次序不能任意颠倒

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

	> 所有的结点都只有左子树的二叉树树叫 <font color = "red">左斜树</font>，所有结点都是只有右子树的二叉树叫 <font color = "red">右斜树</font>，这两者统称为斜树

	斜树有很明显的特点，就是 <font color = "red">每一层都只有一个结点</font>，<font color = "red">结点的个数与二叉树的深度相同</font>

	有人会想，斜树的表现形式不是和线性表是一样的吗？对的，其实线性表结构就可以理解为是树的一种极其特殊的表现形式

	![2020-3-8-13-17-55.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-13-17-55.png)

2. 满二叉树
	
	> 在一棵二又树中，如果所有分支结点都存在左子树和右子树，并且所有叶子都在同一层上，这样的二叉树树称为满二叉树

	单是每个结点都存在左右子树，不能算是满二叉树，因此，满二叉树的特点有：
	- <font color = "red">所有的叶子都在同一层上</font>，做到整棵树的平衡
	- <font color = "red">叶子只能出现在最下一层</font>，出现在其他层就不可能达成平衡
	- 非叶子结点的度一定是 [[2]]
	- 在同样深度的二叉树中，满二叉树的结点个数最多，叶子数最多

	 ![2020-3-8-14-0-6.png](https://raw.githubusercontent.com/NGPONG/Blog/master/img/2020-3-8-14-0-6.png)

3. 完全二叉树

	> 