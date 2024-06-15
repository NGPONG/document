# *Language Intergated Query*

### 前言

#### 1. 关于Linq的简单介绍

> Linq 是一系列直接将查询功能集成到`C#`语言的技术统称，换句话说它是一种统一的数据查询接口，其系列技术提供了一种跨各种数据源和数据格式 (实现自<span style="color:red">IEnumerable</span>或<span style="color:red">IEnumerable&lt;T&gt;</span>接口) 的查询体验，借助 Linq，查询成为了最高级的语言构造，就像类、方法和事件一样，可以使用语言关键字和熟悉的运算符针对强类型化对象集合编写查询

> `Linq查询表达式`是统一的数据查询接口，通过提供了一种跨各种数据源和数据格式使用数据的一致模型，不管是查询 `Linq to Object` 也好还是自定义数据源 `Linq to Provider`，Linq对于数据检索的表达式是不变的，需要变更的只是数据查询提供程序

![2012072522494987.jpg](https://i.loli.net/2019/09/21/3KVXgcnrOZPh9lJ.jpg)

#### 2. Linq查询操作的流程

- 获取数据源
- 创建查询
- 执行查询


### 让我们进一步的理解Linq

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

#### 4. 如何具体的实现Linq查询的功能

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

    - 查询表达式必须以 `from` 开头，且必须以 `select` 或 `group` 结尾，在第一个 `from` 与最后一个 `select` 或 `group` 之间，可以包含以下这些可选子句中的一个或多个： `where` 、 `orderby` 、 `join` 、 `let`

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

动态构建Linq查询的核心关键在于构建一个表达式树，我们在前面说到了Linq主要针对于两种类型，第一种是 `Linq to Object` 第二种是 `Linq to Provider` ，而关于 `Linq to Provider` 所接收的链式查询方法中所录入的Lambda表达式都会被 `System.Linq.Expressions` 所包裹着，具体来讲，Linq是不能够动态的，但是表达式树却能够，所以我们可以自然而然的使用 `IQueryable<T>`，并动态构建一个表达式树将他交由扩展自 `System.Linq.Querable` 的方法自行处理，那可能会有疑问了，关于操作 `IEnumerable` 的 `Linq to Object` 该怎么处理？ `Linq to Object` 的扩展方法的提供商可是 `System.Linq.Enumerable` 和上面所说的扩展自 Enumerable的链式查询方法本身是不被Express表达式树所包裹着的 `System.Linq.Querable` 的方法可是截然不同，其实在 `System.Linq.Enumerable` 为 `IEnumerable<T>` 所提供的扩展方法中还提供了一个 `AsQueryable()`，可以把调用当前方法的 `IEnumerable<T>` 类型的实例转换成 `IQueryable<T>` 类型，这样我们就可以动态构造表达式树，并进行动态的Linq查询了，详情请见 [.NET深入解析LINQ框架（三：LINQ优雅的前奏）](https://www.cnblogs.com/wangiqngpei557/archive/2012/12/04/2801181.html)

#### 6. 一些其他的知识点

##### 6.1 关于数据源在一段Linq查询表达式中的核心地位：

我们通过Linq查询表达式中的 `from` 关键字所引入的就是一个数据源，<span style="color:red;">Linq中的所有操作，不管是筛选(`where`、`let`、……)的过程还是查询(`select`)的过程，都是围绕着数据源所进行的</span>。具体来讲的话，不管我们 `select` 出来的是什么 `范围变量` ，其元素的数量都是与当前表达式所指定的数据源所相关联的，我们可以看一下以下的代码，我们所操作的数据源为 `customers` 其被定义成范围变量 `c` ，经过 `where` 筛选之后的元素数量结果为 `2` 个，所以最后我们所 `let` 出来的元素数量的结果也是 `2` 个

```csharp
var query_Exp = from c in customers
                where c.CustomerID > 6
                let temp = 1
                select temp;
```

6.2 <span id = "一段Linq查询表达式中所引入的多个查询数据源他们之间所形成的关系">一段Linq查询表达式中所引入的多个查询数据源他们之间所形成的关系：</span>

一段Linq查询表达式中，我们是可以通过多个 `from` 关键字来引入多种不同的数据源的，需要注意的是，我们通过 `from` 关键字所多次引入的不同的数据源，它们之间会形成一种相互的关系。具体来讲，如果多次引入的数据源之间并没有什么实际的连接关系，则它们之间则会互相做 `笛卡尔积` ，也就是 `SQLSERVER` 中的 `CROSS JOIN` 关键字的效果，而当所引入的数据源之间都有某个属性是有相对应的关系的时候，他们之间就会相互做一个 `INNER JOIN` 形式的连接，再次强调一点的是，不管是 `笛卡尔积` 也好还是 `INNER JOIN` ，<span style="color:red;">这种关系的成果是直接映射到当前Linq查询表达式中所有引入的数据源的改变的</span>，这是一种机制，具体可以参考 [复合from的用法](#复合from)

##### 6.3 范围变量：
范围变量就是可以作用于当前Linq查询表达式中使用的一个查询变量，它可以是 <span style = "color:red">任意类型</span> ，我们在使用 `from` 关键字的时候，除了引入一个数据源到当前Linq查询表达式之外，还会把当前数据源中的元素的类型当成是该数据源所对应的范围变量，举个例子，所引入的数据源的类型为 `IEnumerable<int>` ，则其所对应的范围变量的类型则为 `int` 类型

### Linq的实际应用

#### 1. from

`from` 是查询表达式的一个开头，更准确地说它是一个 `生成器` ，其目的在于引入一个可迭代类型（数据序列 `Sequence`）作为当前Linq表达式查询的数据源，并将其内部的元素声明为 `范围变量`

![微信截图_20190926204230.png](https://i.loli.net/2019/09/26/t4WqPkrghajBfQ8.png)

#### 2. 复合from<span id="复合from"></span>

针对一个数据源的内部如果存在成员的类型也是一个数据序列的类型的时候，可使用复合from来把这个数据序列作为数据源引入到当前Linq查询表达式当中
```csharp
    var query_Exp = from c in customers
                    from o in c.Orders
                    from d in o.Details
                    select new
                    {
                        d.DetailID,
                        d.ProductID,
                        d.UnitPrice,
                        o.OrderID
                    };

------------------------------------------------------------------------------------------------------------------------------------
    
    // 把两个数据源做笛卡尔积
    char[] upperCase = { 'A', 'B', 'C' };
    char[] lowerCase = { 'x', 'y', 'z' };

    var query_Exp = from upper in upperCase
                     from lower in lowerCase
                     select new 
                     {
                          upper,
                          lower
                     };
```

#### 3. where

用于对范围变量进行筛选和过滤

```csharp
    // 筛选在北京且名称以‘小’开头的顾客
    var query_Exp = from c in customers
                    where c.City == "北京" && c.Name.StartsWith("小")
                    select c;

    var query_Fun = customers.Where(c => c.City == "北京" && c.Name.StartsWith("小"));
```

#### 4. let

构造一个新的范围变量加入到此次查询的逻辑当中，当然我们可以选择性地使用这个新的范围变量

```csharp
    // 把字符串集合中的句子都以空格分割出来，并都转换成小写打印出来
    string[] strings = 
    {
        "A penny saved is a penny earned.",
        "The early bird catches the worm.",
        "The pen is mightier than the sword."
    };

    var query = from sentenct in strings
                let words = sentenct.Split(' ')
                from word in words
                let wordProcess = word.ToLower()
                select wordProcess;
```

#### 5. orderby

对范围变量进行排序

```csharp
    string str = "qwertyuiopasdfghjklzxcvbnm123456789";

    // 为字符串中的每个字符进行倒序排序
    var query_Exp = from s in str.ToCharArray()
                    orderby s descending
                    let sProcess = s.ToString().ToUpper()
                    select sProcess;
    var query_Fun = str.ToCharArray().Select(x => x.ToString().ToUpper()).OrderByDescending(x => x);

    // 为字符串中的每个字符进行正序排序
    var query_Exp = from s in str.ToCharArray()
                    orderby s ascending
                    let sProcess = s.ToString().ToUpper()
                    select sProcess;
    var query_Fun = str.ToCharArray().Select(x => x.ToString().ToUpper()).OrderBy(x => x);
```

#### 6. <span id = "group">group</span>

正如 [前面](#Linq查询表达式开头和结尾) 所说到，Lin请查询表达式是能够以 `Group` 关键字作为结尾的，所以我们在Linq查询表达式的逻辑是使用 `Group` 来处理逻辑的话，那么其下文就不能再继续编写其余的Linq查询表达式了，除非我们使用 `into` 来使当前查询做一个延续，需要注意的是，一旦我们使用了 `into` 关键字之后，所指定的范围变量就会覆盖掉之前所引用的数据源与查询变量了，我们可以理解为这是开始一次新的查询去进行，我们还需要注意一点的是这个分组出来的新的数据源中的成员，具体来讲其类型可以看成是一个键值对的集合，其中有一个 `Key` 属性作为此组合所分组的依据，而另一个属性则包含着这个分组依据所存在的数据集合

```csharp
    // 把字符串集合都按照每个元素的首字母进行分组，筛选出分组结果大于2的元素，并且每个筛选结果元素的末尾要加上 分组成功 的样式
    string[] words = { "apples", "blueberries", "oranges", "bananas", "apricots" };

    var query_Exp = from w in words
                    group w by w[0] into g
                    where g.Count() > 1
                    select (from gProc in g
                            select gProc + $" 分组成功，这个是{g.Key.ToString()}组");

    var query_Func = words.GroupBy(x => x[0])
                          .Where(g => g.Count() > 1)
                          .Select(x => x
                          .Select(y => y + $"分组成功，这个是{x.Key.ToString()}组"));
```

#### 7. into

为 `select`、`join` 或 `group by` 所投影出来的数据源继续引入到当前linq查询的上下文当中，并为其构造一个新的范围变量，对于它的具体用法，简而言之，`into` 可以使 `select` 或 `group by` 操作的 <span style="color:red">延续</span> ，可以为 `join` 的结果进行 <span style="color:red">分组汇总</span>

```csharp
    var query_Exp = from c in customers
                    select c into cContinue
                    from o in cContinue.Orders
                    select o.OrderID;
```

#### 8. join

`join` 关键字的用法主要有两种

第一种用法则和前面所说到的 [复合from的用法](#复合from) 的概念类似，其标志是在 `join` 进行连接后不使用 `into` 关键字为连接后的数据进行分组汇总操作，该用法类似于 `SQLSERVER` 中的 `INNER JOIN` 关键字的用法，只会把所join的两个数据源之间相匹配的元素给找出来，需要注意的是，一旦使用了这种用法后，通过 `join` 连接的两个数据源中的元素的个数都会发生变化，举个例子，数据序列A的元素个数有5个，数据序列B的元素个数有三个，但是两个数据序列之间只有两个元素能够发生相互连接的关系，当我们使用了 `join` 连接两个数据源的时候，我们不管在最后 `select` 的是哪个范围变量所代表的数据源，其元素的个数都为两个，具体可以参考这一 [小节](#一段Linq查询表达式中所引入的多个查询数据源他们之间所形成的关系)

```csharp
    // 查询出顾客名称其所在的城市，还有这个顾客所下的订单的总金额还有这个订单所包含的商品
    var query_Exp = from c in customers
                    from o in c.Orders
                    from d in o.Details
                    join p in products on d.ProductID equals p.ProductID
                    select new
                    {
                        Name = c.Name,
                        City = c.City,
                        Money = d.Quantity * d.UnitPrice,
                        ProductName = p.ProductName
                    };


    var query_Exp2 = from r in
                        from c in customers
                        from o in c.Orders
                        from d in o.Details
                        select new { Name = c.Name, City = c.City, Money = d.Quantity * d.UnitPrice, ProductID = d.ProductID }
                     join t in
                        from p in products
                        select p
                     on r.ProductID equals t.ProductID
                     select new { Name = r.Name, City = r.City, Money = r.Money, ProductName = t.ProductName };


    var query_Fun = customers.SelectMany(c => c.Orders, (c, o) => new { Name = c.Name, City = c.City, Details = o.Details })
                             .SelectMany(oc => oc.Details, (oc, details) => new { Name = oc.Name, City = oc.City, Money = details.Quantity * details.UnitPrice, ProductID = details.ProductID })
                             .Join(products, ocd => ocd.ProductID, p => p.ProductID, (ocd, p) => new { Name = ocd.Name, City = ocd.City, Money = ocd.Money, ProductName = p.ProductName });
```

第二种用法是通过 `into` 关键字来完成，该关键字其实可以根据 `join` 的数据源的不同来达成不同的效果，其一类似于 `INNER JOIN` 的效果，和 [前面小结](#复合from) 所提供的样例无恙，而 `into` 在这其中的作用只是提供一个操作的延续，这里我们要重点说下它的 `into` 的第二种效果，它能够为所 `join` 的两个数据源提供一个分组汇总操作，这种做法的好处其一，<span style="color:red">如果能够正确的分配所连接的两个数据源</span>，那么在 `into` 后所给我们做出的分组汇总结果能使我们更加清晰地知道该字段的具体某个值（ `into` 后的结果有一个 `Key` 属性作为参考，具体可以查看这一 [小节](#group) 的内容）到底连接了多少个所 `join` 的数据序列中的元素，第二就是如果我们有类似 `SQLSERVER` 中 `LEFT JOIN` 的需求，那么通过这种方式也能够很方便的呈现出来

关于 `into` 所提供给 `join` 的分组汇总操作我们还需注意一个问题，其实关于这个问题在上面一节也用红色字体标注出来了，在这个过程当中其实有两种不同类型的数据源，一种是 `连接数据源` 另一种是 `需要被连接的数据源` ，这两种数据源之间的元素所表现得形式是需要区分开来的， `连接数据源` 中的需要和另一个数据源( `需要被连接的数据源` )的连接依据，也就是说需要进行相互匹配的成员的值最好不要出现重复性的数据，如：`0、1、2、1、0` 这样子的数据，我们应当保证该成员的值在当前数据源的元素当中的存在是唯一性的，如： `0、1、2、3、4` ，这样我们才能称呼这个数据源为 `连接数据源` ，那么在具体的编写Linq查询表达式的过程当中， `join` 关键字后面所连接的数据源我们要保证它的类型为 `需要被连接的数据源` ，而已作用于当前Linq查询表达式上下文的数据源，换句话说就是在使用 `join` 关键字之前当前Linq表达式所使用的数据源我们要保证它的类型为 `连接数据源` ，原因其实很简单，因为 `join into` 会把 `连接数据源` 中我们所指定的连接逻辑的成员所包含的元素都一一拿出来与 `需要被连接的数据源` 相互匹配的，还是刚刚的例子，比如说我们所指定的连接逻辑成员为 `ProductId` 成员，那么则会把 `连接数据源` 中 `ProductId` 的值 `0、1、2、3、4` 都拿出来与另一边的 `需要被连接的数据源` 相互匹配，如果存在一样的则把它拿出来并 `into` 到分组汇总的结果当中去，那么在这一个过程当中，如果 `连接数据源` 中的 `ProductId` 的值存在一个重复的话，则可能会出现重复的组的情况，比如说重复的数据 `0、1、2、1、0` 则会出现重复的 `1` 组和 `0` 组

```csharp
    // 标准的 join into 查询
    var query_Exp_Default = from p in products
                            join d in details on p.ProductID equals d.ProductID into dGroup
                            from dGroupProcess in dGroup
                            select dGroupProcess;

    var query_Fun_Default = products.GroupJoin( details,
                            p => p.ProductID,
                            d => d.ProductID,
                            (p, dGroup) => dGroup);

    // 左连接形式的 join into 查询
    var query_Exp_left = from p in products
                            join d in details on p.ProductID equals d.ProductID into dGroup
                            from dGroupPro in dGroup.DefaultIfEmpty()
                            select dGroupPro;

    var query_Fun_left = products.GroupJoin(details,
                         p => p.ProductID,
                         d => d.ProductID,
                         (p, dGroup) => dGroup.DefaultIfEmpty());
```

### 参考文档

- [.NET深入解析LINQ框架（一：LINQ优雅的前奏）](https://www.cnblogs.com/wangiqngpei557/archive/2012/11/10/2764201.html)

- [.NET深入解析LINQ框架（二：LINQ优雅的前奏）](https://www.cnblogs.com/wangiqngpei557/archive/2012/11/22/2783357.html)

- [.NET深入解析LINQ框架（三：LINQ优雅的前奏）](https://www.cnblogs.com/wangiqngpei557/archive/2012/12/04/2801181.html)

- [.NET深入解析LINQ框架（四：IQueryable、IQueryProvider接口详解)](https://www.cnblogs.com/wangiqngpei557/archive/2012/12/11/2813490.html)

- [.NET深入解析LINQ框架（五：IQueryable、IQueryProvider接口详解)](https://www.cnblogs.com/wangiqngpei557/archive/2012/12/14/2817559.html)

- [语言集成查询 (LINQ)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/)

- [查询关键字（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/query-keywords)

- [语言集成查询 (LINQ)](https://docs.microsoft.com/zh-cn/dotnet/csharp/linq/index)

- [[深入学习C#]LINQ查询表达式详解(1)——基本语法、使用扩展方法和Lambda表达式简化LINQ查询](https://blog.csdn.net/honantic/article/details/46472647)

- [[深入学习C#]LINQ查询表达式详解(2)——查询表达式的转换](https://blog.csdn.net/xuchen_wang/article/details/86520995)

- [C#基础之IEnumerable](https://www.cnblogs.com/fangyz/p/5721269.html)

- [.NET复习笔记之LINQ，从IQueryable说起](https://zhuanlan.zhihu.com/p/47776558)

- [打造自己的LINQ Provider（上）：Expression Tree揭秘](https://www.cnblogs.com/Terrylee/archive/2008/08/01/custom-linq-provider-part-1-expression-tree.html)

- [打造自己的LINQ Provider（中）：IQueryable和IQueryProvider](https://www.cnblogs.com/Terrylee/archive/2008/08/25/custom-linq-provider-part-2-IQueryable-IQueryProvider.html)