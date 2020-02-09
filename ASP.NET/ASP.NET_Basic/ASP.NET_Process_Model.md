# *<span>ASP.NET</span> Process Model*

<br/>

### 开始前，对一些其他的术语进行认识

---

#### 1. HTTP.SYS

- 运行在 `Kernel Model` 中的一个组件 ,它负责侦听 [ Listen ] 来自于外部的HTTP请求,根据请求的URL将其分发给相应的应用程序池
- HTTP监听器，是 `TCP/IP` 网络子程序的一部分，用于持续监听HTTP请求
- 为了提供更好的性能， HTTP.SYS 内部建立了一个缓冲区，<span style="color:red">静态的内容现在被缓存于内核模式下，将最近的HTTP请求处理结果保存起来</span>，这使服务响应速度更快
- 所有的服务请求会在 HTTP.SYS 里暂存入队列，而不是由服务程序本身来处理，这样，即使服务程序重启，尚未被处理的请求也不会丢失了
- 只要是通过 HTTP.SYS 管理的端口(基本包括了那些著名的端口，比如80)，都支持端口重用

#### 2. Application Pool 应用程序池

- Application Pool 是将一个或多个应用程序链接到一个或多个工作进程集合的配置
- 每个应用程序池都对应着一个属于它的 `Worker Process` (工作者进程)
- 应用程序池的概念出现在 `IIS 6.0` 当中，而 `IIS 5.0` 中所使用的还是 `Application Domain` 的处理机制
- 一个资源文件请求至服务器时，HTTP.SYS 会根据IIS中的 `Metabase` 查看基于该 Request 的 Resource 属于哪个 Application Pool，如果不存在则创建之

#### 3. Application Domain 应用程序域

- 一个应用程序在其中执行的独立环境，为执行托管代码提供隔离、卸载和安全边界
- 一个工作者进程中承载着一个或多个的应用程序域，这由我们在IIS上在部署网站的时候的配置选择有关
- 当一个请求到达 `ISAPI Extension` 进行初始化CLR、构建 `.NET Framework Runtime` 时候，这个过程其实就是在为WEB应用程序构建一个应用程序域

#### 4. Worker Process 工作者进程

- 运行在一个 <span style="color:res">非托管环境</span> 下的进程
- 工作者进程在不同的IIS版本所对应的文件是不同的
    - IIS 6.0：w3wp.exe
    - IIS 5.0：aspnet_wp.exe
- 一个应用程序池对应一个工作者进程，当一个应用程序池在创建的时候，`WAS Web Administrative service` 会根据 IIS 在 `Metabase` 中所维护的 `Application Pool` 和 Worker Process 的 `Mapping`，找到所对应的 Worker Process 并监听它，如果 Woker Process 不存在，则创建之

- Worker Porcess 会分析当前请求资源文件的后缀并根据在该模块中所对应注册的 `ISAPI Extension Mapping` 找到 `ISAPI Extension` （以*.aspx请求资源为例，所指定的ISAPI Extension 默认为 aspnet_isapi.dll），并通过 `ISAPI Extension` 去初始化CLR，构建 .NETFramework Runtime

#### 5. WAS：Web Administrative Service

- 这是一个监控程序，它一方面可以存取放在 `InetInfo` 元数据库 `Metabase` 中的各种信息，另一方面也负责监控应用程序池中的工作者进程的工作状态况，必要时它会关闭一个老的工作者进程并创建一个新的取而代之
 
#### 6. ISAPI：Internet Server Application Programming Interface 英特网服务应用程序编程接口

- 是运行在 <span style="color:red">非托管环境中</span> 的一组API，它是一种用来开发扩展IIS程序的强有力的方法，虽然 ISAPI 扩展绝不仅仅局限于IIS的开发，但是实际上它用的最广泛的还是开发一些与IIS合作的程序

#### 7. ISAPI Extension
- 是实现了 ISAPI 的拓展，对于 <span>ASP.NET</span> 来说，所使用的 `ISAPI Extension` 则为  `aspnet_isapi.dll`
- 在 `Worker Process` 加载的过程当中，会通过 `aspnet_isapi.dl` 去初始化CLR，加载 .NETFramework Runtime
- `aspnet_isapi.dl` 是承载在 <span style="color:red">非托管环境</span> 当中的，而.NET中的对象则运行在 <span style="color:blue">托管环境之上(CLR)</span>，那么作为托管环境与非托管环境之间沟通的桥梁就是一个非托管指针 `EPC Execution Control Block`
- `aspnet_isapi.dl` 在初始化CLR的过程当中是异步调用的，那么为了解决能够在调用 `ISAPI Extension` 的过程当中也能够获取 .NETFramework 中所返回的 Response，也是通过一个非托管指针EPC Execution Control Block 来实现的

<br/>

### 从更低的角度去看 <span>ASP.NET</span> 的处理流程

---

以 IIS6.0+ 而言，监听HTTP请求的工作被分发至一个运行在 Kernel Module 的组件 HTTP.SYS 上了，IIS会把自己的 WebApplication 的目录注册到 HTTP.SYS 的虚拟路径上 (只是注册了静态资源的路径，不包含动态资源：(`*.aspx`、`*.ashx`、`*.cshtml`、…) ，这步操作实际上是告诉 HTTP.SYS 当前所请求的URL的可访问性，举个例子，当我们浏览一个静态资源页面 `*.html` 出现404的时候，就是通过它来决定的

那么当 HTTP.SYS 捕获到一个可访问的URL的HTTP请求的时候，它的首要工作是先分析当前 `Resource Request` 的IP地址和端口号，紧接着它会在自己内部所建立的一个缓冲区查找该 `Resource Request` 是否在最近被请求过，如果有的话则直接抓取所缓存的数据并返回 Response 给 Client

![图片1.png](https://i.loli.net/2019/12/04/uvnTsmoPFSgeX4y.png)

如果并没有在最近被请求过的话则又会进入一次判断，判断此次 Resource Request 是否是一个动态资源 (`*.aspx`、`*.ashx`、`*.cshtml`、…)，如果不是的话 (`*.html`、`*.jpg`、…) 则直接从 IIS 为 HTTP.SYS 所 `Host` 的虚拟路径下找到资源文件并返回 Response 给 Client 

![图片1.png](https://i.loli.net/2019/12/04/Cp1FBrTEAx6jyg2.png)

那么当 Resource Request 为一个动态网页文件的话则根据 IIS 在 Meta Base 中所维护的 Application Pool 和 Resource 的 Mapping 找到对应的 Application Pool，如果还未运行则创建之，否则把 Request 加入到一个Application Pool Queue 中进行处理，至此，一个 Application Pool 已经创建完毕

在上一节中我们介绍了 WAS，WAS 其主要职责就是根据 IIS 在 Meta Base 中所维护的 Application Pool 和 Worker Porcess 之间的 Mapping 去监听Worker Porcess 的工作状态，如果它不存在，则创建之，当 Worker Process 被创建之后，它会根据 IIS 在 Meta Base 中所维护的 Resource 资源后缀和 ISAPI Extention 的 Mapping ，找到并加载之，就 <span>ASP.NET</span> 而言，其对应的 ISAPI Extention 则为 `aspnet_isapi.dll`

![图片2.png](https://i.loli.net/2019/12/04/SlmKioCghJrNGb8.png)

> 在这里如果仔细阅读的可以理解到，IIS 其实负责维护了三段关系之间的 Mapping，分别是
Application Pool -> 请求资源
Application Pool -> Workder Process
ISAPI -> 请求资源

aspnet_isapi.dll 是一个运行在 <span style="color:red">Unmanaged Environment</span> 中的一个组件，Worker Porcess 通过它经过一系列的 `COM` 级别的调用 (获取Server Variable数据、通过Post Method回传Server数据、…)，调用最终会落定在初始化 `CLR` 的身上

在 CLR 初始化的过程当中，同样会加载两个重要的dll，其一是 `AppManagerAppDomainFactory`，通过它为当前 Web Application 构建出一个 Application Domain ，其二是 `System.Web.Hosting.ISAPIRuntime`，关于这个程序集需要我们重点的关注 ，在 ISAPIRuntime 中有一个 `ProcessRequest` 函数，它是 <span>ASP.NET</span> 的一个入口，前面我们说到 aspnet_isapi.dll 是一个运行在 Unmanaged Environment 中的，也就是说在此刻 Http Resource Request 其实还在 Unmanaged Environment 中，那么 aspnet_isapi.dll 是怎么把 Http Resource Request 带到运行在 <span style="color:blue">Managed Environment</span> 中的呢？是通过一个名为 `EPC` 的 C++ Point，这个EPC也是上面说的 ProcessRequest 入口函数中的一个 Siganature，通过 EPC，aspnet_isapi.dll 成功地把刚刚还运行在U nmanaged Environment 中的 Http Resource Request 带领到 Managed Environment 中（ aspnet_isapi.dll 对 ISAPIRuntime 的调用是基于异步执行的，并且会把请求放入至队列中进行处理）

![图片3.png](https://i.loli.net/2019/12/04/Nbk3Zs9VY1ycaJq.png)

```csharp
[SecurityPermission(SecurityAction.LinkDemand, Unrestricted = true)]
public int ProcessRequest(IntPtr ecb, int iWRType)
{
    IntPtr intPtr = IntPtr.Zero;
    if (iWRType == 2)
    {
        //***************************************
        intPtr = ecb;
        ecb = UnsafeNativeMethods.GetEcb(intPtr);
        //***************************************
    }
    //*******************************************
    ISAPIWorkerRequest iSAPIWorkerRequest = null;
    //*******************************************
    try
    {
        bool useOOP = iWRType == 1;
        iSAPIWorkerRequest = ISAPIWorkerRequest.CreateWorkerRequest(ecb, useOOP);
        iSAPIWorkerRequest.Initialize();
        string appPathTranslated = iSAPIWorkerRequest.GetAppPathTranslated();
        string appDomainAppPathInternal = HttpRuntime.AppDomainAppPathInternal;
        if (appDomainAppPathInternal == null || StringUtil.EqualsIgnoreCase(appPathTranslated, appDomainAppPathInternal))
        {
            //*****************************************************
            HttpRuntime.ProcessRequestNoDemand(iSAPIWorkerRequest);
            //*****************************************************
            return 0;
        }
        HttpRuntime.ShutdownAppDomain(ApplicationShutdownReason.PhysicalApplicationPathChanged, SR.GetString("Hosting_Phys_Path_Changed", appDomainAppPathInternal, appPathTranslated));
        return 1;
    }

    // More ....
}
```

通过上面我们也可以看到 ProcessRequest 入口函数的实现代码，其核心已标明出来，那么我们跟着这个流程继续走（ProcessRequestNoDemand），代码又会落定在ProcessRequestInternal之中，在个函数中主要干了两件非常重要的事情

第一件则是通过 EPC 所构建的 `HttpWorkerRequest` 去创建了 `HttpContext`

```csharp
private void ProcessRequestInternal(HttpWorkerRequest wr)
{
    // More .....

    HttpContext httpContext;
    try
    {
        // **********************************************************
        httpContext = new HttpContext(wr, initResponseWriter: false);
        // **********************************************************
    }
    catch
    {
        try
        {
            wr.SendStatus(400, "Bad Request");
            wr.SendKnownResponseHeader(12, "text/html; charset=utf-8");
            byte[] bytes2 = Encoding.ASCII.GetBytes("<html><body>Bad Request</body></html>");
            wr.SendResponseFromMemory(bytes2, bytes2.Length);
            wr.FlushResponse(finalFlush: true);
            wr.EndOfRequest();
        }
        finally
        {
            Interlocked.Decrement(ref _activeRequestCount);
        }
        return;
    }

    // More ....
}

```

第二件事则是调用了 `GetApplicationInstance` 函数，关于这个函数的内部做了非常多的事情，详情请参照 [ASP.NET Process Model 缩略图](https://github.com/NGPONG/document/blob/master/ASP.NET/ASP.NET%E5%9F%BA%E7%A1%80/ASP.NET%20Process%20Model.png) ，以下只做概括性讲解
- 判断 Application Pool 是否存在未被使用的 `HttpApplication` ，如果不存在则通过反射的方式创建一个基于 Global.asax 的 `HttpApplication`
- 通过 `EnsureAppStartCalled` 函数去触发 HttpApplication 的 `Application_Start Event Handler`
- 通过 `HookupEventHandlersForApplicationAndModules` 函数完成对 Global.asax 中的 Event Handler 的注册
- 通过配置文件 (默认为本机的全局 `web.config`，同样也适配于自己本站 `web.config` 中定义的 `<modules>` ) 中的映射关系去创建 `HttpModule`，并循环执行每个 `HttpModule` 的 `init` 函数，再这个 init 函数内部，不同的 `HttpModule` 会针对 `HttpApplication` 中所分发的不同的 `Event` 去注册独属自己的、当前 **Module** 的逻辑，那么关于为什么在 `HttpApplication` 的第7个事件的时候才会去创建一个 `HttpHandler`，因为不管是 WebForm 也好还是MVC也好，它们都有属于自己的一个 `HttpModule`，并且其内部的 `Init` 函数的内部会注册第7个事件，并指定的 `Event Handler` 中会负责参与创建当前 `Handler` 所对应的 `HttpHandler`，如：MVC 则为 `MvcHandler`
- 通过 `BuildStep` 函数最终为 HttpApplication 完成 对于 19个事件(`默认`) 的注册

```csharp
private void ProcessRequestInternal(HttpWorkerRequest wr)
{
    // More .....

    wr.SetEndOfSendNotification(_asyncEndOfSendCallback, httpContext);
    HostingEnvironment.IncrementBusyCount();
    try
    {
        // More ....

        httpContext.Response.InitResponseWriter();
        
        // *******************************************************************************************
        IHttpHandler applicationInstance = HttpApplicationFactory.GetApplicationInstance(httpContext);
        // *******************************************************************************************
        
        // More ....
    }
    catch (Exception e)
    {
        httpContext.Response.InitResponseWriter();
        FinishRequest(wr, httpContext, e);
    }
}
```

在 `HttpApplication` 、 `HttpModules` 、 `HttpContext` 初始化完成之后，调用会降临到 `BeginProcessRequest` 函数身上

```csharp
private void ProcessRequestInternal(HttpWorkerRequest wr)
{
    // More .....

    wr.SetEndOfSendNotification(_asyncEndOfSendCallback, httpContext);
    HostingEnvironment.IncrementBusyCount();
    try
    {
        // More .....

        // *******************************************************************************************
        IHttpHandler applicationInstance = HttpApplicationFactory.GetApplicationInstance(httpContext);
        // *******************************************************************************************
        
        // More .....

        if (applicationInstance is IHttpAsyncHandler)
        {
            IHttpAsyncHandler httpAsyncHandler2 = httpContext.AsyncAppHandler = (IHttpAsyncHandler)applicationInstance;

            // ****************************************************************************************
            httpAsyncHandler2.BeginProcessRequest(httpContext, _handlerCompletionCallback, httpContext);
            // ****************************************************************************************

        }

        // More .....
    }
    catch (Exception e)
    {
        httpContext.Response.InitResponseWriter();
        FinishRequest(wr, httpContext, e);
    }
}
```

那么在 `BeginProcessRequest` 的内部，会通过 `HttpApplication` 去调用 `ResumeSteps` ，至此开始进入 `Http PipeLine`，依次执行管道中的事件，并且每一个事件处理函数都把 `HttpApplication` 作为 `Sender` 传入

```csharp
internal IAsyncResult BeginProcessRequestNotification(HttpContext context, AsyncCallback cb)
{
    if (_context == null)
    {
        AssignContext(context);
    }
    context.CurrentModuleEventIndex = -1;
    HttpAsyncResult httpAsyncResult = new HttpAsyncResult(cb, context);
    context.NotificationContext.AsyncResult = httpAsyncResult;

    // ***************
    ResumeSteps(null);
    // ***************

    return httpAsyncResult;
}
```

![图片4.png](https://i.loli.net/2019/12/04/FDpmd1fLExGJ8Sb.png)

在 `Http PipeLine` 中，当执行到第8个事件的时候，会根据配置文件 (默认为本机的全局 `web.config`，同样也适配于自己本站 `web.config` 中定义的 `<handlers>` ) 中的映射关系，在结合当前的 Resource Request 去匹配在第7个事件中所创建 HttpHandler，那么在第11至12之间，会调用这个 HttpHandler 的 `ProcessRequest` 函数去进行后续的逻辑处理

![图片6.png](https://i.loli.net/2019/12/04/MBrFxjXk9nHfqop.png)

![图片5.png](https://i.loli.net/2019/12/04/NCalu8nvUQjHKLz.png)

