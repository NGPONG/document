# *<span>ASP.NET</span> MVC*


<br/>

### 目录

- [开始](#前言)
- [路由系统](#路由)
- [Controller & Action](#控制器和Action)
- [Area](#Area)
- [Filter](#Filter)
- [新的视图引擎 - Razor](#Razor)

<br/>

### 前言

<span id="前言"></span>

---

#### <span>ASP.NET</span> MVC 介绍

<span>ASP.NET MVC</span> 是微软2009对外公布的第一个开源的表示层框架，它是微软第一个开源的项目，关于其项目地址请点击 [这里]("https://github.com/aspnet/AspNetWebStack")，目前已经更新至 `6.x` 版本，需要注意的是，MVC 并不一定就泛指 `ASP.NET MVC`，MVC 的开发模式是一种思想，展开来讲的话之前的 `WebForm` 我也能够归纳为 MVC 的一种范畴，我可以把单纯的 `*.aspx` 视为视图的一部分，而关于 `*.aspx` 的后置文件 `*.aspx.cs` 我可以将其视为一种控制器，而控制器中所用到的实体类我也可以将其归纳为模型的一部分

MVC 是一种思想，而关于 `ASP.NET MVC` 这个框架有带给了开发者另一种思想 `约定大于配置`，具体来讲的话，我们但从 `View` 和 `Controller` 身上就能够看得出来，比如说作为 `Controller` 的类必须以 `Controller` 作为结尾，其次，控制器中的具体 `Action` 也要和视图的名字作为匹配，并且视图所在物理路劲的文件夹的位置也要和控制器的名字作匹配，这就是 `ASP.NET MVC` 这个框架所带给我们的约定，约定大于配置，微软约定好了需要这么干那么也需要遵从这一种指示去开发

#### <span>ASP.NET</span> MVC 的构成

MVC 拆解开来的话由三部分所组成，分别是
- Model：主要是存储或者是处理数据的组件
- View：用户接口层组件，主要是将 Model 中的数据展示给用户
- Controller：处理用户交互，从 Model 中获取数据并将数据传给指定的 View

![27.jpg](https://i.loli.net/2019/12/13/Kgc6V5hqlX8DYIB.jpg)

<br/>

### 路由

<span id="路由"></span>

---

#### 介绍

`ASP.NET MVC` 路由的实现依靠于 `System.Web.Routing`

路由是 `ASP.NET MVC` 的一个最为核心的部分，一切都由其作为根源，在这里我们需要区别于传统的 `WebForm` 形式的开发，`WebForm` 中，对 URL 的传入请求通常映射到服务器磁盘上真实的物理文件位置上

而对于 `ASP.NET MVC`，具体的一切请求路径都需要遵循我们所定义的路由路径，并且最后会根据我们所请求的 URL 映射至具体的某个 `Controller` 的 `Action` 身上，当然这句话是相对的，我们其实也可以根据文件路径直接访问 `ASP.NET MVC` 框架所编写的网站的具体某个网站，但是需要注意的是，这句话是不包含 `Views` 文件夹目录下的，因为 `Views` 文件夹目录下所存放的视图文件默认情况下都会给一个叫做 `BlockViewHandler` 的 `HttpHandler` 作为管控，如果所请求的路径包含 `Views` 文件夹下的内容则会弹出 `404` 的错误页，如果需要解决这一问题还需配置下这个 `BlockViewHandler` 的 `path` 仅针对 `*.cshtml` 文件即可

#### 路由路径的注册

`ASP.NET MVC` 对于我们所注册的路由都定义为一个 `Route` -> `RouteData` 对象，并且由一个 `RouteCollection` 的集合对他们进行暂存和管控，关于路由规则的注册就是往集合里面添加一个 `Route` 类型的元素，关于更多信息其实在 [另一篇](https://github.com/NGPONG/document/blob/master/ASP.NET/MVC/ASP.NET%20MVC%20Life%20Cycle.md) 中已经详细介绍过了，这里不再做阐述

在 `ASP.NET MVC` 这个框架项目默认创建出来后，在其 `App_Start` 文件夹下会自动创建一个 `RouteConfig` 的类，并且在 `Global.asax` 中的 `Application_Start` 函数中会自动为我们调用 `RouteConfig` 这个类中关于路由注册的函数实现，所以我们关注点只需要放在 `RouteConfig` 中关于路由注册的函数实现即可，其实从中也可以看出另一点，路由的注册必须要在我们的 `AppDomain` 在第一次启动后就要完成这些预先的设定

`RouteCollection.MapRoute( string routeName, string url, object defaults, object constraints )`：为 `AppDomain` 注册一个自定义路由规则
- routeName：指定该路由规则的名字，注意不可重复
- url：指定请求 URL 的结构识别规则
    - 指定的路由规则可以是静态字段或者是动态的字段，什么是静态/动态字段，以 `{xxx}` 两个括号所包含起来的则为静态字段，反之则为动态字段，例如：`MyRoute/{controller}/{action}`，则指定当前网站所请求的路径都必须包含MyRoute的前缀
    - 正常情况，该参数需要带上 `{controller}` `{action}` 这两种动态字段的，已告诉路由当前所请求的URL具体在哪个控制器，控制器中的哪个action，当然这里指的是一般情况下，`System.Web.Routing` 对于该参数中是否存在 `{controller}` `{action}` 不进行校验，也就是说我们指定该参数的时候也可以不用带有 `{controller}` `{action}`，但是在这个前提下必须要指定 `default` 参数中具体的 `Controller` 和 `Action` 否则连一个页面都打不开，为什么？因为 `ASP.NET MVC` 的请求是要具体映射到某个控制器的某个Action的，其实归根结底，URL 这个参数中还是需要带有 `{controller}` `{action}` 这两个动态字段的，不带上这两个参数的情况下能够正常访问页面的原因只是因为 `System.Web.Routing` 参考了我们在 `default` 参数中对于 `Controller` 和 `Action` 的设置罢了
    - 该参数中可以带上一些自定义的变量，如下格式：`{controller}/{action}/{myParameter}`，如果这么指定的话，我们还需要在 `default` 变量上指定 `myParameter = UrlParameter.Optional` ，这样就可以实现自定义变量的使用了，那么自定义变量是什么？好处是什么，举个例子现在我们指定的路由路径就是 `{controller}/{action}/{myParameter}`，这时一个从 Client 上发来的 URL 请求格式假设为 `localhost/My/MyAction/Value`，那么当具体的 `MyController` 类的 `MyAction` 函数中的 Signature 存在一个 `myParameter` 的参数的时候，就会自动的把 `Value` 这个值复制给 `MyController.MyAction` 中的 `myParameter` 这个参数了
    - 在该参数中还有一种写法为：`{*values}`，其寓意为捕获任何形式的URL字符，简而言之就是在这个动态参数的范围内不管输入的任何字符串的URL形式都是可以通过的，不至于弹出404页面，需要注意的是，<span style="color:red">该参数必须要在路由规则字符串中最后出现的位置</span>，例如：`{controller}/{action}/{*values}`
    - 这里稍微扩展一下，对于一个网站，<span style="color:red">为了SEO友好,一个网址的路由规则 URL 层次不要超过三层</span>
- defaults：路由路径中所编写的相应的动态参数的位置在 Client 所发来的请求 未录入 的情况下才会取 `defaults` 中的设定，除此之外还能够指定自定义变量的属性，例如当我们在路由路径中指定了自定义变量 `myParameter` 的话，如果需要它起作用，那么就需要为这个参数添加 `myParameter = UrlParameter.Optional`
- constraints：该参数主要功能是针对动态参数的具体规则，其接收参数体现在两个方面
    - 单纯的指定一个正则能够达到的效果就是所指定的动态参数在 Client 录入请求 URL 的时候相应位置必须要按照正则表达式进行匹配
    - 指定一个继承并实现与 `IRouteConstraint` 接口的实例，在其所实现的 `Match` 函数中进行校验，这里要扩展下，我们还能通过该参数来校验 HTTP 请求的方式，比如说 `ASP.NET MVC` 默认就提供了 `HttpMethodConstraint` 的实现来达到这一种效果
- namespaces：为重名的控制器注册命名空间，控制器的建立在一个 `ASP.NET MVC WebApplication` 是不能出现重复的情况的，比如说在一个 `Area` 中建立了一个于当前 `ASP.NET MVC` 主目录下重名的控制器，那么当访问这个控制器的 `Action` 的时候则会报错，这时候我们就可以通过设定该属性的值为重名控制器的命名空间则可以避免这个问题

来看下 `RouteConfig` 中关于路由注册的实现

```csharp
public class RouteConfig
{
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}",
            defaults: new { controller = "Home", action = "About" },
            constraints: new { httpMethod = new HttpMethodConstraint("GET")},
            namespaces: new string[] { "ASP.NET_MVC_Study_03.Controllers" }
        );
    }
}
```

#### 路由规则的匹配顺序

前面说到，路由规则都统一集成在 `RouteCollection` 这个集合里面去，我们可以指定一个或多个路由规则，当我们指定了多个规则后，路由对于所请求的 URL 匹配路由规则也有一个先后的顺序，我们说到 `RouteCollection` 是一个集合，那么我们也可以大胆的猜测路由规则的匹配优先度是从该集合中第一个元素开始匹配，如果第一个匹配不上开始取第二个元素路由规则进行匹配支持匹配成功为止，如果该集合遍历完成后都不存在能够成功匹配的 URL 格式，则会返回404的错误页

<br/>

### 控制器和Action

<span id="控制器和Action"></span>

---

#### 控制器的简单介绍

我们所新建的控制器都被统一放在当前 `WebApplication` 的 `Controllers`，并且所建立的控制器必须以 `Controller` 作为结尾

控制器的建立在一个 `ASP.NET MVC WebApplication` 是不能出现重复的情况的，比如说在一个 `Area` 中建立了一个于当前 `ASP.NET MVC` 主目录下重名的控制器，那么当访问这个控制器的 `Action` 的时候则会报错，这时候我们就可以通过 [前面](#路由) 所提到的，在注册路由的时候指定特定重名的控制器的 `namespaces` 则可以解决这个问题

下面开始介绍下 `Controller` 中的 `Action`

#### 指定 Action 的 Http Method

在 `Action` 中，如果我们不具体指定 `Action` 的 Http 方式，则允许以 `GET` `POST` 的方式对该 `Action` 进行调用，具体识别的过程交由 `ASP.NET MVC` 框架来完成，反之，如果我们想让该 `Action` 只能通过某种 Http，在这里 `ASP.NET MVC` 针对不同的 HttpMethod 的请求，为 `Action` 提供了相应的特性，查看以下代码

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult Get()
    {
        return Content("Hello,World!");
    }

    [HttpPost]
    public ActionResult Post()
    {
        return Content("Hello,World!");
    }

    [HttpPut]
    public ActionResult Put()
    {
        return Content("Hello,World!");
    }

    [HttpDelete]
    public ActionResult Delete()
    {
        return Content("Hello,World!");
    }

    [HttpPatch]
    public ActionResult Patch()
    {
        return Content("Hello,World!");
    }
}
```

#### Action 的重载

`ASP.NET MVC` 是不允许任意一个 `Controller` 中的 `Action` 存在重载关系的，当存在重载情况的时候，这个 `WebApplication` 的启动的过程中就会弹出 `操作方法之间不明确` 的黄页，虽然是不支持方法本身的重载，但是 `ASP.NET MVC` 却支持在引用了不同 HttpMethod 的情况下进行重载，如下代码所示

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult MyAction()
    {
        return Content("Hello,World!");
    }

    [HttpPost]
    public ActionResult MyAction()
    {
        return Content("Hello,World!");
    }
}
```

#### 自动的把请求参数录入至 Action 中对应的参数名的参数身上

前面说到，路由路径中我们是可以指定自定义变量的，当一个请求 `URL` 中自定义变量的位置复合当前所请求的 `Action` 的 Signature 的时候，则会把 URL 上对应自定义变量位置上的值录入到 `Action` 中相应的位置，除此之外，`Action` 本身对请求所附带的参数也支持这一特性，举个例子，一个请求中所附带的参数名如果和当前所请求的 `Action` 的 Signature 是一致的话，则会自动地把请求所附带的参数的值赋值到 `Action` 中对应的参数身上，并且这一特性同样也适用于 `Action` 的参数是一个实体的情况，比如说实体的属性和请求所附带的参数名是一致的话，同样会把请求参数的值复制到实体类的示例身上，但是这种情况其实只适用于 `HttpGet` 的 `Action`，如果一个 `Action` 被标识为 `HttpPost` 的情况下，则需要在 Post 请求头上附带上 `Content-Type=application/x-www-form-urlencoded` 才能够适用于这一种特性

#### ActionResult

`Action` 所返回的类型通常情况下需要返回一个实现于 `IActionResult` 的类型，当然并不是说一定需要返回这个，但是返回 `IActionResult` 能够更契合 `ASP.NET MVC` 这个框架的主体

`ASP.NET MVC` 为我们提供了以下几种实现于 `IActionResult` 的类型

- [ContentResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.contentresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [EmptyResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.emptyresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [FileResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.fileresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [HttpStatusCodeResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.httpstatuscoderesult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [JavaScriptResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.javascriptresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [JsonResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.jsonresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [RedirectResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.redirectresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [RedirectToRouteResul](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.redirecttorouteresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
- [ViewResultBase](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.viewresultbase?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
    - [PartialViewResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.partialviewresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)
    - [ViewResult](https://docs.microsoft.com/en-us/dotnet/api/system.web.mvc.viewresult?redirectedfrom=MSDN&view=aspnet-mvc-5.2)

在这里只是针对几种常用的 `IActionResult` 进行注解，需要事先说明的是，这几种实现于 `IActionResult` 类型在控制器的父类 `Controller` 中都封装了相应的函数去返回它们

<span id = "Controller.View">1.</span> `ViewResult：Controller.View`

该种类型的是控制器所对应视图的内容结果，需要注意的是，该函数提供了几种重载的结果，在什么参数都不录入的情况下就是直接返回一个对应视图的结果，进入视图渲染的工作环节，但是我们也可以指定一个任意类型的示例，为 `Razor` 的强类型视图注入我们所指定的类型的实例

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult DefaultAction()
    {
        return View();
    }

    [HttpGet]
    public ActionResult TypeAction()
    {
        return View(new Entity(){
            Name = "NGPONG",
            Age  = 22
        });
    }
}

public class Entity
{
    public string Name { get; set; }
    public string Age { get; set; }
}
```

<span>2.</span> `ContentResult：Controller.Content`

该种类型返回的是一种字符串格式的信息，其实就相当于之前的 `HttpContext.Response.Writer(value)`，将字符串写入 HTTP 响应体中返回给 Client，该函数提供了一种重载的格式可以让我们去指定返回 `Response` 的 `Content-type`

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult DefaultAction1()
    {
        return Content("Hello,World");
    }

    [HttpGet]
    public ActionResult DefaultAction2()
    {
        return Content("Hello,World","text/pain");
    }
}
```

<span>3.</span> `RedirectToRouteResul / RedirectResult：Controller.RedirectToAction / Controller.Redirect`

这里把它放在一起的目的是因为其功能性是一样的，也是重定向至另一个页面，其功能相当于 `HttpContext.Response.Redirect(url)`

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult RedirectToRouteResul()
    {
        return RedirectToAction("Home","Index");
    }

    [HttpGet]
    public ActionResult RedirectResult()
    {
        return Redirect("~/Home/Index");
    }

    [HttpGet]
    public ActionResult Index()
    {
        return View();
    }
}
```

<span>4.</span> `JsonResult：Controller.Json`

该种类型能够将返回的结果进行 `Json` 序列化 ( 采用 `ASP.NET MVC` 框架所提供的序列化器 )，并自动地为我们添加上 `Content-type = application/json` 的 `Response Header`

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult DefaultAction()
    {
        return Json(new Entity(){
            Name = "NGPONG",
            Age  = 22
        });
    }
}

public class Entity
{
    public string Name { get; set; }
    public string Age { get; set; }
}
```

#### ViewBag & ViewData

`ViewBag` 和 `ViewData` 其作用是一样的，只针对当前请求上下文去存放一些数据提供 `View` 去使用，虽然其作用是一样的，但是功能上本质还是存在一定的区别

`ViewBag`

- 动态类型，获取的时候不需要进行数据转换
- `ASP.NET MVC 3` 后才有的新功能
- `基于 .Net Framework 4.0` 的实现 
- 可读性更好
- ViewBag 是属于对 `ViewData` 的进一步的封装，就效率而言 ViewData 是高于 ViewBag 的

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult DefaultAction()
    {
        ViewBag.Title = "WuPeng";
        ViewBag.Description = "Hello,World";
        ViewBag.Age = 22;

        return View();
    }
}
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title></title>
</head>
<body>
    <h1>Hello, My name is @ViewBag.Title, I'm @ViewBag.Age Years old, @ViewBag.Description</h1>
</body>
</html>
```

`ViewData`

- 它是键为 `string` 值为 `Object` 的字典集合，故获取的时候需要进行数据类型的转换
- ViewData 带有的 Model 属性可以作为给强类型视图的 Model 的实例，是实现强类型视图获取 `Model` 实例的一种方式
- 从 `Asp.net MVC 1` 就开始支持了
- 基于 `.Net Framework 3.5` 的实现
- 由于 ViewBage 是基于 ViewData 所封装的属性，所以就效率而言，ViewData 比 ViewBag 高

```csharp
public class DefaultController : Controller
{
    [HttpGet]
    public ActionResult DefaultAction()
    {
        ViewBag[Title] = "WuPeng";
        ViewBag[Description] = "Hello,World";
        ViewBag[Age] = 22;

        return View();
    }
}
```

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title></title>
</head>
<body>
    <h1>Hello, My name is @ViewData["Title"]?.ToString(), I'm @Convert.ToInt32(ViewData[Age]).ToString() Years old, @ViewData[Description]?.ToString()</h1>
</body>
</html>
```

<br/>

### Area

<span id="Area"></span>

---

#### 介绍

随着业务的需要，结构需求会越来越多,`Views` 目录下面的文件夹越来越多，更或者你需要更细结构的页面路径，另外可能是多个人合作开发，有多个Web项目需要进行合并在一个网站中访问，在这时候 `Area` 就起到了很大的作用

`ASP.NET MVC Area` 可以将一个大型的网站划分为相对独立的区域，该区域同样享有 `Model` `Controller` `View` 的结构模式，并且也有着属于自己的一个 `web.config` 和 路由规则

#### 如何创建一个 Area

![Mygif.gif](https://i.loli.net/2019/12/15/ymDEZFlhu5djRcI.gif)

#### Area 中的路由

Area 种路由的注册通过在新建一个 Area 的时候帮我自动生成的 `*AreaRegistration.cs` 来完成，和一个 `ASP.NET MVC` 主项目一样，也是通过 `MapRoute` 函数来完成自己定义的路由规则的注册，需要注意的是，这里关于 `MapRoute` 函数的提供方是换了一个人的，在 `ASP.NET MVC` 上路由注册的由 `RouteCollection` 来提供，而在 `Area` 中却由 `AreaRegistrationContext` 来提供，虽然两者之间的 `Provider` 不同，但是在 `Area` 中路由注册的一些注意事项同样可以沿用在 [前面](#路由) 所说的信息

在 `Area` 中，为了解决可能存在某个控制器会跟 `ASP.NET MVC` 主项目中的控制器重名的情况，在录入路由路径注册的过程中，底层会为 `Area` 所注册的路由添加一个 `DataToken` 以来添加标识，而这个 `DataToken` 就是我们在 [注册路由](#路由) 时候所指定的 `namespaces`，关于更多信息可以查看这一篇 [文章](https://github.com/NGPONG/document/blob/master/ASP.NET/MVC/ASP.NET%20MVC%20Life%20Cycle.md)

下面的代码展示了 `Area` 中注册路由的方式

```csharp
public class MyAreaAreaRegistration : AreaRegistration 
{
    public override string AreaName 
    {
        get 
        {
            return "MyArea";
        }
    }

    public override void RegisterArea(AreaRegistrationContext context) 
    {
        context.MapRoute(
            "MyArea_default",
            "MyArea/{controller}/{action}/{id}",
            new { action = "Index", id = UrlParameter.Optional }
        );
    }
}
```

在这里也稍微扩展下，我们都知道在 `ASP.NET MVC` 上面所注册路由的函数在创建一个 `ASP.NET MVC` 项目的时候，都会自动帮我在 `Application_Start` 中进行调用，以完成路由的注册工作，但是在 `Area` 中的 `RegisterArea` 关于注册路由的函数却没有找到显示的调用，其实其调用是在 `Application_Start` 中为我们自动调用的 `AreaRegistration.RegisterAllAreas` 函数身上，关于其具体实现，可以参考这篇 [文章](https://github.com/NGPONG/document/blob/master/ASP.NET/MVC/ASP.NET%20MVC%20Life%20Cycle.md)

<br/>

### Filter

<span id="Filter"></span>

---

#### 介绍

Filter 是 `ASP.NET MVC` 这个框架对于 `AOP` 思想的体现之一，它的出现能够让我们可以更加自主的控制一个请求的流程

#### Filter 的使用

`ASP.NET MVC` 为我们提供了4种类型的 `Filter` 所对应的接口，每种接口都对应着不同动作时候需要执行的 `Filter`，参考以下表格

Filter Type|Interface|Description 
--------|--------------|------------
Authorization|IAuthorizationFilter|Runs first
Action|IActionFilter|Runs before and after the action method
Result|IResultFilter|Runs before and after the result is executed
Exception|IExceptionFilter|Runs if another filter or action method throws an exception

`Filter` 除了上面提到的这些具体过程的接口之外，还有一个重要的类型 `FilterAttribute`，我们所自定义的 `Filter` 除了需要实现上面所说到的那些接口外，还需要继承于这个类型，无口否认的是，`Filter` 的使用沿用了特性的思想，那么集成这个类的第一个目的也是能够把这个类声明称一个特性，除此之外，只有继承了 `FilterAttribute` 这个特性才能够 `ASP.NET MVC` 框架发现这个特性是属于 `Filter` 的特性

在完成了继承 `FilterAttribute` 并且实现自上面提到的任意一个接口后，只需在相应的 `Action` 的位置引用这个特性 (也可以在 `Controller` 身上使用，如果直接在 `Controller` 上使用则表明该 `Controler` 下所有的 `Action` 都使用这一种 `Filter`) 即可以完成 `Filter` 的工作，以下代码只是展示了一种 `Filter` 所对应的工作

```csharp
public class MyFilterAttribute : FilterAttribute, IActionFilter
{
    public void OnActionExecuted(ActionExecutedContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuted Filter");
    }

    public void OnActionExecuting(ActionExecutingContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuting Filter");
    }
}
```

```csharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }

    [MyFilter]
    public ActionResult About()
    {
        ViewBag.Message = "Your application description page."; 
        return View();
    }
}
```

<br/>

上面所说的直接在控制器或Action中使用一个继承于 `FilterAttribute` 并实现了 `Filter` 接口的特性只能够沿用当前控制器或Action上面，同样的，我们还可以把这个特性沿用到全局控制器下面，我们只需要在 `ASP.NET MVC` 项目在创建的过程中为我们自动创建的在 `App_Start` 目录下的 `FilterConfig` 类中的 `RegisterGlobalFilters` 中添加一个全局 `Filter` 即可，在默认生成完毕情况下，`RegisterGlobalFilters` 这个函数会在 `Application_Start` 进行了调用，所以我们的关注点也就只用放在 `RegisterGlobalFilters` 这个函数身上即可，以下代码展示了注册一个全局 `Filter` 的方式，该 `Filter` 将引用至当前 `WebApplication` 中的所有 `Controller` 和 `Action`，当然也包括 `Area` 中的所有 `Controller` 和 `Action`

```csharp
public class MyFilterAttribute : FilterAttribute, IActionFilter
{
    public void OnActionExecuted(ActionExecutedContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuted Filter");
    }

    public void OnActionExecuting(ActionExecutingContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuting Filter");
    }
}
```

```csharp
public class FilterConfig
{
    public static void RegisterGlobalFilters(GlobalFilterCollection filters)
    {
        filters.Add(new HandleErrorAttribute());

        filters.Add(new MyFilterAttribute());
    }
}
```

```csharp
public class MvcApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        // More....
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
    }
}
```

<br/>

除了能使用特性的方式来完成 `Filter` 的工作外，控制器默认所继承的 `Controller` 也提供了几个对于请求步骤截取的函数进行重写，重写这些函数也可以完成 `Filter` 的工作，需要注意的是，这种方式的 `Filter` 使用会沿用至当前控制器下所有的 `Action`，查看以下代码

```csharp
public class HomeController : Controller
{
    public ActionResult Index()
    {
        return View();
    }

    public ActionResult About()
    {
        ViewBag.Message = "Your application description page."; 
        return View();
    }

    public ActionResult Contact()
    {
        ViewBag.Message = "Your contact page.";

        return View();
    }

    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuting Filter");
        base.OnActionExecuting(filterContext);
    }

    protected override void OnActionExecuted(ActionExecutedContext filterContext)
    {
        filterContext.HttpContext.Response.Write("I'm OnActionExecuted Filter");
        base.OnActionExecuted(filterContext);

    }
}
```

<br/>

### Razor

<span id="Razor"></span>

---

#### 介绍

`Razor` 是 `ASP.NET MVC` 引入的一种新的视图引擎模板，区别于 `WebForm` 的是，Razor的语法简单且清晰，只需要最小化的输入，并且Razor可以 `HTML` 和 `C#` 代码混合进行使用

`Razor` 文件是以 `*.cshtml` 所结尾的，其去除了传统 `WebForm` 视图文件还有一个后置代码文件的规格，并将其分离至 `Controller` 当中，使项目的耦合度更低

#### Razor 的命名空间

和 `WebForm` 一样，一个 `ASP.NET MVC WebApplication` 在启动的过程当中，也会把当前 `WebApplication` 的视图文件编译成一个继承于 `WebViewPage<T>` 或 `ViewStartPage` 的类，并且把他们打包至 `DLL` 类放在系统的临时文件夹目录作为缓存文件缓存起来，但是区别于 `WebForm` 的是，`Razor` 所编译后的 `DLL` 具体来说由三个，分别是 
- `View` 目录下视图文件所生成的
![微信截图_20191212171523.png](https://i.loli.net/2019/12/12/AYDIH4zKsN1wgRm.png)
- `_ViewStart.cshtml` 文件所生成的
![微信截图_20191212232735.png](https://i.loli.net/2019/12/12/RxPWKhb6FG2HufY.png)
- `Shared` 目录下视图文件所生成的
![微信截图_20191212232813.png](https://i.loli.net/2019/12/12/AZUwOXdzt7lIGLJ.png)

关于它们的更多信息，本篇不做更多的阐述，更多请查看这篇 [文章](https://github.com/NGPONG/document/blob/master/ASP.NET/MVC/ASP.NET%20MVC%20Life%20Cycle.md)

#### Razor 语法简单描述

`@`

输出一个变量，或者是输出一个函数的返回值，不能单独用来输出一个值，比如说字符串或数字

`@( )`

解决上一种语法所存在的问题，用来输出一个值，比如说字符串或数字

`@{ }`

标识一个代码块，在这里可以直接编写 `C#` 的代码

`@* *@`

标识注释块

`@section sectionName{ }`

是写在布局页中，部分渲染区域 `sectionName` 里面的内容

`@using namespance`

引入命名空间

`@model namespace.class`

指定强类型视图的具体类型

```csharp
@{
    ViewBag.Title = "Index";
    Layout = null;
}

@using System.Reflection;

@model ASP.NET_MVC_Study_03.Models.CustomEntity

<h1>@Assembly.GetExecutingAssembly().Location</h1>

<h1>@("Hello, World!")</h1>

@*<h1>111</h1>*@

@{ 
    for (int i = 0; i < 10; i++)
    {
        <h1>@i.ToString()</h1>
    }
}
```

#### LayoutPage 布局页
<br/>

<span>1.</span> 简述

`Razor` 的布局页和 `WebForm` 中的模板页是一个概念，也是作为一个子页面嵌套于一个布局页中的使用，作为布局页，其统一放在网站根目录的 `~/Views/Shared` 目录下

<br/>

<span>2.</span> 如何指定布局页

我们可以通过两种方式来指定一个视图页所对应的布局页，第一种方式是一种全局嵌套的方式，在一个 `ASP.NET MVC` 项目在创建后，会自动为我们生成一个 `_ViewStart.cshtml` 的视图，这个视图页在开始渲染当前请求所指定的 `Action` 所映射的视图前开始执行，在这里面我们就可以通过其父类 `ViewStartPage` 的属性 `Layout` 来指定一个布局页

```csharp
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```

第二种方式则是指定部分页面所引用的布局页，和上面一个方法一样，也是通过其父类 `WebViewPage<T>` 的 `Layout` 属性来指定一个布局页，这里要区分开来，在 `_ViewStart.cshtml` 的父类和视图的父类是不一样的

其次在这里，我们也可以通过指定 `Layout` 的属性为 `Null` or `Empty` 来取消在 `_ViewStart.cshtml` 所设定的全局布局页的设定

```csharp
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```

<br/>

<span>3.</span> 视图页的渲染顺序

他们三者之间的渲染顺序是由 _ViewStart.cshtml -> 视图页 - 布局页 进行渲染的，这也印证了为什么能够在视图页中指定 `Layout` 属性的值为为 `Null` or `Empty` 就可以取消在 `_ViewStart.cshtml` 所设定的全局布局页的原理，但是关于更多信息可以参考这篇 [文章](https://github.com/NGPONG/document/blob/master/ASP.NET/MVC/ASP.NET%20MVC%20Life%20Cycle.md)

<br/>

<span>4.</span> 布局页的使用方法

通过在布局页中使用 `@RenderBody()` 来渲染在视图页中的内容，如果视图页使用了该布局也的话，视图页里面的内容则一定会渲染在使用该函数的区域

通过在布局页中使用 `@RenderSection(string sectionName,bool required)` 来标识部分渲染的内容，其中也可以指定 `required` 参数来决定是否必须要渲染这部分区域，除此之外，在相应的引用了这个布局页的视图页中，需要通过语法 `@section sectionName{ }` 来构造布局页中调用 `@RenderSection` 区域的内容

```html
<!DOCTYPE html>

<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <script src="~/Scripts/jquery-3.3.1.min.js"></script>

    <script type="text/javascript">
        @RenderSection("scripts", required: false)
    </script>

</head>
<body>

    @RenderBody()

</body>
</html>
```

```html
@{
    ViewBag.Title = "Index";
    Layout = "~/Views/Shared/_LayoutPage.cshtml";
}

@section scripts{
    $(this).ready(function () {

    window.alert("Hello, I'm the content of section area!")
    });
}


<h1>Hello,World!</h1>
```

<br/>

<span>5.</span> 强类型视图

`ViewData` & `ViewBag` 在前面已经介绍过，它们可以提供给当前请求上下文去存放一些数据提供 `View` 去使用，但是它们还是会存在一个很大的缺陷就是类型的问题，使用 `ViewData` 的时候需要对类型进行转换，而是用 `ViewBag` 由于是动态类型，所以在编写的时候无法让编译器起到一个很好的检查作用，`ASP.NET MVC` 针对这一问题还为开发人员提供了一种利用强类型的方法来将数据或对象传递到视图模板中，这种强类型的方法为你的编码过程提供了很丰富的编辑时的智能输入提示信息与非常好的编译时的检查，通过这种方法后我们所编写的视图页也可以称之为 `强类型视图`

指定 `View` 所使用的强类型模型的实例通过两种方式，第一种就是通过 [上面](#Controller.View) 所提到的在控制器中通过 `Controller.View(object entity)` 函数，在返回视图结果的同时把一个类型的实例也一起返回给视图

```csharp
public class MyController : Controller
{
    [HttpGet]
    public ActionResult Index()
    {
        return View(new MyEntity(){
            Name = "Wupeng",
            Age  = 22
        };
    }
}
```

第二种方式则是通过 `ViewData.Model` 属性来指定强类型模型的实例

```csharp
public class MyController : Controller
{
    [HttpGet]
    public ActionResult Index()
    {
        base.ViewData.Model = new MyEntity(){
            Name = "Wupeng",
            Age  = 22
        };

        return View();
    }
}
```

当我们强类型视图所具体使用的强类型模型的实例后，我们还需要在视图页面上通过 `@model namespace.model` 语法来引入强类型模型的命名空间，接下来只需要通过视图页的父类 `WebViewPage<T>` 所提供的 `Model` 属性 或者是直接通过 `ViewData.Model` 就可以访问我们在刚刚所设定的强类型模型中的属性了，因为在 `Controller` 中又设定了强类型模型的具体实例，所以这里的输出是有效值的

这里也稍微扩展下，为什么视图页的父类中有一个泛型 `WebViewPage<T>`，这个泛型其实就是我们在视图页中通过 `@model namespace.model` 语法所设定的强类型

```html
@{
    ViewBag.Title = "Index";
    Layout = "~/Views/Shared/_LayoutPage.cshtml";
}

<h1> Hello,@Model.Name! </h1>
<h1> I'm @ViewData.Model.Age.ToString() years old </h1>
```
