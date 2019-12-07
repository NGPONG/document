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

可以清楚的看到，`MapRoute` 函数其实是属于 `RouteCollection` 的一个 [扩展方法](https://github.com/NGPONG/document/blob/a9dac43935045a7a03e7a7c7453699d4b2adb9fe/c%23/2019%E5%B9%B4/8%E6%9C%884%E6%97%A5/%E6%89%A9%E5%B1%95%E6%96%B9%E6%B3%95.md)

