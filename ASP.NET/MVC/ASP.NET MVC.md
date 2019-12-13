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

路由是 `ASP.NET MVC` 的一个最为核心的部分，一切都由其作为根源，在这里我们需要区别于传统的 `WebForm` 形式的开发，`WebForm` 中，对 URL 的传入请求通常映射到服务器磁盘上真实的物理文件位置上

而对于 `ASP.NET MVC`，具体的一切请求路径都需要遵循我们所定义的路由路径，并且最后会根据我们所请求的 URL 映射至具体的某个 `Controller` 的 `Action` 身上，当然这句话是相对的，我们其实也可以根据文件路径直接访问 `ASP.NET MVC` 框架所编写的网站的具体某个网站，但是需要注意的是，这句话是不包含 `Views` 文件夹目录下的，因为 `Views` 文件夹目录下所存放的视图文件默认情况下都会给一个叫做 `BlockViewHandler` 的 `HttpHandler` 作为管控，如果所请求的路径包含 `Views` 文件夹下的内容则会弹出 `404` 的错误页，如果需要解决这一问题还需配置下这个 `BlockViewHandler` 的 `path` 仅针对 `*.cshtml` 文件即可

#### 注册一个路由路径


