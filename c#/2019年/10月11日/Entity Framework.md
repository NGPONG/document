# *Entity Framework*

<br/>

### 前言

---

#### 1. 关于 Entity Framework 的简单介绍

Entity Framework 是微软以 `ADO.NET` 为基础所发展出来的对象关系映射 `O/R Mapping` 解决方案，早期该框架被称为 `ObjectSpace` ，属于 `.NET Framework` 的一部分，但 `Version 6` 之后就从  `.NET Framework` 分离出来，演化为今天的 `Entity Framework`

Entity Framework利用抽象化数据结构的方式，将每个数据库对象都转换成应用程序对象( `Entity` )，而数据字段都转换为属性( `Property` )，关系则转换为结合属性( `Association` )，让数据库的 `E/R`（实体-联系图）模型完全的转成对象模型，让程序设计师能用最熟悉的编程语言来调用访问

<div align=center>
<img src = "https://i.loli.net/2019/10/11/EGRxzbAUycviM78.png" />
</div>

<div align=center>
关于 Entity Framework 与 ADO.NET 的关系
</div>

#### 2. ORM 的简单介绍

对象-关系映射 ( `Object Relational Mapping`) 简称ORM，它并不是任何一个产品，而是一套框架的统称，更具体来讲的话它是一种思想

广义上， ORM 指的是面向对象的对象模型和关系型数据库的数据结构之间的相互转换，狭义上， ORM 可以被认为是，基于关系型数据库的数据存储，实现一个虚拟的面向对象的数据访问接口，用来把对象模型表示的对象映射到基于SQL的关系模型数据库结构中去，理想情况下，基于这样一个面向对象的接口，持久化一个OO对象应该不需要了解任何关系型数据库存储数据的实现细节

<br/>

<div align=center>
<img src = "https://i.loli.net/2019/10/11/K8CMkq5Ldyc6auH.png" />
</div>

<div align=center>
ORM 的实体模型
</div>

#### 3. Entity Framework 是如何实现 ORM 的

Entity Framework 采用元数据来描述对象一关系映射细节，元数据采用XML格式所展现出来，并且存放在我们所新建的实体数据模型文件 `*.edmx` 当中，只要提供了持久化类与表的映射关系，ORM框架在运行时就能参照映射文件的信息，把对象持久化到数据库中

![Mygif.gif](https://i.loli.net/2019/10/11/8M5s7hWAtwZ6ja9.gif)

由上图可知，Entity Framework 主要由以下三种模型和具有相应文件扩展构建成的一个 ORM 对象关系模型的框架：
- `SSDL` 概念层：负责向上的对象与属性显露与访问
- `CSDL` 储存层：依不同数据库与数据结构，而显露出实体的数据结构体，和 `Provider` 一起，负责实际对数据库的访问的工作
- `C-S` 映射层：将 概念层 和 储存层 的数据结构关系映射在一起

<div align=center>
<img src = "https://i.loli.net/2019/10/11/7oVYtLsnzPWFcSu.png" />
</div>

#### 4. 关于实体数据模型文件的介绍

实体数据模型文件是针对于 `DataBase First` / `Module First` 这两种开发模式的，它是 `Entity Framework` 框架在 `Visual Studio` 中的一个集成，其中不单单只是提供给了开发人员一个可视化操作的 `Module` 编辑器，所对应的实体类、EF的核心还有 `T4模板` 都被集成在了这个 `*.edmx` 文件当中



![微信截图_20191012212334.png](https://i.loli.net/2019/10/12/4DnTIculd7fH2tv.png)

#### 5. Entity Framework 的核心构成
`System.Data.Entity.DbContext` 是 Entity Framework 的核心，它是 EF 与数据库之间进行通讯的桥梁，在上面的章节提到过，Entity Framework 是微软以 `ADO.NET` 为基础所发展出来的对象关系映射解决方案， 那么可想而知，`System.Data.Entity.DbContext` 里面封装的就是基于 `ADO.NET` 所实现的，用于操作数据库的功能

而在实际开发的过程当中，不管使用哪种 [开发模式](#四种开发模式) 也好，`Visual Studio` 都会为我们生成一个继承于 `System.Data.Entity.DbContext` 的直接映射着数据库上下文的一个类，我们估计可以称它为 `实体操作模型` 

```csharp
    public partial class book_shop3Entities : DbContext
    {
        // 在构造函数中通过传入一个 SQL连接字符串名 给予其父类的构造函数的实参以完成对数据库的初始化操作
        public book_shop3Entities()
            : base("name=book_shop3Entities")
        {
        }

        // 实体创建时的 Event Handler，初始化的过程当中我们可以在此 Event Handler 声明对 EF 的一些全局性的初始化设置
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            throw new UnintentionalCodeFirstException();
        }
    
        // Table
        public virtual DbSet<ActionGroup> ActionGroup { get; set; }
        public virtual DbSet<ActionInfo> ActionInfo { get; set; }
        public virtual DbSet<Articel_Words> Articel_Words { get; set; }
        public virtual DbSet<BookComment> BookComment { get; set; }
        public virtual DbSet<Books> Books { get; set; }
    
        // Stored Procedure or Function
        public virtual int CreateOrder(string orderId, Nullable<int> userId, string address, ObjectParameter totalMoney)
        {
            var orderIdParameter = orderId != null ?
                new ObjectParameter("orderId", orderId) :
                new ObjectParameter("orderId", typeof(string));
    
            var userIdParameter = userId.HasValue ?
                new ObjectParameter("userId", userId) :
                new ObjectParameter("userId", typeof(int));
    
            var addressParameter = address != null ?
                new ObjectParameter("address", address) :
                new ObjectParameter("address", typeof(string));
    
            return ((IObjectContextAdapter)this).ObjectContext.ExecuteFunction("CreateOrder", orderIdParameter, userIdParameter, addressParameter, totalMoney);
        }
        public virtual int createOrderConfirm(string ordernumber, string address, Nullable<int> userId, ObjectParameter totalMoney)
        {
            var ordernumberParameter = ordernumber != null ?
                new ObjectParameter("ordernumber", ordernumber) :
                new ObjectParameter("ordernumber", typeof(string));
    
            var addressParameter = address != null ?
                new ObjectParameter("address", address) :
                new ObjectParameter("address", typeof(string));
    
            var userIdParameter = userId.HasValue ?
                new ObjectParameter("userId", userId) :
                new ObjectParameter("userId", typeof(int));
    
            return ((IObjectContextAdapter)this).ObjectContext.ExecuteFunction("createOrderConfirm", ordernumberParameter, addressParameter, userIdParameter, totalMoney);
        }
    }
```

上面的代码很充分的展示了 `实体操作模型` 所涵盖的内容，还有几点需要重点的强调一下
##### `5.1 实体操作模型的构造函数` 

实体操作模型 是 `System.Data.Entity.DbContext` 的派生类，而 `System.Data.Entity.DbContext` 的构造函数中的实参为DataBase上下文所需的 SQL连接字符串 的名字，相应的在 `实体操作模型` 的构造函数中可以通过上面的代码看到其显示的指定这个 SQL连接字符串 的值

##### `5.2 OnModelCreating` 

OnModelCreating 是 EF 在第一次访问数据库时所执行的 `Event Handler`，通常我们可以在这里设置本次会话中，关于 `DbContext` 的一些全局性的设置

##### `5.3 DbSet` 

DbSet 是一个实体的集合，是数据库中的某个表所涵盖的数据的一个映射，而其所对应泛型则为 `实体`，它是前面所提到的某个表的 `结构`，既然它是一个 `实体` 集合，所以 `DbSet` 还有另外一个名称 `实体集`，关于它的更多讯息请查看 [该节点](#DbSet) 的内容

##### `5.4 存储过程和函数`

存储过程和函数的关系也同样可以映射到 EF 当中，而我们通常把它们都映射成一个 c# 中的方法以便我们调用，需要注意的是，它们同样也是放在关于数据库上下文的 `实体操作模型` 中，充当为它的成员

`DbContext` 其本身同样是提供了一些成员针对于数据库的操作，具体内容请查看这一 [章节](#DbContext) 

<br/><br/>

### Entity Framework 的具体使用

---

#### 1. <span id = "四种开发模式">关于 Entity Framework 的四种开发模式</span>
- `DataBase First`：该模式从数据库作为出发点，先从数据库中设计好表的结构和其所对应的业务逻辑，然后再通过该模式把数据库中已经创建好的结构映射成 `Visual Studio` 的实体数据模型文件 `*.edmx`

![Mygif.gif](https://i.loli.net/2019/10/12/jbInYs1v8QaM3VH.gif)

- `Module First`：该模式从 `Visual Studio` 作为出发点，其刚好是跟 `DataBase First` 模式是相反的，先在实体数据模型文件 `*.edmx` 中通过可视化模型编辑器设计好模型和其所对应的业务逻辑，再通过该模式把所建立好的结构映射到数据库当中去

![Mygif.gif](https://i.loli.net/2019/10/12/y3DIzlaYo1jQ2Jx.gif)

- `Code First`：该模式脱离了传统的 `Visual Studio` 所提供的可视化模型编辑器进行模型编辑还有使用T4模板进行自动的生成模型类的特定，该模式从一个类为出发点，一切从代码中开始，并且也通过代码把我们所设计好的模型映射到数据库中去

![Mygif.gif](https://i.loli.net/2019/10/12/ckFQ3iTuPMpIBoG.gif)

- `DataBase CodeFirst`：该模式和 `Code First` 是一个概念，一切都是通过代码来完成，但是该模式的出发点转换成了数据库，在数据库中设计好相应的表和业务逻辑最后在通过该模式把数据库中的架构映射到代码当中去

![Mygif.gif](https://i.loli.net/2019/10/12/LNnPofOWQsBclCp.gif)

<br/>

#### 2. Entity Framework 的基础性配置
##### 2.1 `DbModelBuilder.Conventions.Remove<PluralizingTableNameConvention>()`：

在使用 `Code First` 模式进行开发的过程当中，如果我们想把自己添加的实体集映射成数据库对应的表的话，默认情况下 EF 都会为所映射的表添加一个复数形式的表明，也就是在表明的后缀添加一个 `s` ，通过在 `OnModelCreating` 这个 `Event Handler` 中使用此函数能够避免该情况的发生，关于更多信息可以参考 [这里](https://stackoverflow.com/questions/8222943/ef-code-first-pluralizingtablenameconvention-for-one-dbset)
```csharp
    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // 创建表的时候，表的后缀名不添加负数形式
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
```

##### 2.2 `DbContext.Configuration`
该属性用于设置当前会话关于 DbContext 的一些配置
- `DbContext.Configuration.ValidateOnSaveEnabled`：取消EF对于所添加/修改的实体完整性的检查，解决 <span style = "color:red">DbEntityValidationException 对一个或多个实体验证失败</span> 抛出的问题
```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 取消EF对于实体完整性的检查
        db.Configuration.ValidateOnSaveEnabled = false;

        CustomerSet customer = new CustomerSet()
        {
            CustomerId = 9
        };
        db.CustomerSet.Attach(customer);
        customer.CustomerName = "Update CustomerName";

        db.SaveChanges();
    }
```

- `DbContext.Configuration.LazyLoadingEnabled`：是否允许延迟加载机制，该延迟加载机制仅针对实体中类型为导航属性的成员，也就是说当我们关闭了该选项后，实体的类型为导航属性成员的值就算我们是查询出来的但是也永远为 `Null` ，需要我们手动显示的为导航属性成员的值进行加载

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 关闭延迟加载
        db.Configuration.LazyLoadingEnabled = false;
        
        var query = from c in db.CustomerSet
                    where c.CustomerId > 10
                    select c;

        // 当关闭了延迟加载后，实体的类型为导航属性成员的值永远为Null，需要我们显式进行加载
        foreach(var item in query)
        {
            var orders = item.Orders;
        }
    }
```

<br/>

#### 3. <span id = "DbSet">再谈 Entity Framework 核心中的 DbSet</span>

前面说到，DbSet 是实体的集合，我们也可以称它为 `实体集` ，区别于其他类型的集合，它是一个特殊的、高效的集合

实体集映射着数据库中具体的某个表，但这只是说明它和具体的某个表之间存在一定得联系，不代表 EF 在加载的过程当中，EF 会把其所对应的表中的数据全部取出来并放在这个实体集里面，他的存在更类似于一个本地缓存数据集的作用，举个例子，当我们在数据库中针对该实体集查出来了两行数据，然后这两行数据就会保存在该实体集的 `Local` 属性当中充当一个缓存的作用，在我们下一次又想针对那两行数据进行查询的时候我们就可以直接通过该实体集的 `Find()` 函数直接取出来，已提升一定的效能

实体集除了能提供一个缓存数据库中所对应实体的数据集之外，还为当前所缓存的数据集都提供一个状态跟踪的功能，跟踪所当前实体集中所缓存的具体某个实体是否存在 `Add`、`Update`、`Deleted` 的操作行为，其目的是为了在后面调用 `DbContext.SavaChanges()` 函数的时候能够更准确的为当前实体集中所对应的某个具体的实体所发生的变化生成相应的 SQL语句，最后再映射回数据库中去，下面将介绍关于实体集的一些用法

##### 3.1 根据实体集在数据库中查询该实体集所映射的表的数据，查询出来的有效数据都会放在本地缓存 `Local` 属性当中
需要注意的是，所查询出来的实体EF所给予的默认跟踪状态都是 `UnChanged`，这时候如果我们再对所查询出来的实体，换句话说已经缓存至实体集中的实体进行其他种类的操作的时候（修改实体的某个成员的值 / 将其移除 / ……），EF 都会时刻监听并更新实体状态为我们所对应的操作种类

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 查询出对应的实体并把它放在本地缓存的实体集当中
        var customer = from c in db.CustomerSet
                        where c.CustomerId > 2
                        select c

        int rowCount = db.CustomerSet.Local.Count;
    }
```

##### 3.2 添加一个实体到实体集中去，并将其状态跟踪为 `Add`

需要注意的问题是，使用 EF 进行添加数据的方式有很多，还有几种方法在下面还会继续介绍，但是不管使用的是什么方式进行数据的添加，<span style="color:red">如果当前实体所映射的表存在主键并且该主键属于标识规范的话</span>，EF 在添加数据成功后都自动的获取到当前所添加的行对应的主键的值，并将其赋值给当前所操作的实体类的所对应的主键的成员，其原理是因为EF所生成SQL，在插入数据后还会查询出当前查数据后的最后的主键的值

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {   
        CustomerSet customerSet = new CustomerSet()
        {
            CustomerName = "EF DataTest",
            CustomerAddress = "Shenzhen.Longhua"
        };

        // 把实体的实例添加到数据集的本地缓存中去，并将其的状态跟踪为 Add
        db.CustomerSet.Add(customer);
        db.SaveChanges();

        int Id = customerSet.CustomerSetId;
    }
```

##### 3.3 在实体集已经缓存的实体中移除一个实体，并将其状态跟踪为 `Deleted`

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 查询出对应的实体并把它放在本地缓存的实体集当中
        var customer = (from c in db.CustomerSet
                        where c.CustomerId == 2
                        select c).FirstOrDefault();

        // 根据查询出来的实体从本地缓存的实体集中删除，并同步至数据库的修改
        db.CustomerSet.Remove(customer);
        db.SaveChanges();
    }
```

##### 3.4 在实体集已经缓存的实体中拿出一个实体，修改其内部成员，并将其状态跟踪为 `Update`

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 查询出对应的实体并把它放在本地缓存的实体集当中
        var customer = (from c in db.CustomerSet
                        where c.CustomerId == 2
                        select c).FirstOrDefault();

        // 修改已经缓存的实体的成员，将其状态跟踪为 Update
        customer.CustomerName = "Update CustomerName"
        db.SaveChanges();
    }
```

##### 3.5 添加一个实体到实体集中去，并将其状态跟踪为 `UnChanged` 

从 `3.1-3.5` 中我们可以看到如何使用实体集去对数据库进行 CRUD 操作，但是它们的操作总归存在一个缺陷，那就是必须要在数据库中先查询一遍我们想操作的某个实体，然后再把它放在实体集的本地缓存 `Local` 属性当中，并对其施加一个跟踪状态，而某些情况下当我们已经知道了想要操作的实体所对应的唯一性标识的数据的时候，这时候我们就可以通过 `Attach()` 函数来避免还要查询以此数据库的情况了

通过实体集的 `Attach()` 函数能够让开发人员手动的添加一个实体至实体集的本地缓存中去，并将其状态跟踪为 `UnChanged`，是的没错，所跟踪的状态就和我们从我们从数据库中所查询出来的实体是一致的，只是 `Attach()` 在这中间少了以此访问数据库的过程

既然 `Attach()` 在操作完成后 EF 默认给予当前所操作的实体的状态为 `UnChanged` 的话，那就意味着这时候我们再对该实体进行一些不同种类的操作的话，EF 都会实时更新其状态为我们所操作的总类，这时候我们在通过 `SaveChanges()` 函数就能够把我们的操作映射至数据库的改变中去

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 取消EF对于实体完整性的检查
        db.Configuration.ValidateOnSaveEnabled = false;


        /* 通过 Attach() 函数进行 CUD操作 */

        // Add
        var customer_Add = new CustomerSet()
        {
            CustomerName = "EF DataTest",
            CustomerAddress = "Shenzhen.Longhua"
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 UnChanged
        db.CustomerSet.Attach(customer_Add);
        // 将已经添加到本地缓存及中的实体的状态手动调整为 Add
        db.Entry<CustomerSet>(customer_Add).State = EntityState.Added;
        db.SaveChanges();


        // Update
        var customer_Update = new CustomerSet()
        {
            CustomerId = 5
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 UnChanged
        db.CustomerSet.Attach(customer_Update);
        // 手动的修改该实体中某个成员的值，这时候EF就会将该实体的状态调整为 Updated
        customer_Update.CustomerName = "Attach Updated";
        db.SaveChanges();


        // Deleted
        var customer_Deleted = new CustomerSet()
        {
            CustomerId = 5
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 UnChanged
        db.CustomerSet.Attach(customer_Deleted);
        // 从本地缓存中删除刚刚通过 Attach 所添加的实体，并将其状态调整为 Deleted
        db.Remove(customer_Deleted);
        db.SaveChanges();
    }
```

##### 3.6 通过 `Local` 属性去获取实体集的本地缓存
实体集的 `local` 属性就是前面所多次提到的本地缓存，所查询出来的数据都会放在这里面作为一个本地缓存的作用，我们针对实体集的 `Add`、`Update`、`Remove` 操作都会直接映射到这个缓存视图的改变

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 取消EF对于实体完整性的检查
        db.Configuration.ValidateOnSaveEnabled = false;

        // 手动的在本地缓存中添加一个实体数据
        var customer_Add = new CustomerSet()
        {
            CustomerName = "EF DataTest",
            CustomerAddress = "Shenzhen.Longhua"
        };
        db.CustomerSet.Attach(customer_Add);

        // 从数据库中查询出对应的实体数据并自动的添加到实体集的本地缓存中去
        var customer_Search = (from c in db.CustomerSet
                               where c.CustomerId == 2
                               select c).FirstOrDefault();

        int localCachedCount = db.CustomerSet.Local.Count;
        if(localCachedCount == 2)
        {
            Console.WriteLine("Success!");
        }
    }
```

##### 3.7 `Find()` 函数优化 EF 的查询操作，减少访问数据库的次数
实体集所内置的 `Find()` 函数并不是IQueryable所扩展，该函数的查询会优先从这个缓存集中获取数据，如果获取不到才会生成相应的 `SQL` 再去数据库中查询

需要注意的是该函数存在一个缺陷，其所纳入的查询条件只能针对实体集的主键进行查询

```csharp
    using (EFDemoEntities db = new EFDemoEntities())
    {
        // 从实体集的本地缓存中查找主键ID为3的实体数据，如果不存在则从数据库中查询出来，并再次放入实体集的本地缓存当中去，以便下一次Find的时候不用再进行数据库的读取操作
        var query = db.CustomerSet.Find(3);
    }
```

<br/>

#### 4. <span id = "DbContext">再谈 Entity Framework 的核心 `DbContext`</span>

关于 `DbContext` 是什么，还有它和 `实体操作模型` 之间的关系在上面已经讲过，这里不再叙述，本章节主要针对于它所内置的成员进行讲解

##### 4.1 DbContext.Database
该成员主要提供了一些针对于数据库的一些操作，例如创建数据库、删除数据库等等

- `DbContext.Database.CreateIfNotExists()`：判断当前 `DbContext` 数据库上下文所指定 SQL连接字符串 的数据库是否存在，否则将创建一个数据库
```csharp
    using (MyTestModule db = new MyTestModule())
    {
        // init database
        db.Database.CreateIfNotExists();
    }
```

- 