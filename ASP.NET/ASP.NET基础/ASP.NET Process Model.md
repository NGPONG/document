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

以 IIS6.0+ 而言，监听HTTP请求的工作被分发至一个运行在 Kernel Module 的组件 HTTP.SYS 上了，IIS会把自己的 WebApplication 的目录注册到 HTTP.SYS 的虚拟路径上 (只是注册了静态资源的路径，不包含动态资源：`*.aspx`、`*.ashx`、`*.cshtml`、…) ，这步操作实际上是告诉 HTTP.SYS 当前所请求的URL的可访问性，举个例子，当我们浏览一个静态资源页面 `*.html` 出现404的时候，就是通过它来决定的

那么当 HTTP.SYS 捕获到一个可访问的URL的HTTP请求的时候，它的首要工作是先分析当前 `Resource Request` 的IP地址和端口号，紧接着它会在自己内部所建立的一个缓冲区查找该 `Resource Request` 是否在最近被请求过，如果有的话则直接抓取所缓存的数据并返回 Response 给 Client

![图片1.png](https://i.loli.net/2019/12/04/uvnTsmoPFSgeX4y.png)

