# *Generic*


<br/>

### 前言

---

#### 泛型的介绍

泛型是所存储或使用的一个或多个类型具有占位符 （ <kbd>类型形参</kbd> ） 意义的类、结构、接口和方法，通过这个类型参数可以提供具体 [[参数化 ]]的类型，事实上，泛型类型的类型参数变成了泛型类型的元数据，运行时在需要的时候会利用它们去构造合适且符合的类型，通过这些类型，我们有可以实例化不同类型的对象，也就是说

> 未绑定泛型类型是来构造泛型类型的蓝图，已构造泛型类型又是实际对象的蓝图

#### 使用泛型所带来的好处

- 泛型将类型安全的负担从开发人员手里转移到编译器，开发人员没有必要编写代码来测试正确的数据类型，因为它会在编译时强制执行，这一定程度上降低了强制类型转换的必要性和运行时错误的可能性
- 使用泛型能够有效地避免对值类型进行装箱和拆箱时所造成的性能损失
- 如果使用的恰当，泛型能够很大的程度上避免代码的重用


<br/>

### 先来讨论下泛型

---

#### .NET 是如何实现泛型的？

泛型的开始，由我们在 IDE 中输入一个 `<T>` 开始，当编译器在编译的过程中遇到我们所写的泛型的时候，会针对该语法，并采用一种特殊的占位符来表示这种类型参数 <kbd>`x</kbd>，在这里 <kbd>x</kbd> 表示为 <kbd>元数</kbd>，其代表着这个泛型类型所具有的类型参数的数量，如下面代码所示，泛型类中一共包含了3个类型参数，那么最终该行代码，或者说这个泛型类被编译成 <kbd>IL</kbd> 后其占位符所存在的元数则为 3

```csharp
public class Person<T,T2,T3,T4,T5>
{
}
```

回到正题，在使用了泛型后，其编译的代码都带有一个特殊的标识符，那么当引用了这个 <kbd>dll</kbd> 的 <kbd>Appdomain</kbd> 在实际运行的过程当中，<kbd>CLR</kbd> 会进行一次二次编译的工作，在二次编译的过程中，<kbd>JIT</kbd> 就会找到 <kbd>IL</kbd> 代码中存在特殊占位符的代码，并把我们所指定的相应的 <kbd>封闭类型</kbd> 填充至相应的占位符的位置，通过这样就可以形成运行时编译录入我们所指定的封闭类型的特点

>泛型是由 <kbd>.NET Framework 2.0</kbd> 才开始引入的新特性，从上面的说明也能够看出 <kbd>Version 1.0</kbd> ~ <kbd>Version 2.0</kbd> 之间也一并更新了 编译器 和 <kbd>CLR</kbd> 的功能才能够独以支持该特性

#### 泛型类中静态成员的特殊性 <span id = "标识1"></span>

因为 <kbd>泛型</kbd> 的引入，导致一个泛型类中的静态成员也发生了一些变化，在以往的认知中，一个类中的静态成员 <kbd>CLR</kbd> 会优先初始化静态变量和静态构造函数，且只有在第一次实例化的时候进行初始化，后续都不在进行初始化，但是在泛型类中，静态成员变量在相同封闭类型间共享，不同的封闭类型间不共享，简而言之就是这个泛型类所采用的不同类型的封闭类型都会针对它去生成一份 <kbd>副本</kbd>，这也导致了一种肉眼可见的一种结果，同一个类里面的静态成员会出现多次初始化的情况，详情可以查看下面的代码

```csharp
class TypeWithField<T>
{
    public static string field;
    public static void PrintField()
    {
        Console.WriteLine(field);
    }
}

class Program
{
    static void Main(string[] args)
    {
        TypeWithField<int>.field = "Int Field";
        TypeWithField<string>.field = "String Field";

        TypeWithField<int>.PrintField();
        TypeWithField<string>.PrintField();

        Console.Read();
    }
}
```

#### 关于在一个泛型成员中，相同类型参数的延续性

比如说我们现在有一个函数包含了一个泛型类型参数 `<T>`，如果在该函数的 <kbd>Signture</kbd> 中同样也存在类型名为 `T` 的参数，那么该函数的参数的类型同样也沿用调用该函数时所设定的具体封闭类型，如下代码

```csharp
namespace Generic
{
    public delegate T1 GenericHandler<T1, T2>(T2 value);

    class Program
    {
        static void Main(string[] args)
        {
            CommonMethod.Get(delegate (int inPut)
            {
                return (++inPut).ToString();
            }, 1);

            Console.ReadKey(true);
        }
    }

    public class CommonMethod
    {
        public static void Get<T1,T2>(GenericHandler<T1,T2> handler,T2 value)
        {
            Console.WriteLine($"Hello! Value test:{handler.Invoke(value)}");
        }
    }
}
```

<br/>

### 泛型的具体实现

---

使用一个泛型成员，我们需要在该成员名结束的地方以 `<T>` 作为标记，以表明该成员需要使用一个名字为 <kbd>T</kbd> 的类型参数，如果需要指定多个的话，则以 <kbd>,</kbd> 为标记，例如：<kbd><T1,T2,T3,T4,T5,……></kbd>

> 一旦声明了一个泛型参数类型给某一个泛型成员，那么在使用它的过程中必须要指定它具体的封闭类型，当然我们也可以隐式进行指定，什么是隐式进行指定？例如一个泛型方法中标明了一个泛型参数类型 `<T>`，并且其带有的参数也是泛型参数 `T` 类型，这时候我们在调用这个函数的时候就不需要另外再指定它泛型的参数类型具体是某个封闭类型了，直接在输入参数中录入相应的实例即可自动识别，如下面代码
```csharp
namespace Generic
{
    class Program
    {
        static void Main(string[] args)
        {
            CommonMethod.Get(new Person() { Name = "NGPONG" });

            Console.ReadKey(true);
        }
    }

    public class Person
    {
        public string Name { get; set; }
    }

    public class CommonMethod
    {
        public static void Get<T>(T t)
        {
            var per = t as Person;

            Console.WriteLine(per.Name);
        }
    }
}
```

泛型的应用领域大致可以把它们分为四种类型，它们分别是

- 泛型方法

```csharp
namespace Generic
{
    class Program
    {
        static void Main(string[] args)
        {
            CommonMethod.Get(new Person() { Name = "NGPONG" });

            Console.ReadKey(true);
        }
    }

    public class Person
    {
        public string Name { get; set; }
    }

    public class CommonMethod
    {
        public static void Get<T>(T t)
        {
            var per = t as Person;

            Console.WriteLine(per.Name);
        }
    }
}
```

- 泛型委托

```csharp
namespace Generic
{
    public delegate T1 GenericHandler<T1, T2>(T2 value);

    class Program
    {
        static void Main(string[] args)
        {
            CommonMethod.Get(delegate (int inPut)
            {
                return (++inPut).ToString();
            }, 1);

            Console.ReadKey(true);
        }
    }

    public class CommonMethod
    {
        public static void Get<T1,T2>(GenericHandler<T1,T2> handler,T2 value)
        {
            Console.WriteLine($"Hello! Value test:{handler.Invoke(value)}");
        }
    }
}
```

- 泛型接口

```csharp
namespace Generic
{
    class Program
    {
        static void Main(string[] args)
        {
            IGenericTest<string> genericTest = new GenericTest<string>();
            genericTest.Get("NGPONG");

            Console.ReadKey(true);
        }
    }

    public class GenericTest<T> : IGenericTest<T>
    {
        public void Get(T value)
        {
            Console.WriteLine(value.ToString());
        }
    }

    public interface IGenericTest<T>
    {
        void Get(T value);
    }
}
```

- 泛型类

```csharp
namespace Generic
{
    public delegate int GenericHandler<T>(T value1,T value2);

    class Program
    {
        static void Main(string[] args)
        {
            var commandString = new CommandMethod<string>();
            var strResult = commandString.GetMaxLength(new string[] { "测试一下", "23333333","666" }, commandString.GetMaxString);

            var commandInt = new CommandMethod<int>();
            var intResult = commandInt.GetMaxLength(new int[] { 1, 2, 3, 4, 5, 6, 7, 8 }, commandInt.GetMaxInt);

            Console.ReadKey(true);
        }
    }

    public class CommandMethod<T>
    {
        public T GetMaxLength(T[] arrary,GenericHandler<T> handler)
        {
            T max = arrary[0];

            for (int i = 0; i < arrary.Length; i++)
            {
                if (handler(arrary[i], max) > 0)
                {
                    max = arrary[i];
                }
            }

            return max;
        }

        public int GetMaxInt(int value1, int value2)
        {
            return value1 - value2;
        }

        public int GetMaxString(string value1, string value2)
        {
            return value1.Length - value2.Length;
        }
    }
}
```

<br/>

### 泛型缓存

---

根据 [上文](#标识1)，依赖于泛型类中静态类型的特性我们可以设定一个以 <kbd>Type</kbd> 作为键的泛型字典缓存，其性能对比单纯的以 <kbd>Type</kbd> 作为键的字典集合要优秀的很多，下面的代码展示除了泛型缓存和字典缓存之间的性能对比

```csharp
namespace GenericCache
{
    class Program
    {
        static void Main(string[] args)
        {
            GenericCacheTest<object>.TestGenericStorage();
            DictionaryCacheTest.TestDictionaryStorage();

            Console.ReadKey(true);
        }
    }

    public static class Cache_Generic<T>
    {
        public static object Instance { get; set; }
    }

    public static class Cache_Dic
    {
        public static Dictionary<Type, object> Instance { get; set; }
    }

    public class GenericCacheTest<T>
    {
        static GenericCacheTest()
        {
            Cache_Generic<T>.Instance = new object();
        }

        public static void TestGenericStorage()
        {
            Stopwatch watch = new Stopwatch();
            watch.Start();

            for (int i = 0; i < 100_000_000; i++)
            {
                var obj = Cache_Generic<T>.Instance;
            }

            watch.Stop();
            Console.WriteLine($"GenericCache:{watch.ElapsedMilliseconds.ToString()}");
        }
    }

    public class DictionaryCacheTest
    {
        static DictionaryCacheTest()
        {
            Cache_Dic.Instance = new Dictionary<Type, object>();

            Cache_Dic.Instance[typeof(object)] = new object();
        }

        public static void TestDictionaryStorage()
        {
            var type = typeof(object);

            Stopwatch watch = new Stopwatch();
            watch.Start();

            for (int i = 0; i < 100_000_000; i++)
            {
                var obj = Cache_Dic.Instance[type];
            }

            watch.Stop();
            Console.WriteLine($"DictionaryCache:{watch.ElapsedMilliseconds.ToString()}");
        }
    }
}
```

```console
GenericCache
        Time Elapsed:   1803ms
        CPU Cycles:     151,015,248
        Gen 0:          0
        Gen 1:          0
        Gen 2:          0

Normal DictionaryCache
        Time Elapsed:   3619ms
        CPU Cycles:     22,475,810,124
        Gen 0:          0
        Gen 1:          0
        Gen 2:          0
```

可以看到执行结果，使用了泛型缓存和使用普通的字典缓存，进行一亿次数据获取的情况下，泛型缓存的速度大约是字典缓存的 <kbd>2倍多</kbd>

就性能而言，我们可以肉眼看得到差距，但是泛型缓存自身也存在很多的缺点，例如：占用的空间（应该）较多、一个类型的键只能全局唯一，不如普通字典的缓存方式来的灵活，另外，除非能够在代码中得到泛型参数，否则同样无法使用泛型字典

<br/>

### 泛型约束

---

泛型约束我们可以称之为 <kbd>权利和义务</kbd>，什么是权利？当我们为一个泛型成员规定了一个泛型约束后，那么他就拥有对于类型参数进行某种约束的 <kbd>权利</kbd>，而使用这个泛型成员也必须按照具体的约束来指定类型参数，这就是一种 <kbd>义务</kbd>

泛型约束可以分为5种类型，他们分别是：

> 每一种类型不可以随意进行搭配，不同的泛型约束类型都拥有一定得局限性，不可随意进行搭配

- 基类约束：所指定的参数类型必须继承与或者就是约束类型的本身
    - 一条约束条文中不可以出现多个
    - 出现的位置必须要出现在最前面

```csharp
public class Person
{
    public string Name { get; set; }
}

public class CommandMethods<T>
    where T : Person
{

}
```

- 接口约束：所指定的参数类型必须实现于指定约束的接口
    - 一条约束条文中可以多次使用接口约束，就其本身可以任意搭配，并无具体的限制要求

```csharp
public interface IPerson
{
    string Name { get; set; }
}

public interface IMan
{
    char Gender { get; set; }
}

public class CommandMethods<T>
    where T : IPerson,IMan
{

}
```

- 无参构造函数约束：所指定的参数类型必须是引用类型切包含无参的构造函数
    - 一条约束条文中不可以出现多个
    - 出现的位置必须要出现在最后面
    - 不可以和值类型约束一同使用

```csharp
public class CommandMethods<T>
    where T : new ()
{

}
```

- 值类型约束：所指定的参数类型必须为值类型
    - 一条约束条文中不可以出现多个
    - 出现的位置必须要出现在最前面
    - 不可以和引用类型约束一起使用
    - 不可以和无参构造函数约束一起使用

```csharp
public class CommandMethods<T>
    where T : struct
{

}
```

- 引用类型约束：所指定的参数类型必须为引用类型
    - 一条约束条文中不可以出现多个
    - 出现的位置必须要出现在最前面
    - 不可以和值类型约束一起使用
    - 不可以和基类约束一起使用

```csharp
public class CommandMethods<T>
    where T : class
{

}
```

<br/>

### 泛型逆变和协变

---

#### 什么是逆变和协变

逆变和协变的概念由 <kbd>.NET Framework 4.0</kbd> 以后针对于泛型所产生的两种概念，在解释它们之前我们先说一个另外一种概念，假设我们现在有两种类型 <kbd>T</kbd> & <kbd>u</kbd>，咱们先不谈这两个类型之间的关系，假设它们之间存在着一种类型安全的隐式转换，并且能够把这一特性带到对于它们的成员 ([[数组]]) 的身上去，我们就可以把这种能力称为 [[可变性 Variance ]] ，我们在此基础上进一步的进行扩展，假设 [[T]] 为 [[U]] 的基类，基于 [[里氏转换原则]] 我们把 U 赋值给 T，那么我们可以称之为这一段关系为 [[协变 covariant]]，反之，如果只是 [[存粹地]] 把 T 赋值给了 U，并且是类型安全的转换，那么我们可以称这一段关系为 [[逆变 contravariant]]

!!!
    简而言之，如果一个可变性和子类到父类转换的方向一样，就称作[[协变]]，而如果和子类到父类的转换方向相反，就叫 [[逆变]]
    
!!! danger 注意
    需要注意的是，协变和逆变的概念在 [[.NET]] 中仅仅是针对 [[泛型]] 所出现的，并且只是针对 [[泛型接口]] 和 [[泛型委托]]，为什么说仅仅只是针对泛型所出现的呢？因为逆变的概念涉及到父类直接转换为子类的应用，在正常的 [[csharp]] 语言的角度考虑是不允许的，由于 [[里式原则]] 的存在，基类想转换为一个有效的派生类必须要基类是已经经过派生类赋值的基础上才能够进行

#### 逆变和协变的具体实现

在 .NET 中，逆变和协变的存在通常是用来指定泛型成员之间互相进行安全的类型转换的问题的，并且仅仅只是针对 [[泛型接口]] 和 [[泛型委托]]，在这里我们使用 [[泛型接口]] 来举例子，查看以下代码，假设有一个接口 `Interface<T>`，分别对它进行了两种参数类型的指定 `Interface<string>` 和 `Interface<object>`，

```csharp
namespace Contravariant_and_Covariant
{
    class Program
    {
        static void Main(string[] args)
        {
            IPerson<string> personStr = new Student<string>();

            IPerson<object> personObj = new Student<object>();

            personObj = personStr;

            Console.ReadKey(true);
        }
    }

    public interface IPerson<T>
    {
        void Test(T);
    }

    public class Student<T> : IPerson<T>
    {
        public void Test(T)
        {
            return default(T);
        }
    }
}
```

在这里有一个接口 `IPerson<T>` 并且有一个实现于它的成员 `Student<T>`，我们可以看到 `Student<T>` 分别指定了两种不同类型参数分别是 [[string]] 和 [[object]]，但是在下一行代码 [[personObj = personStr]] 会编译不通过，因为这就涉及到了泛型成员类型转换间的问题了，我们知道，[[string]] -> [[object]] 之间是一种 [[协变]] 的关系，那我们就要在相应的泛型成员中通过 [[out]] 关键字来强调这一种关系，反之 [[object]] -> [[string]] 属于一种逆变的关系，那我们就需要在相应的泛型成员中通过 [[in]] 关键字来强调着一种关系

!!!
    在一个支持 [[in & out]] 的泛型成员中使用了这两个关键字对类型参数进行修饰后，我们并不能单纯的称为这个泛型成员是支持 [[逆变 & 协变]] 的，更准确的说法应该为这个泛型成员中通过 [[in]] 所修饰的参数类型是支持 [[逆变]] 的，反之，通过 [[out]] 所修饰的参数类型是支持 [[协变]] 的

!!!
    虽然通过 [[in & out]] 关键字能够达到 [[逆变 & 协变]] 之间转换的目的，但是使用它们还存在的另外的一些限制
    - 使用 [[in]] 所修饰的类型参数只能用作于 输入参数
    - 使用 [[out]] 所修饰的类型参数只能用作与 输出参数

以下代码展示了 [[in]] 和 [[out]] 的具体使用，也展现了不同类型参数之间进行 [[协变]] 和 [[逆变]] 的关系

```csharp
namespace Contravariant_and_Covariant
{
    class Program
    {
        static void Main(string[] args)
        {
            IPerson<string,object> personStr = new Student<string, object>();

            IPerson<object,string> personObj = new Student<object, string>();

            personStr = personObj;

            Console.ReadKey(true);
        }
    }

    public interface IPerson<in TIn,out TOut>
    {
        TOut Test(TIn value);
    }

    public class Student<TIn,TOut> : IPerson<TIn, TOut>
    {
        public TOut Test(TIn value)
        {
            return default(TOut);
        }
    }
}
```

下面还有几点关于泛型逆变和协变需要注意的一些问题

!!!danger 注意
    - 仅有泛型接口和泛型委托支持对类型参数的可变性，泛型类或泛型方法是不支持的
    - 值类型不参与协变或反变，因为.NET泛型，每个值类型会生成专属的封闭构造类型，与引用类型版本不兼容
    - 在泛型成员中使用了 [[协变 & 逆变]] 后如果存在声明的属性时要注意，可读写的属性会将类型同时用于参数 [[输入]] 和返回值 [[输出]]，因此只有只读属性才允许使用 [[out]] 类型参数，只写属性能够使用 [[in]] 参数
