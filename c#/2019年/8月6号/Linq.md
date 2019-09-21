# *Language Intergated Query*

<br/>

### 前言

---

#### 1. 关于Linq的简单介绍

> Linq 是一系列直接将查询功能集成到`C#`语言的技术统称，换句话说它是一种统一的数据查询接口，其系列技术提供了一种跨各种数据源和数据格式 (实现自<span style="color:red">IEnumerable</span>或<span style="color:red">IEnumerable&lt;T&gt;</span>接口) 的查询体验，借助 Linq，查询成为了最高级的语言构造，就像类、方法和事件一样，可以使用语言关键字和熟悉的运算符针对强类型化对象集合编写查询

> `Linq查询表达式`是统一的数据查询接口，通过提供了一种跨各种数据源和数据格式使用数据的一致模型，不管是查询 `Linq to Object` 也好还是自定义数据源 `Linq to Provider`，Linq对于数据检索的表达式是不变的，需要变更的只是数据查询提供程序

![2012072522494987.jpg](https://i.loli.net/2019/09/21/3KVXgcnrOZPh9lJ.jpg)

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

> <span style="color:red;">其次需要注意的是</span>，我们可以分别进入 `System.Linq.Enumerable` / `System.Linq.Queryable` 的内部进行查看，细看可以发现，这两个命名空间所扩展的链式查询方法的功能是一模一样的，区别在于 `System.Linq.Queryable` 所扩展的链式查询方法还使用了 `System.Linq.Expressions` 为查询方法所需要录入查询逻辑表达式进行了封装，<span style="color:red;">其目的就是为了把当前所录入的查询逻辑表达式的数据结构解析成表达式树，并在需要的时候来读取相关的逻辑结构进行查询操作</span>

#### 3. Linq 查询的分类

Linq查询的数据源主要是分为了两种类型
- `Linq to Object`：这种类型的查询操作是针对派生并实现自 `System.Linq.IEnumerable` 类型的数据源的，举个很常见的例子就是我们经常所操作的集合
- `Linq to Provider`：这种类型的查询操作是针对派生并实现自 `System.Linq.IQueryable` 类型的数据源的，通常来讲，我们更愿意称这种模式为自定义查询，诸如 `Linq to XML`、`Linq to JSON`、`Linq to SQL` 其实都是属于 `Linq to Provider` 的统称

#### 3. 如何具体的实现Linq查询的功能

其实如果我们想为一个数据源执行 Linq 查询操作的话其实很简单，其分可分为两种不同的书写模式来完成这一操作

- 链式查询方法：链式查询方法其实就是 Linq 查询的核心，由 `System.Linq.Enumerable` 和 `System.Linq.Queryable` 这两种命名空间为链式查询方法针对不同的数据源提供了扩展函数的支持
```csharp
    List<int> list = new List<int>() { 1, 2, 3, 4, 5, 6, 7 };

    var query = list
        .Where(x => x > 3)
        .Select(q => q);

    foreach (var item in query)
    {
        Console.WriteLine(item);
    }

    Console.ReadLine();
```

- 查询表达式：关于查询表达式需要讨论的问题比较多，<u>我们细分来看看</u>
    - 查询表达式是构建在 `CTS` 之上的一种由 `编辑器` 负责处理的语法，它本身是不属于托管语言的范畴的，换句话说它是承载在托管语言之上的一种语言，当我们对程序中编写了 `查询表达式` 后，是编辑器负责对查询表达式进行处理，而不是 `CLR` 负责对 Linq 进行处理，编辑器将 Linq 处理成框架所实现的基本接口集。简而言之，Linq 是语法糖层面的，它不是 `C#` 更不是 `CLR` 的基本内核的支持
    - 每一个查询表达式都有其所对应着的链式查询方法，在上面说到，Linq查询表达式 是构建在CTS上的一种语言，关于它的处理是交由编译器去完成的，也就是说当编译器在编译的过程当中，我们所书写的 Linq查询表达式 最终都会被编译器只能识别成其所对应的链式查询方法，举个例子，我们在 `where` 关键字在最终会被编译成 `where<T>` 的查询方法。另外补充一点，编译器在为查询表达式进行解析的过程当中还会分析当前所查询的数据源的类型，如果是 `Linq to Object` 类型的查询，编译器则最终所编译后的链式查询方法是由 `System.Linq.Enumerable` 所提供的，反之，如果是 `Linq to Provider` 类型的数据查询操作，编译器除了会把查询表达式编译成 `System.Linq.Queryable` 所扩展的链式查询方法外，还会调用数据源所实现自 `System.Linq.IQueryable` 的 `Provider` 函数，以提供为所录入的查询逻辑表达式的表达式树的解析工作，并在需要的时候来读取解析完成后的表达式树中的相关逻辑结构
```csharp
    List<int> list = new List<int>() { 1, 2, 3, 4, 5, 6, 7 };

    var query = from l in list
                where l > 3
                select l;

    foreach (var item in query)
    {
        Console.WriteLine(item);
    }

    Console.ReadLine();
```



#### 3. 延迟执行

使用Linq查询后的结果我们可以称它为一个查询变量，但是更准确的说法是 `一个可迭代的对象` ，在 [迭代模式篇章](/c%23/2019年/8月6号/迭代器模式.md) 说到过，一个可迭代对象并就是所指向的数据序列的本身，我们需要对它进行迭代工作后才能完整的获取数据源，这其实也就认证了 `延迟执行` 这句话