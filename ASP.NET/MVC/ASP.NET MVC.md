# *<span>ASP.NET</span> MVC*

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
            constraints: new { httpMethod = new HttpMethodConstraint("GET")}
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

