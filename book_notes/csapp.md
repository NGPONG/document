# *Computer Systems: A Programmer's Perspective*

<br/>

## Chapter <font color = "red">1</font>: A Tour of Computer Systems

### 了解系统硬件的组成 (1.4)

---


<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-14-00-18-51.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Hardware organization of a typical system</div>
</center>


#### 1. 总线 ( $bus$ )

穿整个系统的是一组电子管道，称作总线 $(bus)$，它携带字节信息并负责在各个组件之间传递，通常总线被设计成传送 **定长** 的字节块，也就是字 $(word)$

!!! info
    字通常描述一个字节数，也可称为 字长，它是一个基本的系统参数，在各个系统中都不尽相同，现在的大多数机器字长要么是4个字节(32bit)， 要么是8个字节(64bit)

#### 2. I/O设备 ( $I/O \ Devices$ )

I/O设备 是系统与外部世界的联系通道，它们都通过一个<u>**控制器**</u>或<u>**适配器**</u>与 I/O总线($I/O \ bus$) 相连，构建了信息传递的渠道；在图中，我们能够看到的 I/O设备 有: 键盘和鼠标、显示器、磁盘驱动器

!!! info
    控制器和适配器之间的区别主要在于它们的封装方式

    控制器是 I/O设备 本身或者系统的主印制电路板（通常称作主板）上的芯片组

    适配器则是一块插在主板插槽上的卡

#### 3. 主存 ( $Main \ Memory$ )

主存是一个临时存储设备，在处理器执行程序时，用来存放程序和程序处理的数据

从物理上来说，主存是由一组动态随机存取存储器( $DRAM$ )芯片组成的；从逻辑上来说，存储器是一个线性的字节数组，每个字节都有其唯一的地址(数组索引)，这些地址是从零开始
的

#### 4. 处理器 ( $Processor$ )

中央处理单元( $CPU$ )，简称处理器，是解释(或执行)存储在主存中指令的引擎；其内部核心其实是一个大小为一个字的存储设备(或寄存器)，称为<u>**程序计数器**</u>( $PC$ )，在任何时刻，PC 都指向主存中的某条机器语言指令在主存中的地址

系统通电开始，直到系统断电，处理器一直在不断地执行程序计数器指向的指令，再更新程序计数器，使其指向下一条指令；这使得处理器看上去是按照一个非常简单的指令执行模型(由<u>**指令集架构**</u>决定)来操作的，在这个模型中，指令按照严格的顺序执行

处理器在执行指令时，处理器会从 PC 指向的内存处读取指令并解释指令中的位( $bit$ )，最后执行该指令所指示的一些简单操作；这些简单操作都会围绕着主存( $Main \ Memory$ )、寄存器文件( $Register \ File$ )和算术／逻辑单元( $ALU$ )所进行

!!! info
    PC 是计算机科学中使用的一个术语，他是一个抽象的概念，在 x86 兼容 CPU 上，<font color = "red">CS段寄存器</font> 和 <font color = "red">IP寄存器</font> 是实现计算机科学中的 PC 这个概念的具体设施

<br/>

### 了解高速缓存与存储设备层次结构 (1.5, 1.6)

---

<u>**根据机械原理，较大的存储设备要比较小的存储设备运行得慢，而快速设备的造价远高于同类的低速设备处理器。**</u> 比如说，在访问寄存器与主存之间，二者的时间差异高达100倍，但是，仅依赖于典型寄存器的几百字节的容量根本无法处理系统上的巨量信息，然而盲目扩张寄存器的大小反而会导致设备造价的急速升高

针对于这种相互矛盾且尴尬的差异，系统设计者将一种称为<u>**高速缓存存储器**</u>( $cache \   memory$ ) 的存储器设立在了寄存器与主存之间，它对比寄存器拥有更大的容量，对比主存拥有更快的读写速度

高速缓存的功能，我们可以总结为: 它作为暂时的集结区域，存放处理器近期可能会需要的信息。

在现代处理器当中，通常都附带了编号为 $L1, L2, L3$ 的高速缓存，它们的区别在于位于序列前面的缓存它能够拥有更快的读写速度，但是相对应的他会失去一些能够容纳信息的容量，比如说：位于处理器芯片上的 $L1$ 高速缓存的容量可以达到数万字节，访问速度几乎和访问寄存器文件一样快；一个容量为数十万到数百万字节的更大的 $L2$ 高速缓存通过一条特殊的总线连接到处理器，进程访问 $L2$ 高速缓存的时间要比访问 $L1$ 高速缓存的时间长5倍，但是这仍然比访问主存的时间快5～10

使得高速缓存能够拥有近似寄存器访问速度的原因，除了因为其是通过一种叫做静态随机访问存储器 ( $SRAM$ )硬件技术去实现的之外，它还利用了高速缓存的<u>**局部性原理**</u>，即程序具有访问局部区域里的数据和代码的趋势

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-00-28-29.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Cache memories</div>
</center>

<br/>

在处理器和一个较大较慢的设备(例如主存)之间插入一个更小更快的存储设备(例如高速缓存)的想法已经成为一个普遍的观念

实际上，<u>**每个计算机系统中的存储设备都被组织成了一个存储器层次结构**</u>。其主要思想是上一层的存储器作为低一层存储器的高速缓存，例如：寄存器文件就是 $L1$ 的高速缓存，$L1$ 是 $L2$ 的高速缓存，$L2$ 是 $L3$ 的高速缓存，$L3$ 是 主存 的高速缓存，而主存又是硬盘的高速缓存。在某些具有分布式文件系统的网络系统中，本地磁盘就是存储在其他系统中磁盘上的数据的高速存在

如下图所示(寄存器文件记为 $L0$ )，在这个层次结构中，从上至下，设备的访问速度越来越慢、容量越来越大，并且每字节的造价也越来越便宜

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-00-47-54.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">An example of a memory hierarchy</div>
</center>

<br/>

### 了解操作系统管理硬件 (1.7)

---

仅依靠一个程序自身的能力是无法访问系统中的其他硬件设备的，取而代之的，它需要依靠于<u>**操作系统所提供的一些<font color="red">服务</font>**</u>去进行。我们可以把操作系统看成是应用程序和硬件之间插入的一层软件，所有应用程序对硬件的操作尝试都必须通过操作系统来完成

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-01-01-26.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Layered view of a computer system</div>
</center>

<br/>

操作系统通过几个基本的抽象概念(进程、虚拟内存和文件)来实现两个基本功能: (1) 防止硬件被失控的应用程序滥用；(2) 向应用程序提供简单一致的机制来控制复杂而又通常大不相同的低级硬件设备。如下图所示，文件是对I/O设备的抽象表示；虚拟内存是对主存和磁盘I/O设备的抽象表示，进程则是对处理器、主存和I/O 设备的抽象表示。

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-01-07-21.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Abstractions provided by an operating system.</div>
</center>

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-01-09-00.png">
</center>

<br/>

### 了解进程与线程 (1.7.1， 1.7.2)

---

#### 1. 进程

程序在现代系统上运行时，操作系统会提供一种假象，就好像系统上只有这个程序在运行并独占地使用处理器、主存和I/O设备；而处理器看上去就像在不间断地一条接一条地执行着这个独占程序中的代码指令，即该程序的代码和数据是系统内存中唯一的对象。这些假象则是通过进程的概念来实现的

<u>**进程是操作系统对一个正在运行的程序的一种抽象**</u>，它是计算机科学中最重要和最成功的概念之一

现代处理器(单核)虽然唯有且仅有一个 PC 寄存器，但是它依然能够在某一时刻下同时运行着多个进程。操作系统通过一种暂且称之为进程间切换的机制，以至于让多个进程之间的指令可以被处理器交错执行，因为处理器的执行速度过快，它能够在人肉眼无法察觉的时间范围内将 PC 寄存器从此刻正在执行的进程切换到另一个进程身上，以此来完成一种 *并发执行* 的假象。对于这种机制，我们会将它称之为 **<font color="red">上下文切换</font>**( $context\ \ switch$ )

**<font color="red">上下文</font>**$(context)$，即<u>**操作系统保持跟踪进程运行所需的所有状态信息**</u>，比如说 PC 和寄存器文件的当前值，以及主存的内容等等。在任何一个时刻，单处理器系统都只能执行一个进程的代码，当操作系统决定要把控制权从当前进程转移到某个新进程时，就会进行上下文切换，即保存当前进程的上下文、恢复新进程的上下文，然后将控制权传递到新进程。新进程就会从它上次停止的地方开始

!!! INFO
    从一个进程到另一个进程的转换是由操作系统内核（kernel）管理的

    内核是操作系统代码常驻主存的部分。当应用程序需要操作系统的某些操作时，比如读写文件，它就执行一条特殊的系统调用（system call）指令，将控制权传递给内核。然后内核执行被请求的操作并返回应用程序
    
    注意，<u>**内核不是一个独立的进程**</u>，相反，它是系统管理全部进程所用代码和数据结构的集合

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-16-23-24-37.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Process context switching.</div>
</center>


#### 2. 线程

尽管通常我们认为一个进程只有单一的控制流，但是在现代系统中，<u>**一个进程实际上可以由多个称为线程的<font color="red">执行单元</font>组成，每个线程都运行在进程的上下文中，并共享同样的代码和全局数据**</u>

由于网络服务器中对并行处理的需求，线程成为越来越重要的编程模型，因为多线程之间比多进程之间更容易共享数据，也因为线程一般来说都比进程更高效，当有多处理器可用的时候，多线程也是一种使得程序可以运行得更

<br/>

### 了解虚拟内存 (1.7.3)

---

虚拟内存是一个抽象概念，它为每个进程提供了一个假象，即每个进程都在独占地使用主存。每个进程看到的内存都是一致的，称为<u><font color = "red">**虚拟地址空间**</font></u>

下图展示了一张 linux 虚拟地址空间内存模型图，在地址空间最上面的区域是保留给操作系统中的代码和数据的，这对所有进程来说都是一样，地址空间的底部区域存放用户进程定义的代码和数据

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-17-00-01-27.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;"> Process virtual address space.</div>
</center>

<br/>

每个进程看到的虚拟地址空间由大量准确定义的区构成，每个区都有专门的功能

- **Program code and data**: 对所有的进程来说，代码是从同意固定地址开始，紧接着的是和C全局变量相对应的数据位置

- **Heap**: 代码和数据区后紧随着的是运行时堆，代码和数据区在进程一开始运行时就被指定了大小，与此不同，当调用像 $malloc$ 和 $free$ 这样的C标准库函数时，堆可以在运行时动态地扩展和收缩

- **Shared libraries**: 大约在地址空间的中间部分是一块用来存放像C标准库和数学库这样的共享库的代码和数据的区域

- **Stack**: 位于用户虚拟地址空间顶部的是<u>**<font color = "red">用户栈</font>**</u>，编译器用它来实现函数调用。和堆一样，用户栈在程序执行期间可以动态地扩展和收缩，例如当每次我们调用一个函数时栈就会增长；从一个函数返回时栈就会收缩

- **Kernel virtual memory**: 地址空间顶部的区域是为内核保留的，不允许应用程序读写这个区域的内容或者直接调用内核代码定义的函数，相反，它们必须调用内核来执行这些操作

<br/>

### 了解文件 (1.7.4)

---

<u>**文件就是字节序列仅此而已**</u>。每个I/O设备，包括磁盘、键盘、显示器甚至网
络，都可以看成是文件。系统中的所有输入输出都是通过使用一小组称为 Unix I/O 的系统函数调用读写文件来实现的

文件这个简单而精致的概念是非常强大的，因为它向应用程序提供了一个统一的视图，来看待系统中可能含有的所有各式各样的I/O设备。例如，处理磁盘文件内容的应用程序员可以非常幸福，因为他们无须了解具体的磁盘技术，<u>**进一步说，同一个程序可以在使用不同磁盘技术的不同系统上运行**</u>(同时还解决了平台差异化的问题，提升了程序的便携性)

<br/>

### Amdahl定律 (1.9.1)

---

$Gene\ Amdahl$，计算领域的早期先锋之一，对提升系统某一部分性能所带来的效果做
出了简单却有见地的观察，这个观察被称为 Amdahl 定律( $Amdahl's\ law$ )。<u>**该定律的主要思想是，当我们对系统的某个部分加速时，其对系统整体性能的影响取决于该部分的重要性和加速程度**</u>

若系统执行某应用程序需要时间为 $T_{old}$，假设系统某部分所需执行时间与该时间的比例为 $\small \alpha$，而该部分性能提升比例为 $\small \kappa$，即该部分初始所需时间为 ${\small \alpha} \, T_{old}$，现在所需时间为 $({\small \alpha}\,T_{old})/\,{\small \kappa}$，因此，总的执行时间应为: 

$$
    T_{new} = (1-{\small \alpha})\,T_{old} + ({\small \alpha}\,T_{old}\,/\, {\small \kappa}) = T_{old}\,[\,(1-{\small \alpha}) + {\small \alpha} \, / \, {\small \kappa}\,]
$$

由此，可以计算加速比 $S=T_{old} \, / \, T_{new}$ 为: 

$$

    S=	\frac {1}{(1- {\small \alpha} \, + \, {\small \alpha} \, / \, {\small \kappa})}

$$

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-17-00-49-04.png">
</center>


<br/>

### 并发和并行 (1.9.2)

---

数字计算机的整个历史中，有两个需求是驱动进步的持续动力∶ 一个是我们想要计算机做得更多，另一个是我们想要计算机运行得更快

当处理器能够同时做更多的事情时，这两个因素都会改进。<u>**我们用的术语 并发($concurrency$) 是一个通用的概念，指一个同时具有多个活动的系统；而术语 并行($parallism$）指的是用并发来使一个系统运行得更快**</u>。并行可以在计算机系统的多个抽象层次上运用，在此，我们按照系统层次结构中由高到低的顺序重点强调三个层次

#### 1. 线程级并发 ( $Thread-Level\ \ Concurrency$ )

构建在进程这个抽象之上，我们能够设计出同时有多个程序执行的系统，这就导致了并发。使用线程，我们甚至能够在一个进程中执行多个<font color = "red">控制流</font>

对于处理器的配置不同，我们通常会将处理器的类型分为 <u>**单处理器系统**</u> 和 <u>**多处理器系统**</u>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-18-01-30-11.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Categorizing different processor configurations.</div>
</center>

<br/>

在以前，大多数实际的计算都是由一个处理器来完成的，我们将这种系统架构称之为 <u>**单处理器系统**</u>。即便只有一个核心在工作，单处理器系统依然提供了多个用户同时与系统交互的能力，虽然实际上只是通过 $context\ switch$ 机制来模拟出来的一种假象，但是这样的能力，我们依然可以将它理解为是一种并发执行行为的体现

当构建一个由单操作系统内核控制的多处理多处理器器组成的系统时，我们便可以得到一个<u>**多处理器系统**</u>。随着现代计算机的发展，还有 <u>**多核处理器**</u> 与 <u>**超线程技术**</u> ( $hypertheading$ ) 的出现，多处理器系统日趋成熟且常见

多核处理器是将多个 $CPU$ (<u>**核心**</u>）集成到了一个集成电路芯片上，并且每个核心之间都拥有自己的高速缓存( $L1$, $L2$, $L3$ )，其中的 L1高速缓存分为两个部分———个保存最近取到的指令，另一个存放数据。因为核心都在独立运行，与单处理器系统不同，多处理器系统是能够实现真正的并发执行的行为

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-18-01-29-21.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Multi-core processor organization.</div>
</center>

!!! info

    超线程，有时称为同时多线程 ( $simultaneous\ multi-threading$ )，是一项允许一个 CPU 执行多个控制流的技术。它涉及 CPU某些硬件有多个备份，比如程序计数器和寄存器文件，而其他的硬件部分只有一份，比如执行浮点算术运算的单元。常规的处理器需要大约 20000 个时钟周期做不同线程间的转换，而超线程的处理器可以在单个周期的基础上决定要执行哪一个线程。这使得 CPU 能够更好地利用它的处理资源

    我们可以总结多线程技术拥有这几个优势: (1) 对于硬件部分存在多种备份方式 (2) 更高效的上下文切换的时机选择

#### 2. 指令级井行 ( $Instruction-Level \ \ Parallelism$ )

在较低的抽象层次上，现代处理器可以同时执行多条指令的属性称为 <u>**指令级并行**</u>

早期的微处理器，如 1978 年的 Intel 8086，需要多个 (通常是 3 ～ 10 个) 时钟周期来执行一条指令。最近的处理器通过一种名为 <u>**流水线**</u> ( $pipeling$ ) 的技术，可以保持每个时钟周期执行 2 ～ 4 条指令的执行速率，<u>**虽然每条指令的开始与结束会需要更多的时间，但是该项技术能够 CPU 执行指令的吞吐量变的更高，以从另一个角度上换来如此高的执行速率**</u>

!!! info

    如果处理器可以达到比一个周期一条指令更快的执行速率，就称之为超标量 ( $super\ scalar$) 处理器

#### 3. 单指令多数据并行 ( $Single-Instruction, Multiple-Data(SIMD)\ \ Parallelism$ )

在最低层次上，许多现代处理器拥有特殊的硬件，允许一条指令产生多个可以并行执
行的操作，这种方式称为 <u>**单指令、多数据**</u>，即 <u>**SIMD 并行**</u>

!!! info
    较新几代的 Intel 和 AMD 处理器都具有并行地对 8 对单精度浮点数 ( C数据类型 float ) 做加法的指令，供这些 $SIMD$ 指令多是为了提高处理影像、声音和视频数据应用的执行速度

<br/>

<br/>

## Chapter <font color = "red">2</font>: Representing and Manipulating Information

### 寻址和字节顺序 (2.1.3)

---

对于跨越多字节的程序对象，我们必须建立两个规则，(1)这个对象的地址是什么; (2)这个对象在内存中如何排列这个对象所表示的字节数据。<u><font color = "red">**在几乎所有的机器上，多字节对象都被存储为连续的字节序列，对象的地址为所使用字节中最小的地址**</font></u>

考虑一个 $w$ 位的整数，其位表示为 $[x_{w-1},x_{w-2},\ldots,x_1,x_0]$ ，其中 $x_{w-1}$ 是最高有效位，而 $x_0$ 是最低有效位。假设 $w$ 是 $8$ 的倍数，这些位就能被分组成为字节，其中最高有效字节包含位 $[x_{w-1},x_{w-2},\ldots,x_{w-8}]$ ，而最低有效字节包含位 $[x_7,x_7,\ldots,x_0]$ ，其他字节包含中间的位。从内存中按照从最低有效字节到最高有效字节的顺序存储对象的模式则称为 <u><font color = "red">**小端法**</font></u> ( $little\ endian$ )
，它的最低有效字节表示在最前面；从内存中按照从最高有效字节到最低有效字节的顺序存储对象的模式则称为 <u><font color = "red">**大端法**</font></u> ( $big\ endian$ )，它的最高有效字节表示在最前面

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-22-00-46-56.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">x = 0x01234567; &x = 0x100; </div>
</center>

<br/>

在大多数时候，字节顺序对于开发者来说是完全不可见的，无论为哪种类型的机器所编译的程序都会得到同样的结果。
但是在某些时候需要我们引起一些注意，比如当我们编写网络程序的时候出现两端机器都采用不同字节序的情况，还有就是在阅读表示 <u>**整数数据**</u> 的字节序列的时候，例如下面摘自某个通过小端序机器所生成程序的机器级代码的文本表示

```text
" 0x200b43 实际存储为 [43 0b 20 00]，即最低位在左边，最高位在右边
4004d3: 01 05 43 0b 20 00    add %eax,0x200b43(%rip)
```

我们要重点理解字节顺序的含义，它是<u><font color = "red">**以我们写入的值作为字节顺序存储的依据，而不是实际使用的类型**</font></u>，比方说我们有一个字符串 `"123456"`，它在机器级表示为 `31(1) 32(2) 33(3) 34(4) 35(5) 36(6)`，而不是与上面例子所相反的结果。从这其实也可以看出，文本数据比二进制数据具有更强的平台独立性

<br/>

### C语言中的位运算 (2.1.6, 2.1.7, 2.1.8, 2.1.9)

---

#### 1. 布尔代数于位级运算符

<u><font color = "red">**二进制值是计算机编码、 存储和操作信息的核心**</font></u>，所以围绕数值 $0$ 和 $1$ 的研究已经演化出了丰富的数学知识体系。这起源于 1850 年前后*乔治·布尔* ( $George\ Boole,\ 1815—1864$ ) 的工作，因此也 称为*布尔代数*( $Boolean\ algebra$ )。布尔注意到通过将逻辑值 $TRUE$ 和 $FALSE$ 编码为二进制值 $1$ 和 $0$，能够设计出一种代数，以研究逻辑推理的基本原则

布尔运算 $\sim$ 对应于逻辑运算 $NOT$，在命题逻辑中用符号 $\neg$ 表示。也就是说，当 $P$ 不是真的时候，我们就说 $\neg P$ 是真的，反之亦然；相应地，当 $P$ 等于 $0$ 时，$\sim P$ 等于 $1$，反之亦然

布尔运算 $\&$ 对应于逻辑运算 $AND$，在命题逻辑中用符号 $\bigwedge$ 表示。当 $P$ 和 $Q$ 都为真时，我们说 $P \bigwedge Q$ 为真；相应地，只有当 $p = 1$ 且 $q = 1$ 时，$p\ \& \ q$ 才等于 $1$

布尔运算 $\mid$ 对应于逻辑运算 $OR$，在命题逻辑中用符号 $\bigvee$ 表示。当 $P$ 或者 $Q$ 为真时，我们说 $P \bigvee Q$成立；相应地，当 $p = 1$ 或者 $q = 1$ 时，$p \mid q$ 等于 $1$

布尔运算 $\wedge$ 对应于逻辑运算异或，在命题逻辑中用符号 $\bigoplus$ 表示。当 $P$ 或者 $Q$ 为真但不同时为真时，我们说 $P \bigoplus Q$ 成立；相应地，当 $p = 1$ 且 $q = 0$，或者 $p = 0$ 且 $q = 1$ 时，$p \wedge q$ 等于 $1$

<br/>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-04-14-01-27-00.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Operations of Boolean algebra.</div>
</center>

<br/>

我们可以将上述 4 个布尔运算扩展到位向量的运算，位向量就是固定长度为 $w$ 且仅由 $0$
和 $1$ 组成的串

假设 $a$ 和 $b$ 分别表示位向量 $[a_{w-1},a_{w-2},\ldots,a_1,a_0]$ 和 $[b_{w-1},b_{w-2},\ldots,b_1,b_0]$。例如当我们需要计算 $a\ \& \ b$ 时，则再定义一个也定义为一个长度为 $w$ 的位向量，其中第 $i$ 个元素等于 $a_{i} \  \& \  b_{i}$，$0\ll i \ll w$；这种方式同样可以应用于运算 $\mid$、 $\wedge$ 和 $\sim$ 身上

希伯来文(Hebrew) | 希腊文(Greek) | 英语(English)
 :-: | :-: | :-:
弥赛亚(Messiah) | 基督(Christ) | 受膏者(the anointed one)