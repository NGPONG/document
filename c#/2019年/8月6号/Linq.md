# *Language Intergated Query*

<br/>

### 前言

---

#### 1. 关于Linq的简单介绍

> Linq 是一系列直接将查询功能集成到`C#`语言的技术统称，换句话说它是一种统一的数据查询接口，其系列技术提供了一种跨各种数据源和数据格式 (实现自<span style="color:red">IEnumerable</span>或<span style="color:red">IEnumerable&lt;T&gt;</span>接口) 的查询体验，借助 Linq，查询成为了最高级的语言构造，就像类、方法和事件一样，可以使用语言关键字和熟悉的运算符针对强类型化对象集合编写查询

> `Linq查询表达式`是统一的数据查询接口，通过提供了一种跨各种数据源和数据格式使用数据的一致模型，不管是查询 `Linq to Object` 也好还是自定义数据源 `Linq to Provider`，Linq对于数据检索的表达式是不变的，需要变更的只是数据查询提供程序


#### 2. Linq查询操作的流程

- 获取数据源
- 创建查询
- 执行查询

<br/>

### 让我们进一步的理解Linq

---

#### 1. 关于Linq的核心设计模式

- `链式设计模式` 提供了对链式查询方法的支持
- `lambda表达式` 提供了对查询时候所需要的自定义查询逻辑的支持
- `表达式树` 提供了对自定义数据源所需要的自定义查询逻辑的支持
- `DLR动态语言运行时` 提供了编译器对我们所编写的Linq查询表达式的编译和支持

#### 2. Linq查询操作所在的核心命名空间
Linq 是在 `.NET Framework 3.5` 中所引入的，其核心程序集名为：`System.Core.dll`，那么在这之中又包含了两个命名空间是直接关系到 Linq 查询的，其名为：
- `System.Linq.Enumerable`：该命名空间负责为 `IEnumerable<T>` 提供了针对于 `Linq to Object` 数据源查询所需要的链式扩展方法
- `System.Linq.Queryable`：该命名空间负责为 `IQueryable<T>` 提供了针对于 `Linq to Provider` 数据源查询所需要的链式扩展方法
- `System.Linq.Expressions`：为 `System.Linq.Queryable` 所扩展的查询方法对应录入查询逻辑的参数进行封装

> <span style="color:red;">这里需要注意一点</span>，可能通过上面的阅读可能会产生疑惑，Linq 所提供的核心命名空间是只针对泛型版本的数据源(`IEnumerable<T>` / `IQueryable<T>`)扩展了链式查询方法进行查询吗？其实不然，不管是 `Linq to Object` 也好还是 `Linq to Provider` 也罢，其所对应的核心命名空间(`System.Linq.Enumerable` / `System.Linq.Queryable`)都针对于普通版本的数据源(`IEnumerable` / `IQueryable`)扩展了 `cast<T>()` 和 `OfType<T>()` 这两种函数使其转换成对应的泛型版本的实例，以让开发人员去使用 `System.Linq.Enumerable` / `System.Linq.Queryable` 为其泛型版本的数据源所扩展的链式查询方法来进行Linq查询操作。通过这点其实也可以总结出为什么我们使用Linq查询表达式针对 `IEnumerable` / `IQueryable` 类型进行查询操作的时候，`from` 关键字后面总是需要跟着一个需要转换的类型，其实这一步操作在编译器对该段表达式编译后的结果只是一个调用 `cast<T>()` 和 `OfType<T>()` 函数的过程。

> <span style="color:red;">其次需要注意的是</span>，我们可以分别进入 `System.Linq.Enumerable` / `System.Linq.Queryable` 的内部进行查看，细看可以发现，这两个命名空间所扩展的链式查询方法的功能是一模一样的，区别在于 `System.Linq.Queryable` 所扩展的链式查询方法还使用了 `System.Linq.Expressions` 为查询方法所提供的需要录入查询逻辑的参数进行了封装

#### 3. 延迟执行

使用Linq查询后的结果我们可以称它为一个查询变量，但是更准确的说法是 `一个可迭代的对象` ，在 [迭代模式篇章](/c%23/2019年/8月6号/迭代器模式.md) 说到过，一个可迭代对象并就是所指向的数据序列的本身，我们需要对它进行迭代工作后才能完整的获取数据源，这其实也就认证了 `延迟执行` 这句话

#### 4. 如何具体的实现Linq查询的功能

其实如果我们想为一个数据源执行 Linq 查询操作的话其实很简单，其分可分为两种不同的书写模式来完成这一操作

- 链式查询方法：


#### 5. Linq 查询的分类

Linq查询的数据源主要是分为了两种类型
- 一种是我们经常所操作的集合，如果我们使用他们来充当我们 Linq 查询的数据源的话，更准确地说这种做法名为：`Linq to Object`。