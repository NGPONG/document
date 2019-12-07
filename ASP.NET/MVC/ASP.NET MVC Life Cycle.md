# *<span>ASP.NET</span> MVC Life Cycle*

<br/>

### 路由的开始

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

此外还要补充一点的是，`RouteData` 中存在一个属性 `RouteHandler`，其类型也是 `MvcRouteHandler`，并且其引用是和生成这个 `RouteData` 的 `Route` 是存在着对应关系的

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

### Area 的注册

---

`ASP.NET MVC Area` 可以将一个大型的网站划分为相对独立的区域，该区域同样享有 `Model` `Controller` `View` 的结构模式，并且也有着属于自己的一个 `web.config` 和路由规则，关于如何创建 `Area` 可以参考这一篇[文章](https://www.codeproject.com/Articles/1139669/How-to-Create-an-Area-in-ASP-NET-MVC-Application)

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

关于 `DataToken` 还是有必要稍微展开讲一下，故名起义，这个成员就是属于 `Route` 的一个Token，其目的主要是为了避免 `Area` 和原始的母 MVC 项目存在同名 `Controller` 的情况，可以看出其实际上只有在 `Area` 中能够达到实际作用

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

### Controller的激活

---


