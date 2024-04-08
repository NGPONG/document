<style>
.center 
{
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
</style>

## Abstract

Today, shared libraries are ubiquitous. Developers use them for multiple reasons and create them just as they would create application code. This is a problem, though, since on many platforms some additional techniques must be applied even to generate decent code. Even more knowledge is needed to generate optimized code. This paper introduces the required rules and techniques. In addition, it introduces the concept of ABI (Application Binary Interface) stability and shows how to manage it.

如今，共享库无处不在。开发者出于多种原因使用它们，并且创建它们就像创建应用程序代码一样。然而，这是一个问题，因为在许多平台上，即使是生成合理的代码，也必须应用一些额外的技术。生成优化代码则需要更多的知识。本文介绍了所需的规则和技术。此外，它还介绍了ABI（应用程序二进制接口）稳定性的概念，并展示了如何管理它。

## 1. Preface

For a long time, programmers collected commonly used code in libraries so that code could be reused. This saves development time and reduces errors since reused code only has to be debugged once. With systems running dozens or hundreds of processes at the same time, reuse of the code at link-time solves only part of the problem. Many processes will use the same pieces of code which they import from libraries. With the memory management systems in modern operating systems, it is also possible to share the code at run-time. This is done by loading the code into physical memory only once and reusing it in multiple processes via virtual memory. Libraries of this kind are called shared libraries.

长期以来，程序员们将常用的代码收集在库中，以便代码可以被重用。这节省了开发时间，并且由于重用的代码只需调试一次，因此减少了错误。在同时运行数十个或数百个进程的系统中，链接时的代码重用只解决了部分问题。许多进程将使用它们从库中导入的相同的代码片段。在现代操作系统的内存管理系统中，也可以在运行时共享代码。这是通过将代码仅加载一次到物理内存中，并通过虚拟内存在多个进程中重用它来完成的。这种类型的库被称为共享库。

The concept is not very new. Operating system designers implemented extensions to their system using the infrastructure they used before. The extension to the OS could be done transparently for the user. But the parts the user directly has to deal with initially created problems.

这个概念其实并不新鲜。操作系统的设计者们通过利用他们先前使用的基础架构，对系统进行了拓展。这种对操作系统的拓展，对用户来说是透明的。但是，用户直接需要处理的部分在最开始就引发了一些问题。

The main aspect is the binary format. This is the format which is used to describe the application code. Long gone are the days that it was sufficient to provide a memory dump. Multi-process systems need to identify different parts of the file containing the program such as the text, data, and debug information parts. For this, binary formats were introduced early on. Commonly used in the early Unix-days were formats such as a.out or COFF. These binary formats were not designed with shared libraries in mind and this clearly shows.

主要问题在于二进制格式。这种格式被用来描述应用程序的代码。过去，仅提供内存转储对于代码描述已经足够了。但是，随着系统需要同时运行数十甚至数百个进程，仅在链接时重用代码变得不够用。系统需要识别程序文件中不同的部分，比如文本、数据和调试信息。为此，二进制格式在很早前就被引入了。早期Unix使用的二进制格式，如 a.out(assembler output) 或 COFF(Common Object File Format)，并未设计时考虑共享库，这一点在它们的设计中非常明显。

### 1.1 A Little Bit of History

The binary format used initially for Linux was an a.out variant. When introducing shared libraries certain design decisions had to be made to work in the limitations of a.out. The main accepted limitation was that no relocations are performed at the time of loading and afterward. The shared libraries have to exist in the form they are used at run-time on disk. This imposes a major restriction on the way shared libraries are built and used: every shared library must have a fixed load address; otherwise, it would not be possible to generate shared libraries which do not have to be relocated.

Linux最初使用的二进制格式是a.out的一个变种。在引入共享库时，必须在a.out的限制条件下做出一些设计决策。主要接受的限制是，在加载和加载后不进行重定位。也就是说，共享库在磁盘上的存储形式必须与它在运行时被加载到内存中的形式完全一致(内存布局)。这对共享库的构建和使用方式施加了重大限制：每个共享库都必须有一个固定的加载地址；换句话说，如果没有这些固定的加载地址，就无法创建不需要在运行时进行地址重定位的共享库。

The fixed load addresses had to be assigned and this has to happen without overlaps and conflicts and with some future safety by allowing growth of the shared library. It is therefore necessary to have a central authority for the assignment of address ranges which in itself is a major problem. But it gets worse: given a Linux system of today with many hundred of DSOs (Dynamic Shared Objects) the address space and the virtual memory available to the application gets severely fragmented. This would limit the size of memory blocks which can be dynamically allocated which would create unsurmountable problems for some applications. It would even have happened by today that the assignment authority ran out of address ranges to assign, at least on 32-bit machines.

固定加载地址必须在避免重叠和冲突的过程中被分配，并且通过允许共享库的增长来确保未来的安全性。因此，需要有一个集中的管理器来分配地址范围，这本身就是一个重大问题而且情况会变得更糟：鉴于今天的Linux系统拥有数百个DSO（动态共享对象），应用程序可用的地址空间和虚拟内存变得极其碎片化。这将限制可以动态分配的内存块的大小，从而为某些应用程序创造了无法克服的问题。甚至在今天，该系统(集中的管理器)可能已经用尽了可分配的地址范围，至少在32位机器上是这样。

We still have not covered all the drawbacks of the a.out shared libraries. Since the applications using shared libraries should not have to be relinked after changing a shared library it uses, the entry points, i.e., the function and variable addresses, must not change. This can only be guaranteed if the entry points are kept separate from the actual code since otherwise limits on the size of a function would be hard-coded. A table of function stubs which call the actual implementation was the solution used on Linux. The static linker got the address of each function stub from a special file (with the filename extension .sa). At run-time a file ending in .so.X.Y.Z was used and it had to correspond to the used .sa file. This in turn requires that an allocated entry in the stub table always had to be used for the same function. The allocation of the table had to be carefully taken care of. Introducing a new interface meant appending to the table. It was never possible to retire a table entry. To avoid using an old shared library with a program linked with a newer version, some record had to be kept in the application: the X and Y parts of the name of the .so.X.Y.Z suffix was recorded and the dynamic linker made sure minimum requirements were met.

在a.out格式的共享库中，我们还没有讨论到所有的缺点。由于使用共享库的应用程序在更换它使用的共享库后不应该需要重新链接，因此，入口点（$Entry\ Point$），即函数和变量地址，不应该发生变化。这只能通过将入口点与实际代码分开来保证，否则函数的大小（指的是字节码的长度）限制将被硬编码。Linux上使用的解决方案是调用实际实现的函数存根表($Function\ Stub$)。静态链接器从一个特殊的文件（文件名扩展为 `.sa`）中获取每个函数存根的地址。在运行时，会使用以 `.so.X.Y.Z` 结尾的文件，它必须与使用的 .sa 文件对应。这反过来要求存根表中分配的条目必须始终用于相同的函数。表的分配必须谨慎处理。引入新接口意味着要追加到表中。永远不可能取消表项。为了避免使用旧的共享库来连接新版本的程序，应用程序中必须保留一些记录：.so.X.Y.Z 后缀名的X和Y部分被记录下来，动态链接器确保满足最低要求。

The benefits of the scheme are that the resulting program runs very fast. Calling a function in such shared libraries is very efficient even for the first call. It can be implemented with only two absolute jumps: the first from the user code to the stub, and the second from the stub to the actual code of the function. This is probably faster than any other shared library implementation, but its speed comes at too high a price: 

该方案的好处是，结果程序运行非常快。即使是第一次调用，也能在这种共享库中非常高效地调用函数。它可以仅通过两次绝对跳转来实现：第一次从用户代码跳到存根，第二次从存根跳到函数的实际代码。这可能比任何其他共享库实现都要快，但代价太高：

1. a central assignment of address ranges is needed; 

    需要对地址范围进行集中分配；

2. collisions are possible (likely) with catastrophic results; 

    发生冲突可能（很可能）造成灾难性后果；

3. the address space gets severely fragmented.

    地址空间变得严重碎片化。


For all these reasons and more, Linux converted early on to using ELF (Executable Linkage Format) as the binary format. The ELF format is defined by the generic specification (gABI) to which processor-specific extensions (psABI) are added. As it turns out the amortized cost of function calls is almost the same as for a.out but the restrictions are gone.

由于这些原因以及更多其他原因，Linux很早就转换为使用 `ELF`（$Executable\ Linkage\ Format$）作为二进制格式。ELF 格式由通用规范（gABI）定义，其中添加了特定于处理器的扩展（psABI）。事实证明，函数调用的平摊成本几乎与a.out相同，但限制已经消失。

### 1.2 The Move To ELF

For programmers the main advantage of the switch to ELF was that creating ELF shared libraries, or in ELF-speak DSOs, becomes very easy. The only difference between generating an application and a DSO is in the final link command line. One additional option (--shared in the case of GNU ld) tells the linker to generate a DSO instead of an application, the latter being the default. In fact, DSOs are little more than a special kind of binary; the difference is that they have no fixed load address and hence require the dynamic linker to actually become executable. With Position Independent Executable (PIEs) the difference shrinks even more.

对程序员来说，转向 ELF 格式的主要优势是制作 ELF 共享库（或ELF中的动态共享对象，DSO）变得非常简单。生成应用程序和DSO之间的唯一区别在于最终的链接命令行选项。一个额外的选项（对于 GNU ld 来说是 --shared）会指示链接器生成一个 DSO 而不是应用程序，后者是默认选项。实际上，DSO 只是一种特殊类型的二进制文件；它们之间的区别在于DSO没有固定的加载地址，因此需要动态链接器才能真正变为可执行的。随着位置无关可执行文件（PIE）的出现，这种差异（即 DSO 与早期普通的二进制可执行程序）变得更小。

This, together with the introduction of GNU Libtool which will be described later, has led to the wide adoption of DSOs by programmers. Proper use of DSOs can help save large amounts of resources. But some rules must be followed to get any benefits, and some more rules have to be followed to get optimal results. Explaining these rules will be the topic of a large portion of this paper.

这一转变，加上之后介绍的 GNU Libtool 的引入，促使程序员们广泛采用了DSO。适当地使用DSO可以帮助节约大量资源。但是，要想从中获益，必须遵循一些规则；而要获得最优结果，则需要遵循更多的规则。本文的很大一部分将用于解释这些规则。

Not all uses of DSOs are for the purpose of saving resources. DSOs are today also often used as a way to structure programs. Different parts of the program are put into separate DSOs. This can be a very powerful tool, especially in the development phase. Instead of relinking the entire program it is only necessary to relink the DSO(s) which changed. This is often much faster.

并非所有使用DSO的目的都是为了节省资源。如今，DSO也经常被用作结构化程序的一种方式。程序的不同部分被放入不同的DSO中。这可以是一个非常强大的工具，特别是在开发阶段。不需要重新链接整个程序，只需要重新链接更改了的DSO(s)。这通常要快得多。

Some projects decide to keep many separate DSOs even in the deployment phase even though the DSOs are not reused in other programs. In many situations it is certainly a useful thing to do: DSOs can be updated individually, reducing the amount of data which has to be transported. But the number of DSOs must be kept to a reasonable level. Not all programs do this, though, and we will see later on why this can be a problem.

即使在部署阶段，一些项目也决定保留许多独立的DSO，尽管这些DSO在其他程序中不被重用。在许多情况下，这无疑是一个有益的做法：DSO可以单独更新，从而减少了需要传输的数据量。但是，必须将DSO的数量保持在合理的水平。不是所有的程序都这么做，我们稍后将看到这为什么会成为一个问题。

Before we can start discussing all this some understanding of ELF and its implementation is needed.

在我们开始讨论所有这些之前，需要对ELF及其实现有一定的了解。

### 1.3 How Is ELF Implemented?

The handling of a statically linked application is very simple. Such an application has a fixed load address which the kernel knows. The load process consists simply of making the binary available in the appropriate address space of a newly created process and transferring control to the entry point of the application. Everything else was done by the static linker when creating the executable.

处理静态链接的应用程序非常简单。这样的应用程序有一个内核已知的固定加载地址（虽然ASLR技术会在程序实际启动的时候再次被动态分配，但是地址也是固定的）（虽然是固定的地址，但是由于内核使用虚拟地址空间技术管理内存，所以也就不存在冲突的说法）。加载过程简单到只需要将二进制文件放置在新创建的进程的适当地址空间内，并将控制权转移给应用程序的入口点。其他所有事情都是在创建可执行文件时由静态链接器完成的。

Dynamically linked binaries, in contrast, are not complete when they are loaded from disk. It is therefore not possible for the kernel to immediately transfer control to the application. Instead some other helper program, which obviously has to be complete, is loaded as well. This helper program is the dynamic linker. The task of the dynamic linker is it, to complete the dynamically linked application by loading the DSOs it needs (the dependencies) and to perform the relocations. Then finally control can be transferred to the program.

与之相反，动态链接的二进制文件在从磁盘加载时是不完整的。因此，内核不能立即将控制权交给应用程序。相反，必须加载另一个已经是完整状态的助手程序。这个助手程序就是动态链接器。动态链接器的任务是，通过加载应用程序所需的DSO（即依赖关系）并执行重定位作业，来完成动态链接应用程序的构建。之后，控制权才最终可以交给程序。

This is not the last task for the dynamic linker in most cases, though. ELF allows the relocations associated with a symbol to be delayed until the symbol is needed. This lazy relocation scheme is optional, and optimizations discussed below for relocations performed at startup immediately affect the lazy relocations as well. So we ignore in the following everything after the startup is finished.

然而，在大多数情况下，这不是动态链接器的最后一个任务。ELF允许将与符号相关的重定位延迟到需要符号时才进行。这种延迟重定位方案是可选的，并且接下来要讨论的，在启动时执行的重定位优化也会立即影响到延迟重定位。因此，我们在后续的讨论中将不再涉及启动完成后的任何内容。

### 1.4 Startup: In The Kernel

Starting execution of a program begins in the kernel, normally in the execve system call. The currently executed code is replaced with a new program. This means the address space content is replaced by the content of the file containing the program. This does not happen by simply mapping (using mmap) the content of the file. ELF files are structured and there are normally at least three different kinds of regions in the file:

程序的执行起始于内核，通常是在 `execve` 系统调用中。当前执行的代码会被新程序替换。这意味着地址空间的内容被包含程序的文件内容所替换。这一替换并非仅通过映射（使用 `mmap`）文件的内容来实现。ELF文件是有结构的，并且通常至少有三种不同类型的区域：

- Code which is executed; this region is normally not writable;

    被执行的代码；这个区域通常是不可写的；


- Data which is modified; this region is normally not executable;

    被修改的数据；这个区域通常是不可执行的；

- Data which is not used at run-time; since not needed it should not be loaded at startup.

    在运行时不被使用的数据；由于不需要，它不应该在启动时被加载

Modern operating systems and processors can protect memory regions to allow and disallow reading, writing, and executing separately for each page of memory. It is preferable to mark as many pages as possible not writable since this means that the pages can be shared between processes which use the same application or DSO the page is from. Write protection also helps to detect and prevent unintentional or malignant modifications of data or even code.

现代操作系统和处理器可以保护内存区域，分别允许和禁止对每个内存页的读取、写入和执行。最好将尽可能多的页面标记为不可写，因为这意味着这些页面可以在使用同一应用程序或DSO的进程之间共享。写保护也有助于检测和防止数据甚至代码的无意或恶意修改。

For the kernel to find the different regions, or segments in ELF-speak, and their access permissions, the ELF file format defines a table which contains just this information, among other things. The ELF Program Header table, as it is called, must be present in every executable and DSO. It is represented by the C types Elf32_Phdr and Elf64_Phdr which are defined as can be seen in figure 1.

为了让内核找到不同的区域（或在ELF中称为 $segments$）及其访问权限，ELF文件格式定义了一个包含这些信息（以及其他信息）的表。这个表被称为ELF程序头部表($ELF\ Program\ Header\ table$)，它必须存在于每个可执行文件和DSO中。它由C类型 `Elf32_Phdr` 和 `Elf64_Phdr` 表示，这些类型的定义如图1所示。

<div class="center">

```c
typedef struct {
  Elf64_Word p_type;    /* Segment type */
  Elf64_Word p_flags;   /* Segment flags */
  Elf64_Off p_offset;   /* Segment file offset */
  Elf64_Addr p_vaddr;   /* Segment virtual address */
  Elf64_Addr p_paddr;   /* Segment physical address */
  Elf64_Xword p_filesz; /* Segment size in file */
  Elf64_Xword p_memsz;  /* Segment size in memory */
  Elf64_Xword p_align;  /* Segment alignment */
} Elf64_Phdr;
```

</div>

<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 1: ELF Program Header C Data Structure</div>
</center>

<br/>

To locate the program header data structure another data structure is needed, the ELF Header. The ELF header is the only data structure which has a fixed place in the file, starting at offset zero. Its C data structure can be seen in figure 2. The e_phoff field specifies where, counting from the beginning of the file, the program header table starts. The e_phnum field contains the number of entries in the program header table and the e_phentsize field contains the size of each entry. This last value is useful only as a run-time consistency check for the binary.

为了定位程序头部数据结构，需要另一个数据结构，即ELF头部（$ELF\ Header$）。ELF头部是文件中唯一固定位置的数据结构，从偏移量0开始。其C数据结构如图2所示。`e_phoff`字段指定了程序头部表从文件开始的位置，`e_phnum`字段包含程序头部表中的条目数，`e_phentsize`字段包含每个条目的大小。这最后一个值只在运行时用于二进制文件的一致性检查。

<div class="center">

```c
typedef struct {
  unsigned char	e_ident[EI_NIDENT];	/* Magic number and other info */
  Elf64_Half e_type;                /* Object file type */
  Elf64_Half e_machine;             /* Architecture */
  Elf64_Word e_version;             /* Object file version */
  Elf64_Addr e_entry;               /* Entry point virtual address */
  Elf64_Off	e_phoff;                /* Program header table file offset */
  Elf64_Off	e_shoff;                /* Section header table file offset */
  Elf64_Word e_flags;               /* Processor-specific flags */
  Elf64_Half e_ehsize;              /* ELF header size in bytes */
  Elf64_Half e_phentsize;           /* Program header table entry size */
  Elf64_Half e_phnum;               /* Program header table entry count */
  Elf64_Half e_shentsize;           /* Section header table entry size */
  Elf64_Half e_shnum;               /* Section header table entry count */
  Elf64_Half e_shstrndx;            /* Section header string table index */
} Elf64_Ehdr;
```

</div>

<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 2: ELF Header C Data Structure</div>
</center>

<br/>

The different segments are represented by the program header entries with the PT LOAD value in the p_type field. The p_offset and p_filesz fields specify where in the file the segment starts and how long it is. The p_vaddr and p_memsz fields specify where the segment is located in the process’ virtual address space and how large the memory region is. The value of the p_vaddr field itself is not necessarily required to be the final load address. DSOs can be loaded at arbitrary addresses in the virtual address space. But the relative position of the segments is important. For pre-linked DSOs the actual value of the p_vaddr field is meaningful: it specifies the address for which the DSO was pre-linked. But even this does not mean the dynamic linker cannot ignore this information if necessary.

不同的段在程序头部条目中由 `PT_LOAD` 值在 `p_type` 字段中表示。 `p_offset` 和 `p_filesz` 字段指定段在文件中的起始位置以及其长度。`p_vaddr` 和 `p_memsz` 字段指定了段在进程虚拟地址空间中的位置和内存区域的大小。`p_vaddr` 字段的值不必然是最终加载地址。DSO可以在虚拟地址空间的任意地址被加载。但段之间的相对位置是重要的。对于预链接的DSO，`p_vaddr` 字段的实际值是有意义的：它指定了DSO被预链接的地址。但即便如此，如果有必要，动态链接器也可以忽略这些信息。

The size in the file can be smaller than the address space it takes up in memory. The first `p_filesz` bytes of the memory region are initialized from the data of the segment in the file, the difference is initialized with zero. This can be used to handle BSS sections, sections for uninitialized variables which are according to the C standard initialized with zero. Handling uninitialized variables this way has the advantage that the file size can be reduced since no initialization value has to be stored, no data has to be copied from disc to memory, and the memory provided by the OS via the mmap interface is already initialized with zero.

在ELF文件中定义的段（如代码段或数据段）在文件中占用的实际字节大小可能比它们在程序运行时占用的内存大小要小。对于这些段的内存区域，它们开始的 `p_filesz` 字节将会从文件中读取数据来初始化。如果内存区域的大小（`p_memsz`）大于文件中的大小（`p_filesz`），那么剩下的部分将会用零来填充。这种方式特别适用于处理BSS段（$Block\ Started\ by\ Symbol$），即那些在程序启动时需要被初始化为零的未初始化变量。通过这种方式，文件的大小可以被减少，因为不需要在文件中存储这些变量的初始零值；当程序加载到内存时，操作系统会保证这部分内存被零填充。这不仅减少了文件大小，还省去了将初始零值从磁盘复制到内存的过程，因为操作系统通过`mmap`接口提供的内存页默认就是零初始化的。

The p_flags finally tells the kernel what permissions to use for the memory pages. This field is a bitmap with the bits given in the following table being defined. The flags are directly mapped to the flags mmap understands.

`p_flags` 字段最终告诉内核使用何种权限来处理内存页。这个字段是一个位图，其中的位按下表给出的定义。这些标志直接映射到 `mmap` 能理解的标志上。


<div class="center">

| p_flags | Value | mmap flag | Description |
|  :---:  | :---: |   :---:   |    :---:    |
| PF_X    | 1     | PROT_EXEC | Execute Permission |
| PF_W    | 2     | PROT_WRITE | Write Permission |
| PF_R    | 4     | PROT_READ | Read Permission |

</div>

After mapping all the PT LOAD segments using the appropriate permissions and the specified address, or after freely allocating an address for dynamic objects which have no fixed load address, the next phase can start. The virtual address space of the dynamically linked executable itself is set up. But the binary is not complete. The kernel has to get the dynamic linker to do the rest and for this the dynamic linker has to be loaded in the same way as the executable itself (i.e., look for the loadable segments in the program header). The difference is that the dynamic linker itself must be complete and should

在使用适当的权限和指定的地址映射完所有`PT_LOAD`段之后，或者为没有固定加载地址的动态对象自由分配地址之后，下一个阶段才可以开始。动态链接的可执行文件自身的虚拟地址空间就此建立。但是，二进制文件还不完整。内核必须让动态链接器来完成剩下的工作，为此动态链接器必须以与可执行文件本身相同的方式被加载（即，在程序头中查找可加载的段）。不同之处在于动态链接器本身必须是完整的，并且应该能够自由重定位。

Which binary implements the dynamic linker is not hardcoded in the kernel. Instead, the program header of the application contains an entry with the tag PT_INTERP. The p_offset field of this entry contains the offset of a NUL-terminated string which specifies the file name of this file. The only requirement on the named file is that its load address does not conflict with the load address of any possible executable it might be used with. In general, this means that the dynamic linker has no fixed load address and can be loaded anywhere; this is just what dynamic binaries allow.

可执行文件并不直接包含动态链接器（如 `ld-linux.so`）的路径。相反，ELF文件的程序头中有一个特殊的条目，标记为 `PT_INTERP`。这个条目指向一个字符串，这个条目的 `p_offset` 字段包含一个NUL终止字符串的偏移量，该字符串是动态链接器文件的路径。当内核准备运行一个动态链接的可执行文件时，它会查看这个 `PT_INTERP` 条目，找到动态链接器的路径，并将其加载到内存中。动态链接器的加载地址不是硬编码的，意味着它可以被加载到进程的虚拟地址空间中的任意位置，这为动态链接器提供了灵活性，并允许它处理来自不同可执行文件的不同需求。

Once the dynamic linker has also been mapped into the memory of the to-be-started process we can start the dynamic linker. Note it is not the entry point of the application to which control is transferred to. Only the dynamic linker is ready to run. Instead of calling the dynamic linker right away, one more step is performed. The dynamic linker somehow has to be told where the application can be found and where control has to be transferred to once the application is complete. For this a structured way exists. The kernel puts an array of tag-value pairs on the stack of the new process. This auxiliary vector contains beside the two aforementioned values several more values which allow the dynamic linker to avoid several system calls. The elf.h header file defines a number of constants with a AT prefix. These are the tags for the entries in the auxiliary vector.

一旦动态链接器被映射到即将启动的进程的内存中，接下来就可以启动动态链接器了。需要注意的是，此时程序的控制权并不是直接转移到应用程序的主入口点。而是，只有动态链接器处于就绪状态，准备开始其工作。另外，在开始动态链接器的调用之前还需要执行一个步骤。需要以某种方式告诉动态链接器应用程序在哪里可以找到，以及一旦应用程序准备完毕后控制权需要转移到哪里。为此，存在一种结构化的方式。内核在新进程的堆栈上放置一个标签 `tag-value pairs`。这个辅助向量除了上述两个值外，还包含几个其他值，这些值允许动态链接器避免多次系统调用。`elf.h`头文件定义了一系列带有`AT`前缀的常量，这些是辅助向量中条目的标签。

After setting up the auxiliary vector, the kernel is finally ready to transfer control to the dynamic linker in user mode. The entry point is defined in the e_entry field of the ELF header of the dynamic linker.

在设置好辅助向量之后，内核最终准备好将控制权以用户模式转移给动态链接器。入口点在动态链接器的ELF头的 `e_entry` 字段中定义。

### 1.5 Startup in the Dynamic Linker

The second phase of the program startup happens in the dynamic linker. Its tasks include:

程序启动的第二阶段发生在动态链接器中。其任务包括：

- Determine and load dependencies;

    确定并加载依赖项；找出应用程序需要的所有动态共享对象（DSO）并将它们加载到内存中。

- Relocate the application and all dependencies;

    重定位应用程序及所有依赖项；调整应用程序和其依赖的代码和数据，使它们能够正确地引用彼此的地址。简单来说，就是确保程序中的所有指针和引用指向正确的内存位置。

- Initialize the application and dependencies in the correct order.

    按正确的顺序初始化应用程序及依赖项；在程序开始执行前，以正确的顺序初始化所有加载的组件，确保所有初始化函数都被调用。

In the following we will discuss in more detail only the relocation handling. For the other two points the way for better performance is clear: have fewer dependencies. Each participating object is initialized exactly once but some topological sorting has to happen. The identify and load process also scales with the number dependencies; in most (all?) implementations this does not scale linearly.

接下来，我们将更详细地讨论重定位处理。对于其他两点，提高性能的方法很明确：减少依赖项的数量。每个参与的对象文件都会被精确地初始化一次，但需要进行某种形式的拓扑排序。随着依赖项数量的增加，识别和加载过程也会相应扩大；在大多数（可能所有？）实现中，这种扩大不是线性的。

The relocation process is normally the most expensive part of the dynamic linker’s work. It is a process which is asymptotically at least $O(R + n * r)$ where R is the number of relative relocations, r is the number of named relocations, and n is the number of participating DSOs (plus the main executable). Deficiencies in the ELF hash table function and various ELF extensions modifying the symbol lookup functionality may well increase the factor to $O(R + r * n * \log_2 s)$ where s is the number of symbols. This should make clear that for improved performance it is significant to reduce the number if relocations and symbols as much as possible. After explaining the relocation process we will do some estimates for actual numbers.

重定位过程通常是动态链接器工作中最为昂贵的部分。从理论上讲，这一过程的时间复杂度至少为 $O(R + n * r)$，其中 $R$ 是相对重定位的数量，$r$ 是命名重定位的数量，$n$ 是参与的DSO数量（加上主执行文件）。ELF哈希表函数的不足以及对符号查找功能的各种ELF扩展可能会将这一因子提高到 $O(R + r * n * \log_2 s)$，其中 $s$ 是符号的数量。所以我们明确的是，为了提高性能，尽可能减少重定位和符号的数量是很重要的。在解释了重定位过程之后，我们将对实际的数字进行一些估算。

### 1.5.1 The Relocation Process

Relocation in this context means adjusting the application and the DSOs, which are loaded as the dependencies, to their own and all other load addresses. There are two kinds of dependencies:

在这里讨论的重定位，指的是将应用程序（特指那些在编译和链接过程结束后形成的、可以直接被操作系统加载并执行的程序，即可执行程序）及其作为依赖加载的DSO调整到它们自身和所有其他加载地址上的过程。存在两种类型的依赖关系：

- Dependencies to locations which are known to be in the own object. These are not associated with a specific symbol since the linker knows the relative position of the location in the object. Note that applications do not have relative relocations since the load address of the code is known at link-time and therefore the static linker is able to perform the relocation.

    依赖于已知位于本对象文件（指的是程序中的一个组件或编译单元，即编译后的中间文件 .o 或 .obj）内的位置。这些依赖不关联特定的符号，因为链接器知道这个位置在对象文件内的相对位置（同一个编译单元或代码库内的地址引用，这些地址在链接时相对于对象文件自身已知）。需要注意的是，应用程序没有相对位置的重定位，因为代码的加载地址在链接时已经确定（指的是链接的对象文件，也就是说对象文件已经通过相对重定位实现了地址的确定，所以也就意味着应用程序本身也就不需要相对重定位），因此静态链接器能够执行这种重定位。

- Dependencies based on symbols. The reference of the definition is generally, but not necessarily, in a different object than the definition.

    基于符号的依赖。这种依赖的引用通常（但不总是）位于定义所在的不同对象中；这涉及到跨DSO的引用，例如一个DSO中的函数调用另一个DSO中的函数。这种重定位比较复杂，因为它要处理不同对象之间的关系，需要在运行时确定符号的最终地址。

The implementation of relative relocations is easy. The linker can compute the offset of the target destination in the object file at link-time. To this value the dynamic linker only has to add the load address of the object and store the result in the place indicated by the relocation. At runtime the dynamic linker has to spend only a very small and constant amount of time which does not increase if more DSOs are used.

相对重定位的实施相对简单。链接器可以在链接时计算目标位置在对象文件中的偏移量。动态链接器只需将这个偏移量加上对象文件的加载地址，并将结果存储在重定位指示的位置。在运行时，动态链接器只需要花费很少且固定的时间来处理，这个时间不会因为使用更多的DSO而增加。

The relocation based on a symbol is much more complicated. The ELF symbol resolution process was designed very powerful so that it can handle many different problems. All this powerful functionality adds to the complexity and run-time costs, though. Readers of the following description might question the decisions which led to this process. We cannot argue about this here; readers are referred to discussions of ELF. Fact is that symbol relocation is a costly process and the more DSOs participate or the more symbols are defined in the DSOs, the longer the symbol lookup takes.

基于符号的重定位则复杂得多。ELF符号解析过程设计得非常强大，以便它可以处理许多不同的问题。然而，这种强大的功能增加了复杂性和运行时的成本。读者可能会对导致此过程的决策提出疑问。在这里我们无法就此进行讨论；读者可以参考有关ELF的讨论。事实是，符号重定位是一个成本高昂的过程，参与的DSO越多或DSO中定义的符号越多，符号查找所需的时间就越长。

The result of any relocation will be stored somewhere in the object with the reference. Ideally and generally the target location is in the data segment. If code is incorrectly generated by the user, compiler, or linker relocations might modify text or read-only segments. The dynamic linker will handle this correctly if the object is marked, as required by the ELF specification, with the DF_TEXTREL set in the DT_FLAGS entry of the dynamic section (or the existence of the DT_TEXTREL flag in old binaries). But the result is that the modified page cannot be shared with other processes using the same object. The modification process itself is also quite slow since the kernel has to reorganize the memory handling data structures quite a bit.

重定位的结果会存放在与引用相关的对象文件内的某处。通常，这样的目标位置应该在数据段中。但如果因为编程、编译或链接过程中的错误，导致重定位错误地修改了代码段或只读数据段，那么动态链接器需要按照ELF规范来正确处理这种情况。具体来说，如果对象文件的动态段(dynamic section)中的 `DT_FLAGS` 设置了 `DF_TEXTREL` 标志，或者在较老的二进制文件中存在 `DT_TEXTREL` 标志，那么动态链接器就知道如何正确地处理这些修改。然而，这种修改会有两个后果：首先，因为页面内容被修改，这些页面就不能和其他使用相同代码的进程共享了，每个进程都需要有自己的副本。其次，这种修改过程通常比较慢，因为操作系统需要对内存管理的数据结构进行大量的调整来应对这些修改。


### 1.5.2 Symbol Relocations

The dynamic linker has to perform a relocation for all symbols which are used at run-time and which are not known at link-time to be defined in the same object as the reference. Due to the way code is generated on some architectures, it is possible to delay the processing of some relocations until the references in question are actually used. This is on many architectures the case for calls to functions. All other kinds of relocations always have to be processed before the object can be used. We will ignore the lazy relocation processing since this is just a method to delay the work. It eventually has to be done and so we will include it in our cost analysis. To actually perform all the relocations before using the object is used by setting the environment variable LD_BIND_NOW to a non-empty value. Lazy relocation can be disabled for an individual object by adding the -z now option to the linker command line. The linker will set the DF_BIND_NOW flag in the DT_FLAGS entry of the dynamic section to mark the DSO. This setting cannot be undone without relinking the DSOs or editing the binary, though, so this option should only be used if it is really wanted.

动态链接器必须对所有在运行时被使用、并且在链接时不确定是否在引用同一对象文件内定义的符号进行重定位。由于某些架构生成代码的方式，一些重定位的处理可以延迟到实际使用相关引用时再进行。对于许多架构来说，函数调用就是这种情况。除此之外，所有类型的重定位都必须在对象文件被使用之前完成处理。我们将不讨论延迟重定位处理，因为那只是延迟工作的一种方式。这项工作最终是必须完成的，因此我们会把它计入我们的成本分析中。通过将环境变量 `LD_BIND_NOW` 设置为非空值，可以确保在使用对象文件之前完成所有重定位。为了禁用个别对象文件的延迟重定位，可以在链接命令行中添加 `-z now` 选项。链接器会在动态段的 `DT_FLAGS` 条目中设置 `DF_BIND_NOW` 标志来标记该DSO。不过，除非重新链接DSO或编辑二进制文件，否则无法取消这一设置，因此这个选项应该只在真正需要时使用。

The actual lookup process is repeated from start for each symbol relocation in each loaded object. Note that there can be many references to the same symbol in different objects. The result of the lookup can be different for each of the objects so there can be no short cuts except for caching results for a symbol in each object in case more than one relocation references the same symbol. The lookup scope mentioned in the steps below is an ordered list of a subset of the loaded objects which can be different for each object itself. The way the scope is computed is quite complex and not really relevant here so we refer the interested reader to the ELF specification and section 1.5.4. Important is that the length of the scope is normally directly dependent on the number of loaded objects. This is another factor where reducing the number of loaded objects is increasing performance.

实际的符号查找过程对于每个加载的对象中的每次符号重定位都需要重复进行。注意，由于不同对象可能引用相同的符号，而且每个对象对于同一符号的解析结果可能不同，导致动态链接器必须分别对每个引用进行处理，不能简单地跳过或假定它们是相同的。因此，除了对每个对象中相同符号的多次重定位结果进行缓存以外，没有捷径。查找范围是一个有序的、包含了一部分已加载对象的子集的列表，这个列表对每个对象自身可能不同。这个范围的计算方式相当复杂，这里不做深入讨论，有兴趣的读者可以参考ELF规范和1.5.4节。重要的是，查找范围的长度通常直接依赖于已加载对象的数量。这意味着，减少已加载对象的数量是提高性能的一个途径（减少程序运行时需要加载和处理的动态共享对象（DSO）或库的数量）。

There are today two different methods for the lookup process for a symbol. The traditional ELF method proceeds in the following steps:

现今，有两种不同的方法用于符号的查找过程。传统的ELF方法按以下步骤进行：

1. Determine the hash value for the symbol name.

    确定符号名称的哈希值。

2. In the first/next object in the lookup scope:

    在查找范围中的第一个/下一个对象中：

    * 2.a Determine the hash bucket for the symbol using the hash value and the hash table size in the object.

        使用哈希值和对象中的哈希表大小确定符号的哈希桶。

    * 2.b Get the name offset of the symbol and using it as the NUL-terminated name.

        获取符号的名称偏移，并将其作为NUL终止的名称。

    * 2.c Compare the symbol name with the relocation name.

        将符号名称与重定位名称进行比较。

    * 2.d If the names match, compare the version names as well. This only has to happen if both, the reference and the definition, are versioned. It requires a string comparison, too. If the version name matches or no such comparison is performed, we found the definition we are looking for.

        如果名称匹配，也比较版本名称。这只在引用和定义都有版本时才需要进行。这也需要进行字符串比较。如果版本名称匹配或没有进行此类比较，则找到了我们正在查找的定义。


    * 2.e If the definition does not match, retry with the next element in the chain for the hash bucket.

        如果定义不匹配，使用哈希桶链中的下一个元素重试。

    * 2.f If the chain does not contain any further element there is no definition in the current object and we proceed with the next object in the lookup scope.

        如果链中没有更多元素，表示当前对象中没有定义，我们继续处理查找范围中的下一个对象。


3. If there is no further object in the lookup scope the lookup failed.

   如果查找范围中没有更多对象，则查找失败。


<div class="center">

```
Histogram for bucket list length in section [ 2] ’.hash’ (total of 1023 buckets):
    Addr: 0x42000114 Offset: 0x000114 Link to section: [ 3] ’.dynsym’
    Length Number % of total Coverage
    0   132   12.9%
    1   310   30.3%   15.3%
    2   256   25.0%   40.6%
    3   172   16.8%   66.0%
    4   92    9.0%    84.2%
    5   46    4.5%    95.5%
    6   14    1.4%    99.7%
    7   1     0.1%    100.0%
    Average number of tests: successful lookup: 1.994080
                           unsuccessful lookup: 1.981427
```

</div>

<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 3: Example Output for eu-readelf -I libc.so</div>
</center>

<br/>

<div class="center">

```
Histogram for bucket list length in section [ 2] ’.hash’ (total of 191 buckets):
    Addr: 0x00000114 Offset: 0x000114 Link to section: [ 3] ’.dynsym’
    Length Number % of total Coverage
    0   103   53.9%
    1   71    37.2%  67.0%
    2   16    8.4%   97.2%
    3   1     0.5%   100.0%
    Average number of tests: successful lookup: 1.179245
                           unsuccessful lookup: 0.554974
```

</div>

<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 4: Example Output for eu-readelf -I libnss files.so</div>
</center>