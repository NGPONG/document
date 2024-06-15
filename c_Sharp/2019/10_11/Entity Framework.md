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

![![Mygif.gif](httpsi.loli.net201910118M5s7hWAtwZ6ja9.gif)](https://i.loli.net/2019/10/18/7yxe5qAN4btvsX2.gif)

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

- `DbContext.Configuration.LazyLoadingEnabled`：是否允许延迟加载机制，该延迟加载机制仅针对实体中类型为 [导航属性](#关于导航属性) 的成员，也就是说当我们关闭了该选项后，实体的类型为导航属性成员的值就算我们是查询出来的但是也永远为 `Null` ，需要我们手动显示的为导航属性成员的值进行加载

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
        // 使用该方式让 EF 进行对数据的修改只能是在知道实体主键的情况下才能够适用
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

- `DbContext.Database.ExecuteSqlCommand(string sql, params object[] parameters)`：执行关于 新增 / 删除 / 修改 类型的 SQL 语句，并返回受影响的行数，支持参数化格式
```csharp

    using (MyTestModule db = new MyTestModule())
    {
        // init database
        SqlParameter[] parameters = 
        {
            new SqlParameter("@Name","NGPONG"),
            new SqlParameter("@Address","Shenzhen.LongHUa")
        }

        int count = db.Database.ExecuteSqlCommand("insert into UserInfo (Name,Address) values(@Name,@Address)",parameters);
    }
```

- `DbContext.Database.SqlQuery<T>(string sql, params object[] parameters)`：执行关于 查询 类型的 SQL 语句，支持参数化格式，所指定的泛型则为查询 SQL 所返回的结果集所对应的实体类型
```csharp

    using (MyTestModule db = new MyTestModule())
    {
        // init database
        SqlParameter[] parameters = 
        {
            new SqlParameter("@Name","NGPONG"),
            new SqlParameter("@Address","Shenzhen.LongHUa")
        }

        var userInfos = db.Database.SqlQuery<UserInfo>("select * from UserInfo where Name = @Name and Address = @Address)",parameters);
    }
```

##### 4.2 DbContext.Entry&#60;TEntity&#62;(TEntity entity)：
如果所指定类型 `TEntity` 的实体的实例 `entity` 已经存在于实体集的本地缓存当中并且 EF 已经对其状态进行了监听跟踪的工作，则该函数永远只会返回已经在实体集中缓存的实体实例的 `Entry` 类型

如果所添加的实体实例是还未缓存至本地实体集当中的话，该函数则会把该实体实例添加到本地缓存当中去，然后设置其跟踪状态为 `Detached` ，并返回其 `Enrty` 类型，而这个跟踪状态的含义就意味着这个实体实例的本身包括其内部的成员都会被认为是 脏 的，不会被 EF 所跟踪，造成的后果在修改的过程当中尤为明显，因为通过该方式所添加到缓存集的实体会被认为是 脏 的，所以就算我们对已经调用了该函数的实体进行某个成员的修改，该实体也不会被 EF 跟踪为修改的状态，只有我们手动的更改该函数所返回类型的 State 属性为修改状态的时候才会进行相应的修改SQL的执行，但是一旦通过这种方式的话，那么所有字段都会被更新上去 `包括我们不想修改的`，指定当前实体被 EF 所跟踪的状态才能够完成操作，如果这时候我们再去把
entry所添加的实体的状态跟踪为 Modified 的时候，那么所有字段（包括我们不想修改的）都会被一起更新上去，<span style = "color:red">所以在使用 EF 进行修改操作的时候要慎用该函数进行操作</span>

```csharp
    using (MyTestModule db = new MyTestModule())
    {
        // Add
        var customer_Add = new CustomerSet()
        {
            CustomerName = "EF DataTest",
            CustomerAddress = "Shenzhen.Longhua"
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 Detached
        db.CustomerSet.Attach(customer_Add);
        // 将已经添加到本地缓存及中的实体的状态手动调整为 Add
        db.Entry<CustomerSet>(customer_Add).State = EntityState.Added;
        db.SaveChanges();


        // Deleted
        var customer_Deleted = new CustomerSet()
        {
            CustomerId = 5
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 Detached
        db.CustomerSet.Attach(customer_Deleted);
        // 从本地缓存中删除刚刚通过 Attach 所添加的实体，并将其状态调整为 Deleted
        db.Entry<CustomerSet>((customer_Deleted).State = EntityState.Deleted;
        db.SaveChanges();
    }
```

##### 4.3 DbContext.SaveChanges()：

通过该函数能够把调用该函数之前，对实体的所有合法的操作都会生成相应的　SQL语句　并执行，最后都映射回数据库当中

需要注意的是，该函数在默认情况下会为我们开启事务的管控功能，并且其生成的SQL语句都会为我们参数化，防止SQL注入

```csharp
    using (MyTestModule db = new MyTestModule())
    {
        // Deleted
        var customer_Deleted = new CustomerSet()
        {
            CustomerId = 5
        };
        // 把实体添加至实体集的本地缓存当中去，并将其状态跟踪为 Detached
        db.CustomerSet.Attach(customer_Deleted);
        // 从本地缓存中删除刚刚通过 Attach 所添加的实体，并将其状态调整为 Deleted
        db.Entry<CustomerSet>((customer_Deleted).State = EntityState.Deleted;
        db.SaveChanges();
    }
```

<br/>

#### <span id = "关于导航属性">5. 关于导航属性</span>

导航属性并不是数据库中某个表的外键，但是他又和外键存在着密切的关系，主要原因是因为他的存在是由 .Net 所抽象出来的，去目的在于能够在使用 EF 的过程当中，能够通过某个实体成员的导航属性更加准确的访问到其外部所涵盖的实体关系，举个例子，顾客和订单存在着外键对应关系，一个顾客下面会有多个订单，一个订单只属于一个顾客，那么它们所对应的实体模型中，顾客实体就存在一个订单类型集合的导航属性，而订单实体中就存在一个顾客类型的导航属性

##### 5.1 导航属性的延迟加载

在使用 EF 开发的过程当中，我们可以手动的选择是否开启延迟加载的功能，但是这个延迟加载的功能仅仅只是针对导航属性的，而导航属性延迟加载的效果莫过于我们只有在访问该导航属性成员的时候才会去数据库中查询对应表的数据出来，需要注意的是，一个查询出来的实体集数据序列的本身是无法同步这一设置的，因为它们延迟加载的机制是由 `IQueryable` / `IQueryable<T>` 所决定的

当我们需要为导航属性开启延迟加载的话，必须要注意以下几点是否是正确实施的，只要有任意一点配置不正确则无法开启导航属性延迟加载的机制，简而言之就是实体集数据序列中导航属性类型的成员的值永远为 `Null`
- 实体类是由 `Public` 修饰，不能是封闭类，也就是说，不能带有 `Sealded` 修饰符
- 实体类型所声明的导航属性必须使用 `virtual` 关键字进行标注
- 必须已经打开了EF全局配置中延迟加载的特性

##### 5.2 导航属性的贪婪加载

通过在查询某个实体集的过程当中，通过该实体集的 `Include(string path)` 函数来把导航属性的内容也一起贪婪加载出来
```csharp
    using (MyTestModule db = new MyTestModule())
    {
        // Products为Order实体的导航属性，通过Include表示在查询过程当中把所Include的成员也一起查询出来
        var query = from o in db.Order.Include("Products")
                             where o.OrderId > 2
                             select o;

        foreach (var item in query)
        {
            foreach (var p in item.Products)
            {
                
            }
        }
    }
```

##### 5.3 导航属性的显示加载

这种模式其实和默认的延迟加载的机制是非常类似的，具体做法则为通过所查询出来数据序列中类型为导航属性的实体实例的 `Entry` 类型的 `Reference` (针对单个) / `Collection` (针对集合)，进行对导航属性的显示加载，这种做法和默认的延迟加载的机制区别就在于，我们可以手动的去管控这个导航属性加载的时机

```csharp
    using (MyTestModule db = new MyTestModule())
    {
        // Collection
        var query = from o in db.Order
                    where o.OrderId > 2
                    select o;

        foreach (var item in query)
        {
            // 显示加载导航属性
            db.Entry<Order>(item).Collection(o => o.Products).Load();

            foreach (var product in item.Products)
            {
                
            }
        }


        // Reference
        var query = from p in db.Product
                    where p.ProductId > 2
                    select p;

        foreach (var item in query)
        {
            // 显示加载导航属性
            db.Entry<Product>(item).Reference(p => p.Order).Load();

            Order order = item.Order;
        }
    }
```

<br/>

#### 6. 在 Entity Framework 中使用事务

EF 中使用事务主要有三种方式

##### 6.1 DbContext.SaveChanges() 默认的为我们的 `CRUD` 进行时事务的管控
```csharp
    using (EFDemo2Entities db = new EFDemo2Entities())
    {
        // 该事务管理SavaChanges()前所有的修改

        Order order = new Order()
        {
            OrderName = "NGPONG Test",
            CreateDate = DateTime.Now
        };
        db.Order.Add(order);

        Product product = new Product()
        {
            ProductName = "NGPONG TrantestPro",
            OrderId = 100 // 模拟错误，OrderId为外键，输入一个不存在的OrderId
        };
        db.Product.Add(product);

        // SaveChanges默认的会为我们进行事务的管控
        db.SaveChanges();
    }
```

##### 6.2 DbContext.Database 中存在几组函数能够让我们更自由的对事务进行控制
```csharp
    using (EFDemo2Entities db = new EFDemo2Entities())
    {
        // Begin Trans
        using (var dbTrans = db.Database.BeginTransaction())
        {
            // 该实现方式可以手动管理事务，对比单纯的SaveChanges()灵活，并且能够管理在一个上下文档中多次SavaeChanges()的回滚
            try
            {
                // 前两次通过SaveChanges()是已经成功执行的，但是第三次通过SQL语句进行插入数据的时候失败了，所以前两次的成功执行也一起回滚了
                Order order = new Order()
                {
                    OrderName = "NGPONG Test",
                    CreateDate = DateTime.Now
                };
                db.Order.Add(order);
                db.SaveChanges();

                Product product = new Product()
                {
                    ProductName = "NGPONG Test Pro",
                    Order = order
                };
                db.Product.Add(product);
                db.SaveChanges();

                // 故意制造失败的情况，OrderId字段有着非空约束
                db.Database.ExecuteSqlCommand(
                    "insert into Product (ProductName) values(@ProductName)",
                    new SqlParameter[] { new SqlParameter("@ProductName", "NGPONG ProductTest") });

                // Commit Trans
                dbTrans.Commit();
            }
            catch (Exception objException)
            {
                // Rollback Trans
                dbTrans.Rollback();
            }
        }
    }
```

##### 6.3 通过 `TransactionScope` 这一类型进行事务的控制操作，其用法类似于 `6.2` 节点所提供的方案
```csharp
    using (EFDemo2Entities db = new EFDemo2Entities())
    {
        // Begin Trans
        using (TransactionScope dbTrans = new TransactionScope())
        {
            try
            {
                // 前两次通过SaveChanges()是已经成功执行的，但是第三次通过SQL语句进行插入数据的时候失败了，所以前两次的成功执行也一起回滚了
                Order order = new Order()
                {
                    OrderName = "NGPONG Test",
                    CreateDate = DateTime.Now
                };
                db.Order.Add(order);
                db.SaveChanges();

                Product product = new Product()
                {
                    ProductName = "NGPONG Test Pro",
                    Order = order
                };
                db.Product.Add(product);
                db.SaveChanges();

                // 故意制造失败的情况，OrderId字段有着非空约束
                db.Database.ExecuteSqlCommand(
                    "insert into Product (ProductName) values(@ProductName)",
                    new SqlParameter[] { new SqlParameter("@ProductName", "NGPONG ProductTest") });

                // Commit Trans
                dbTrans.Complete();
            }
            catch (Exception objException)
            {
                // Rollback Trans
                Transaction.Current.Rollback();
            }
        }
    }
```

<br/>

#### 7. DbContext 实例创建所需要注意的问题

如果EF的实例在同一个上下文被创建了多次是存在脏读的问题的，现在我们假设一个环境，函数A和B，函数B被函数A所调用，函数A中创建了一个EF的实例，并使用它对数据进行了一些比如说在一个函数A中我们创建了一个 `DbContext` 实例，并使用它对 `UserInfo` 表进行了一些查询操作，这时候调用了函数B，而函数B有创建了一个新的 `DbContext` 实例，并使用它对 `UserInfo` 表进行了删除的操作，这时候因为函数A中 `DbContext` 的实例和函数B中的实例是不同的，我们无法让EF去追踪一个最新的结果，函数A中的 `DbContext` 对于 `UserInfo` 表的追踪操作这时候是不同步函数B中 `DbContext` 的实例的，简而言之就是发生了 `脏读` 的情况

如果保证 DbContext 的实例在整个 `AppDomain` 中全局只有一个，即 `单例模式` ，虽然看似是解决了第一种情况所存在的 脏读 问题，但是随之而来却又诞生了第二种问题，因为所有线程都是使用的一个实例，假设在这里同样是两个线程 `ThreadA` 和 `ThreadB`，ThreadA 对 `UserInfo` 表中的数据进行了一些修改但是还未调用 `SaveChanges()` 同步到数据库，也许因为某些原因它还不想那么快进行同步，但是 `ThreadB` 也对 UserInfo 的数据进行了修改并且完成了执行 SavaChanges() 的工作，这时候 ThreadA 中还不想那么快同步到数据库中的数据也一并同步上去了，此外，这一情景只是针对两个线程，我们尝试把这个问题的角度转换到 `WebApplication` 身上，因为一个用户的请求都会从线程池中抽取一个线程进行逻辑处理，而在这个过程当中因为一直使用的是一个 `DbContext` 实例面向多个线程的操作，实体集的本地缓存的占用也会不断的进行增加，简而言之内存的损耗也会因为 `DbContext` 的实例无法即时释放，随之时间的推移，不断的累加

为了解决上述的两种问题，可以沿用 `HttpContext` 的 `线程内唯一对象` 的思想去处理

但还有一个问题需要注意，线程内唯一对象还是会存在一定的脏读问题，拟定在一个上下文中有两个线程 `ThreadA` 和 `ThreadB`，因为是不同线程，并且 `DbContext` 仅仅只是一个 `线程内唯一对象` ，所以它们分别创建了不同的 `DbContext` 对象去对数据库进行操作，ThreadA读取了 `UserInfo` 表的数据并尝试对他进行修改， ThreadB 也进行了相同的操作并且先于 ThreadA `SavaChanges()` 完成，即所修改的数据已经同步到数据库中的 UserInfo 表当中，这时候 ThreadA 所创建的 DbContext 实例 在这个过程当中是无法实时同步到 ThreadB 针对 UserInfo 所操作的数据的，那么在这一情景下， ThreadA 就会发生 `脏读` 的情况，诚然，这种方式是线程的不同步所造成的，这种情况我们应该针对其他方案去施以控制

##### 7.1 HttpContext.Items
因为 HttpContext 本身就是一个 `线程内唯一对象` ，而其提供的 `Items` 成员又能够用于存储只是 <span style="color:red">应用于当前上下文</span> 所需要存储的一些数据，那么我们就可以把 `DbContext` 交由它来进行处理，关于该属性的具体使用其实在其他篇章已经讲过，详情可以参考 [这里](https://github.com/NGPONG/document/blob/master/ASP.NET/ASP.NET%E5%9F%BA%E7%A1%80/HttpContext.docx) 

##### 7.2 CallContext
CallContext 也是能够提供 `线程内唯一对象` 的作用，下面代码的示例只是为了演示 CallContext 是否能正确的提供给每个逻辑执行线程唯一的数据槽这一项功能，具体的更多细节可以参考这一篇文章 [CallContext](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.remoting.messaging.callcontext?view=netframework-4.8) 

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            CallContext.SetData("Person", new Person() { Name = "NGPONG" });
            CallContext.LogicalSetData("Person_Logic", new Person() { Name = "NGPONG_Logic" });

            Task.Run(()=> 
            {
                var per = CallContext.GetData("Person");
                var per_Logic = CallContext.LogicalGetData("Person_Logic");
            });

            Console.ReadLine();
        }
    }

    public class Person
    {
        public string Name { get; set; }
    }
```
