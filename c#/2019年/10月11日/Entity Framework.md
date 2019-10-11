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

由上图可知，Entity Framework 主要由 `三层关系` 构建成的一个 ORM 对象关系模型的框架，其中：
- `SSDL` 概念层：负责向上的对象与属性显露与访问
- `CSDL` 储存层：依不同数据库与数据结构，而显露出实体的数据结构体，和 `Provider` 一起，负责实际对数据库的访问的工作
- `C-S` 映射层：将 概念层 和 储存层 的数据结构关系映射在一起


<br/>

### Entity Framework 的具体使用

---
