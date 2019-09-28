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
    - 查询表达式从 `from` 开始，以 `select` 或者 `group` 结束，在这两个表达式之间可以添加零个或者多个 `from` 、 `let` 、 `where` 、 `join`、 `orderby` 、……
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

#### 4. 延迟执行
其实关于延迟执行的问题，如果探讨仔细后，我们就可以理解 `Linq to Object` 和 `Linq to Provider` 之间的异同和其核心运行模型了

- 那么首先来是 `Linq to Object`：

使用Linq查询后的结果我们可以称它为一个查询变量，但是更准确的说法是 `一个可迭代的对象` ，一个可迭代对象并非就是所指向的数据序列的本身，我们需要对它进行迭代工作（获取可迭代对象所包含的迭代器，并不断地调用其实现的 `MoveNext()` 函数和 `Current` 属性）后才能完整的获取数据源，以此来完成一次延迟执行的操作，当然，关于这点其实无可厚非，在 [迭代模式篇章](/c%23/2019年/8月6号/迭代器模式.md) 章节中其实对其已经分析的很清楚明朗，但是我们需要注意的是，<span style="color:red">以上这些步骤仅仅只是针对一个可迭代对象的</span>，而在Linq中，我们每一个查询步骤的背后，其实都返回了一个 可迭代对象，这里就拿Linq查询表达式来说，我们在 `where` 的时候其实是返回来一个 可迭代对象，在 `select` 又是返回了一个可迭代对象，需要补充的是，Linq 的核心设计模式是 <span style="color:red">碎片化的环路执行对象模型</span>，简而言之上一次(`where`)所返回的可迭代对象所包含的内容会同步到下一个需要操作(`select`)的模型中去，通过多个可迭代对象共同构成了一个核心运行模型，这才是 `Linq to Object` 延迟执行的原理所在

![微信截图_20190924212318.png](https://i.loli.net/2019/09/24/z8N5cnV2Sy46fuE.png)

- 其次就来讨论下 `Linq to Provider`：

我们在前面说到，`System.Linq.Queryable` 针对于 `IQueryable<T>` 所提供的扩展方法是离不开 `System.Linq.Expressions` 表达式树的，每一个所库宗翰的链式查询方法中需要我们录入查询逻辑的 `Lambda表达式` 中总是会被一个表达式树所包裹着，其目的就是为了把当前所录入的查询逻辑 `Lambda表达式` 解析成一个表达式树，并保存起来，那么在下一次(`select`)调用的时候会同步到上一步(`where`)操作所解析出来的表达式树，并在此次操作完成之前把此次的表达式树又跟踪到刚刚所保存的上一次操作所解析出来的表达式树中，这些操作的最后其实都是保存在 `IQueryable` 中的属性 `Expression` 当中，其实我们可以想象到，在所有的查询表达式亦或是链式查询方法在解析完成的最后，`Expression` 属性所保存的表达式树的最终形态是多么的雄大，需要补充一点的是，`Linq to Provider` 同样享有着 <span style="color:red">碎片化的环路执行对象模型</span> 这一种模式

其实我们可能还存在疑惑，这些表达式的解析和执行的工作都是交给谁来完成的呢？其实就是交由 `IQueryable` 当中的 `IQueryProvider` 类型的属性 `Provdier` 去完成的，`Provdier` 在前面为当前链式查询方法所录入的查询逻辑 `Lambda表达式` 进行解析，并把解析后的表达式树同步到 `IQueryable` 中的 `Expression` 属性当中，当我们真正的对 `Linq to Provider` 所返回的结果进行遍历的过程当中，`Provdier` 才会去执行 `Expression` 所保存下来的表达式树，换句话来说，前面所做的过程其实都是为一个表达式树所做铺垫，在其的基础上增添不同的点缀，在最终遍历的过程中才把这颗庞大的表达式树交由一个提供者去执行操作，`Linq to Provider` 就是以此来完成延迟执行的对象模型的

```csharp
    public interface IQueryable : IEnumerable
    {
        //获取实例的表达式树 
        Expression Expression { get; }

        //获取表达式树执行之后返回的类型
        Type ElementType { get; }

        //获取实例数据源对应的query provider
        IQueryProvider Provider { get; }

        // 通过 IQueryable 所包含的 Provider 去执行最终所保存下来的表达式树
        public IEnumerator<T> GetEnumerator() 
        {
            return (Provider.Execute<T>(Expression) as IEnumerable<T>).GetEnumerator(); 
        } 

        // 通过 IQueryable 所包含的 Provider 去执行最终所保存下来的表达式树
        System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator() 
        { 
            return (Provider.Execute(Expression) as IEnumerable).GetEnumerator(); 
        }
    }
```

![微信截图_20190924215933.png](https://i.loli.net/2019/09/24/qv8wgPA234VTn5G.png)

#### 5. 动态Linq查询
动态构建Linq查询的核心关键在于构建一个表达式树，我们在前面说到了Linq主要针对于两种类型，第一种是 `Linq to Object` 第二种是 `Linq to Provider` ，而关于 `Linq to Provider` 所接收的链式查询方法中所录入的Lambda表达式都会被 `System.Linq.Expressions` 所包裹着，具体来讲，Linq是不能够动态的，但是表达式树却能够，所以我们可以自然而然的使用 `IQueryable<T>`，并动态构建一个表达式树将他交由扩展自 `System.Linq.Querable` 的方法自行处理，那可能会有疑问了，关于操作 `IEnumerable` 的 `Linq to Object` 该怎么处理？ `Linq to Object` 的扩展方法的提供商可是 `System.Linq.Enumerable` 和上面所说的扩展自 Enumerable的链式查询方法本身是不被Express表达式树所包裹着的 `System.Linq.Querable` 的方法可是截然不同，其实在 `System.Linq.Enumerable` 为 `IEnumerable<T>` 所提供的扩展方法中还提供了一个 `AsQueryable()`，可以把调用当前方法的 `IEnumerable<T>` 类型的实例转换成 `IQueryable<T>` 类型，这样我们就可以动态构造表达式树，并进行动态的Linq查询了，详情请见 
[.NET深入解析LINQ框架（三：LINQ优雅的前奏）](https://www.cnblogs.com/wangiqngpei557/archive/2012/12/04/2801181.html)

#### 5. 一些其他的知识点
- 数据源：

我们通过Linq查询表达式中的 `from` 关键字所引入的就是一个数据源，<span style="color:red;">Linq中的所有操作，不管是筛选(`where`、`let`、……)的过程还是查询(`select`)的过程，都是围绕着数据源所进行的</span>

一段Linq查询表达式中，我们是可以引入多个数据源的，需要注意的是，多次引入的数据源除了第一次引入(开头使用 `from` 表达式所引入的数据源)外，都需要一个父级数据源作为承载，并且当引入过后，子级数据源与父级数据源之间的关系是相互的，也就是说，当父级数据源所声明的范围变量经过了筛选，那么其改变也会映射至子级数据源当中，相对的子级数据源所映射的范围变量经过了筛选，那其改变也会映射至父级数据源当中去

- 范围变量：



<br/>

### Linq的实际应用

---

#### 1. from
`from` 是查询表达式的一个开头，更准确地说它是一个 `生成器` ，其目的在于引入一个可迭代类型（数据序列 `Sequence`）作为当前Linq表达式查询的数据源，并将其内部的元素声明为 `范围变量`

在这里需要解释一下 `范围变量` 这个名词，`范围变量` 顾名思义则是只作用于当前Linq查询中的一个变量，它可以是 <span style="color:red;">任意类型</span>

![微信截图_20190926204230.png](https://i.loli.net/2019/09/26/t4WqPkrghajBfQ8.png)

#### 2. where
用于对范围变量进行筛选和过滤

```csharp
    // 筛选在北京且名称以‘小’开头的顾客
    var query_Exp = from c in customers
                    where c.City == "北京" && c.Name.StartsWith("小")
                    select c;

    var query_Fun = customers.Where(c => c.City == "北京" && c.Name.StartsWith("小"));
```

#### 3. let
构造一个新的范围变量加入到此次查询的逻辑当中，当然我们可以选择性地使用这个新的范围变量

```csharp
    // 把集合中的CustomerID都+1 并且查询出来
    var query_Exp = from c in customers
                    let cTemp = c.CustomerID + 1
                    where cTemp > 3
                    select cTemp;
```

#### 4. 