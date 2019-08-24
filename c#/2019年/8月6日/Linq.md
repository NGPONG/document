# *Language Intergated Query*

<br/>

### 前言

---

#### 1. 关于Linq的简单介绍
&nbsp;&nbsp;&nbsp;1.1 Linq 是一系列直接将查询功能集成到`C#`语言的技术统称，换句话说它是一种统一的数据查询接口，其系列技术提供了一种跨各种数据源和数据格式 (实现自<span style="color:red">IEnumerable</span>或<span style="color:red">IEnumerable&lt;T&gt;</span>接口) 的查询体验，借助 Linq，查询成为了最高级的语言构造，就像类、方法和事件一样，可以使用语言关键字和熟悉的运算符针对强类型化对象集合编写查询

&nbsp;&nbsp;&nbsp;1.2 `Linq查询表达式`是统一的数据查询接口，通过提供了一种跨各种数据源和数据格式使用数据的一致模型，不管是查询`Linq to Object`也好还是 `Linq to Sql` 、 `Linq to Entities` 、 `LINQ to XML` 亦或是一些自定义的数据源，Linq对于数据检索的表达式是不变的，需要变更的只是数据查询提供程序

