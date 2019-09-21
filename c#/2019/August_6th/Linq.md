# *Language Intergated Query*

<br/>

### 前言

---

#### 1. 关于Linq的简单介绍

&nbsp;&nbsp;&nbsp;1.1 Linq 是一系列直接将查询功能集成到`C#`语言的技术统称，换句话说它是一种统一的数据查询接口，其系列技术提供了一种跨各种数据源和数据格式 (实现自<span style="color:red">IEnumerable</span>或<span style="color:red">IEnumerable&lt;T&gt;</span>接口) 的查询体验，借助 Linq，查询成为了最高级的语言构造，就像类、方法和事件一样，可以使用语言关键字和熟悉的运算符针对强类型化对象集合编写查询

&nbsp;&nbsp;&nbsp;1.2 `Linq查询表达式`是统一的数据查询接口，通过提供了一种跨各种数据源和数据格式使用数据的一致模型，不管是查询`Linq to Object`也好还是 `Linq to Sql` 、 `Linq to Entities` 、 `LINQ to XML` 亦或是一些自定义的数据源，Linq对于数据检索的表达式是不变的，需要变更的只是数据查询提供程序

&nbsp;&nbsp;&nbsp;1.4 Linq 是在 `.NET Framework 3.5` 中所引入的，其核心程序集名为：`System.Core.dll`，那么在这之中又包含了两个命名空间是直接关系到 Linq 查询的，其名为：
- System.Linq.Enumerable：该命名空间负责为 `IEnumerable<T>` 提供了针对于 `Linq to Object` 查询
- System.Linq.Queryable：
- System.Linq.Expressions：

#### 2. Linq查询操作的流程

- 获取数据源
- 创建查询
- 执行查询

#### 3. 延迟执行

使用Linq查询后的结果我们可以称它为一个查询变量，但是更准确的说法是 `一个可迭代的对象` ，在[迭代模式篇章](/c#/2019/August 6th/Lterator Pattern.md)说到过，一个可迭代对象并就是所指向的数据序列的本身，我们需要对它进行迭代工作后才能完整的获取数据源，这其实也就认证了 `延迟执行` 这句话

[迭代模式篇章](/document/c#/2019/August_6th/Lterator_Pattern.md)
<br/>

### 深入理解Linq

---

#### 1. 关于Linq的核心设计模式

- `链式设计模式` 提供了对链式查询方法的支持
- `lambda表达式` 提供了对查询时候所需要的自定义查询逻辑的支持
- `表达式树` 提供了对自定义数据源所需要的自定义查询逻辑的支持
- `DLR动态语言运行时` 提供了编译器对我们所编写的Linq查询表达式的编译和支持

#### 2. Linq 查询的分类

Linq查询的数据源主要是分为了两种类型
- 一种是我们经常所操作的集合，如果我们使用他们来充当我们 Linq 查询的数据源的话，更准确地说这种做法名为：`Linq to Object`。