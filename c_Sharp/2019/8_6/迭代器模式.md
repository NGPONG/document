# *迭代器模式*

<br/>

### 关于迭代器模式

---

#### 1. 迭代器模式是什么
> 迭代器模式是 `Linq` 的核心模式，关于 `Linq` 的实现都是围绕着迭代器模式所进行的

> 迭代器模式是设计模式中行为模式 `behavioral pattern` 的一个例子，他是一种简化对象间通讯的模式，也是一种非常容易理解和使用的模式，简单来说，迭代器模式使得你能够获取到序列中的所有元素而不用关心它的序列结构，这一点使我们能够非常高效的构建数据处理通道 `data pipeline` ，即数据能够进入处理通道，进行一系列的变换，或者过滤

#### 2. <span id="迭代器模式的构成">迭代器模式的构成</span>
迭代器模式由 `可迭代的对象` 和 `迭代器` 所构成，它们之间的关系是 <span style="color:red">前者囊括后者</span> 的关系，也就是说一个需要被迭代的对象里面有属于它自己的迭代器，而在.NET中，关于可迭代对象和迭代器的体现分别通过了 `IEnumerable` / `IEnumerable<T>` 和 `IEnumerator` / `IEnumerator<T>` 两种不同类型的接口所抽象出来
- 可迭代对象：实现于 `IEnumerable` / `IEnumerable<T>` 的对象，其内部除了提供了需要被迭代的数据序列之外，还提供了对实现 `IEnumerator` / `IEnumerator<T>` 迭代器的类的支持，需要注意的是，实现于 `IEnumerable` 可迭代对象其所对应的数据序列的类型则为 `object`，相对的，其泛型版本所对应的数据序列的类型则为泛型的类型定义
- 迭代器：实现于 `IEnumerator` / `IEnumerator<T>` 的对象，其定义类似于数据库中的游标，只能够向前进行移动，它除了能够记录当前数据序列所迭代到的位置之外还提供了相应的属性提供我们去获取当前所迭代到的元素的值

#### 3. 关于可迭代对象和迭代器所对应的接口的抽象成员解释
```csharp
    /// <summary>
    /// 可迭代对象
    /// </summary>
    public interface IEnumerable
    {
        /// <summary>
        /// 获取可迭代对象的迭代器
        /// </summary>
        /// <returns>迭代器</returns>
        IEnumerator GetEnumerator();
    }

    /// <summary>
    /// 泛型可迭代对象
    /// </summary>
    public interface IEnumerable<out T> : IEnumerable
    {
        /// <summary>
        /// 获取泛型可迭代对象的泛型迭代器
        /// </summary>
        /// <returns>泛型迭代器</returns>
        IEnumerator<T> GetEnumerator();
    }


------------------------------------------------------------


    /// <summary>
    /// 迭代器
    /// </summary>
    public interface IEnumerator
    {
        /// <summary>
        /// 当前所迭代到的元素的值
        /// </summary>
        object Current { get; }
        /// <summary>
        /// 数据序列的游标位置往下移动一位
        /// </summary>
        /// <returns>是否迭代完成</returns>
        bool MoveNext();
        /// <summary>
        /// 重置迭代器数据序列的游标位置的记录
        /// </summary>
        void Reset();
    }

    /// <summary>
    /// 泛型迭代器
    /// </summary>
    public interface IEnumerator<out T> : IDisposable, IEnumerator
    {
        /// <summary>
        /// 当前所迭代到的元素的值
        /// </summary>
        T Current { get; }
    }
```
#### 4. <span id="关于迭代器模式的特性">关于 迭代器模式 的特性</span>
- 当我们通过 迭代器模式 获取数据的时候，区别于直接操作集合或数组，迭代器模式 去获取数据的时候永远只是获取当前可迭代对象所绑定的迭代器的游标所指向的数据序列中的某一行的数据，而直接操作集合或数组则是一次性把我们所期望的数据都加载到内存当中以便我们去获取，我们需要明白的是，迭代器模式 是 `直线性` 的方式去获取数据的，无可否认，其速度是优于直接操作一个集合或数组的，因为传统的 迭代器模式 去获取数据需要不断的调用 `MoveNext()` 函数和 `Current` 属性去获取数据，并且每次所操作的数据只是针对当前可迭代对象的迭代器的游标所指向的数据序列的某一行，这对内存的压力显而易见是较少的，而直接操作一个集合或数组的话是一次性把我们所期望的数据都加载到内存当中，可想而知这样对内存的压力就较大了，其实通过这点也印证了经常听人所说的 `foreach` 操作集合要比 `for` 要快，因为 `foreach` 的操作实现其实就类比了传统的 迭代器模式 获取数据的方式，只是这部分相对与开发人员来是不可见的。当然，万物皆有利弊，使用 迭代器模式 去操作数据的话有一个致命的缺点就是该方式是 `直线性` 的获取数据，也就是说我们不能在迭代的过程当中去获取当前游标所指向的数据序列的位置的上一个位置数据的值，亦或者说我们想直接获取一个数据序列中具体某个位置的元素的值，这是 迭代器模式 无法做到的，当然集合和数组是可以很好的规避掉这一个缺点
- 当我们通过某种方式（Linq、自定义函数、直接声明可迭代对象的实例、迭代方法、迭代访问器）去获取了一个可迭代对象后，这个可迭代对象并不就是我们所期望的数据序列的本身，我们需要获取该可迭代对象所绑定的迭代器，并不断地调用该迭代器的 `MoveNext()` 函数和 `Current` 属性，直至调用完成，也就是 `MoveNext` 函数返回false的时候，我们才能够完完整整的获取到这个可迭代对象所表示的数据序列。我们在前面说到，Linq的核心其实就是 迭代器模式，而关于Linq的 `延时加载` 其实并没有什么魔力，它的实现机制和刚刚所说到的是一个道理，其实细心观察的话可以发现，我们每次进行查询linq操作的时候，返回的总是一个可迭代的对象，结合前面所说，因为它是一个可迭代对象所以它并不就是我们所期望的数据序列的本身，简而言之， Linq 所查询的最终结果并不是我们所期望的最终结果，也就是上面所提到的 `延迟加载`。但与传统的 迭代器模式 存在一点 <span style="color:red;font-size:12px">不同</span> 的是，linq查询我们使用查询方法或查询表达式所使用的、转换成的 `lambda` 表达式是封装在linq所返回的可迭代对象所绑定的迭代器所实现的 `MoveNext()` 函数内部的，由于 `foreach` 关键字其实就是对 迭代器模式 的迭代流程的简化，那么 `foreach` 每一次进行迭代工作的时候也就是在调用这个可迭代对象所绑定的迭代器的`MoveNext()` 函数和 `Current` 属性，因为我们所指定的 `lambda` 都封装在 `MoveNext()` 里头，所以每一次的调用 `MoveNext()` 的过程当中，都会执行我们所执行的数据筛选逻辑进行筛选工作

<br/>

### foreach

---

#### 1. <span id="foreach前言">前言</span>
> 在 c# 1.0 以前，我们要为一个已经定义好的 迭代器模式 进行迭代工作是非常繁琐的事情，首先我们需要创建一个可迭代对象的实例，证明我当前需要迭代的是哪位成员，其次，通过可迭代对象所实现相关接口的 `GetEnumerator()` 函数去获取当前可迭代对象所内置的迭代器，然后我们再通过一个循环，并且不断地调用迭代器所实现相关接口的 `MoveNext()` 函数把游标指向下一个数据序列，再调用迭代器的 `Current` 属性去获取当前所指向的数据序列的值，以此来完成一个 迭代器模式 的迭代工作

> 但是在 c# 1.0 以后，微软为我们提供了 `foreach` 语法糖让我们更好的为 迭代器模式 进行迭代工作

#### 2. foreach 语法糖使用前提条件
使用 `foreach` 进行迭代的成员，必须为可迭代对象，也就是实现了 `IEnumerable` / `IEnumerable<T>` 的实例成员，并且其所实现的  `GetEnumerator()` 能够返回一个正确的、合法的、实现了 `IEnumerator` / `IEnumerator<T>` 接口并且与当前可迭代对象有对应关系的迭代器

#### 3. foreach 语法糖为我们简化迭代的原理
- 首次迭代的时候，调用可迭代对象的 `GetEnumerator()` 去获取迭代器
- 调用迭代器的 `MoveNext()` 函数使当前游标所指向的数据序列位置往下进一位，并调用迭代器的 `Current` 属性来获取当前所指向的数据序列的值，该工作持续到 `MoveNext()` 函数返回 `false` 为止
- 在退出 `foreach` 语句块之前，或者说因为某种原因而推出 `foreach` 语句块（发生异常），则会在退出之前调用迭代器所实现的 `Dispose()` 函数来为当前迭代器进行内存的释放工作

以上则是 `foreach` 语法糖来简化迭代的实现原理，可以看出，它的实现原理和在该节点的[前言](#foreach前言)部分所介绍的我们如何去手动实现迭代的工作是一致的，只是该语法糖为我们开发人员隐藏了这一部分的内容

<br/>

### yield

---

#### 1. 前言
> 在 c# 2.0 以前，微软虽然为我们提供了 `foreach` 的语法糖去解决我们手动实现可迭代对象的繁琐迭代工作，但迭代器模式的本身还是存在另外一个问题引起开发者们苦恼的，那就是如果我们想手写一个 迭代器模式，那么就不得不考虑迭代对象和其所对应的迭代器的声明工作
在 [开头部分](#迭代器模式的构成) 我们说到了 迭代器模式 其实是包含了两个部分，为了OO设计单一职责的原则，通常把这两个部分都抽象出具体的类出来 <span style="color:red;font-size:12px">虽然也可以通过一个类来实现 能够被迭代的对象和迭代器 的工作，但是如果是自己手写迭代模式的话，不大推荐这一种做法</span>，简而言之就是建立不同的包含职责的类去抽象这一定义
那么我们先从迭代器来说起，在前面说到，在c#中迭代器的具体体现是通过继承 `IEnumerator` / `IEnumerator<T>` 接口并实现相关的抽象成员所体现出来的，那么我们就需要实现它的 `MoveNext()` 函数来让当前数据序列的游标位置往下递加、`Current` 属性获取当前游标所指向的元素的值、`Reset()` 函数来重置数据序列游标的计数工作，因为 迭代器模式 所设计的目的并不是一次性的返回所有的数据，所以迭代器为了保证下一次所迭代的数据是最新的数据，我们还要为当前所迭代到的或者说当前游标所指向的数据序列的元素做一个标识，不难发现，这些工作其实都是在间接地把迭代器的实例声明成了一个 状态机
有了迭代器之外我们还需要一个能够承载这个迭代器的可迭代对象，关于可迭代对象在具体实现c#中是通过继承 `IEnumerable` / `IEnumerable<T>` 接口并实现相关的抽象成员所体现出来的，那么我们就需要实现接口的 `GetEnumerator()` 函数去获取这个可迭代对象所绑定的迭代器，其次，就是需要提供一个迭代器可用的存储数据的数据序列，当然这个是根本
说到这里，我们也能够体会出手动编写 迭代器模式 具体的实现的时候是一件多么繁琐的事情了


> 在 c# 2.0 以后通过关键字 `yield` 可以帮我们解决了声明 迭代器模式 繁琐工作的问题

#### 2. yield 的介绍
yield 关键字拥有着两种组成的模式，其一是：`yield return <expression>`，其二则是：`yield break`，我们可以在一个自定义函数或是一个类中的 `get` 访问器中使用该关键字，当我们在上述的这两种环境中使用了 yield 关键字时，其实本意就是告诉 编译器 我们所定义的自定义函数或者是 `get` 访问器不再是原有的含义，而是一个可执行的 <span style="color:red;">迭代块</span>，更准确地来说它们是 <span style="color:red;">迭代函数</span> / <span style="color:red;">迭代访问器</span>

迭代块所返回的是一个 迭代器模式，具体来讲的话，所返回的类型为：`IEnumerable` / `IEnumerable<T>`、`IEnumerator` / `IEnumerator<T>`，也就是说我们可以为迭代块所返回的类型进行迭代工作

一个已经声明好的迭代块被调用后其，其内部的逻辑是 `延时加载` ，只有我们真正的对其返回的 迭代器模式 进行迭代的过程当中才会加载迭代块内部的代码逻辑，其实关于这方面并没有什么魔力，我们需要明白，yield 出现在 c# 2.0 以后其目的是为了解决我们前面说到的关于声明可迭代对象和其所绑定的迭代器及其繁琐的问题，简而言之它就是一个语法糖，当我们在一个我们所自定义的函数或者是 `get` 访问器中使用了这种语法糖并且还施以调用后，我们可以反编译看下编译器为了这个迭代块做了什么样的变化（以下仅示范自定义函数的情况）

![微信截图_20190821223311](https://i.imgur.com/n6kLrlN.png)

我们可以看到，当一个自定义函数被 yield 这个语法糖定义成迭代块的时候，编译器会为我们自动生成一个已经实现了可迭代对象和迭代器的类，换句话说，这个编译器为我们生成的类就是我们前面所说到的 迭代器模式 ，那么为什么关于迭代块的内部逻辑为什么只有在对调用迭代块后所返回的 迭代器模式 进行迭代的过程中其内部的逻辑才会执行呢？因为编译器把我们在迭代块中所声明的代码逻辑都封装在了其实现于迭代器的 `MoveNext()` 函数中，这时候结合[上面](#关于迭代器模式的特性)的所说，我相信对这个未知的黑箱已稍加明朗了，在这里还需要另外提的一点，编译器关于 yield 的解析其实是非常的智能的，如果我们只是在一个自定义函数中使用了 yield 关键字的话，编译器为我们所生成的类就会是一个即实现了可迭代对象的抽象又实现了迭代器的抽象的类，而当我们在一个已经定义好了，并且已经实现过了可迭代对象的类中中的 `GetEnumerator()` 函数中使用了该关键字的话，那么编译器为我们所生成的类就只是一个实现了迭代器抽象的类，简而言之编译器会根据我们目前需要什么，就去生成实现哪方面工作的类

回过头来继续讲一下 `yield return` 和 `yield break` 声明在了一个自定义函数或类中的 `get` 访问器中，具体起到了哪些作用
- **yield return：** 我们在为迭代块进行迭代的过程当中，每一次迭代所返回的数据其实就是 `yield return` 出来的数据，需要明白的是，在迭代的过程当中，每一次 `yield return` 后会记录当前迭代块当前所 `yield return` 出来的代码逻辑位置，以便在下一次的迭代工作中在所记录的位置继续下一步的逻辑运算，其实关于这部分并没有什么神秘的地方，我们所直观看到的事情并不就一定是事情的原委，仔细看上面的关于反编译后的结果图就会明白其详细的原理，因为我们所 `yield return` 的代码其实都是封装在编译器所自动为我们生成的实现于迭代器的 `MoveNext()` 函数中，并且其有一个属性会记录当前游标所指向的数据序列的位置，可能对于我们直观的debug的过程来说，我们看到迭代块在debug的时候的确是在 `yield return` 中保留了原有逻辑位置，实际上其内部还是我们在[上面](#关于迭代器模式的特性)所说到的关于 迭代器模式 进行迭代工作的原理的那一套工作
- **yield break：** 关于 `yield break` 就稍微简单点，直观来讲他就是退出此次迭代的工作，并且保留了游标在退出前最后所指向的数据序列的值

#### 3. yield 关键字使用时的注意事项
- 返回类型必须为 `IEnumerable` / `IEnumerable<T>`、`IEnumerator` / `IEnumerator<T>`
- 声明的过程中不能含有任何的 `in`、`ref` 或 `out` 关键字
- 当返回类型为默认的可迭代对象(`IEnumerable`)或是迭代器(`IEnumerator`)的时候，那么 `yield return` 所返回的类型则为 `object`，相反，其所对应的泛型版本则需要 `return` 相应的泛型类型
- yield 关键不能出现在Lambda表达式和匿名方法中

#### 4. yield 的简单使用
```csharp
    class Program
    {
        static void Main(string[] args)
        {
            // 通过 foreach 语法糖来为 yield 所返回的迭代器模式进行迭代工作
            foreach (var item in GetIterationSampleByPro)
            {
                Console.WriteLine(item);
            }

            // 传统的方式为 yield 所返回的迭代器模式进行迭代工作
            var enumeratoer = GetIterationSample(true).GetEnumerator();
            while (enumeratoer.MoveNext())
            {
                Console.WriteLine(enumeratoer.Current);
            }
        }

        /// <summary>
        /// 迭代访问器
        /// </summary>
        public static IEnumerable<string> GetIterationSampleByPro
        {
            get
            {
                yield return "GetIterationSampleByPro：Hello，World";

                for (int i = 0; i < 3; i++)
                {
                    yield return $"GetIterationSampleByPro：Is the {i.ToString()} time continue to work!";
                }
            }
        }
        /// <summary>
        /// 迭代方法
        /// </summary>
        /// <returns></returns>
        public static IEnumerable<string> GetIterationSample(bool isBreak)
        {
            yield return "GetIterationSample：Hello，World";

            if(isBreak)
                yield break;
            else
                yield return "GetIterationSample：It's continue to work!";
        }
    }
```

#### 5. 关于迭代块中出现了 try-catch-finally 语句块的具体注意事项
- **try**
    - yield return：不能够出现在 `try-catch` 语句块中的 `try` 块中，但是允许出现在 `try-finally` 语句块中的 `try` 块中 
    - yield break：可以出现在任何的 `try` 语句块中
- **catch**
    - yield return：不能出现在任何的 `catch` 块中
    - yield break：可以出现在任何的 `catch` 块中
- **finally**
    - yield return：无法出现在任何的 `finally` 块中
    - yield break：可以出现在任何的 `finally` 块中
- **Extend** 
当我们在调用了迭代块所返回的 迭代器模式 所实现的 `Dispost()` 函数的时候，如果迭代块中存在 `finally` 语句块则该 `finally` 块中的代码会被执行，那么我们除了手动的调用所实现的 `Dispost()` 函数，还有什么条件下会出现调用 `Dispost()` 函数的情况呢
    - 通过using()释放了一个迭代块的内存占用
    - 手动调用迭代块的Dispose函数
    - foreach在进行迭代的过程当中，自然退出或非自然退出前
