# *<span>ASP.NET</span> MVC Life Cycle*


<br/>

### 目录

- [路由的开始](#路由的开始)
- [Area 的注册](#Area的注册)
- [Controller 的激活](#Controller的激活)
- [Filter 和 Action 的执行](#Filter和Action的执行)
- [ActionResult 的执行](#标识4)
- [View 的渲染](#View的渲染)

<br/>

### 路由的开始
<span id="路由的开始"></span>

---

路由系统是 `ASP.NET MVC` 的核心，其隶属于 `System.Web.Routing.dll` 组件，关于MVC的入口也是从它开始

`System.Web.Routing.dll` 的加载是通过一个独属于 `ASP.NET Mvc Process Model` 名为 `UrlRoutingModule` 的 `HttpModule` 去完成的，查看以下源码

```csharp
public class UrlRoutingModule : IHttpModule
{
    protected virtual void Init(HttpApplication application)
    {
        if (application.Context.Items[_contextKey] == null)
        {
            application.Context.Items[_contextKey] = _contextKey;

            // 注册了 HttpApplication 第7个管道事件
            application.PostResolveRequestCache += OnApplicationPostResolveRequestCache;
        }
    }
}
```

可以看到，在其实现 `HttpModule` 的 `Init` 函数的内部，为 `HttpApplication` 管道事件的第7个事件注册了自己的逻辑，以下代码则为其实现

```csharp
public class UrlRoutingModule : IHttpModule
{
	public virtual void PostResolveRequestCache(HttpContextBase context)
	{
		RouteData routeData = RouteCollection.GetRouteData(context);
		if (routeData == null)
		{
			return;
		}
		IRouteHandler routeHandler = routeData.RouteHandler;
		if (routeHandler == null)
		{
			throw new InvalidOperationException(string.Format(CultureInfo.CurrentCulture, SR.GetString("UrlRoutingModule_NoRouteHandler")));
		}
		if (routeHandler is StopRoutingHandler)
		{
			return;
		}
		RequestContext requestContext = new RequestContext(context, routeData);
		context.Request.RequestContext = requestContext;
		IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);
		if (httpHandler == null)
		{
			throw new InvalidOperationException(string.Format(CultureInfo.CurrentUICulture, SR.GetString("UrlRoutingModule_NoHttpHandler"), new object[1]
			{
				routeHandler.GetType()
			}));
		}
		if (httpHandler is UrlAuthFailureHandler)
		{
			if (!FormsAuthenticationModule.FormsAuthRequired)
			{
				throw new HttpException(401, SR.GetString("Assess_Denied_Description3"));
			}
			UrlAuthorizationModule.ReportUrlAuthorizationFailure(HttpContext.Current, this);
		}
		else
		{
			context.RemapHandler(httpHandler);
		}
	}
}
```
<span id="跳转1"></span>
我们将 `PostResolveRequestCache(HttpContextBase context)` 这个函数分开进行讲解，首先关注点先转移至获取 `RouteData` 的身上，`RouteData` 这个类是路由系统的核心类之一，该类是路由系统根据我们在 `Application_Start` 中所录入的路由集合再结合当前所请求的 URL 地址成功解析出来的路由数据，其内部包含着一个关键的属性 `RouteHandler`，其隶属于 `MvcRouteHandler` 类型，这个类型我们放在后面讲解，现在先不做进一步的解析
```csharp
public class UrlRoutingModule : IHttpModule
{
	public virtual void PostResolveRequestCache(HttpContextBase context)
	{
		RouteData routeData = RouteCollection.GetRouteData(context);

		// More ....
	}
}
```

再进入 `RouteCollection.GetRouteData(HttpContext context)` 内部之前，我们先看一下 `RouteCollection`，它是属于 `UrlRoutingModule` 的一个类型为 `RouteCollection` 的属性，那么它是从何而来的呢？我们进入该属性的内部深入研究一下

```csharp
public class UrlRoutingModule : IHttpModule
{
	public RouteCollection RouteCollection
	{
		get
		{
			if (_routeCollection == null)
			{
				_routeCollection = RouteTable.Routes;
			}
			return _routeCollection;
		}
		set
		{
			_routeCollection = value;
		}
	}
}
```

通过上面的代码，我们会发现一个很神奇的问题，`RouteCollection` 的实现是通过内部又引用了另一个类 `RouteTable` 来完成，那么这个 `RouteTable` 又是从何而来的呢？讲到这里，思维又要重新回到我们项目上了，在一个标准的 `ASP.NET MVC` 工程项目建立好后，VS 会自动的为我们在 `Application_Start()` 中录入注册路由的代码，而在代码里面，我们就会发现刚刚所说的那个 `RouteTable`

```csharp
public class MvcApplication : System.Web.HttpApplication
{
	protected void Application_Start()
	{
		//More....
		
		RouteConfig.RegisterRoutes(RouteTable.Routes);

		// More...
	}
}

public class RouteConfig
{
	public static void RegisterRoutes(RouteCollection routes)
	{
		routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

		routes.MapRoute(
			name: "Default",
			url: "{controller}/{action}/{*values}",
			defaults: new { controller = "Home", action = "About" },
			constraints: new { httpMethod = new HttpMethodConstraint("GET")}
		);
	}
}
```

通过上面的代码应该就很清楚了，一个 `ASP.NET MVC Application` 在刚开始启动的过程当中，就通过 `MapRoute` 函数为 `RouteTable.Routes` 录入了我们所定义的路由规则，在这里我们还可以继续对 `RouteCollection.MapRoute` 的深入，下面所贴出来的代码是 `ASP.NET MVC` 框架所开源出来的，这里就省去了还要反编译查看的过程

```csharp
public static class RouteCollectionExtensions
{
	public static Route MapRoute(this RouteCollection routes, string name, string url, object defaults, object constraints, string[] namespaces)
	{
		if (routes == null)
		{
			throw new ArgumentNullException("routes");
		}
		if (url == null)
		{
			throw new ArgumentNullException("url");
		}

		// ***************************************************************
		Route route = new Route(url, new MvcRouteHandler())
		{
			Defaults = CreateRouteValueDictionaryUncached(defaults),
			Constraints = CreateRouteValueDictionaryUncached(constraints),
			DataTokens = new RouteValueDictionary()
		};
		// ***************************************************************

		ConstraintValidation.Validate(route);

		if ((namespaces != null) && (namespaces.Length > 0))
		{
			route.DataTokens[RouteDataTokenKeys.Namespaces] = namespaces;
		}

		// *********************
		routes.Add(name, route);
		// *********************

		return route;
	}
}
```

可以清楚的看到，`MapRoute` 函数其实是属于 `RouteCollection` 的一个 [扩展方法](https://github.com/NGPONG/document/blob/a9dac43935045a7a03e7a7c7453699d4b2adb9fe/c%23/2019%E5%B9%B4/8%E6%9C%884%E6%97%A5/%E6%89%A9%E5%B1%95%E6%96%B9%E6%B3%95.md)，并且我们这里还发现一个新的类型 `Route`，可以发现，`RouteCollection` 中的元素的类型就是为 `Route`

`Route` 是 `ASP.NET MVC` 中的核心类之一，关于该类我们还是有一些需要进行关注的地方的

`Route` 是继承于抽象类 `RouteBase` 的，并且重写了 `RouteBase` 中的 `GetRouteData(HttpContextBase httpContext)` 函数，关于该函数没有必要进行下一步的深究，只需了解它能够返回一个在前面所说到的 `RouteData` 即可

```csharp
public class Route : RouteBase
{
	public override RouteData GetRouteData(HttpContextBase httpContext)
	{
		string virtualPath = httpContext.Request.AppRelativeCurrentExecutionFilePath.Substring(2) + httpContext.Request.PathInfo;
		RouteValueDictionary routeValueDictionary = _parsedRoute.Match(virtualPath, Defaults);
		if (routeValueDictionary == null)
		{
			return null;
		}
		RouteData routeData = new RouteData(this, RouteHandler);
		if (!ProcessConstraints(httpContext, routeValueDictionary, RouteDirection.IncomingRequest))
		{
			return null;
		}
		foreach (KeyValuePair<string, object> item in routeValueDictionary)
		{
			routeData.Values.Add(item.Key, item.Value);
		}
		if (DataTokens != null)
		{
			foreach (KeyValuePair<string, object> dataToken in DataTokens)
			{
				routeData.DataTokens[dataToken.Key] = dataToken.Value;
			}
			return routeData;
		}
		return routeData;
	}
}
```

初次之外，我们把关注点放在 `Route` 的构造函数上，我们会发现除了需要录入一个我们所定义的路由规则的 URL 之外，还创建了一个 `MvcRouteHandler` 的类，并且还把当前的属性 `RouteHandler` 引用成为我们所传入的 `MvcRouteHandler` 类型 ( 实现于 `IRouteHandler` 接口，关于该类的具体作用还需要放在后面进行解析 )，结合 [前面](#跳转1) 所说，关于这点应该是比较清晰的了
```csharp
public static class RouteCollectionExtensions
{
	public static Route MapRoute(this RouteCollection routes, string name, string url, object defaults, object constraints, string[] namespaces)
	{
		// More....

		// ***************************************************************
		Route route = new Route(url, new MvcRouteHandler())
		{
			Defaults = CreateRouteValueDictionaryUncached(defaults),
			Constraints = CreateRouteValueDictionaryUncached(constraints),
			DataTokens = new RouteValueDictionary()
		};
		// ***************************************************************

		// More....
	}
}

public class Route : RouteBase
{
	public Route(string url, IRouteHandler routeHandler)
	{
		Url = url;
		RouteHandler = routeHandler;
	}
}
```

在回到 `PostResolveRequestCache(HttpContextBase context)` 的进一步解析前，最后还要回头来看下 `RouteData` 对象，在前面说到，`RouteData` 是路由系统的核心对象之一，`Route` 对象也不例外，其实 `Route` 和 `RouteData` 之间是存在着千丝万缕的关系的，`Route` 就相当于我们在 `Application_Start` 的时候所录入的自定义路由规则，而 `RouteData` 就是当前所请求的 URL 能够成功匹配到所定义的一个 `Route`，当当前所请的 URL 地址符合某个 `Route` 的时候，就会调用这个 `Route` 的 `GetRouteData` 函数去返回一个与之对应的 `RouteData`，以下代码则为 `RouteCollection.GetRouteData(HttpContext context)` 函数的内部逻辑，可以清楚的看到，`ASP.NET MVC` 框架匹配路由的方式是偏离所有所定义的路由规则 ( `RouteTable.Routes` -> `RouteCollection` ) 去完成校验的

此外还要补充一点的是，`RouteData` 中存在一个属性 `RouteHandler`，其类型也是 `MvcRouteHandler`，并且其引用是和生成这个 `RouteData` 的 `Route` 是存在着对应关系的，最后，稍微剧透一下，`RouteData` 这个类在后面的 `Controller的激活` 步骤中起到了非常关键性的作用，我们脑子里先警醒着这个 `RouteData` 先

```csharp
public class RouteCollection : Collection<RouteBase>
{
	public RouteData GetRouteData(HttpContextBase httpContext)
	{
		if (httpContext == null)
		{
			throw new ArgumentNullException("httpContext");
		}
		if (httpContext.Request == null)
		{
			throw new ArgumentException(SR.GetString("RouteTable_ContextMissingRequest"), "httpContext");
		}
		if (base.Count == 0)
		{
			return null;
		}
		bool flag = false;
		bool flag2 = false;
		if (!RouteExistingFiles)
		{
			flag = IsRouteToExistingFile(httpContext);
			flag2 = true;
			if (flag)
			{
				return null;
			}
		}
		using (GetReadLock())
		{
			using (IEnumerator<RouteBase> enumerator = GetEnumerator())
			{
				while (enumerator.MoveNext())
				{
					RouteBase current = enumerator.Current;
					RouteData routeData = current.GetRouteData(httpContext);
					if (routeData != null)
					{
						if (!current.RouteExistingFiles)
						{
							if (!flag2)
							{
								flag = IsRouteToExistingFile(httpContext);
								flag2 = true;
							}
							if (flag)
							{
								return null;
							}
						}
						return routeData;
					}
				}
			}
		}
		return null;
	}
}
```

```csharp
public class RouteData
{
	private IRouteHandler _routeHandler;

	public IRouteHandler RouteHandler
	{
		get
		{
			return _routeHandler;
		}
		set
		{
			_routeHandler = value;
		}
	}
}
```

通过上面对于 `Route` `RouteData` `RouteTable` `RouteCollection` 所做的铺垫，我们终于可以回到 PostResolveRequestCache(HttpContextBase context) 的内部了，下面的代码还是 `PostResolveRequestCache` 的源码，不过我只把关键的步骤展示出来

```csharp
public class UrlRoutingModule : IHttpModule
{
	public virtual void PostResolveRequestCache(HttpContextBase context)
	{
		// Step 1
		RouteData routeData = RouteCollection.GetRouteData(context);
		
		// More....

		// Step 2
		IRouteHandler routeHandler = routeData.RouteHandler;
		
		// More....

		if (routeHandler is StopRoutingHandler)
		{
			return;
		}

		RequestContext requestContext = new RequestContext(context, routeData);
		context.Request.RequestContext = requestContext;

		// Step 3
		IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);

		// Step 4
		context.RemapHandler(httpHandler);
	}
}
```

`Step 1` 在前面已经说过，其主要目的就是根据当前所请求的URL结合我们在 `Application_Start` 中所定义的路由集合去匹配一个 `RouteData` 对象

`Step 2` 则是根据 `Step 1` 所返回的 `RouteData` 的 `RouteHandler` 属性返回一个实现于 `IRouteHandler` 的 `MvcRouteHandler`

`Step 3` 则是调用了 `MvcRouteHandler` 所实现自 `IRouteHandler` 的 `GetHttpHandler` 函数，去获取一个 `HttpHandler`，这里其实也印证了之前的疑问，`MvcRouteHandler` 的具体作用是什么？`MvcRouteHandler` 的作用则是获取一个实现自 `IHttpHandler` 的 `MvcHandler`，我们可以往下看下 `GetHttpHandler` 函数内部的实现

```csharp
public class MvcRouteHandler : IRouteHandler
{
	protected virtual IHttpHandler GetHttpHandler(RequestContext requestContext)
	{
		requestContext.HttpContext.SetSessionStateBehavior(GetSessionStateBehavior(requestContext));
		return new MvcHandler(requestContext);
	}
}
```

方法非常简单，但是我们也可以从中了解到，在返回 `MvcHandler` 的之前就对 `Session` 状态进行设置了

当 `Step 3` 获取了一个实现自 `IHttpHandler` 的 `MvcHandler` 后，紧接着在 `Step 4` 就会调用 `HttpContext` 的 `RemapHandler` 函数，使刚刚所创建出来的 `MvcHandler` 接过这次请求的后续处理流程

<span style="color:red">紧接着当管道事件走到第11事件与第12个事件之间</span>，就会拿出刚刚所所 `Remap` 的 `HttpHandler` (MvcHandler)，并调用其实现自 `IHttpHandler` 的 `BeginProcessRequest` 函数，进入后续的 `Controller` 激活的步骤

<br/>

<span id = "Area的注册"></span>

### Area 的注册

---

`ASP.NET MVC Area` 可以将一个大型的网站划分为相对独立的区域，该区域同样享有 `Model` `Controller` `View` 的结构模式，并且也有着属于自己的一个 `web.config` 和路由规则，关于如何创建 `Area` 可以参考这一篇 [文章](https://www.codeproject.com/Articles/1139669/How-to-Create-an-Area-in-ASP-NET-MVC-Application)

我们都知道，一个 `Area` 路由规则的注册是通过在新建 `Area` 工程项目的时候，VS自动帮我创建的 `AreaNameAreaRegistration.cs` 这个类中去完成的，我们看一下这个类中的具体实现

```csharp
public class AreaTestAreaRegistration : AreaRegistration 
{
	public override string AreaName 
	{
		get 
		{
			return "AreaTest";
		}
	}

	public override void RegisterArea(AreaRegistrationContext context) 
	{
		context.MapRoute(
			"AreaTest_default",
			"AreaTest/{controller}/{action}/{id}",
			new { action = "Index", id = UrlParameter.Optional }
		);
	}
}
```

可以清楚的看到，Area 的路由注册也是通过 `MapRoute` 来完成的，但需要注意的是，这里的 MapRoute 和前面所讲的 MapRoute 是不同的成员提供的，Area 中的 MapRoute 由 `AreaRegistrationContext` 这个类所提供出来，那我们进入这个类中再仔细看下它的内部实现和之前的 `MapRoute` 有何不同

```csharp
public class AreaRegistrationContext
{
	public Route MapRoute(string name, string url, object defaults, object constraints, string[] namespaces)
	{
		if (namespaces == null && Namespaces != null)
		{
			namespaces = Namespaces.ToArray();
		}

		Route route = Routes.MapRoute(name, url, defaults, constraints, namespaces);
		route.DataTokens[RouteDataTokenKeys.Area] = AreaName;

		// disabling the namespace lookup fallback mechanism keeps this areas from accidentally picking up
		// controllers belonging to other areas
		bool useNamespaceFallback = (namespaces == null || namespaces.Length == 0);
		route.DataTokens[RouteDataTokenKeys.UseNamespaceFallback] = useNamespaceFallback;

		return route;
	}
}
```

可以看到它和之前的 `MapRoute` 存在的区别不是很大，也是调用了在上文所说的 `RouteCollection.MapRoute` 的函数来完成 `Route` 的最终创建，有一点不同的是，Area 在创建完成之后还特别针对当前 `Area` 的命名空间为上一步所创建的 `Route` 写入了一个 `DataToken`

关于 `DataToken` 还是有必要稍微展开讲一下，故名起义，这个成员就是属于 `Route` 的一个Token，其目的主要是为了避免 `Area` 和原始的母 MVC 项目存在同名 `Controller` 的情况，具体如何实现的呢？其内部存放的就是我们在注册路由的时候所指定的 `namespaces` 属性的值，以该 `namespaces` 来区分一个 `WebApplication` 下，出现 `Area` 的控制器和主目录的控制器重名的情况，可以看出其实际上只有在 `Area` 中能够达到实际作用

回过头来我们已经知道了 `Area` 的路由规则是如何建立的，但是还存在着一个问题，`AreaNameAreaRegistration.RegisterArea(AreaRegistrationContext context)` 这个函数又是在哪里会调用的呢？其实也是在 `Application_Start` 当中被调用，具体体现在

```csharp
public class MvcApplication : System.Web.HttpApplication
{
	protected void Application_Start()
	{
		AreaRegistration.RegisterAllAreas();

		// More...
	}
}
```

我们进入到 `AreaRegistration.RegisterAllAreas()` 这个函数的内部一探究竟

```csharp
public abstract class AreaRegistration
{
	internal static void RegisterAllAreas(RouteCollection routes, IBuildManager buildManager, object state)
	{
		List<Type> areaRegistrationTypes = TypeCacheUtil.GetFilteredTypesFromAssemblies(TypeCacheName, IsAreaRegistrationType, buildManager);
		foreach (Type areaRegistrationType in areaRegistrationTypes)
		{
			AreaRegistration registration = (AreaRegistration)Activator.CreateInstance(areaRegistrationType);
			registration.CreateContextAndRegister(routes, state);
		}
	}
}
```

具体含义其实不复杂，首先枚举出当前当前 `AppDomain` 中所有继承于 `AreaRegistration` 的子类 (VS自动帮我创建的 `AreaNameAreaRegistration.cs` 这个类就是 `AreaRegistration` 的派生类)，然后调用了其 `CreateContextAndRegister(RouteCollection routes, object state)` 函数，我们继续深究进入这个函数的内部

```csharp
public abstract class AreaRegistration
{
	internal void CreateContextAndRegister(RouteCollection routes, object state)
	{
		AreaRegistrationContext context = new AreaRegistrationContext(AreaName, routes, state);

		string thisNamespace = GetType().Namespace;
		if (thisNamespace != null)
		{
			context.Namespaces.Add(thisNamespace + ".*");
		}

		RegisterArea(context);
	}
}
```

这时候仿佛就豁然开朗了，看到了熟悉的 `RegisterArea(AreaRegistrationContext context)` 函数，明白了其具体的调用的过程

在这里也稍微总结一下，在 `Application_Start` 中，通过 `AreaRegistration.RegisterAllAreas()` 函数，找到当前 `AppDomain` 下所有派生于 `AreaRegistration` 的子类，并调用其 `RegisterArea` 方法完成 `Area` 的路由注册，最后呈现的效果是则为在全局路由表 `RouteTable` 中添加一条属于 `Area` 的路由规则，并且，再添加完成路由规则后还会把当前的 `AreaRegistration` 派生类的命名空间和 `UseNamespaceFallback` 加到 `Route` 的 `DataToken` 中，关于这个 `DataToken` 的具体使用我们放在下一节点揭开其面纱

<br/>

<span id = "Controller的激活"></span>

### Controller 的激活

---

续 [路由的开始](#路由的开始)，请求流程走到了 `BeginProcessRequest` 函数，再该函数内部紧接着又调用了 `ProcessRequestInit` 函数，该函数的工作可以视为 `Controller激活` 的一个入口，在这里，`Controller` 的实例被真正的构建出来，我们深入了解下该函数的内部

```csharp
public class MvcHandler : IHttpAsyncHandler, IHttpHandler, IRequiresSessionState
{
	private void ProcessRequestInit(HttpContextBase httpContext, out IController controller, out IControllerFactory factory)
	{
		// If request validation has already been enabled, make it lazy. This allows attributes like [HttpPost] (which looks
		// at Request.Form) to work correctly without triggering full validation.
		// Tolerate null HttpContext for testing.
		HttpContext currentContext = HttpContext.Current;
		if (currentContext != null)
		{
			bool? isRequestValidationEnabled =  .IsValidationEnabled(currentContext);
			if (isRequestValidationEnabled == true)
			{
				ValidationUtility.EnableDynamicValidation(currentContext);
			}
		}

		AddVersionHeader(httpContext);
		RemoveOptionalRoutingParameters();

		// Get the controller type
		string controllerName = RequestContext.RouteData.GetRequiredString("controller");

		// Instantiate the controller and call Execute
		factory = ControllerBuilder.GetControllerFactory();
		controller = factory.CreateController(RequestContext, controllerName);
		if (controller == null)
		{
			throw new InvalidOperationException(
				String.Format(
					CultureInfo.CurrentCulture,
					MvcResources.ControllerBuilder_FactoryReturnedNull,
					factory.GetType(),
					controllerName));
		}
	}
}
```
前面的代码可以无视，主要是做一些请求验证的处理工作，我们把关注点从 `Get the controller type` 这一句注释开始，在这里我们又看到了熟悉的 `RouteData` 对象，它是当前所请求的 URL 成功匹配到自定义路由规则 `Route` 所创建的 `RouteData`，在这里通过它去调用 `RouteData.GetRequiredString(string valueName)` 去获取 `Controller` 具体的名字

紧接着，通过 `ControllerBuilder.GetControllerFactory()` 获取一个 `Controller工厂`，在这里需要稍微扩展以下 `ControllerBuilder` 这个类，可以清楚的看到 `ASP.NET MVC` 关于 `Controller` 的获取采用了抽象工厂的模式，默认情况下所返回的工厂为 `DefaultControllerFactory` (接下来深入的流程也是会围绕着这个类进行展开)，另外还要扩充一点的是，`ControllerBuilder` 对于工厂的实例化采取了 `Dependency Injection` (依赖注入) 的方式来展开，具体可以查看以下代码

```csharp
public class ControllerBuilder
{
	internal ControllerBuilder(IResolver<IControllerFactory> serviceResolver)
	{
		_serviceResolver = serviceResolver ?? new SingleServiceResolver<IControllerFactory>(
													() => _factoryThunk(),
													new DefaultControllerFactory { ControllerBuilder = this },
													"ControllerBuilder.GetControllerFactory");
	}
}
```

回到正题，我们看一下，在获取了一个 `Controller工厂` 后，我们看一下它的 `IControllerFactory.CreateController(RequestContext requestContext, string controllerName)` 又做了什么，先上一下源码

```csharp
public class DefaultControllerFactory : IControllerFactory
{
	public virtual IController CreateController(RequestContext requestContext, string controllerName)
	{
		// More....

		// Step 1
		Type controllerType = GetControllerType(requestContext, controllerName);

		// Step 2
		IController controller = GetControllerInstance(requestContext, controllerType);

		return controller;
	}
}
```

还是分开来一步一步讲解把，先从 `Step 1` 开始，我们进入 `GetControllerType` 这个函数的内部看下具体做了什么

```csharp
public class DefaultControllerFactory : IControllerFactory
{
	protected internal virtual Type GetControllerType(RequestContext requestContext, string controllerName)
	{
		// More....

		// first search in the current route's namespace collection
		object routeNamespacesObj;
		Type match;
		if (routeData != null && routeData.DataTokens.TryGetValue(RouteDataTokenKeys.Namespaces, out routeNamespacesObj))
		{
			IEnumerable<string> routeNamespaces = routeNamespacesObj as IEnumerable<string>;
			if (routeNamespaces != null && routeNamespaces.Any())
			{
				HashSet<string> namespaceHash = new HashSet<string>(routeNamespaces, StringComparer.OrdinalIgnoreCase);
				match = GetControllerTypeWithinNamespaces(routeData.Route, controllerName, namespaceHash);

				// the UseNamespaceFallback key might not exist, in which case its value is implicitly "true"
				if (match != null || false.Equals(routeData.DataTokens[RouteDataTokenKeys.UseNamespaceFallback]))
				{
					// got a match or the route requested we stop looking
					return match;
				}
			}
		}

		// then search in the application's default namespace collection
		RouteBase route = routeData == null ? null : routeData.Route;
		if (ControllerBuilder.DefaultNamespaces.Count > 0)
		{
			HashSet<string> namespaceDefaults = new HashSet<string>(ControllerBuilder.DefaultNamespaces, StringComparer.OrdinalIgnoreCase);
			match = GetControllerTypeWithinNamespaces(route, controllerName, namespaceDefaults);
			if (match != null)
			{
				return match;
			}
		}

		// if all else fails, search every namespace
		return GetControllerTypeWithinNamespaces(route, controllerName, null /* namespaces */);
	}
}
```

在这里我们又看到了熟悉的 `RouteData.DataTokens` ，没错这里也是判断 `DataTokens` 中是否存在与当前相符的 `NameSpace`，如果存在则调用 `GetControllerTypeWithinNamespaces` 函数对 `Controller` 进行下一步的检索工作，我们看下这个函数具体做了什么

```csharp
public class DefaultControllerFactory : IControllerFactory
{
	private Type GetControllerTypeWithinNamespaces(RouteBase route, string controllerName, HashSet<string> namespaces)
	{
		// Once the master list of controllers has been created we can quickly index into it
		ControllerTypeCache.EnsureInitialized(BuildManager);

		ICollection<Type> matchingTypes = ControllerTypeCache.GetControllerTypes(controllerName, namespaces);
		switch (matchingTypes.Count)
		{
			case 0:
				// no matching types
				return null;

			case 1:
				// single matching type
				return matchingTypes.First();

			default:
				// multiple matching types
				throw CreateAmbiguousControllerException(route, controllerName, matchingTypes);
		}
	}
}
```

这里可以看到 `ControllerTypeCache.EnsureInitialized` 这个函数，我们在进入其内部看看发生了什么

```csharp
internal sealed class ControllerTypeCache
{
	public void EnsureInitialized(IBuildManager buildManager)
	{
		if (_cache == null)
		{
			lock (_lockObj)
			{
				if (_cache == null)
				{
					List<Type> controllerTypes = TypeCacheUtil.GetFilteredTypesFromAssemblies(TypeCacheName, IsControllerType, buildManager);
					var groupedByName = controllerTypes.GroupBy(
						t => t.Name.Substring(0, t.Name.Length - "Controller".Length),
						StringComparer.OrdinalIgnoreCase);
					_cache = groupedByName.ToDictionary(
						g => g.Key,
						g => g.ToLookup(t => t.Namespace ?? String.Empty, StringComparer.OrdinalIgnoreCase),
						StringComparer.OrdinalIgnoreCase);
				}
			}
		}
	}
}
```

在这里，我们似乎又看到了一个熟悉的函数 `TypeCacheUtil.GetFilteredTypesFromAssemblies`，不过距离上次看到是在 `Area` 部分当中用于当前 `AppDomain` 中所有派生自 `AreaRegistration` 的类型，在这里我们深入研究下具体它干了些什么

```csharp
internal static class TypeCacheUtil
{
	public static List<Type> GetFilteredTypesFromAssemblies(string cacheName, Predicate<Type> predicate, IBuildManager buildManager)
	{
		TypeCacheSerializer serializer = new TypeCacheSerializer();

		// first, try reading from the cache on disk
		List<Type> matchingTypes = ReadTypesFromCache(cacheName, predicate, buildManager, serializer);
		if (matchingTypes != null)
		{
			return matchingTypes;
		}

		// if reading from the cache failed, enumerate over every assembly looking for a matching type
		matchingTypes = FilterTypesInAssemblies(buildManager, predicate).ToList();

		// finally, save the cache back to disk
		SaveTypesToCache(cacheName, matchingTypes, buildManager, serializer);

		return matchingTypes;
	}
}
```

在这里就没有继续往下研究的必要了，有一点需要补充下，`ASP.NET MVC` 针对请求次数过多和反射耗费性能的问题，针对反射出来的类也需要缓存在一个文件中进行保存，而这个函数其内部的思想就体现了解决这一问题所在，具体的流程就是就是从缓存文件中获取到一个 `Type` 的集合，`ASP.NET MVC` 对应 `Controller` 的缓存文件则为 `Mvc-ControllerTypeCache.xml` ，下面可以看下它里面的内容稍微扩充一下

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--This file is automatically generated. Please do not modify the contents of this file.-->
<typeCache lastModified="11/4/2012 8:52:26 PM" mvcVersionId="a5d58bd9-3a4a-4d1d-a7ce-9cef11e4c380">
  <assembly name="MVCApp, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null">
    <module versionId="c7b3d847-7853-44f3-87d0-9cc040c4cb53">
      <type>MVCApp.Areas.Admin.Controllers.HomeController</type>
      <type>MVCApp.Controllers.HomeController</type>
    </module>
  </assembly>
</typeCache>
```

另外还要补充一点的是，`GetFilteredTypesFromAssemblies` 函数的 Signature `Predicate<Type> predicate` 也是很有意思，我们可以看下它具体的实现

```csharp
internal static bool IsControllerType(Type t) 
{
	return t != null &&
		   t.IsPublic &&
		   t.Name.EndsWith("Controller", StringComparison.OrdinalIgnoreCase) &&
		   !t.IsAbstract &&
		   typeof(IController).IsAssignableFrom(t);
}
```

可以看出，一个类如果要能够成为 `Controller`，类名必须以 `Controller` 结尾，必须是public的，必须实现IController接口

我们把关注点再回到 `EnsureInitialized` 函数的身上，在获取到当前 `AppDomain` 中 `Controller` 所反射的 `Type` 的集合 `controllerTypes` 之后，还会对这个集合进行一些过滤条件的动作，比如说根据 `Controller` 的名字进行分组，再根据 `NameSpace` 进行下一步的筛选，其目的在于为后面可以更加的方便的获取到 `Controller` 的信息

我们再把关注点回到 `GetControllerTypeWithinNamespaces` 函数的身上，在调用完成 `ControllerTypeCache.EnsureInitialized` 对 `Controller` 的 `Type` 进行一些预初始化工作后，紧接着又调用了 `ControllerTypeCache.GetControllerTypes` 去真正的获取当前所请求的 URL 所对应的 `Controller`，关于其内部实现就不做进一步的展示了，具体含义只是做一些后期的校验和匹配工作

目光再回到 `GetControllerType` 函数身上，我们发现这里一共是调用了三次 `GetControllerTypeWithinNamespaces` 函数，上面所讲的是针对正常情况下的获取 `Controller` 的流程，后面两次也不做详细探讨，如果第一次 `GetControllerTypeWithinNamespaces` 返回了 null，并且 `UseNamespaceFallback` 设为true，那么会进行下一步的搜索，否则返回 null，下一步的搜索就是在项目的 `DefaultNamespace` 下进行搜索，对于没有 `Namespace` 的 `RouteData`，默认就是在这里搜索的。最后在所有的 `Namespace` 中搜索

回到 `CreateController` 函数身上，在调用完成 `GetControllerType` 获取到具体的 `Controller` 的 `Type` 以后，紧接着又调用了 `GetControllerInstance` 函数去真正进入创建 `Controller` 实例的流程，我们可以看下以下代码

```csharp
public class DefaultControllerFactory : IControllerFactory
{
	protected internal virtual IController GetControllerInstance(RequestContext requestContext, Type controllerType)
	{
		if (requestContext == null)
		{
			throw new ArgumentNullException("requestContext");
		}
		if (controllerType == null)
		{
			throw new HttpException(404,
									String.Format(
										CultureInfo.CurrentCulture,
										MvcResources.DefaultControllerFactory_NoControllerFound,
										requestContext.HttpContext.Request.Path));
		}
		if (!typeof(IController).IsAssignableFrom(controllerType))
		{
			throw new ArgumentException(
				String.Format(
					CultureInfo.CurrentCulture,
					MvcResources.DefaultControllerFactory_TypeDoesNotSubclassControllerBase,
					controllerType),
				"controllerType");
		}

		return ControllerActivator.Create(requestContext, controllerType);
	}
}
```

在这里也可以看到，如果在调用 `GetControllerType` 所获取到的 `Controller` 的 `Type` 为空，则弹出 `404` 的错误，在这里也要稍微扩充一点，`ASP.NET MVC` 区别于 `WebForm` 的是，一切请求都是基于所注册的路由路径来完成的，而 `WebForm` 的请求是基于 `文件的虚拟路径来完成的`，回到正题，最终的调用落定在 `ControllerActivator.Create` 身上

关于 `ControllerActivator` 这个类的实例化，`ASP.NET MVC` 再一次采用了 `Dependency Injection` 的机制，它继承于 `IControllerActivator` 接口，默认情况下它为 `DefaultControllerActivator` 的实例，那么在 `Create` 函数的内部紧着又调用了 `IDependencyResolver.GetService` 函数，这个函数的内部完成了 `Controller` 的最终实例化工作，我们可以看下以下代码

```csharp
private class DefaultDependencyResolver : IDependencyResolver
{
	public object GetService(Type serviceType)
	{
		// Since attempting to create an instance of an interface or an abstract type results in an exception, immediately return null
		// to improve performance and the debugging experience with first-chance exceptions enabled.
		if (serviceType.IsInterface || serviceType.IsAbstract)
		{
			return null;
		}

		try
		{
			// 终于等到你
			return Activator.CreateInstance(serviceType);
		}
		catch
		{
			return null;
		}
	}
}
```

至此，`Controller` 的创建步骤已然完成，那么步骤又重新回到了 `BeginProcessRequest` 身上，我们将在下一个节点继续介绍 `Filter 和 Action 的执行`

<br/>

<span id = "Filter和Action的执行"></span>

### Filter 和 Action 的执行

---

继续 `BeginProcessRequest` 这个函数之旅，在获取完成 `Controller` 之后，会先查看所返回的 `Controller` 类型是否是异步Controller `IAsyncController`，这里又是一个新的类型，但是我们不对其做分析，我们来分析 `Synchronous Controller` 的进一步实现

在 `Synchronous Controller` 模式下，会调用上面所说的所返回的 `Controller` 的 `Execute` 函数进入下一步的流程，该函数实现自 `Controller` 的最高基类 `ControllerBase`，我们来看下该函数的具体实现

```csharp
public abstract class ControllerBase : IController
{
	protected virtual void Execute(RequestContext requestContext)
	{
		// More....

		VerifyExecuteCalledOnce();
		Initialize(requestContext);

		using (ScopeStorage.CreateTransientScope())
		{
			ExecuteCore();
		}
	}
}
```

在该函数中，首先除了做了一些函数安全检查之外，还通过 `Initialize` 函数实例化了 `ControllerContext` 类，该类包含着当前请求上下文和 `Controller` 的实现，最后我们把关注点放在 `ExecuteCore` 函数身上，该函数是一个抽象函数，真正的实现在 `ControllerBase` 的下级派生类 `Controller` 身上，先看看下其内部的实现

```csharp
public abstract class Controller : ControllerBase, IActionFilter, IAuthenticationFilter, IAuthorizationFilter, IDisposable, IExceptionFilter, IResultFilter, IAsyncController, IAsyncManagerContainer
{
	protected override void ExecuteCore()
	{
		// If code in this method needs to be updated, please also check the BeginExecuteCore() and
		// EndExecuteCore() methods of AsyncController to see if that code also must be updated.

		PossiblyLoadTempData();
		try
		{
			string actionName = GetActionName(RouteData);
			if (!ActionInvoker.InvokeAction(ControllerContext, actionName))
			{
				HandleUnknownAction(actionName);
			}
		}
		finally
		{
			PossiblySaveTempData();
		}
	}
}
```

该函数首先会加载临时数据，在这里不做讨论，在下面有一个需要重点关注且稍显复杂的地方 `InvokeAction`，他是接口 `IActionInvoker` 的函数，具体的实现来自于 `ControllerActionInvoker`，该函数是 `Action` 的发现和 `Filter` 执行的一个核心，我们进入该函数具体探究下其实现

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	public virtual bool InvokeAction(ControllerContext controllerContext, string actionName)
	{
		// More...

		ControllerDescriptor controllerDescriptor = GetControllerDescriptor(controllerContext);
		ActionDescriptor actionDescriptor = FindAction(controllerContext, controllerDescriptor, actionName);

		if (actionDescriptor != null)
		{
			FilterInfo filterInfo = GetFilters(controllerContext, actionDescriptor);

			try
			{
				AuthenticationContext authenticationContext = InvokeAuthenticationFilters(controllerContext, filterInfo.AuthenticationFilters, actionDescriptor);

				if (authenticationContext.Result != null)
				{
					// An authentication filter signaled that we should short-circuit the request. Let all
					// authentication filters contribute to an action result (to combine authentication
					// challenges). Then, run this action result.
					AuthenticationChallengeContext challengeContext = InvokeAuthenticationFiltersChallenge(
						controllerContext, filterInfo.AuthenticationFilters, actionDescriptor,
						authenticationContext.Result);
					InvokeActionResult(controllerContext, challengeContext.Result ?? authenticationContext.Result);
				}
				else
				{
					AuthorizationContext authorizationContext = InvokeAuthorizationFilters(controllerContext, filterInfo.AuthorizationFilters, actionDescriptor);
					if (authorizationContext.Result != null)
					{
						// An authorization filter signaled that we should short-circuit the request. Let all
						// authentication filters contribute to an action result (to combine authentication
						// challenges). Then, run this action result.
						AuthenticationChallengeContext challengeContext = InvokeAuthenticationFiltersChallenge(
							controllerContext, filterInfo.AuthenticationFilters, actionDescriptor,
							authorizationContext.Result);
						InvokeActionResult(controllerContext, challengeContext.Result ?? authorizationContext.Result);
					}
					else
					{
						if (controllerContext.Controller.ValidateRequest)
						{
							ValidateRequest(controllerContext);
						}

						IDictionary<string, object> parameters = GetParameterValues(controllerContext, actionDescriptor);
						ActionExecutedContext postActionContext = InvokeActionMethodWithFilters(controllerContext, filterInfo.ActionFilters, actionDescriptor, parameters);

						// The action succeeded. Let all authentication filters contribute to an action result (to
						// combine authentication challenges; some authentication filters need to do negotiation
						// even on a successful result). Then, run this action result.
						AuthenticationChallengeContext challengeContext = InvokeAuthenticationFiltersChallenge(
							controllerContext, filterInfo.AuthenticationFilters, actionDescriptor,
							postActionContext.Result);
						InvokeActionResultWithFilters(controllerContext, filterInfo.ResultFilters,
							challengeContext.Result ?? postActionContext.Result);
					}
				}
			}
			catch (ThreadAbortException)
			{
				// This type of exception occurs as a result of Response.Redirect(), but we special-case so that
				// the filters don't see this as an error.
				throw;
			}
			catch (Exception ex)
			{
				// something blew up, so execute the exception filters
				ExceptionContext exceptionContext = InvokeExceptionFilters(controllerContext, filterInfo.ExceptionFilters, ex);
				if (!exceptionContext.ExceptionHandled)
				{
					throw;
				}
				InvokeActionResult(controllerContext, exceptionContext.Result);
			}

			return true;
		}

		// notify controller that no method matched
		return false;
	}
}
```

我们一步一步来解读，首先通过了 `GetControllerDescriptor(ControllerContext controllerContext)` 函数获取了一个 `ControllerDescriptor` 的抽象类，其具体的实现为 `ReflectedControllerDescriptor`，接下来又调用了 `FindAction(ControllerContext controllerContext, ControllerDescriptor controllerDescriptor, string actionName)` 函数，其中 `ControllerDescriptor` Signature的具体引用则为刚刚所获取到的 `ReflectedControllerDescriptor`，我们先来看下这个函数的具体实现

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual ActionDescriptor FindAction(ControllerContext controllerContext, ControllerDescriptor controllerDescriptor, string actionName)
	{
		// More....

		ActionDescriptor actionDescriptor = controllerDescriptor.FindAction(controllerContext, actionName);
		return actionDescriptor;
	}
}
```

通过上面的代码可以看到，通过 `controllerDescriptor` 这个参数调用了 `FindAction(ControllerContext controllerContext, string actionName)` 并返回了一个 `ActionDescriptor` 的抽象类，前面说过，`controllerDescriptor` 的具体实现为 `ReflectedControllerDescriptor`，我们先进入 `FindAction` 这个函数的内部一叹究，看看 `ActionDescriptor` 的具体实现到底什么

```csharp
public class ReflectedControllerDescriptor : ControllerDescriptor
{
	public override ActionDescriptor FindAction(ControllerContext controllerContext, string actionName)
	{
		// More....

		MethodInfo matched = _selector.FindActionMethod(controllerContext, actionName);
		if (matched == null)
		{
			return null;
		}

		return new ReflectedActionDescriptor(matched, actionName, this);
	}
}
```

在这里，我们可以看到通过 `ControllerContext` 和 `ActionName` 调用了 `_selector` 的 `FindActionMethod` 函数去获取到了一个 `MethoInfo`，关于 `MethodInfo` 我们可以理解成 `Reflected` 了指定 `Controller` 的所有 `Action`，并于 `ActionName` 进行匹配后拿到一个当前请求 URL 所对应的具体 `Action` 的 `MethodInfo`，还有另外需要补充的是，`_selector` 是一个 `ActionMethodSelectorBase` 的抽象类，其具体实现为 `ActionMethodSelector`，当然这里仅做了解

当获取完成 `MethodInfo` 后，将其封装至 `ReflectedActionDescriptor` 类中，那我们也能够了解到，`ActionDescriptor` 的具体实现则为 `ReflectedActionDescriptor` 并且其内部封装了具体 `Action` 的 `Reflected` 

致此，我们可以把思绪转回到 `InvokeAction` 这个函数身上，看看下一步具体还做什么

紧接着下一步，调用了 `GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)` 函数去获取 `Filters`，关于该函数内部的具体调用落定在 `FilterProviders.Providers.GetFilters` 身上，在这里先说两句题外话，我们都知道 `ASP.NET MVC` 为我们提供了三种去注册过滤器的方式，分别是
- 通过 `Application_Start` 中注册一个全局 `Filter`
- 自己实现 `ASP.NET MVC` 所提供的4种不同功能的过滤器的接口，并在具体的 `Action` 或 `Controller` 中使用该特性
- `Controller` 的基类 `Controller` 本身也有几个抽象方法提供了过滤器的功能，通过重写它们以实现这一目标

回过头来，这三种方式其实就是对应着三种 `FilterProvider` 其具体的表现可以从 `FilterProviders` 这个类的静态构造函数中看得出来，以下是它的代码

```csharp
public static class FilterProviders
{
	static FilterProviders()
	{
		Providers = new FilterProviderCollection();

		// Global
		Providers.Add(GlobalFilters.Filters); 

		// Attribute
		Providers.Add(new FilterAttributeFilterProvider()); 

		// Override abstract method
		Providers.Add(new ControllerInstanceFilterProvider()); 
	}
}
```

当获取到当前 `AppDomain` 中所注册的 `Filter` 后，就开始正式进入 `Action` 和 `Filter` 的执行关键步骤了，以下再贴一次 `InvokeAction(ControllerContext controllerContext, string actionName)` 这个函数的代码，以方便观看 

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	public virtual bool InvokeAction(ControllerContext controllerContext, string actionName)
	{
		// More...

		ControllerDescriptor controllerDescriptor = GetControllerDescriptor(controllerContext);
		ActionDescriptor actionDescriptor = FindAction(controllerContext, controllerDescriptor, actionName);

		if (actionDescriptor != null)
		{
			FilterInfo filterInfo = GetFilters(controllerContext, actionDescriptor);

			try
			{
				// More...

				AuthorizationContext authorizationContext = InvokeAuthorizationFilters(controllerContext, filterInfo.AuthorizationFilters, actionDescriptor);
				if (authorizationContext.Result != null)
				{
					// An authorization filter signaled that we should short-circuit the request. Let all
					// authentication filters contribute to an action result (to combine authentication
					// challenges). Then, run this action result.
					AuthenticationChallengeContext challengeContext = InvokeAuthenticationFiltersChallenge(
						controllerContext, filterInfo.AuthenticationFilters, actionDescriptor,
						authorizationContext.Result);
					InvokeActionResult(controllerContext, challengeContext.Result ?? authorizationContext.Result);
				}
				else
				{
					if (controllerContext.Controller.ValidateRequest)
					{
						ValidateRequest(controllerContext);
					}

					IDictionary<string, object> parameters = GetParameterValues(controllerContext, actionDescriptor);
					ActionExecutedContext postActionContext = InvokeActionMethodWithFilters(controllerContext, filterInfo.ActionFilters, actionDescriptor, parameters);

					// The action succeeded. Let all authentication filters contribute to an action result (to
					// combine authentication challenges; some authentication filters need to do negotiation
					// even on a successful result). Then, run this action result.
					AuthenticationChallengeContext challengeContext = InvokeAuthenticationFiltersChallenge(
						controllerContext, filterInfo.AuthenticationFilters, actionDescriptor,
						postActionContext.Result);
					InvokeActionResultWithFilters(controllerContext, filterInfo.ResultFilters,
						challengeContext.Result ?? postActionContext.Result);
				}
			}
			// More ...
			catch (Exception ex)
			{
				// something blew up, so execute the exception filters
				ExceptionContext exceptionContext = InvokeExceptionFilters(controllerContext, filterInfo.ExceptionFilters, ex);
				if (!exceptionContext.ExceptionHandled)
				{
					throw;
				}
				InvokeActionResult(controllerContext, exceptionContext.Result);
			}

			return true;
		}

		// notify controller that no method matched
		return false;
	}
}
```

还忘了一件事，在继续下一步了解之前，先来回顾下 `ASP.NET MVC` 所提供的 4 种 `Filter` 的类型

Filter Type | Interface | Description 
 - | - | - | - | -
 Authorization | IAuthorizationFilter | Runs first
 Action | IActionFilter | Runs before and after the action method 
 Result | IResultFilter | Runs before and after the result is executed
 Exception | IExceptionFilter | Runs if another filter or action method throws an exception

首先进入的是 `InvokeAuthorizationFilters(ControllerContext controllerContext,IList<IAuthorizationFilter> filters, ActionDescriptor actionDescriptor)` 函数，该函数的实现为 `Filter` 的第一次过滤，也就是身份验证，先看一下他的源码

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual AuthorizationContext InvokeAuthorizationFilters(ControllerContext controllerContext, IList<IAuthorizationFilter> filters, ActionDescriptor actionDescriptor)
	{
		AuthorizationContext context = new AuthorizationContext(controllerContext, actionDescriptor);
		foreach (IAuthorizationFilter filter in filters)
		{
			filter.OnAuthorization(context);
			// short-circuit evaluation when an error occurs
			if (context.Result != null)
			{
				break;
			}
		}

		return context;
	}
}
```

在 `Authorization Filter` 内部，我们可以看到一个熟悉的类型 `AuthorizationContext` ，我们知道，如果想要在所实现的 `IAuthorizationFilter` 的 `OnAuthorization(AuthorizationContext context)` 跳转到其他网页的时候，需要指定 `context.Result` 为具体的 `IActionResult` 才能达到跳转的目的，我们先离开这个函数返回 `InvokeAction` 再看看，可以看到调用了 `InvokeAuthorizationFilters` 函数后紧接着返回的就是 `AuthorizationContext`，并且在下一步会判断 `AuthorizationContext` 实例的 `Result` 属性是否为 `NULL`，如果不是的话则会执行 `Result` 所指定的 `IActionResult`，否则的话会继续下一步的执行

紧接着，会调用 `GetParameterValues(ControllerContext controllerContext, ActionDescriptor actionDescriptor)` 获取 `Action` 种具体的参数 `Signature`，在该函数内部还进行了 `Model Binding` 的操作，这也是 `ASP.NET MVC` 的另一个特性之一，我会放在下一个节点说

接下来的调用落定在 `InvokeActionMethodWithFilters(ControllerContext controllerContext, IList<IActionFilter> filters, ActionDescriptor actionDescriptor, IDictionary<string, object> parameters)` 函数身上，在该函数中主要是干了两件事，执行在封装在 `ReflectedActionDescriptor` 中具体 `Action` 的 `MethodInfo`，其次，会根据执行的顺序分辨执行 `IActionFilter` 所实现的具体函数，我们先来看下源代码

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual ActionExecutedContext InvokeActionMethodWithFilters(ControllerContext controllerContext, IList<IActionFilter> filters, ActionDescriptor actionDescriptor, IDictionary<string, object> parameters)
	{
		ActionExecutingContext preContext = new ActionExecutingContext(controllerContext, actionDescriptor, parameters);
		Func<ActionExecutedContext> continuation = () =>
													new ActionExecutedContext(controllerContext, actionDescriptor, false /* canceled */, null /* exception */)
													{
														Result = InvokeActionMethod(controllerContext, actionDescriptor, parameters)
													};

		// need to reverse the filter list because the continuations are built up backward
		Func<ActionExecutedContext> thunk = filters.Reverse().Aggregate(continuation,
																		(next, filter) => () => InvokeActionMethodFilter(filter, preContext, next));
		return thunk();
	}
}
```

关于这个函数大量用到了匿名函数和委托，读起来还是稍微有点难度的，但是我们来拆开里具体看看，首先从 `Func<ActionExecutedContext> continuation` 这个变量看是，可以看到其具体的是又封装了一个 `ActionExecutedContext` 类型，并且我们可以看到该类型中的 `Result` 的属性为 `InvokeActionMethod(controllerContext, actionDescriptor, parameters)` 所调用的结果，那么我们就进入该函数内部具体看看发生了什么

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual ActionResult InvokeActionMethod(ControllerContext controllerContext, ActionDescriptor actionDescriptor, IDictionary<string, object> parameters)
	{
		object returnValue = actionDescriptor.Execute(controllerContext, parameters);
		ActionResult result = CreateActionResult(controllerContext, actionDescriptor, returnValue);
		return result;
	}
}
```

在这里可以看到调用了 `ActionDescriptor` 的抽象函数 `Execute`，在前面说到，`ActionDescriptor` 的具体实现则为 `ReflectedActionDescriptor` 并且其内部封装了具体 `Action` 的 `Reflected`，那么我们进入其内部看看调用情况

```csharp

public override object Execute(ControllerContext controllerContext, IDictionary<string, object> parameters)
{
	// More....

	// Performance sensitive so avoid Linq or delegates.
	ParameterInfo[] parameterInfos = MethodInfo.GetParameters();
	object[] parametersArray = new object[parameterInfos.Length];
	for (int i = 0; i < parameterInfos.Length; i++)
	{
		ParameterInfo parameterInfo = parameterInfos[i];
		object parameter = ExtractParameterFromDictionary(parameterInfo, parameters, MethodInfo);
		parametersArray[i] = parameter;
	}

	ActionMethodDispatcher dispatcher = DispatcherCache.GetDispatcher(MethodInfo);
	object actionReturnValue = dispatcher.Execute(controllerContext.Controller, parametersArray);
	return actionReturnValue;
}
```

具体 `Action` 调用的地方体现在 `object actionReturnValue = dispatcher.Execute(controllerContext.Controller, parametersArray);` 这一句话，不过其实我们也没有继续深究的必要了，让我们返回到 `InvokeActionMethodWithFilters` 这个函数的身上来

在封装好调用 `Action` 的委托 `continuation` 后，紧接着调用了 `InvokeActionMethodFilter(IActionFilter filter, ActionExecutingContext preContext, Func<ActionExecutedContext> continuation)` 函数，在该函数中，`Action` 和 `Filter` 才真正被执行，我们看下其具体的实现

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	internal static ActionExecutedContext InvokeActionMethodFilter(IActionFilter filter, ActionExecutingContext preContext, Func<ActionExecutedContext> continuation)
	{
		filter.OnActionExecuting(preContext);
		
		// More....
		
		try
		{
			postContext = continuation();
		}
		catch (ThreadAbortException)
		{
			// More....

			filter.OnActionExecuted(postContext);
			throw;
		}
		catch (Exception ex)
		{
			// More....

			filter.OnActionExecuted(postContext);
			
			// More....
		}
		if (!wasError)
		{
			filter.OnActionExecuted(postContext);
		}
		return postContext;
	}
}
```

在这里其实省略的很多不必要的代码，但是我们可以看到，`continuation` 这个参数的 Signature 的类型则为我们在上面所说的封装好调用 `Action` 的委托，并且也可以清楚的看到 `IActionFilter` 的具体执行顺序，在这里需要注意的是最后返回的是 `ActionExecutedContext` 类型的 `postContext`，其对应着 `InvokeAction(ControllerContext controllerContext, string actionName)` 函数对应调用 `InvokeActionMethodWithFilters(controllerContext, filterInfo.ActionFilters, actionDescriptor, parameters)` 的返回值 `postActionContext`，这个返回值其实就是 `ActionResult`，会在后面关于到 `IResultFilter` 的执行

紧接着 `InvokeAction` 的执行，调用落定在了 `InvokeActionResultWithFilters(ControllerContext controllerContext, IList<IResultFilter> filters, ActionResult actionResult)` 这个函数身上，其实这个函数的实现和刚刚所说的 `InvokeActionMethodWithFilters(ControllerContext controllerContext, IList<IActionFilter> filters, ActionDescriptor actionDescriptor, IDictionary<string, object> parameters)` 的实现是非常类似的，也是做了两件事情，第一件事就是处理在 `InvokeActionMethodWithFilters` 所返回的 `ActionResult`，其实就是调用相应 `Filter` 的具体步骤，我们先来看下其具体的实现

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual ResultExecutedContext InvokeActionResultWithFilters(ControllerContext controllerContext, IList<IResultFilter> filters, ActionResult actionResult)
	{
		ResultExecutingContext preContext = new ResultExecutingContext(controllerContext, actionResult);

		int startingFilterIndex = 0;
		return InvokeActionResultFilterRecursive(filters, startingFilterIndex, preContext, controllerContext, actionResult);
	}
}
```

在这里紧接着又调用了 `InvokeActionResultFilterRecursive(IList<IResultFilter> filters, int filterIndex, ResultExecutingContext preContext, ControllerContext controllerContext, ActionResult actionResult)`

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	private ResultExecutedContext InvokeActionResultFilterRecursive(IList<IResultFilter> filters, int filterIndex, ResultExecutingContext preContext, ControllerContext controllerContext, ActionResult actionResult)
	{
		// Performance-sensitive

		// For compatbility, the following behavior must be maintained
		//   The OnResultExecuting events must fire in forward order
		//   The InvokeActionResult must then fire
		//   The OnResultExecuted events must fire in reverse order
		//   Earlier filters can process the results and exceptions from the handling of later filters
		// This is achieved by calling recursively and moving through the filter list forwards

		// If there are no more filters to recurse over, create the main result
		if (filterIndex > filters.Count - 1)
		{
			InvokeActionResult(controllerContext, actionResult);
			return new ResultExecutedContext(controllerContext, actionResult, canceled: false, exception: null);
		}

		// More....

		filter.OnResultExecuting(preContext);

		// More....

		try
		{
			// More....

			postContext = InvokeActionResultFilterRecursive(filters, nextFilterIndex, preContext, controllerContext, actionResult);
		}
		catch (ThreadAbortException)
		{
			// More....

			filter.OnResultExecuted(postContext);
			throw;
		}
		catch (Exception ex)
		{
			// More....

			filter.OnResultExecuted(postContext);
			throw;
			// More....
		}

		filter.OnResultExecuted(postContext);
		
		// More....
	}
}
```

<span i="标识1"></span>
在这里同样是省略了很多不必要的代码，但是其思想和 `InvokeActionMethodFilter` 函数是类似的，可以清楚的看到 `IResultFilter` 的执行顺序和处理 `ActionResult` 的步骤 (`InvokeActionResultFilterRecursive`)

还剩下一个 `IExceptionFilter` 未提起，其实他也就是在 `InvokeAction` 发生异常的时候就会调用该 `Filter` 的一些逻辑，这里就跳过不研究它了

关于 `Action 和 Filter` 执行步骤最后再说一下，关于这部分的调用其实较为复杂，看到一个博客园的博友总结了一张关于调用顺序的图，这里也稍微引用以下

![201211100745323404.png](https://i.loli.net/2019/12/11/UftGaqjlmiBr7XD.gif)


<br/>

<span id = "标识4"></span>

### ActionResult 的执行

---

在正式进入正题之前，先来回顾下 `ActionResult` 具体有哪些类型

- System.Web.Mvc.ContentResult
- System.Web.Mvc.EmptyResult 
- System.Web.Mvc.FileResult
- System.Web.Mvc.HttpStatusCodeResult
- System.Web.Mvc.JavaScriptResult
- System.Web.Mvc.JsonResult
- System.Web.Mvc.RedirectResult
- System.Web.Mvc.RedirectToRouteResult
- System.Web.Mvc.ViewResultBase

在本节中，仅针对 `System.Web.Mvc.ViewResultBase` 进行分析

回到正题，接 [上一节](#标识1) 的内容，我们知道 `ActionResult` 的真正执行是在 `InvokeActionResultFilterRecursive` 函数里头执行的，具体体现在 `InvokeActionResult(ControllerContext controllerContext, ActionResult actionResult)` 我们先看下其内部构造

```csharp
public class ControllerActionInvoker : IActionInvoker
{
	protected virtual void InvokeActionResult(ControllerContext controllerContext, ActionResult actionResult)
	{
		actionResult.ExecuteResult(controllerContext);
	}
}
```

在该函数中，紧跟着又调用了 `ActionResult` 的抽象函数 `ExecuteResult`，其具体实现体现在 `ViewResultBase` 这个类身上，我们进入其内部看下到底发生了什么

```csharp
public abstract class ViewResultBase : ActionResult
{
	public override void ExecuteResult(ControllerContext context)
	{
		// More....

		if (String.IsNullOrEmpty(ViewName))
		{
			ViewName = context.RouteData.GetRequiredString("action");
		}

		ViewEngineResult result = null;

		if (View == null)
		{
			result = FindView(context);
			View = result.View;
		}

		TextWriter writer = context.HttpContext.Response.Output;
		ViewContext viewContext = new ViewContext(context, View, ViewData, TempData, writer);
		View.Render(viewContext, writer);

		if (result != null)
		{
			result.ViewEngine.ReleaseView(context, View);
		}
	}
}
```

首先是会先找到视图的名字，在 `ASP.NET MVC` 中，构造控制器所对应的视图必须要和控制器中具体的 `Action` 名字相同，在这里我们也可以发现，其寻找视图名字的方式也就是寻找控制器中具体 `Action` 的名字来实现的，当成功匹配到视图名后，紧接着调用了 `FindView(ControllerContext context)` 函数去具体的匹配视图的物理路劲，需要注意的是，该函数在 `ViewResultBase` 这个类中隶属于一个抽象函数，其具体的实现体现在 `ViewResultBase` 的派生类 `ViewResult` 身上，我们进入其内部看下 `FindView(ControllerContext context)` 这个函数的具体实现

```csharp
 public class ViewResult : ViewResultBase
{
	protected override ViewEngineResult FindView(ControllerContext context)
	{
		ViewEngineResult result = ViewEngineCollection.FindView(context, ViewName, MasterName);
		if (result.View != null)
		{
			return result;
		}

		// we need to generate an exception containing all the locations we searched
		StringBuilder locationsText = new StringBuilder();
		foreach (string location in result.SearchedLocations)
		{
			locationsText.AppendLine();
			locationsText.Append(location);
		}
		throw new InvalidOperationException(String.Format(CultureInfo.CurrentCulture,
														  MvcResources.Common_ViewNotFound, ViewName, locationsText));
	}
}
```

<span id="标识2"></span>
调用又落定在 `ViewEngineCollection` 这个类的 `FindView(ControllerContext controllerContext, string viewName, string masterName)` 函数身上，在这里我们先关注下 `ViewEngineCollection` 这个类，`ViewEngineCollection` 在默认情况下有两个元素，分别是 `WebFormViewEngine` 和 `RazorViewEngine`，在这里我们只关注 `RazorViewEngine`，而关于 `RazorViewEngine` 的构造函数似乎又出现了一些比较有意思的东西，见下面代码

```csharp
public class RazorViewEngine : BuildManagerViewEngine
{
	public RazorViewEngine(IViewPageActivator viewPageActivator)
		: base(viewPageActivator)
	{
		AreaViewLocationFormats = new[]
		{
			"~/Areas/{2}/Views/{1}/{0}.cshtml",
			"~/Areas/{2}/Views/{1}/{0}.vbhtml",
			"~/Areas/{2}/Views/Shared/{0}.cshtml",
			"~/Areas/{2}/Views/Shared/{0}.vbhtml"
		};
		AreaMasterLocationFormats = new[]
		{
			"~/Areas/{2}/Views/{1}/{0}.cshtml",
			"~/Areas/{2}/Views/{1}/{0}.vbhtml",
			"~/Areas/{2}/Views/Shared/{0}.cshtml",
			"~/Areas/{2}/Views/Shared/{0}.vbhtml"
		};
		AreaPartialViewLocationFormats = new[]
		{
			"~/Areas/{2}/Views/{1}/{0}.cshtml",
			"~/Areas/{2}/Views/{1}/{0}.vbhtml",
			"~/Areas/{2}/Views/Shared/{0}.cshtml",
			"~/Areas/{2}/Views/Shared/{0}.vbhtml"
		};

		ViewLocationFormats = new[]
		{
			"~/Views/{1}/{0}.cshtml",
			"~/Views/{1}/{0}.vbhtml",
			"~/Views/Shared/{0}.cshtml",
			"~/Views/Shared/{0}.vbhtml"
		};
		MasterLocationFormats = new[]
		{
			"~/Views/{1}/{0}.cshtml",
			"~/Views/{1}/{0}.vbhtml",
			"~/Views/Shared/{0}.cshtml",
			"~/Views/Shared/{0}.vbhtml"
		};
		PartialViewLocationFormats = new[]
		{
			"~/Views/{1}/{0}.cshtml",
			"~/Views/{1}/{0}.vbhtml",
			"~/Views/Shared/{0}.cshtml",
			"~/Views/Shared/{0}.vbhtml"
		};

		FileExtensions = new[]
		{
			"cshtml",
			"vbhtml",
		};
	}
}
```

我们会发现，`RazorViewEngine` 在初始化的过程中，为其最上级基类的 `Area view location` 和 `Default view loaction` 中注册了相应的一些磁盘物理路径，通过这点其实也能够看出 `ASP.NET MVC` 框架在寻找视图的具体路径时候的顺序，此外需要补充的是这些 `xxxLocationFormats` 是隶属于 `RazorViewEngine` 的最上级基类 `VirtualPathProviderViewEngine` 中的属性成员

说了这么多，回到 `ViewEngineCollection` 的 `FindView` 函数，先看看其具体的实现

```csharp
public class ViewEngineCollection : Collection<IViewEngine>
{
	public virtual ViewEngineResult FindView(ControllerContext controllerContext, string viewName, string masterName)
	{
		// More....

		return Find(e => e.FindView(controllerContext, viewName, masterName, true),
					e => e.FindView(controllerContext, viewName, masterName, false));
	}
}
```

在这里我们会发现又调用了 `Find` 函数，不过我们其实是不需要研究其内部的实现的，前面说到 `ViewEngineCollection` 在默认情况下有两个元素，分别是 `WebFormViewEngine` 和 `RazorViewEngine`，那么这两个 `e` 我们也可以理解为对应的 `ViewEngine` 的实现，需要注意的是，这里并不是属于对应的 `ViewEngine` 的实现，而是隶属于他们共同的最终父类 `VirtualPathProviderViewEngine` 的实现，只是它们继承了共同的最上级基类，这属于抽象化的一种体现，废话不多说，我们来看下 `VirtualPathProviderViewEngine.FindView` 函数的具体实现

<span id = "标识3"></span>

```csharp
public abstract class VirtualPathProviderViewEngine : IViewEngine
{
	public virtual ViewEngineResult FindView(ControllerContext controllerContext, string viewName, string masterName, bool useCache)
	{
		// More....

		string[] viewLocationsSearched;
		string[] masterLocationsSearched;

		string controllerName = controllerContext.RouteData.GetRequiredString("controller");
		string viewPath = GetPath(controllerContext, ViewLocationFormats, AreaViewLocationFormats, "ViewLocationFormats", viewName, controllerName, CacheKeyPrefixView, useCache, out viewLocationsSearched);
		string masterPath = GetPath(controllerContext, MasterLocationFormats, AreaMasterLocationFormats, "MasterLocationFormats", masterName, controllerName, CacheKeyPrefixMaster, useCache, out masterLocationsSearched);

		if (String.IsNullOrEmpty(viewPath) || (String.IsNullOrEmpty(masterPath) && !String.IsNullOrEmpty(masterName)))
		{
			return new ViewEngineResult(viewLocationsSearched.Union(masterLocationsSearched));
		}

		return new ViewEngineResult(CreateView(controllerContext, viewPath, masterPath), this);
	}
}
```

在这里直观上看可以发现通过 `GetPath(ControllerContext controllerContext, string[] locations, string[] areaLocations, string locationsPropertyName, string name, string controllerName, string cacheKeyPrefix, bool useCache, out string[] searchedLocations)` 函数获得了两种不同的路劲，这里我们只需要关注 `viewPath` 的返回结果即可，其次，我们会发现调用这个 `GetPath` 函数的时候会把 [前面说到的]("#标识2") 的所注册的  `Area view location` 和 `Default view loaction` 都传进去了，我们先进入其内部看看具体的实现

```csharp
public abstract class VirtualPathProviderViewEngine : IViewEngine
{
	private string GetPath(ControllerContext controllerContext, string[] locations, string[] areaLocations, string locationsPropertyName, string name, string controllerName, string cacheKeyPrefix, bool useCache, out string[] searchedLocations)
	{
		searchedLocations = _emptyLocations;

		// More....

		string areaName = AreaHelpers.GetAreaName(controllerContext.RouteData);
		bool usingAreas = !String.IsNullOrEmpty(areaName);
		List<ViewLocation> viewLocations = GetViewLocations(locations, (usingAreas) ? areaLocations : null);

		if (viewLocations.Count == 0)
		{
			throw new InvalidOperationException(String.Format(CultureInfo.CurrentCulture,
																MvcResources.Common_PropertyCannotBeNullOrEmpty, locationsPropertyName));
		}

		bool nameRepresentsPath = IsSpecificPath(name);
		string cacheKey = CreateCacheKey(cacheKeyPrefix, name, (nameRepresentsPath) ? String.Empty : controllerName, areaName);

		if (useCache)
		{
			// More....
		}
		else
		{
			return nameRepresentsPath
				? GetPathFromSpecificName(controllerContext, name, cacheKey, ref searchedLocations)
				: GetPathFromGeneralName(controllerContext, viewLocations, name, controllerName, areaName, cacheKey, ref searchedLocations);
		}
	}
}
```

在这里其实还运用到了缓存的机制，通过访问缓存来找物理路径，不过这里我们不参与其具体的实现，我们先把关注点先从上面开始

首先是查看当前 `Controller` 是不是在 `Area` 中的，接下来就会获取 `GetViewLocations(string[] viewLocationFormats, string[] areaViewLocationFormats)` 来获取视图具体的位置，我们先看下该函数的实现

```csharp
public abstract class VirtualPathProviderViewEngine : IViewEngine
{
	private static List<ViewLocation> GetViewLocations(string[] viewLocationFormats, string[] areaViewLocationFormats)
	{
		List<ViewLocation> allLocations = new List<ViewLocation>();

		if (areaViewLocationFormats != null)
		{
			foreach (string areaViewLocationFormat in areaViewLocationFormats)
			{
				allLocations.Add(new AreaAwareViewLocation(areaViewLocationFormat));
			}
		}

		if (viewLocationFormats != null)
		{
			foreach (string viewLocationFormat in viewLocationFormats)
			{
				allLocations.Add(new ViewLocation(viewLocationFormat));
			}
		}

		return allLocations;
	}
}
```

在该函数中，会把 `Controller` 中所有 `Action` 的路径都会放在一个集合当中，包括 `Area` 的，在这里其实我们也可以发现一些隐藏的比较有意思的事情，比如说我们在 `Area` 的控制器中具体某个 `Action` 的视图并不在 `Area` 对应视图的目录下，但是却在默认的 `Views` 文件夹下，那么也能够正确查找到对应视图名字的视图文件的，因为这里的路径实现实际上包括了 `Area location` 和 `View location`

回到 `VirtualPathProviderViewEngine` 中，在初始化完成物理路径的集合后，会通过 `GetPathFromGeneralName(ControllerContext controllerContext, List<ViewLocation> locations, string name, string controllerName, string areaName, string cacheKey, ref string[] searchedLocations)` 来为路径做一些格式化操作，具体就是把虚拟路径映射为具体的物理路径的操作

至此，我们可以把目光回到 `VirtualPathProviderViewEngine.FindView` 函数身上，具体代码在[这里](#标识3)，在获取完成视图的物理路径后，紧接着会调用 `CreateView(ControllerContext controllerContext, string viewPath, string masterPath)` 去构建一个视图对象，需要注意的是，这个函数也是一个抽象函数，其具体的实现体现在其对应的派生类 `ViewEngine` 当中，我们进入其内部一探究竟

```csharp
public class RazorViewEngine : BuildManagerViewEngine
{
	protected override IView CreateView(ControllerContext controllerContext, string viewPath, string masterPath)
	{
		var view = new RazorView(controllerContext, viewPath,
									layoutPath: masterPath, runViewStartPages: true, viewStartFileExtensions: FileExtensions, viewPageActivator: ViewPageActivator)
		{
			DisplayModeProvider = DisplayModeProvider
		};
		return view;
	}
}
```

其实意思也很简单，只是根据具体的路径生成了一个实现于 `IView` 接口的具体视图对象罢了，在这里由于该篇文章是围绕着 `Razor` 所展开，所以这里是 `RazorView` 对象

最后又把生成的视图对象再封装至 `ViewEngineResult` 这个类当中，并携带着这个结果逐步退回到 `ViewResultBase.ExecuteResult` 函数身上，开始利用所生成的视图对象 `RazorView` 去完成一些 `HTML` 的渲染工作 


<br/>

<span id = "View的渲染"></span>

### View 的渲染

---

为什么要把视图的渲染这一模块的的内容要和上一节的 [ActionResult 的执行](#标识4) 的内容要分开来呢？ 其实本身视图的渲染工作适合 `ActionResult` 的执行是密不可分的，要具体来讲他们的步骤的话则是 `ActionResult` 的执行才带来了视图的渲染工作，但是关于视图渲染部分的内容在源码中所带给我们的体现是挺抽象的，并且竟然他作为 `ASP.NET MVC Life Cycle` 这篇文章的末尾，那我需要好好的对它进行分析一番，废话不多说，咱们进入正题

承接上文，上面说到在成功获取到一个 `RazorView` 后就开始利用它去完成 `HTML` 的渲染工作，为了上下文的承接性，我再上一次该函数的代码

```csharp
public abstract class ViewResultBase : ActionResult
{
	public override void ExecuteResult(ControllerContext context)
	{
		if (context == null)
		{
			throw new ArgumentNullException("context");
		}
		if (String.IsNullOrEmpty(ViewName))
		{
			ViewName = context.RouteData.GetRequiredString("action");
		}

		ViewEngineResult result = null;

		if (View == null)
		{
			result = FindView(context);
			View = result.View;
		}

		TextWriter writer = context.HttpContext.Response.Output;
		ViewContext viewContext = new ViewContext(context, View, ViewData, TempData, writer);
		View.Render(viewContext, writer);

		if (result != null)
		{
			result.ViewEngine.ReleaseView(context, View);
		}
	}
}
```

在获取完成一个 `View` ( `RazorView` ) 后，紧接着创建了一个 `TextWriter` 和 `ViewContext`，关于 `TextWriter` 没什么可说，就是一个字符串输出器，而 `ViewContext` 我们可以把它理解成做一些初始化的信息，并把一些初始化信息 (`HttpContext`, `IView`, ……) 都封装在一起，接下来就开始调用 `RazorView` 的 `Render(ViewContext viewContext, TextWriter writer)` 具体来讲的话，该函数应该是属于 `RazorView` 的基类 `BuildManagerCompiledView` 中的函数才对，我们先看下其实现

```csharp
public abstract class BuildManagerCompiledView : IView
{
	public virtual void Render(ViewContext viewContext, TextWriter writer)
	{
		// More....

		object instance = null;

		Type type = BuildManager.GetCompiledType(ViewPath);
		if (type != null)
		{
			instance = ViewPageActivator.Create(_controllerContext, type);
		}

		if (instance == null)
		{
			throw new InvalidOperationException(
				String.Format(
					CultureInfo.CurrentCulture,
					MvcResources.CshtmlView_ViewCouldNotBeCreated,
					ViewPath));
		}

		RenderView(viewContext, writer, instance);
	}
}
```

在这里其实我们需要了解另一个信息，`ASP.NET MVC` 和 `WebForm` 一样，我们所写入的视图文件其实都是属于一个具体的类型，当我们第一次请求一个视图页的时候，`.NET Runtime` 就会统一编译一次当前项目下的所有视图文件，并把它们打包至一个命名空间内，所编译后生成的 `dll` 会放在 `ASP.NET` 的临时目录下，关于该临时目录如何查找这里不做阐述，在这个命名空间类下所包含的不同的类对应着我们不同的视图页面，需要注意的是，`View` `StartPage` `LayoutPage` 都会生成不同的 `dll` ，并且其所继承的基类也是不同，关于 `View` 和 `LayoutPage` 所生成的类 都是继承于 `WebViewPage<T>` 的，而 `WebViewPage<T>` 又派生自 `WebViewPage`，暂且记住这个类型，在后面起到非常关键的作用，而 `StartPage` 所生成的类则是继承于 `ViewStartPage`，暂且也要记住这个类，后面关于 `StartPage` 的生成也有关键作用

我们先来看一下这些 `dll` 在反编译后的具体结构

- View <span id="ViewPic"></span>
![微信截图_20191212171523.png](https://i.loli.net/2019/12/12/AYDIH4zKsN1wgRm.png)

- StartPage <span id="startPagePic"></span>
![微信截图_20191212232735.png](https://i.loli.net/2019/12/12/RxPWKhb6FG2HufY.png)

- LayoutPage
![微信截图_20191212232813.png](https://i.loli.net/2019/12/12/AZUwOXdzt7lIGLJ.png)


竟然知道视图文件都会统一编译成一个 `dll` 并且都对应着相应的类型这个信息后，我们再来看一下 `Render` 函数中的这一句 `Type type = BuildManager.GetCompiledType(ViewPath)` 这一句也同样是印证了刚刚的说法，没错，这里就是在系统中找到对应的临时dll的路径，将他反射出来并拿到相应的 `Type`，当然这里的 `Type` 仅仅是当前请求 URL 中 `Action` 所对应的视图文件的某个类而已，在拿到具体某个类的 `Type` 后，会调用一个 `IViewPageActivator` 的 `Create` 方法来创建实例，这里采用了依赖注入的手法，但是在默认情况下，也只是调用反射来创建一个实例而已，在 `ASP.NET MVC` 框架中，这种地方已经出现多次了

当视图文件所对应的 `Type` 的实例创建成功后，紧接着又调用了 `RenderView(ViewContext viewContext, TextWriter writer, object instance)` 函数，但是这个函数在 `BuildManagerCompiledView` 中是一个抽象体现，其具体的实现在 `BuildManagerCompiledView` 的派生类 `RazorView` 身上，我们先看下其具体的实现

```csharp
public class RazorView : BuildManagerCompiledView
{
	protected override void RenderView(ViewContext viewContext, TextWriter writer, object instance)
	{
		// More....

		WebViewPage webViewPage = instance as WebViewPage;
		if (webViewPage == null)
		{
			throw new InvalidOperationException(
				String.Format(
					CultureInfo.CurrentCulture,
					MvcResources.CshtmlView_WrongViewBase,
					ViewPath));
		}

		// An overriden master layout might have been specified when the ViewActionResult got returned.
		// We need to hold on to it so that we can set it on the inner page once it has executed.
		webViewPage.OverridenLayoutPath = LayoutPath;
		webViewPage.VirtualPath = ViewPath;
		webViewPage.ViewContext = viewContext;
		webViewPage.ViewData = viewContext.ViewData;

		webViewPage.InitHelpers();

		// More....

		WebPageRenderingBase startPage = null;
		if (RunViewStartPages)
		{
			startPage = StartPageLookup(webViewPage, RazorViewEngine.ViewStartFileName, ViewStartFileExtensions);
		}
		webViewPage.ExecutePageHierarchy(new WebPageContext(context: viewContext.HttpContext, page: null, model: null), writer, startPage);
	}
}
```

首先，我们可以发现把当视图文件所对应的 `Type` 的实例转换成了其基类 `WebViewPage` 类型，然后我们把关注点放在 `StartPageLookup` 这个函数身上，我们先看下其 `RazorViewEngine.ViewStartFileName` 这个参数的值是什么

```csharp
public class RazorViewEngine : BuildManagerViewEngine
{
	internal static readonly string ViewStartFileName = "_ViewStart";
}
```

通过这点我们也可以大胆的去猜测，该函数其具体作用就是寻找我们的 `_ViewStart.cshtml` 所生成的临时类加载并渲染其逻辑，我们进去其内部看下它的具体实现

```csharp
public abstract class StartPage : WebPageRenderingBase
{
	internal static WebPageRenderingBase GetStartPage(WebPageRenderingBase page, IVirtualPathFactory virtualPathFactory, string appDomainAppVirtualPath,
															string fileName, IEnumerable<string> supportedExtensions)
	{
		// Build up a list of pages to execute, such as one of the following:
		// ~/somepage.cshtml
		// ~/_pageStart.cshtml --> ~/somepage.cshtml
		// ~/_pageStart.cshtml --> ~/sub/_pageStart.cshtml --> ~/sub/somepage.cshtml
		WebPageRenderingBase currentPage = page;
		var pageDirectory = VirtualPathUtility.GetDirectory(page.VirtualPath);

		// Start with the requested page's directory, find the init page,
		// and then traverse up the hierarchy to find init pages all the
		// way up to the root of the app.
		while (!String.IsNullOrEmpty(pageDirectory) && pageDirectory != "/" && PathUtil.IsWithinAppRoot(appDomainAppVirtualPath, pageDirectory))
		{
			// Go through the list of supported extensions
			foreach (var extension in supportedExtensions)
			{
				var virtualPath = VirtualPathUtility.Combine(pageDirectory, fileName + "." + extension);

				// Can we build a file from the current path?
				if (virtualPathFactory.Exists(virtualPath))
				{
					var parentStartPage = virtualPathFactory.CreateInstance<StartPage>(virtualPath);
					parentStartPage.VirtualPath = virtualPath;
					parentStartPage.ChildPage = currentPage;
					parentStartPage.VirtualPathFactory = virtualPathFactory;
					currentPage = parentStartPage;

					break;
				}
			}

			pageDirectory = currentPage.GetDirectory(pageDirectory);
		}

		// At this point 'currentPage' is the root-most StartPage (if there were
		// any StartPages at all) or it is the requested page itself.
		return currentPage;
	}
}
```

<span id="标注6"></span>
结合代码和注释，我们可以总结出 `ASP.NET MVC` 关于 `_ViewStart.cshtml` 这个文件的查找规则，首先从当前请求 `URL` 指向的 `Action` 所对应的视图文件的目录开始查找，以此向上层开始遍历，如果找到了的话就创建这个 `_ViewStart.cshtml` 的实例 `StartPage`，并且把当前请求 `URL` 指向的 `Action` 所对应的 `WebViewPage` 封装至 `StartPage` 的 `ChildPage` 属性中，需要关注的地方就这两点，需要特别留意的还是那个 `ChildPage` 属性，这里，我们先暂时返回到 `RenderView(ViewContext viewContext, TextWriter writer, object instance)` 函数中，在获取完成 `StartPage` 后，紧接着调用来到了 `ExecutePageHierarchy(WebPageContext pageContext, TextWriter writer, WebPageRenderingBase startPage)` 函数的调用

在开始进入这个函数之前，我们先说两句题外话，我们先理解下这个函数的意思 `ExecutePageHierarchy` 页面层次执行，那么在 `ASP.NET MVC` 中，什么又是页面的层级关系呢？我先来举个例子，假设目前我们有三个 `cshtml` 文件，它们分别是

- View

```c#
@{
    ViewBag.Title = "Code in View";
}
```

- _ViewStart.cshtml

```csharp
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}
```

- _Layout.cshtml

```csharp
@{ 
    ViewBag.ToView = "Data from Layout";
}
<div>
    Data In View: @ViewBag.Title
</div>
<div>
    @RenderBody();    
</div>
```

<span id = "标注9"></span>
在这样子的方式下，我们会发现 `_Layout.cshtml` 中的 `Data In View` 是能够正确打印出来的，这点无可厚非，那么如果我们换一个角度，要在 `View` 界面上输出 `ViewBag.ToView` 呢？答案是什么都没有，从这点其实我们也可以看出其加载顺序是从 `StartPage` -> `Page` -> `LayoutPage`，在这里我们就可以认为是一种层次的关系

好了解释完成层次关系后，我们正式进入到 `ExecutePageHierarchy(WebPageContext pageContext, TextWriter writer, WebPageRenderingBase startPage)` 函数的内部实现中去，在这里也稍微扩充一下，虽然该函数是通过 `WebViewPage` 对象所调用，但是其具体的实现还是在它的基类 `WebPageBase` 身上
```csharp
public abstract class WebPageBase : WebPageRenderingBase
{
	public void ExecutePageHierarchy(WebPageContext pageContext, TextWriter writer, WebPageRenderingBase startPage)
	{
		PushContext(pageContext, writer);

		if (startPage != null)
		{
			if (startPage != this)
			{
				var startPageContext = WebPageContext.CreateNestedPageContext<object>(parentContext: pageContext, pageData: null, model: null, isLayoutPage: false);
				startPageContext.Page = startPage;
				startPage.PageContext = startPageContext;
			}
			startPage.ExecutePageHierarchy();
		}
		else
		{
			ExecutePageHierarchy();
		}
		PopContext();
	}
}
```

在这里我们可以看到几个熟悉的字样 `Push` 和 `Pop`，那么我们可以理解为使用了 栈 的技术，前面说到层级关系的具体含义，微软为了解决这一问题也是使用了 栈 的技术，在这里其实可以总结为一共使用了两种 栈 ，分别是 `OutputStack`，它的职责主要是 <span style="color:red">压入渲染HTML的TextWriter</span>，其次这只是一个页面的处理过程，一个页面之中还会有 `Partial View` 和  `Action` 等，这些的处理方式都是一样的，因此还需要一个栈来记录处理到了哪个（子）页面，因此还有一个栈，称之为 `TemplateStack`，<span style="color:red">里面压入的是PageContext，PageContext维护了view的必要信息，比如Model之类的，当然也包括上面提到的OutputStack</span>

在这里我们可以先看一下其入栈 `PushContext(WebPageContext pageContext, TextWriter writer)` 的实现

```csharp
public abstract class WebPageBase : WebPageRenderingBase
{
	public void PushContext(WebPageContext pageContext, TextWriter writer)
	{
		_currentWriter = writer;
		PageContext = pageContext;
		pageContext.Page = this;

		InitializePage();

		// Create a temporary writer
		_tempWriter = new StringWriter(CultureInfo.InvariantCulture);

		// Render the page into it
		OutputStack.Push(_tempWriter);
		SectionWritersStack.Push(new Dictionary<string, SectionWriter>(StringComparer.OrdinalIgnoreCase));

		// If the body is defined in the ViewData, remove it and store it on the instance
		// so that it won't affect rendering of partial pages when they call VerifyRenderedBodyOrSections
		if (PageContext.BodyAction != null)
		{
			_body = PageContext.BodyAction;
			PageContext.BodyAction = null;
		}
	}
}
```

入栈成功后，紧接着我们回到 `WebPageBase.ExecutePageHierarchy` 函数身上，下面有判断了一次是否由 [上面](#标注6) 所说的生成的 `StartPage`，如果存在，则调用其所实现的 `ExecutePageHierarchy` 函数了，需要注意的是，这个 `StartPage` 这个类和上面所说的 `WebPageBase` 其实都继承了相同的基类 `WebPageRenderingBase` (在这个基类的基础上又继承了 `WebPageExecutingBase`) ，而 `ExecutePageHierarchy` 是隶属于 `WebPageRenderingBase` 的一个抽象函数，在这里 `StartPage` 实现了这个抽象函数，需要搞清楚的是 `WebPageBase` 虽然也有一个 `ExecutePageHierarchy` 函数，但是其实现并不是实现于其父类中的抽象函数 `WebPageRenderingBase`，而是属于 `WebPageBase` 自己的函数成员，而关于 `WebPageBase` 在哪里实现了其基类的 `WebPageRenderingBase` 的抽象函数的问题，答案就在 `WebPageBase` 的派生类 `WebViewPage` 身上，说了这么多，我们还是先看下 `StartPage.ExecutePageHierarchy` 函数的具体实现 

```csharp
public abstract class StartPage : WebPageRenderingBase
{
	public override void ExecutePageHierarchy()
	{
		// Push the current pagestart on the stack. 
		TemplateStack.Push(Context, this);
		try
		{
			// Execute the developer-written code of the InitPage
			Execute();

			// If the child page wasn't explicitly run by the developer of the InitPage, then run it now.
			// The child page is either the next InitPage, or the final WebPage.
			if (!RunPageCalled)
			{
				RunPage();
			}
		}
		finally
		{
			TemplateStack.Pop(Context);
		}
	}
}
```

在这里我们又看到了刚刚提到的 `TemplateStack` 栈，并且调用了 `StartPage` 所 [临时生成的类](#startPagePic) 的 `Execute()` 函数以完成里面逻辑的加载，当加载完成后又调用了 `RunPage()` 函数，我们看下其内部的实现

```csharp
public abstract class StartPage : WebPageRenderingBase
{
	public void RunPage()
	{
		RunPageCalled = true;
		ChildPage.ExecutePageHierarchy();
	}
}
```

在这里，似乎又看到了一个在 [前面]("#标注6") 说到的熟悉的成员 `ChildPage`，我们知道 `ChildPage` 其实就是当前所请求 URL 指向的 `Action` 所对应视图的 `WebViePage`，并且也说过 `WebViewPage` 的 `ExecutePageHierarchy` 函数其实是实现于 `WebPageRenderingBase` 的抽象函数 `ExecutePageHierarchy`，那么我们进入其 `ExecutePageHierarchy` 看看发生了什么

```csharp
public abstract class WebViewPage : WebPageBase, IViewDataContainer, IViewStartPageChild
{
	public override void ExecutePageHierarchy()
	{
		// Change the Writer so that things like Html.BeginForm work correctly
		TextWriter oldWriter = ViewContext.Writer;
		ViewContext.Writer = Output;

		base.ExecutePageHierarchy();

		// Overwrite LayoutPage so that returning a view with a custom master page works.
		if (!String.IsNullOrEmpty(OverridenLayoutPath))
		{
			Layout = OverridenLayoutPath;
		}

		// Restore the old View Context Writer
		ViewContext.Writer = oldWriter;
	}
}
```

在这里，我们发现如果在渲染当前所请求 URL 指向的 `Action` 所对应视图的 `WebViePage` 中指定了 `Layout` 的话，会为 `WebViewPage` 的 `Layout` 属性赋值，除此之外，在这里我们发现它又调用了其父类的 `ExecutePageHierarchy` 的重载函数，我们进入其内部看看

```csharp

public override void ExecutePageHierarchy()
{
	// Unlike InitPages, for a WebPage there is no hierarchy - it is always
	// the last file to execute in the chain. There can still be layout pages
	// and partial pages, but they are never part of the hierarchy.

	// (add server header for falcon debugging)
	// call to MapPath() is expensive. If we are not emiting source files to header, 
	// don't bother to populate the SourceFiles collection. This saves perf significantly.
	
	// More....

	TemplateStack.Push(Context, this);
	try
	{
		// Execute the developer-written code of the WebPage
		Execute();
	}
	finally
	{
		TemplateStack.Pop(Context);
	}
}
```

在这里又看到了对于 `TemplateStack` 栈的操作，并且调用了 `View` 所 [临时生成的类](#ViewPic) 的 `Execute()` 函数以完成里面逻辑的加载

至此，我们可以回到 `WebPageBase` 的 `ExecutePageHierarchy(WebPageContext pageContext, TextWriter writer, WebPageRenderingBase startPage)` 函数身上，我们关注下最后的 出栈 操作

```csharp
public abstract class WebPageBase : WebPageRenderingBase
{
	public void PopContext()
	{
		// Using the CopyTo extension method on the _tempWriter instead of .ToString()
		// to avoid allocating large strings that then end up on the Large object heap.
		OutputStack.Pop();

		if (!String.IsNullOrEmpty(Layout))
		{
			string layoutPagePath = NormalizeLayoutPagePath(Layout);

			// If a layout file was specified, render it passing our page content.
			OutputStack.Push(_currentWriter);
			RenderSurrounding(
				layoutPagePath,
				_tempWriter.CopyTo);
			OutputStack.Pop();
		}
		else
		{
			// Otherwise, just render the page.
			_tempWriter.CopyTo(_currentWriter);
		}

		VerifyRenderedBodyOrSections();
		SectionWritersStack.Pop();
	}
}
```

也是和压栈一样差不多的逻辑，但是这里换成了出栈，这里我们仔细观察会发现，只有当前面提到 `Layout` 不为空的时候才进入以下的逻辑，那么我就可以大胆的猜测，在该代码块的内部其实就是渲染 `LayoutPage` 的一个工作，我们把关注点放在 `RenderSurrounding(
layoutPagePath,_tempWriter.CopyTo)` 函数身上，我们发现有一个参数是 `layoutPagePath`，这时候也能更加肯定自己的猜测了，但是我们还是进入其内部看看具体的操作

```csharp
public abstract class WebPageBase : WebPageRenderingBase
{
	private void RenderSurrounding(string partialViewName, Action<TextWriter> body)
	{
		// Save the previous body action and set ours instead.
		// This value will be retrieved by the sub-page being rendered when it runs
		// Render(ViewData, TextWriter).
		var priorValue = PageContext.BodyAction;
		PageContext.BodyAction = body;

		// Render the layout file
		Write(RenderPageCore(partialViewName, isLayoutPage: true, data: new object[0]));

		// Restore the state
		PageContext.BodyAction = priorValue;
	}
}
```

继续进入内部看下 `RenderPageCore(partialViewName, isLayoutPage: true, data: new object[0])` 的下一步调用

```csharp
public abstract class WebPageBase : WebPageRenderingBase
{
	private HelperResult RenderPageCore(string path, bool isLayoutPage, object[] data)
	{
		if (String.IsNullOrEmpty(path))
		{
			throw new ArgumentException(CommonResources.Argument_Cannot_Be_Null_Or_Empty, "path");
		}

		return new HelperResult(writer =>
		{
			path = NormalizePath(path);
			WebPageBase subPage = CreatePageFromVirtualPath(path, Context, VirtualPathFactory.Exists, DisplayModeProvider, DisplayMode);
			var pageContext = CreatePageContextFromParameters(isLayoutPage, data);

			subPage.ConfigurePage(this);
			subPage.ExecutePageHierarchy(pageContext, writer);
		});
	}
}
```

在这里我们可以看到其根据 `LayoutPage` 的具体路径又创建了一个 `WebPageBase`，并且通过它又调用了一次 `ExecutePageHierarchy` 函数，以这种递归的方式完成 `LayoutPage` 的视图渲染和加载，关于其 `ExecutePageHierarchy` 函数内部的逻辑和 [前面](#标注9) 所说的是一模一样的，只是之前的角色是通过 `View` 的 `WebViewPage` 去实现，而这里换成了 `LayoutPage` 的 `WebViewPage` 去实现罢了

至此我们 `ASP.NET MVC Life Cycle` 的整体流程就已经结束了，但还是跟我最后一次慢慢退出这些函数

- `WebPageBase.ExecutePageHierarchy(WebPageContext pageContext, TextWriter writer, WebPageRenderingBase startPage)`
- `RazorView.RenderView(ViewContext viewContext, TextWriter writer, object instance)`
- `BuildManagerCompiledView.Render(ViewContext viewContext, TextWriter writer)`
- `ViewResultBase.ExecuteResult(ControllerContext context)`
- `ControllerActionInvoker.InvokeActionResult(ControllerContext controllerContext, ActionResult actionResult)`
- 回到 [ActionResult 的执行](#标识4)


<br/>

### 参考文档

---

- [深入ASP.NET MVC 之一：IIS到路由表](https://www.cnblogs.com/yinzixin/archive/2012/10/30/2745713.html)
- [深入ASP.NET MVC之二：路由模块如何工作](https://www.cnblogs.com/yinzixin/archive/2012/11/05/2754483.html#)
- [深入ASP.NET MVC之三：Controller的激活](https://www.cnblogs.com/yinzixin/archive/2012/11/06/2754484.html)
- [深入ASP.NET MVC之四：Filter和Action的执行](https://www.cnblogs.com/yinzixin/archive/2012/11/10/2763625.html)
- [深入ASP.NET MVC之七：ActionResult的执行（View的加载和渲染）](https://www.cnblogs.com/yinzixin/archive/2012/12/05/2799459.html)
- [ASP.Net请求处理机制初步探索之旅 - Part 2 核心](https://www.cnblogs.com/edisonchou/p/4195259.html)
- [ASP.Net请求处理机制初步探索之旅 - Part 3 管道](https://www.cnblogs.com/edisonchou/p/4201855.html)
- [ASP.Net请求处理机制初步探索之旅 - Part 5 ASP.Net MVC请求处理流程](https://www.cnblogs.com/edisonchou/p/4226558.html)