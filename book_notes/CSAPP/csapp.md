# *Computer Systems: A Programmer's Perspective*


<br/>


## Chapter <font color = "red">1</font> A Tour of Computer Systems

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

<br/>

### 了解进程与线程 (1.7.1， 1.7.2)

---

#### 进程

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

<br/>

#### 线程

尽管通常我们认为一个进程只有单一的控制流，但是在现代系统中，<u>**一个进程实际上可以由多个称为线程的<font color="red">执行单元</font>组成，每个线程都运行在进程的上下文中，并共享同样的代码和全局数据**</u>

由于网络服务器中对并行处理的需求，线程成为越来越重要的编程模型，因为多线程之间比多进程之间更容易共享数据，也因为线程一般来说都比进程更高效，当有多处理器可用的时候，多线程也是一种使得程序可以运行得更

<br/>

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

<br/>

### 了解文件 (1.7.4)

---

<u>**文件就是字节序列仅此而已**</u>。每个I/O设备，包括磁盘、键盘、显示器甚至网
络，都可以看成是文件。系统中的所有输入输出都是通过使用一小组称为 Unix I/O 的系统函数调用读写文件来实现的

文件这个简单而精致的概念是非常强大的，因为它向应用程序提供了一个统一的视图，来看待系统中可能含有的所有各式各样的I/O设备。例如，处理磁盘文件内容的应用程序员可以非常幸福，因为他们无须了解具体的磁盘技术，<u>**进一步说，同一个程序可以在使用不同磁盘技术的不同系统上运行**</u>(同时还解决了平台差异化的问题，提升了程序的便携性)

<br/>

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

<br/>

### 并发和并行 (1.9.2)

---

<br/>

<br/>

<br/>

## Chapter <font color = "red">2</font> Representing and Manipulating Information