# *Generic*


<br/>

### 前言

---

#### 泛型的介绍

泛型是所存储或使用的一个或多个类型具有占位符 （ [[类型形参]] ） 意义的类、结构、接口和方法，通过这个类型参数可以提供具体 [[参数化 ]]的类型，事实上，泛型类型的类型参数变成了泛型类型的元数据，运行时在需要的时候会利用它们去构造合适且符合的类型，通过这些类型，我们有可以实例化不同类型的对象，也就是说

> 未绑定泛型类型是来构造泛型类型的蓝图，已构造泛型类型又是实际对象的蓝图

#### 使用泛型所带来的好处

- 泛型将类型安全的负担从开发人员手里转移到编译器，开发人员没有必要编写代码来测试正确的数据类型，因为它会在编译时强制执行，这一定程度上降低了强制类型转换的必要性和运行时错误的可能性
- 使用泛型能够有效地避免对值类型进行装箱和拆箱时所造成的性能损失
- 如果使用的恰当，泛型能够很大的程度上避免代码的重用


<br/>

### 先来讨论下泛型

---

#### .NET 是如何实现泛型的？

泛型的开始，由我们在 IDE 中输入一个 `<T>` 开始，当编译器在编译的过程中遇到我们所写的泛型的时候，会针对该语法，并采用一种特殊的占位符来表示这种类型参数 <kbd>`x</kbd>，在这里 [[x]] 表示为 [[元数]]，其代表着这个泛型类型所具有的类型参数的数量，如下面代码所示，泛型类中一共包含了3个类型参数，那么最终该行代码，或者说这个泛型类被编译成 [[IL]] 后其占位符所存在的元数则为 3

```csharp
public class Person<T,T2,T3,T4,T5>
{
}
```

回到正题，在使用了泛型后，其编译的代码都带有一个特殊的标识符，那么当引用了这个 [[dll]] 的 [[Appdomain]] 在实际运行的过程当中，[[CLR]] 会进行一次二次编译的工作，在二次编译的过程中，[[JIT]] 就会找到 [[IL]] 代码中存在特殊占位符的代码，并把我们所指定的相应的 [[封闭类型]] 填充至相应的占位符的位置，通过这样就可以形成运行时编译录入我们所指定的封闭类型的特点

!!!
    泛型是由 [[.NET Framework 2.0]] 才开始引入的新特性，从上面的说明也能够看出 [[Version 1.0]] ~ [[Version 2.0]] 之间也一并更新了 编译器 和 [[CLR]] 的功能才能够独以支持该特性

#### 泛型类中静态成员的特殊性

因为 [[泛型]] 的引入，导致一个泛型类中的静态成员也发生了一些变化，在以往的认知中，一个类中的静态成员 [[CLR]] 会优先初始化静态变量和静态构造函数，且只有在第一次实例化的时候进行初始化，后续都不在进行初始化，但是在泛型类中，静态成员变量在相同封闭类型间共享，不同的封闭类型间不共享，简而言之就是这个泛型类所采用的不同类型的封闭类型都会针对它去生成一份 [[副本]]，这也导致了一种肉眼可见的一种结果，同一个类里面的静态成员会出现多次初始化的情况，详情可以查看下面的代码

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

比如说我们现在有一个函数包含了一个泛型类型参数 `<T>`，如果在该函数的 [[Signture]] 中同样也存在类型名为 `T` 的参数，那么该函数的参数的类型同样也沿用调用该函数时所设定的具体封闭类型，如下代码

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

使用一个泛型成员，我们需要在该成员名结束的地方以 `<T>` 作为标记，以表明该成员需要使用一个名字为 [[T]] 的类型参数，如果需要指定多个的话，则以 [[,]] 为标记，例如：[[<T1,T2,T3,T4,T5,……>]]

!!! danger
    一旦声明了一个泛型参数类型给某一个泛型成员，那么在使用它的过程中必须要指定它具体的封闭类型，当然我们也可以隐式进行指定，什么是隐式进行指定？例如一个泛型方法中标明了一个泛型参数类型 `<T>`，并且其带有的参数也是泛型参数 `T` 类型，这时候我们在调用这个函数的时候就不需要另外再指定它泛型的参数类型具体是某个封闭类型了，直接在输入参数中录入相应的实例即可自动识别，如下面代码

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

我们大致可以把泛型的应用分为四种类型，它们分别是

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