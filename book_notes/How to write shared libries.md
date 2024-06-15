---
attachment: "[[How to write shared libries.canvas|Mind map]]"
tags:
  - "#elf"
  - "#linux"
  - "#shared_library"
  - "#c"
---
# 原文

https://www.akkadia.org/drepper/dsohowto.pdf

# Abstract

Today, shared libraries are ubiquitous. Developers use them for multiple reasons and create them just as they would create application code. This is a problem, though, since on many platforms some additional techniques must be applied even to generate decent code. Even more knowledge is needed to generate optimized code. This paper introduces the required rules and techniques. In addition, it introduces the concept of ABI (Application Binary Interface) stability and shows how to manage it.

如今，共享库无处不在。开发者出于多种原因使用它们，并且创建它们就像创建应用程序代码一样。然而，这是一个问题，因为在许多平台上，即使是生成合理的代码，也必须应用一些额外的技术。生成优化代码则需要更多的知识。本文介绍了所需的规则和技术。此外，它还介绍了ABI（应用程序二进制接口）稳定性的概念，并展示了如何管理它。

# 1. Preface

For a long time, programmers collected commonly used code in libraries so that code could be reused. This saves development time and reduces errors sincereused code only has to be debugged once. With systems running dozens or hundreds of processes at the same time reuse of the code at link-time solves only part of the problem. Many processes will use the same pieces of code which they import for libraries. With the memory management systems in modern operating systems it is also possible to share the code at run-time. This is done by loading the code into physical memory only once and reusing it in multiple processes via virtual memory.  Libraries of this kind are called shared libraries.

在信息技术的早期，程序员们通过将代码集中在库中来实现代码重用，从而节省了资源并提高了效率。随着系统的发展，需要同时运行多个进程，单纯在链接时重用代码已经无法满足需求。为此，现代操作系统采用了内存管理技术，允许在运行时共享代码，即通过虚拟内存技术实现多个进程中的代码重用，而这些代码只在物理内存中加载一次。然而，操作系统的这种扩展并不总是一帆风顺。特别是考虑到早期 UNIX 系统所使用的二进制格式，如 a.out 或 COFF，并没有为共享库的需求而设计，这在实际应用中带来了不少问题。这些格式缺乏灵活性，并且在共享库环境下可能导致效率低下或兼容性问题 。 ^4b1c2e

The concept is not very new. Operating system designers implemented extensions to their system using the infrastructure they used before. The extension to the OS could be done transparently for the user. But the parts the user directly has to deal with initially created problems.

共享库的概念由来已久。操作系统设计师通过以前的基础设施扩展了他们的系统，并以透明的方式为用户提供这些扩展功能。但起初给用户带来了问题。

The main aspect is the binary format. This is the format which is used to describe the application code. Long gone are the days that it was sufficient to provide a memory dump. Multi-process systems need to identify different parts of the file containing the program such as the text, data, and debug information parts. For this, binary formats were introduced early on. Commonly used in the early Unix-days were formats such as a.out or COFF. These binary formats were not designed with shared libraries in mind and this clearly shows.

主要问题在于二进制格式。这种格式被用来描述应用程序的代码。过去，仅提供内存转储对于代码描述已经足够了。但是，随着系统需要同时运行数十甚至数百个进程，仅在链接时重用代码变得不够用。系统需要识别程序文件中不同的部分，比如文本、数据和调试信息。为此，二进制格式在很早前就被引入了。早期Unix使用的二进制格式，如 a.out(assembler output) 或 COFF(Common Object File Format)，并未设计时考虑共享库，这一点在它们的设计中非常明显。

## 1.1 A Little Bit of History

The binary format used initially for Linux was an a.out variant. When introducing shared libraries certain design decisions had to be made to work in the limitations of a.out. The main accepted limitation was that no ***relocations*** are performed at the time of loading and afterward. The shared libraries have to exist in the form they are used at run-time on disk. This imposes a major restriction on the way shared libraries are built and used: every shared library must have a fixed load address; otherwise, it would not be possible to generate shared libraries which do not have to be relocated.

Linux最初使用的二进制格式是 a.out 的一个变种。在引入共享库时，必须在a.out的限制条件下做出一些设计决策。主要接受的限制是，在加载和加载后不进行***重定位***。也就是说，共享库在磁盘上的存储形式必须与它在运行时被加载到内存中的形式完全一致(内存布局)。这对共享库的构建和使用方式施加了重大限制：每个共享库都必须有一个固定的加载地址；换句话说，如果没有这些固定的加载地址，就无法创建不需要在运行时进行地址重定位的共享库。

The fixed load addresses had to be assigned and this has to happen without overlaps and conflicts and with some future safety by allowing growth of the shared library. It is therefore necessary to have a central authority for the assignment of address ranges which in itself is a major problem. But it gets worse: given a Linux system of today with many hundred of DSOs (Dynamic Shared Objects) the address space and the virtual memory available to the application gets severely fragmented. This would limit the size of memory blocks which can be dynamically allocated which would create unsurmountable problems for some applications. It would even have happened by today that the assignment authority ran out of address ranges to assign, at least on 32-bit machines.

固定加载地址必须在避免重叠和冲突的过程中被分配，并且通过允许共享库的增长来确保未来的安全性。因此，需要有一个集中的管理器来分配地址范围，这本身就是一个重大问题而且情况会变得更糟：鉴于今天的Linux系统拥有数百个DSO（动态共享对象），应用程序可用的地址空间和虚拟内存变得极其碎片化。这将限制可以动态分配的内存块的大小，从而为某些应用程序创造了无法克服的问题。甚至在今天，该系统(集中的管理器)可能已经用尽了可分配的地址范围，至少在32位机器上是这样。

We still have not covered all the drawbacks of the a.out shared libraries. Since the applications using shared libraries should not have to be relinked after changing a shared library it uses, the entry points, i.e., the function and variable addresses, must not change. This can only be guaranteed if the entry points are kept separate from the actual code since otherwise limits on the size of a function would be hard-coded. A table of function stubs which call the actual implementation was the solution used on Linux. The static linker got the address of each function stub from a special file (with the filename extension .sa). At run-time a file ending in .so.X.Y.Z was used and it had to correspond to the used .sa file. This in turn requires that an allocated entry in the stub table always had to be used for the same function. The allocation of the table had to be carefully taken care of. Introducing a new interface meant appending to the table. It was never possible to retire a table entry. To avoid using an old shared library with a program linked with a newer version, some record had to be kept in the application: the X and Y parts of the name of the .so.X.Y.Z suffix was recorded and the dynamic linker made sure minimum requirements were met.

在 a.out 格式的共享库中，我们还没有讨论到所有的缺点。由于使用共享库的应用程序在更换它使用的共享库后不应该需要重新链接，因此，入口点（Entry Point），即函数和变量地址，不应该发生变化。这只能通过将入口点与实际代码分开来保证，否则函数的大小（指的是字节码的长度）限制将被硬编码。Linux上使用的解决方案是调用实际实现的函数存根表（Function Stub）。静态链接器从一个特定的扩展名为 [.sa](https://stackoverflow.com/questions/8876231/what-is-a-sa-file-for-gcc-shared-library) 的文件中获取每个函数存根的地址。运行时则使用以 .so.X.Y.Z 为后缀的文件，并确保它与 .sa 文件相对应。这样就要求为存根表分配的每一个条目必须始终对应同一个函数。表的分配需要精心管理。添加新的接口意味着需要向表中添加条目，表条目一旦创建就无法被删除。为了防止程序在链接了新版本后仍使用旧的共享库，必须在应用程序中保留某种记录，记录下 .so 文件名中的 X 和 Y 部分，并且动态链接器要确保这些最低要求得到满足。

The benefits of the scheme are that the resulting program runs very fast. Calling a function in such shared libraries is very efficient even for the first call. It can be implemented with only two absolute jumps: the first from the user code to the stub, and the second from the stub to the actual code of the function. This is probably faster than any other shared library implementation, but its speed comes at too high a price: 

该方案的好处是，结果程序运行非常快。即使是第一次调用，也能在这种共享库中非常高效地调用函数。它可以仅通过两次绝对跳转来实现：第一次从用户代码跳到存根，第二次从存根跳到函数的实际代码。这可能比任何其他共享库实现都要快，但代价太高：

1. a central assignment of address ranges is needed; 

      需要中央分配地址范围；这是为了避免不同的共享库或模块尝试使用相同的内存地址而产生冲突。这种中心化的管理可能变成一个瓶颈，并且难以维护，特别是随着系统的依赖关系和规模的增长。

2. collisions are possible (likely) with catastrophic results; 

      可能（并且很可能）发生冲突，带来灾难性的后果；如果两个或更多的库被分配了重叠的地址范围，可能会导致运行时错误，这些错误难以诊断和解决。这些冲突可能会损坏数据或导致程序崩溃，从而带来“灾难性”的后果，尤其是在稳定性和数据完整性至关重要的环境中。

3. the address space gets severely fragmented.

      地址空间严重碎片化；随着时间的推移，随着库的加载和卸载，内存地址空间可能会变得碎片化。这种碎片化可能导致内存使用效率低下，并限制应用程序分配大块连续内存块的能力，这种内存通常对性能至关重要。

For all these reasons and more, Linux converted early on to using ELF (Executable Linkage Format) as the binary format. The ELF format is defined by the generic specification (gABI) to which processor-specific extensions (psABI) are added. As it turns out the amortized cost of function calls is almost the same as for a.out but the restrictions are gone.

由于这些原因以及更多其他原因，Linux很早就转换为使用 ELF（Executable Linkage Format）作为二进制格式。ELF 格式由通用规范（gABI）定义，其中添加了特定于处理器的扩展（psABI）。事实证明，函数调用的平摊成本几乎与a.out相同，但限制已经消失。

## 1.2 The Move To ELF

For programmers the main advantage of the switch to ELF was that creating ELF shared libraries, or in ELF-speak DSOs, becomes very easy. The only difference between generating an application and a DSO is in the final link command line. One additional option (--shared in the case of GNU ld) tells the linker to generate a DSO instead of an application, the latter being the default. In fact, DSOs are little more than a special kind of binary; the difference is that they have no fixed load address and hence require the dynamic linker to actually become executable. With Position Independent Executable (PIEs) the difference shrinks even more.

程序员转向 ELF 最大的好处，是制作 ELF 格式的共享库（即DSO）变得轻而易举。创建应用程序和DSO的差别，仅仅在于最终的链接命令行中多了一个选项（GNU ld中的--shared），这个选项会指示链接器生成DSO而非默认的应用程序。说到底，DSO其实就是一种特别的二进制文件，它们之所以与众不同，是因为 DSO 没有固定的加载地址，必须依赖动态链接器才能变为可执行的。随着位置无关执行文件（PIE）的出现，这种区别进一步缩小了。

This, together with the introduction of GNU Libtool which will be described later, has led to the wide adoption of DSOs by programmers. Proper use of DSOs can help save large amounts of resources. But some rules must be followed to get any benefits, and some more rules have to be followed to get optimal results. Explaining these rules will be the topic of a large portion of this paper.

正是这些优势，加上GNU Libtool的引入——这个工具稍后将详细介绍，促使程序员广泛采用了DSO。正确地使用DSO可以显著节约资源。但是，要从中受益，必须遵循一些基本规则，想要达到最佳效果，则需要更多规则。本文将详细解释这些规则。

Not all uses of DSOs are for the purpose of saving resources. DSOs are today also often used as a way to structure programs. Different parts of the program are put into separate DSOs. This can be a very powerful tool, especially in the development phase. Instead of relinking the entire program it is only necessary to relink the DSO(s) which changed. This is often much faster.

使用DSO并不总是为了节约资源。如今，DSO还常被用于程序的结构化设计中，将程序的不同部分放入不同的DSO。这在开发阶段尤其有用，因为此时无需重新编译整个程序，只需重新链接有所更改的DSO，这通常更快。

Some projects decide to keep many separate DSOs even in the deployment phase even though the DSOs are not reused in other programs. In many situations it is certainly a useful thing to do: DSOs can be updated individually, reducing the amount of data which has to be transported. But the number of DSOs must be kept to a reasonable level. Not all programs do this, though, and we will see later on why this can be a problem.

即便到了部署阶段，有些项目依然保留多个独立、分离的DSO，（而不是将它们合并成更少的几个或一个大的DSO），即使这些DSO不会在其他程序中重用。在许多情况下，这样做确实有益，因为可以单独更新各个DSO，减少传输数据的量。但DSO的数量还是需要控制在合理的范围内，并不是所有的程序都遵循这一原则，我们将在后文探讨为何过多的DSO可能会引发问题。

Before we can start discussing all this some understanding of ELF and its implementation is needed.

在我们深入讨论之前，我们需要先了解ELF及其背后的实现原理。

## 1.3 How Is ELF Implemented?

The handling of a statically linked application is very simple. Such an application has a fixed load address which the kernel knows. The load process consists simply of making the binary available in the appropriate address space of a newly created process and transferring control to the entry point of the application. Everything else was done by the static linker when creating the executable.

处理静态链接的应用程序非常简单。这样的应用程序有一个内核已知的固定加载地址（虽然ASLR技术会在程序实际启动的时候再次被动态分配，但是基于基地址的偏移是固定的）（虽然是固定的地址，但是由于内核使用虚拟地址空间技术管理内存，所以也就不存在冲突的说法）。加载过程简单到只需要将二进制文件放置在新创建的进程的适当地址空间内，并将控制权转移给应用程序的入口点。其他所有事情都是在创建可执行文件时由静态链接器完成的。

Dynamically linked binaries, in contrast, are not complete when they are loaded from disk. It is therefore not possible for the kernel to immediately transfer control to the application. Instead some other helper program, which obviously has to be complete, is loaded as well. This helper program is the ***dynamic linker***. The task of the dynamic linker is it, to complete the dynamically linked application by loading the DSOs it needs (the dependencies) and to perform the relocations. Then finally control can be transferred to the program.

与之相反，动态链接的二进制文件在从磁盘加载时是不完整的。因此，内核不能立即将控制权交给应用程序。相反，必须加载另一个已经是完整状态（即它包含了执行所需任务所必须的全部功能和资源）的辅助程序。这个辅助程序就是***动态链接器***。动态链接器的任务是，通过加载应用程序所需的DSO（即依赖关系）并执行重定位作业，来完成动态链接应用程序的构建。之后，控制权才最终可以交给程序。

This is not the last task for the dynamic linker in most cases, though. ELF allows the relocations associated with a symbol to be delayed until the symbol is needed. This lazy relocation scheme is optional, and optimizations discussed below for relocations performed at startup immediately affect the lazy relocations as well. So we ignore in the following everything after the startup is finished.

然而，在大多数情况下，这不是动态链接器的最后一个任务。ELF允许将与符号相关的重定位延迟到需要符号时才进行。这种延迟重定位方案是可选的，并且接下来要讨论的，在启动时执行的重定位优化也会立即影响到延迟重定位。因此，我们在后续的讨论中将不再涉及启动完成后的任何内容。

## 1.4 Startup: In The Kernel

Starting execution of a program begins in the kernel, normally in the execve system call. The currently executed code is replaced with a new program. This means the address space content is replaced by the content of the file containing the program. This does not happen by simply mapping (using mmap) the content of the file. ELF files are structured and there are normally at least three different kinds of regions in the file:

程序的执行起始于内核，通常是在 `execve` 系统调用中。当前执行的代码会被新程序替换。这意味着地址空间的内容被包含程序的文件内容所替换。这并非使用 `mmap` 简单的映射文件内容到内存中就能实现。

ELF文件是有结构的，并且通常至少有三种不同类型的区域：

- Code which is executed; this region is normally not writable;

    被执行的代码；这个区域通常是不可写的；例如 `.text`

- Data which is modified; this region is normally not executable;

    被修改的数据；这个区域通常是不可执行的；例如 `.data` 和 `.bss`

- Data which is not used at run-time; since not needed it should not be loaded at startup.

    在运行时不被使用的数据；由于不需要，它不应该在启动时被加载；例如 `.debug`

Modern operating systems and processors can protect memory regions to allow and disallow reading, writing, and executing separately for each page of memory. It is preferable to mark as many pages as possible not writable since this means that the pages can be shared between processes which use the same application or DSO the page is from. Write protection also helps to detect and prevent unintentional or malignant modifications of data or even code.

现代操作系统和处理器可以保护内存区域，允许或禁止对每个内存页的读取、写入和执行。最好将尽可能多的页面标记为不可写，因为这意味着使用相同应用程序或 DSO 的进程可以共享这些页面。写保护也有助于检测和防止数据甚至代码的无意或恶意修改。

For the kernel to find the different regions, or segments in ELF-speak, and their access permissions, the ELF file format defines a table which contains just this information, among other things. The ELF Program Header table, as it is called, must be present in every executable and DSO. It is represented by the C types Elf32_Phdr and Elf64_Phdr which are defined as can be seen in figure 1.

为了让内核找到不同的区域（或在ELF中称为segments）及其访问权限，ELF文件格式定义了一个包含这些信息（以及其他信息）的表。这个表被称为ELF程序头部表(ELF Program Header table)，它必须存在于每个可执行文件和DSO中。它由C类型 `Elf32_Phdr` 和 `Elf64_Phdr` 表示，这些类型的定义如图1所示。

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
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 1: ELF Program Header C Data Structure</div>
</center>

To locate the program header data structure another data structure is needed, the ELF Header. The ELF header is the only data structure which has a fixed place in the file, starting at offset zero. Its C data structure can be seen in figure 2. The e_phoff field specifies where, counting from the beginning of the file, the program header table starts. The e_phnum field contains the number of entries in the program header table and the e_phentsize field contains the size of each entry. This last value is useful only as a run-time consistency check for the binary.

为了定位（初始化）程序头部条目，我们需要另一个数据结构，即ELF头部（ELF Header）。ELF头部是文件中唯一固定位置的数据结构，即从文件首部偏移量 0 处开始。其 C 数据结构如图 2 所示。`e_phoff`字段指定了程序头部表从文件开始的位置，`e_phnum`字段包含程序头部表中的条目数，`e_phentsize`字段包含每个条目的大小。这最后一个值只在运行时用于二进制文件的一致性检查。

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
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 2: ELF Header C Data Structure</div>
</center>

The different segments are represented by the program header entries with the PT LOAD value in the p_type field. The p_offset and p_filesz fields specify where in the file the segment starts and how long it is. The p_vaddr and p_memsz fields specify where the segment is located in the process’ virtual address space and how large the memory region is. The value of the p_vaddr field itself is not necessarily required to be the final load address. DSOs can be loaded at arbitrary addresses in the virtual address space. But the relative position of the segments is important. For ***prelinked*** DSOs the actual value of the p_vaddr field is meaningful: it specifies the address for which the DSO was prelinked. But even this does not mean the dynamic linker cannot ignore this information if necessary.

程序中不同的段通过程序头部的条目来表示，其中 `p_type` 字段的值为 `PT_LOAD` 表示需要加载的段。`p_offset` 和 `p_filesz` 字段指定了段在文件中的起始位置及其长度。`p_vaddr` 和 `p_memsz` 字段则指定了该段在进程虚拟地址空间中的位置以及内存区域的大小。因为DSO可以在虚拟地址空间的任意地址被加载，所以我们不应视 `p_vaddr` 字段的值为最终加载的地址，但它能很好的指示段与段之间的相对位置，这同样是一个很重要的信息。特别地，对于预链接的DSO（动态共享对象），`p_vaddr` 字段的值是有具体意义的，因为它指定了DSO预链接时所基于的地址。这意味着预链接的DSO是为了在这个特定地址上运行而优化的。然而，即使DSO被预链接到一个特定地址，动态链接器在必要的情况下也可以选择忽略这个预链接地址，将DSO加载到内存的其他位置。

The size in the file can be smaller than the address space it takes up in memory. The first `p_filesz` bytes of the memory region are initialized from the data of the segment in the file, the difference is initialized with zero. This can be used to handle BSS sections, sections for uninitialized variables which are according to the C standard initialized with zero. Handling uninitialized variables this way has the advantage that the file size can be reduced since no initialization value has to be stored, no data has to be copied from disc to memory, and the memory provided by the OS via the mmap interface is already initialized with zero.

在ELF文件中定义的段（如代码段或数据段）在文件中占用的实际字节大小可能比它们在程序运行时占用的内存大小要小。对于这些段的内存区域，它们开始的 `p_filesz` 字节将会从文件中读取数据来初始化。如果内存区域的大小（`p_memsz`）大于文件中的大小（`p_filesz`），那么剩下的部分将会用零来填充。这种方式特别适用于处理BSS段（Block Started by Symbol），即那些在程序启动时需要被初始化为零的未初始化变量。通过这种方式，文件的大小可以被减少，因为不需要在文件中存储这些变量的初始零值；当程序加载到内存时，操作系统会保证这部分内存被零填充。这不仅减少了文件大小，还省去了将初始零值从磁盘复制到内存的过程，因为操作系统通过`mmap`接口提供的内存页默认就是零初始化的。

The p_flags finally tells the kernel what permissions to use for the memory pages. This field is a bitmap with the bits given in the following table being defined. The flags are directly mapped to the flags mmap understands.

`p_flags` 字段最终告诉内核使用何种权限来处理内存页。这个字段是一个位图，其中的位按下表给出的定义。这些标志直接映射到 `mmap` 能理解的标志上。

| p_flags | Value | mmap flag  |    Description     |
| :-----: | :---: | :--------: | :----------------: |
|  PF_X   |   1   | PROT_EXEC  | Execute Permission |
|  PF_W   |   2   | PROT_WRITE |  Write Permission  |
|  PF_R   |   4   | PROT_READ  |  Read Permission   |

After mapping all the PT LOAD segments using the appropriate permissions and the specified address, or after freely allocating an address for dynamic objects which have no fixed load address, the next phase can start. The virtual address space of the dynamically linked executable itself is set up. But the binary is not complete. The kernel has to get the dynamic linker to do the rest and for this the dynamic linker has to be loaded in the same way as the executable itself (i.e., look for the loadable segments in the program header). The difference is that the dynamic linker itself must be complete and should be freely relocatable.

在使用适当的权限和指定的地址映射完所有`PT_LOAD`段之后，或者为没有固定加载地址的动态对象自由分配地址之后，下一个阶段才可以开始。使用动态链接的可执行文件自身的虚拟地址空间就此建立，但是其二进制文件还不完整（此时依赖于外部共享库还未准备好）。内核必须让动态链接器来完成剩下的工作，为此，动态链接器必须像可执行文件本身一样被加载（即，检查程序头部中的可加载段）。不同之处在于动态链接器本身必须是完整的，并且应该能够自由重定位。

Which binary implements the dynamic linker is not hardcoded in the kernel. Instead, the program header of the application contains an entry with the tag PT_INTERP. The p_offset field of this entry contains the offset of a NUL-terminated string which specifies the file name of this file. The only requirement on the named file is that its load address does not conflict with the load address of any possible executable it might be used with. In general, this means that the dynamic linker has no fixed load address and can be loaded anywhere; this is just what dynamic binaries allow.

动态链接器（如 `ld-linux.so`）的具体实现并未在内核中硬编码。而是通过应用程序的程序头中一个标记为 `PT_INTERP` 的特殊条目来指定，该条目的 `p_offset` 字段包含一个指向一个以 NUL 结尾的字符串的偏移，该字符串指定了动态链接器文件的路径。对命名文件的唯一要求是它的加载地址不能与可能使用它的任何可执行文件的加载地址发生冲突。通常，这意味着动态链接器没有固定的加载地址，可以被加载到任何位置；这正是动态二进制文件所允许的。

Once the dynamic linker has also been mapped into the memory of the to-be-started process we can start the dynamic linker. Note it is not the entry point of the application to which control is transferred to. Only the dynamic linker is ready to run. Instead of calling the dynamic linker right away, one more step is performed. The dynamic linker somehow has to be told where the application can be found and where control has to be transferred to once the application is complete. For this a structured way exists. The kernel puts an array of tag-value pairs on the stack of the new process. This ***auxiliary vector*** contains beside the two aforementioned values several more values which allow the dynamic linker to avoid several system calls. The elf.h header file defines a number of constants with a AT prefix. These are the tags for the entries in the auxiliary vector.

当动态链接器同样被映射到待启动进程的内存中后，我们便可以启动动态链接器。需要注意的是，此时并非将控制权直接转交给应用程序的入口点，而是首先启动动态链接器。在直接激活动态链接器之前，我们还需要完成一个额外的步骤。这一步骤涉及到告知动态链接器应用程序所在的位置以及在应用程序完全加载后应该将控制权转移至何处。为了实现这一点，内核使用了一种结构化的方法：在新进程的栈上放置一个 tag-value pair 数组，称为***辅助向量***。这个辅助向量不仅包括之前提到的两个值，还包含其他几个值，这些额外的信息可以帮助动态链接器省去多次系统调用。`elf.h` 头文件定义了许多以 `AT` 为前缀的常量，这些都是辅助向量中的标签。

After setting up the auxiliary vector, the kernel is finally ready to transfer control to the dynamic linker in user mode. The entry point is defined in the e_entry field of the ELF header of the dynamic linker.

在设置好辅助向量后，内核最终准备好将控制权以用户模式转移给动态链接器。动态链接器的入口点是在其ELF头部的`e_entry`字段中定义的。

## 1.5 Startup in the Dynamic Linker

The second phase of the program startup happens in the dynamic linker. Its tasks include:

在动态链接器中，程序启动的第二阶段开始进行。这一阶段的任务包括：

- Determine and load dependencies;

    确定并加载依赖项；找出应用程序需要的所有动态共享对象（DSO）并将它们加载到内存中。

- Relocate the application and all dependencies;

    重定位应用程序及所有依赖项；调整应用程序和其依赖的代码和数据，使它们能够正确地引用彼此的地址。

- Initialize the application and dependencies in the correct order.

    按正确的顺序初始化应用程序及依赖项；在程序开始执行前，以正确的顺序初始化所有加载的组件，确保所有初始化函数都被调用。

In the following we will discuss in more detail only the relocation handling. For the other two points the way for better performance is clear: have fewer dependencies. Each participating object is initialized exactly once but some topological sorting has to happen. The identify and load process also scales with the number dependencies; in most (all?) implementations this does not scale linearly.

在接下来的内容中，我们将更详细地讨论重定位处理。对于其他两点，提高性能的方法很明确：减少依赖项的数量。每个参与的对象仅被初始化一次，但这一过程需要进行拓扑排序来确定初始化的顺序。识别和加载依赖的过程也随着依赖数量的增加而变得更加复杂；在大多数（或全部）实现中，这种增加并不呈线性，而是随着依赖数量的增加显著增长。

The relocation process is normally the most expensive part of the dynamic linker’s work. It is a process which is asymptotically at least $O(R + n×r)$ where R is the number of relative relocations, r is the number of named relocations, and n is the number of participating DSOs (plus the main executable). Deficiencies in the ELF hash table function and various ELF extensions modifying the symbol lookup functionality may well increase the factor to $O(R + r×n×\log_2 s)$ where s is the number of symbols. This should make clear that for improved performance it is significant to reduce the number if relocations and symbols as much as possible. After explaining the relocation process we will do some estimates for actual numbers.

重定位过程通常是动态链接器工作中最为昂贵的部分。从理论上讲，这一过程的时间复杂度至少为 $O(R + n×r)$，其中 $R$ 是相对重定位的数量，$r$ 是命名重定位的数量，$n$ 是参与的DSO数量（加上主执行文件）。ELF哈希表函数的不足以及对符号查找功能的各种ELF扩展可能会将系数提高到 $O(R + r×n×\log_2 s)$，其中 $s$ 是符号的数量。由此可以看出，为了提高性能，尽可能减少重定位次数和符号数是非常重要的。在解释了重定位过程后，我们将会对的实际数值进行评估。

### 1.5.1 The Relocation Process

Relocation in this context means adjusting the application and the DSOs, which are loaded as the dependencies, to their own and all other load addresses. There are two kinds of dependencies:

在这里讨论的重定位，指的是将应用程序（特指那些在编译和链接过程结束后形成的、可以直接被操作系统加载并执行的程序，即可执行程序）及其作为依赖加载的DSO调整到它们自身和所有其他加载地址上的过程。在这里，存在两种类型的依赖关系：

- Dependencies to locations which are known to be in the own object. These are not associated with a specific symbol since the linker knows the relative position of the location in the object. 

    依赖于已知位于本对象文件（指的是程序中的一个组件或编译单元，即编译后的中间文件 .o 或 .obj）内的位置；这种内部位置的依赖不需要链接到一个外部的符号（即不需要程序外部的信息），因为链接器已经知道这些位置在可执行文件中的相对位置。

     Note that applications do not have relative relocations since the load address of the code is known at link-time and therefore the static linker is able to perform the relocation.

    由于采用静态链接的应用程序在链接时已确定代码的加载地址，因此不需要在运行时进行相对重定位，链接器可以直接完成所有必要的重定位工作。

- Dependencies based on symbols. The reference of the definition is generally, but not necessarily, in a different object than the definition.

    基于符号的依赖，当一个程序依赖某个符号时，这个符号通常位于不同的编译单元或代码对象中，尽管也有例外情况；这涉及到跨DSO的引用，例如一个DSO中的函数调用另一个DSO中的函数。这种重定位比较复杂，因为它要处理不同对象之间的关系，需要在运行时确定符号的最终地址。

The implementation of relative relocations is easy. The linker can compute the offset of the target destination in the object file at link-time. To this value the dynamic linker only has to add the load address of the object and store the result in the place indicated by the relocation. At runtime the dynamic linker has to spend only a very small and constant amount of time which does not increase if more DSOs are used.

相对重定位的实现比较简单。链接器在链接时可以计算出目标位置在对象文件中的偏移量。动态链接器只需将此偏移量加上对象的加载地址，并将计算结果存储在重定位指示的内存位置（也就是说我们具体需要被更新的位置）。在运行时，动态链接器仅需花费很少且恒定的时间来完成这一过程，这个时间不会因为使用了更多的动态共享对象（DSO）而增加。 ^a6c493

The relocation based on a symbol is much more complicated. The ELF symbol resolution process was designed very powerful so that it can handle many different problems. All this powerful functionality adds to the complexity and run-time costs, though. Readers of the following description might question the decisions which led to this process. We cannot argue about this here; readers are referred to discussions of ELF. Fact is that symbol relocation is a costly process and the more DSOs participate or the more symbols are defined in the DSOs, the longer the symbol lookup takes.

基于符号的重定位则复杂得多。ELF符号解析过程设计得非常强大，以便它可以处理许多不同的问题。然而，这种强大的功能增加了复杂性和运行时的成本。读者可能会对导致此过程的决策提出疑问。在这里我们无法就此进行讨论；读者可以参考有关ELF的讨论。事实是，符号重定位是一个成本高昂的过程，参与的DSO越多或DSO中定义的符号越多，符号查找所需的时间就越长。

The result of any relocation will be stored somewhere in the object with the reference. Ideally and generally the target location is in the data segment. If code is incorrectly generated by the user, compiler, or linker relocations might modify text or read-only segments. The dynamic linker will handle this correctly if the object is marked, as required by the ELF specification, with the DF_TEXTREL set in the DT_FLAGS entry of the dynamic section (or the existence of the DT_TEXTREL flag in old binaries). But the result is that the modified page cannot be shared with other processes using the same object. The modification process itself is also quite slow since the kernel has to reorganize the memory handling data structures quite a bit.

任何重定位的结果通常会被存储在引用该位置的对象中的某个地方。理想情况下，目标位置应该在数据段（GOT/PLT）。如果用户、编译器或链接器生成的代码有误，重定位可能会修改文本段或只读段。按照ELF规范的要求，如果对象在动态部分的DT_FLAGS条目中设置了DF_TEXTREL标志（或者在旧的二进制文件中存在DT_TEXTREL标志），动态链接器可以正确处理这些修改。但这样做的结果是，被修改的页面无法与使用同一对象的其他进程共享。此外，由于内核需要重新组织内存处理数据结构，这一修改过程也会比较慢。。

### 1.5.2 Symbol Relocations

The dynamic linker has to perform a relocation for all symbols which are used at run-time and which are not known at link-time to be defined in the same object as the reference. Due to the way code is generated on some architectures, it is possible to delay the processing of some relocations until the references in question are actually used. This is on many architectures the case for calls to functions. All other kinds of relocations always have to be processed before the object can be used. We will ignore the ***lazy relocation processing*** since this is just a method to delay the work. It eventually has to be done and so we will include it in our cost analysis. To actually perform all the relocations before using the object is used by setting the environment variable LD_BIND_NOW to a non-empty value. Lazy relocation can be disabled for an individual object by adding the -z now option to the linker command line. The linker will set the DF_BIND_NOW flag in the DT_FLAGS entry of the dynamic section to mark the DSO. This setting cannot be undone without relinking the DSOs or editing the binary, though, so this option should only be used if it is really wanted.

动态链接器需要对所有在运行时使用但在链接时无法确定是否在同一对象中定义的符号执行重定位。由于某些架构的代码生成方式，可以延迟处理这些重定位，直到实际需要使用这些引用时。这在许多架构中适用于函数调用，而其他所有类型的重定位必须在对象使用前完成。我们将忽略延迟重定位处理，因为这只是延迟工作的一个方法，最终还是必须完成。因此，我们会在成本分析中包括它。要在使用对象之前执行所有重定位，可以将环境变量LD_BIND_NOW设置为非空值。通过在链接器命令行中添加-z now选项，可以为单个对象禁用延迟重定位。链接器会在动态部分的DT_FLAGS条目中设置DF_BIND_NOW标志以标记DSO。不过，除非重新链接DSO或编辑二进制文件，否则无法取消这一设置，因此这个选项应该只在真正需要时使用。

The actual lookup process is repeated from start for each symbol relocation in each loaded object. Note that there can be many references to the same symbol in different objects. The result of the lookup can be different for each of the objects so there can be no short cuts except for caching results for a symbol in each object in case more than one relocation references the same symbol. The ***lookup scope*** mentioned in the steps below is an ordered list of a subset of the loaded objects which can be different for each object itself. The way the scope is computed is quite complex and not really relevant here so we refer the interested reader to the ELF specification and section 1.5.4. Important is that the length of the scope is normally directly dependent on the number of loaded objects. This is another factor where reducing the number of loaded objects is increasing performance.

每个加载对象中的每个符号重定位的实际查找过程都会从头开始。不同对象中可能有多个对同一符号的引用，每个对象的查找结果可能不同，因此无法采用任何捷径，除非在每个对象中缓存符号的结果，以应对多个重定位引用同一个符号的情况。下文中提到的**查找范围**是根据每个对象的依赖关系和加载顺序来确定的一个对象有序子集，每个对象的查找范围可能都不同。计算范围的方式相当复杂，具体细节可以参考 ELF 规范和第 [[#1.5.4 Lookup Scope|1.5.4节]]。重要的是，查找范围的长度通常直接依赖于加载的对象数量。因此，减少加载对象的数量有助于提高性能。

There are today two different methods for the lookup process for a symbol. The traditional ELF method proceeds in the following steps:

目前，符号查找过程有两种不同的方法。传统的ELF方法按以下步骤进行：

1. Determine the hash value for the symbol name.

    计算符号名称的哈希值。

2. In the first/next object in the lookup scope:

    在查找范围中的第一个或下一个对象中进行以下操作：

    * 2.a Determine the hash bucket for the symbol using the hash value and the hash table size in the object.

        使用哈希值和对象中的哈希表大小确定符号所属的哈希桶。

      * 2.b Get the name offset of the symbol and using it as the NUL-terminated name.

        如果哈希值匹配，获取符号的名称偏移，并将其作为以NUL结尾的名称使用。

    * 2.c Compare the symbol name with the relocation name.

        将符号名称与需要重定位的名称进行比较。

    * 2.d If the names match, compare the version names as well. This only has to happen if both, the reference and the definition, are versioned. It requires a string comparison, too. If the version name matches or no such comparison is performed, we found the definition we are looking for.

        如果名称匹配，还需要比较版本名称。这只有在引用和定义都有版本时才需要进行，并且需要进行字符串比较。如果版本名称匹配或不需要进行版本比较，则找到了所需的定义。

    * 2.e If the definition does not match, retry with the next element in the chain for the hash bucket.
    
        如果定义不匹配，尝试哈希桶链中的下一个元素。
    
    * 2.f If the chain does not contain any further element there is no definition in the current object and we proceed with the next object in the lookup scope.

        如果链中没有更多元素，表示当前对象中没有定义，继续处理查找范围中的下一个对象。

3. If there is no further object in the lookup scope the lookup failed.

      如果查找范围中没有更多对象，则查找失败。  

```cpp
auto LookupSymbol(symbolName, lookupScope) {
    // 1. Determine the hash value for the symbol name.
    auto hashValue = computeHash(symbolName);

    // 2. In the first/next object in the lookup scope:
    for (auto& object : lookupScope) {
        // 2.a Determine the hash bucket for the symbol using the hash value and the hash table size in the object.
        auto hashBucket = determineHashBucket(hashValue, object.hashTableSize);

        // 2.b 遍历哈希桶链中的每个元素
        while (hashBucket != nullptr) {
            // Get the name offset of the symbol and using it as the NUL-terminated name.
            auto nameOffset = getNameOffset(object, hashBucket);
            auto symbolNameInObject = getSymbolName(object, nameOffset);

            // Compare the symbol name with the relocation name.
            if (symbolNameInObject == symbolName) {
                // If the names match, compare the version names as well. This only has to 
                // happen if both, the reference and the definition, are versioned. It requires 
                // a string comparison, too. If the version name matches or no such comparison 
                // is performed, we found the definition we are looking for.
                if (hasVersion(symbolName) && hasVersion(symbolNameInObject)) {
                    if (compareVersions(symbolName, symbolNameInObject)) {
                        return symbolNameInObject;
                    }
                } else {
                    return symbolNameInObject;
                }
            }
            
            // Retry with the next element in the chain for the hash bucket.
            hashBucket = getNextElement(object, hashBucket);
        }
    }

    // If there is no further object in the lookup scope the lookup failed.
    return nullptr;
}

```

Note that there is no problem if the scope contains more than one definition of the same symbol. The symbol lookup algorithm simply picks up the first definition it finds. Note that a definition in a DSO being ***weak*** has no effects. Weak definitions only play a role in static linking. Having multiple definitions has some perhaps surprising consequences. Assume DSO ‘A’ defines and references an interface and DSO ‘B’ defines the same interface. If now ‘B’ precedes ‘A’ in the scope, the reference in ‘A’ will be satisfied by the definition in ‘B’. It is said that the definition in ‘B’ interposes the definition in ‘A’. This concept is very powerful since it allows more specialized implementation of an interface to be used without replacing the general code. One example for this mechanism is the use of the LD_PRELOAD functionality of the dynamic linker where additional DSOs which were not present at link-time are introduced at run-time. But interposition can also lead to severe problems in ill-designed code. More in this in section 1.5.4.

需要注意的是，即使查找范围内存在多个相同符号的定义，这也不会造成问题。符号查找算法只会选取第一个找到的定义。此外，DSO 中的弱符号定义不会影响查找结果，因为弱符号定义仅在静态链接时有作用。拥有多个定义可能会带来一些意想不到的后果。比如，假设 DSO‘A’ 定义并引用了一个接口，而 DSO‘B’ 也定义了相同的接口。如果在查找范围内 ‘B’ 的位置在 ‘A’ 之前，那么‘A’对该接口的引用将会使用 ‘B’ 的定义。这种情况称为 ‘B’ 的定义替代了 ‘A’ 的定义。这个特性非常强大，因为它允许在不改变通用代码的前提下使用更为专业的接口实现。一个实际应用例子是动态链接器的 LD_PRELOAD 功能，它允许在运行时引入链接时不存在的额外DSO。但如果设计不当，这种替代机制也可能导致严重问题。更多信息请参见 [[#1.5.4 Lookup Scope |1.5.4节]]。

Looking at the algorithm it can be seen that the performance of each lookup depends, among other factors, on the length of the hash chains and the number of objects in the lookup scope. These are the two loops described above. The lengths of the hash chains depend on the number of symbols and the choice of the hash table size. Since the hash function used in the initial step of the algorithm must never change these are the only two remaining variables. Many linkers do not put special emphasis on selecting an appropriate table size. The GNU linker tries to optimize the hash table size for minimal lengths of the chains if it gets passed the -O option (note: the linker, not the compiler, needs to get this option).

通过分析算法，我们可以看出查找的性能除了其他因素外主要取决于哈希链的长度（`while (hashBucket != nullptr)`）和查找范围内的对象数量（`for (auto& object : lookupScope)`），这也正是之前描述的两个循环过程。哈希链的长度取决于符号的数量和哈希表大小（buckets 的数量，会直接影响哈希冲突的频率，例如：较小的哈希表大小和较多的符号数量则引起冲突的概率会增加，那么也会导致哈希冲突链变长）的选择。由于算法中的哈希函数在初始步骤中不能更改，这两个选择则成为唯一的因素。许多链接器在选择哈希表大小时没有特别关注这一点。然而，如果 GNU 链接器接收到 `-O` 选项（注意：是链接器而非编译器需要这个选项），它会尝试优化哈希表的大小以最小化哈希链的长度。

> [!tip]
> A note on the current implementation of the hash table optimization. The GNU binutils linker has a simple minded heuristic which often favors small table sizes over short chain length. For large projects this might very well increase the startup costs. The overall memory consumption will be sometimes significantly reduced which might compensate sooner or later but it is still advised to check the effectiveness of the optimization. A new linker implementation is going to be developed and it contains a better algorithm.
> 
> 关于当前哈希表优化实现的一点说明。GNU binutils 链接器使用了一种简单的启发式方法，通常更倾向于选择较小的哈希表大小，而不是更短的哈希链长度。对于大型项目，这种选择可能会显著增加启动时间。不过，这种方法通常会显著减少整体内存消耗，随着时间推移，这一好处可能会弥补启动成本的增加。但仍然建议对优化的有效性进行检查。目前正在开发一个新的链接器实现，其中包含一个更好的优化算法。

To measure the effectiveness of the hashing two numbers are important:

要衡量哈希算法效率的有效性，有两个指标非常重要：

- The average chain length for a successful lookup.

    成功查找的平均链长度：这个指标衡量的是，在哈希表中成功找到目标时，平均需要遍历的哈希链的长度。较短的平均成功查找链长度通常意味着哈希表的性能较好，因为它表明大多数查找操作都能快速定位到目标元素。

- The average chain length for an unsuccessful lookup.

    失败查找的平均链长度：这个指标衡量的是，当哈希表中没有找到目标元素时，平均需要遍历的哈希链的长度。这个指标同样重要，因为在实际应用中，查找操作不总是成功的，一个高效的哈希表应该在查找失败时也尽可能减少查找路径的长度。

It might be surprising to talk about unsuccessful lookups here but in fact they are the rule. Note that “unsuccessful” means only unsuccessful in the current objects. Only for objects which implement almost everything they get looked in for is the successful lookup number more important. In this category there are basically only two objects on a Linux system: the C library and the dynamic linker itself.

在这里讨论不成功的符号查找可能会令人感到意外，但事实上，这种情况是很常见的。需要注意的是，“不成功”仅指在当前对象中未能找到所需符号。只有那些几乎实现了所有被查找内容的对象，成功查找的次数才具有更大的意义。在Linux系统中，这类对象主要有两个：C标准库和动态链接器本身。

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
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 3: Example Output for eu-readelf -I libc.so</div>
</center>

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
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 4: Example Output for eu-readelf -I libnss files.so</div>
</center>

Some versions of the readelf program compute the value directly and the output is similar to figures 3 and 4. The data in these examples shows us a number of things. Based on the number of symbols (2027 versus 106) the chosen table size is radically different. For the smaller table the linker can afford to "waste" $53.9\%$ of the hash table entries which contain no data. That’s only 412 bytes on a gABI-compliant system. If the same amount of overhead would be allowed for the libc.so binary the table would be 4 kilobytes or more larger. That is a big difference. The linker has a fixed cost function integrated which takes the table size into account.

某些版本的 readelf 程序直接计算哈希表的值，输出结果类似于图3和图4。这些例子中的数据展示了多个信息。根据符号数量（2027 与 106），选择的哈希表大小有显著差异。对于较小的哈希表，链接器可以 "浪费" 53.9% 的哈希表条目，这些条目没有存储任何数据。在符合 gABI 规范的系统中，这仅为 412字节。如果 libc.so 二进制文件（这是一个符号数量很多的 DSO）也允许相同的开销量（浪费了 53.9% 哈希条目），那么哈希表的大小将增加 4KB 或更多。这是一个显著的差异。链接器中集成了一个固定成本函数，该函数将哈希表的大小考虑在内，优化和选择合适的哈希表大小，以在内存使用和查找效率之间取得平衡。

The increased relative table size means we have significantly shorter hash chains. This is especially true for the average chain length for an unsuccessful lookup. The average for the small table is only 28% of that of the large table.

相对的，选择较大的哈希表则哈希链的长度会显著缩短。尤其是在不成功查找的情况下，较大的哈希表的平均链长仅为较小哈希表的28%。

What these numbers should show is the effect of reducing the number of symbols in the dynamic symbol table. With significantly fewer symbols the linker has a much better chance to counter the effects of the suboptimal hashing function.

这些指标应该展示的是减少动态符号表中符号数量的效果。显著减少符号数量，链接器有更好的机会抵消次优哈希函数的影响。

Another factor in the cost of the lookup algorithm is connected with the strings themselves. Simple string comparison is used on the symbol names which are stored in a string table associated with the symbol table data structures. Strings are stored in the C-format; they are terminated by a NUL byte and no initial length field is used. This means string comparisons has to proceed until a non-matching character is found or until the end of the string. This approach is susceptible to long strings with common prefixes. Unfortunately, this is not uncommon.

查找算法成本的另一个因素与字符串本身有关。符号名称存储在与符号表关联的字符串表中，并使用简单的字符串比较。字符串以C语言格式存储，即以NUL字节结尾，且没有长度相关的字段。这意味着在进行字符串比较时，需要一直比较到找到不匹配字符或者到达字符串末尾为止。这种方法容易受到长字符串的影响，特别是那些具有共同前缀的长字符串。遗憾的是，这种情况很常见。

```c
namespace some_namespace {
  class some_longer_class_name {
    int member_variable;
  public:
    some_longer_class_name (int p);
    int the_getter_function (void);
  };
}
```

The name mangling scheme used by the GNU C++ compiler before version 3.0 used a mangling scheme which put the name of a class member first along with a description of the parameter list and following it the other parts of the name such as namespaces and nested class names. The result is a name which distinguishable in the beginning if the member names are different. For the example above the mangled names for the two members functions look like this figure 5.

GNU C++ 编译器在 3.0 版本之前使用了一种名称混淆（mangling）方案，该方案首先将类成员名称和参数列表的描述放在一起，然后是名称的其他部分，如命名空间和嵌套类名称。因此，如果成员名称不同，这些名称在开头就能区分。对于前面提到的例子，这两个成员函数的改编名称如图 5 所示。

```
__Q214some_namespace22some_longer_class_namei
the_getter_function__Q214some_namespace22some_longer_class_name
```
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 5: Mangled names using pre-gcc 3 scheme</div>
</center>

In the new mangling scheme used in today's gcc versions and all other compilers which are compatible with the common C++ ABI, the names start with the namespaces and class names and end with the member names. Figure 6 shows the result for the little example. The mangled names for the two member functions differs only after the 43rd character. This is really bad performance-wise if the two symbols should fall into the same hash bucket.

然而，在今天的 GCC 版本和所有兼容通用 C++ ABI 的编译器中采用的新代码混淆方案中，名称是以命名空间和类名开始，以成员名结束。图 6 展示了一个简单示例的结果。这意味着，如果两个成员函数的混淆名称落入同一个哈希桶中，直到第 43 个字符之后它们才开始有所不同。这在性能上是非常不利的，因为相似的长字符串会导致哈希冲突和查找效率降低。

```
_ZN14some_namespace22some_longer_class_nameC1Ei
_ZN14some_namespace22some_longer_class_name19the_getter_functionEv
```
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 6: Mangled names using the common C++ ABI scheme</div>
</center>

Ada has similar problems. The standard Ada library for gcc has all symbols prefixed with "ada_", then the package and sub-package names, followed by the function name. Figure 7 shows a short excerpt of the list of symbols from the library. The first 23 characters are the same for all the names.

[Ada 语言](https://zh.wikipedia.org/wiki/Ada) 同样面临着代码混淆的问题。GCC 的标准 Ada 库将所有符号都以 `ada_` 为前缀，然后是包和子包的名称，最后是函数名。这导致大量符号在前缀部分完全相同，增加了查找时的比较负担。

```text
ada__calendar__delays___elabb
ada__calendar__delays__timed_delay_nt
ada__calendar__delays__to_duration
```
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 7: Names from the standard Ada library</div>
</center>

The length of the strings in both mangling schemes is worrisome since each string has to be compared completely when the symbol itself is searched for. The names in the example are not extraordinarily long either. Looking through the standard C++ library one can find many names longer than 120 characters and even this is not the longest. Other system libraries feature names longer than 200 characters and complicated, “well designed” C++ projects with many namespaces, templates, and nested classes can feature names with more than 1,000 characters. One plus point for design, but minus 100 points for performance.

这两种混淆方案中，字符串的长度尤其令人关注，因为在查找符号时，必须对每个字符串进行完整的比较。这不仅增加了查找的复杂度，也影响了性能。在标准 C++ 库中，不难找到名称超过 120 个字符的例子，而且这还不是极限。一些复杂的、设计良好的 C++ 项目可能包含超过 1000 个字符的名称，这虽然在设计上是一个加分点，但在性能上却大大减分。

With the understanding of the hashing function and the specifics of string lookup, let's examine a real-world example: OpenOffice.org. The package comprises 144 separate Dynamic Shared Objects (DSOs). During startup, approximately 20,000 relocations are executed. Many relocations result from `dlopen` calls and hence cannot be optimized through prelinking. The number of string comparisons required during symbol resolution serves as a reasonable measure of the startup overhead. We will now calculate an approximation of this value.

了解哈希函数和字符串查找的细节后，让我们来看一个实际例子：[OpenOffice.org](https://www.openoffice.org/)。这个软件包包含 144 个独立的动态共享对象（DSO）。在启动过程中，大约有 20,000 次重定位操作被执行。许多重定位操作是由 `dlopen` 调用引起的，因此无法通过使用预链接（prelink）来优化。在符号解析期间所需进行的字符串比较次数可以作为评估启动开销的一个合理标准。我们现在将对这个数值进行估算。

The average chain length for an unsuccessful lookup across all DSOs of the OpenOffice.org 1.0 release on IA-32 is 1.1931. This implies that for each symbol lookup, the dynamic linker must perform an average of 72 × 1.1931 = 85.9032 string comparisons. For 20,000 symbols, this totals 1,718,064 string comparisons. The average length of an exported symbol defined in the DSOs of OpenOffice.org is 54.13 characters. Assuming that only 20% of the string is searched before a mismatch is found (which is optimistic since every symbol name is compared in full at least once to match itself), this equates to more than 18.5 million characters needing to be loaded from memory and compared. It's no surprise that startup is slow, especially since other costs have been disregarded.

在所有 OpenOffice.org 1.0 版本的 DSO 中，失败查找的平均链长度为 1.1931。这意味着，对于每次符号查找，动态链接器平均需要执行 72 × 1.1931 = 85.9032 次字符串比较。对于 20,000 个符号，总共需要进行 1,718,064 次字符串比较。在 OpenOffice.org 的 DSO 中定义的导出符号的平均长度为 54.13。即使我们假设在找到不匹配之前只搜索了前 20% 的字符串（这是一个乐观的猜测，因为每个符号名称至少被完整比较一次以匹配自身），这也意味着需要从内存中加载并比较超过 18500000 个字符。难怪启动如此缓慢，特别是因为我们忽略了其他成本。

To calculate the number of lookups the dynamic linker performs, one can enlist the help of the dynamic linker itself. If the environment variable `LD_DEBUG` is set to `symbols`, one simply counts the number of lines starting with `symbol=`. It's advisable to redirect the dynamic linker’s output to a file with `LD_DEBUG_OUTPUT`. The number of string comparisons can then be estimated by multiplying the count by the average hash chain length. Since the collected output includes the name of the file being examined, it would even be possible to obtain more accurate results by using the exact hash chain length for the object.

要计算动态链接器执行的查找次数，可以利用动态链接器自身的功能。如果将环境变量 `LD_DEBUG` 设置为`symbols`，只需要计算链接器的输出中以`symbol=`开头的行数。建议使用`LD_DEBUG_OUTPUT`将动态链接器的输出重定向到文件中。然后，可以通过将行数乘以平均哈希链长度来估算字符串比较的次数。由于收集到的输出包含了正被查看的文件的名称，甚至可以通过乘以对象的确切哈希链长度来获得更准确的结果。

Altering any of the factors—'number of exported symbols', 'length of the symbol strings', 'number and length of common prefixes', 'number of DSOs', and 'hash table size optimization'—can dramatically reduce costs. Generally, the percentage of time the dynamic linker spends on relocations during startup is around 50-70% if the binary is already in the file system cache, and about 20-30% if the file has to be loaded from disk. Therefore, it's worthwhile to focus on these issues, and the remainder of the text will introduce methods to address them. Remember to pass `-O1` to the linker to generate the final product.

通过改变导出符号的数量、符号字符串的长度、常见前缀的数量和长度、DSO的数量和哈希表大小优化中的任何一个因素都可以显著降低成本。一般来说，如果二进制文件已经在文件系统缓存中，动态链接器在启动过程中花费在重定位上的时间大约是 50-70%，如果文件需要从磁盘加载，则大约是 20-30%。因此，值得在这些问题上花时间，在文本的剩余部分，我们将介绍如何做到这一点，直至目前需要记住的是：向链接器传递 -O1 参数以生成最终程序。

### 1.5.3 The GNU-style Hash Table

All the optimizations proposed in the previous section still leave symbol lookup as a significant factor. A lot of data has to be examined and loading all this data in the CPU cache is expensive. As mentioned above, the original ELF hash table handling has no more flexibility so any solution would have to replace it. This is what the GNU-style hash table handling does. It can peacefully coexist with the old-style hash table handling by having its own dynamic section entry (`DT_GNU_HASH`). Updated dynamic linkers will use the new hash table instead of the old, therefore provided completely transparent backward compatibility support. The new hash table implementation, like the old, is self-contained in each executable and DSO so it is no problem to have binaries with the new and some with only the old format in the same process.

尽管前一节讨论的优化方法显著减少了启动时符号查找的成本，但符号查找仍然是启动性能中的重要因素。因为需要检查大量数据，并将所有这些数据加载到 CPU 缓存中代价很高。如前所述，原始的 ELF 哈希表处理方式缺乏灵活性，因此需要替代方案来改进性能。这就是 GNU 风格哈希表要实现的一个目标。它通过增加自己的动态段条目（`DT_GNU_HASH`）与旧风格的哈希表处理共存。更新后的动态链接器将使用新哈希表而不是旧的，从而完全透明地支持向后兼容。新的实现与旧的类似，每个可执行文件和 DSO 都各自包含其所需的哈希表数据，因此在同一进程中同时存在使用新格式和仅使用旧格式的二进制文件并不会有问题。

The main cost for the lookup, especially for certain binaries, is the comparison of the symbol names. If the number of comparisons which actually have to be performed can be reduced we can gain big. A second possible optimization is the layout of the data. The old-style hash table with its linked list of symbol table entries is not necessarily good to the CPU cache. CPU caches work particularly well when the used memory locations are consecutive. A linked list can jump wildly around and render CPU cache loading and prefetching less effective.

对于某些二进制文件而言，查找的主要成本是符号名称的比较。如果能减少实际需要进行的比较次数，我们就能获得很大的收益。另一个可能的优化方向是数据的布局。旧式哈希表使用符号表条目的链表，这种结构对 CPU 缓存并不友好。当内存位置是连续时，CPU 缓存的效果最佳。而链表结构会导致内存位置不连续，从而降低CPU缓存加载和预取的效率。

The GNU-style hash tables set out to solve these problem and more. Since compatibility with existing runtime environments could be maintained by providing the old-style hash tables in parallel no restrictions of the changes were needed. The new lookup process is therefore slightly different:

新的 GNU 哈希表旨在解决这些以及其他更多问题。通过并行提供旧式哈希表，新哈希表可以保持与现有运行环境的兼容性，因此不需要对更改进行任何限制。因此，新的查找过程有所不同：

1. Determine the hash value for the symbol name.

    计算符号名称的哈希值。

2. In the first/next object in the lookup scope:

    在查找范围中的第一个或下一个对象中进行以下操作：：

    2.a The hash value is used to determine whether an entry with the given hash value is present at all. This is done with a 2-bit Bloom filter. If the filter indicates there is no such definition the next object in the lookup scope is searched.

    使用哈希值来确定是否存在具有给定哈希值的条目。这是通过2位布隆过滤器完成的。如果过滤器显示没有这样的定义，则继续搜索查找范围内的下一个对象。

   2.b Determine the hash bucket for the symbol using the hash value and the hash table size in the object. The value is a symbol index.

    使用哈希值和对象中的哈希表大小确定符号的哈希桶位置。这个位置对应的是符号表中的一个索引。

   2.c Get the entry from the chain array corresponding to the symbol index. Compare the value with the hash value of the symbol we are trying to locate. Ignore bit 0.

    从与符号索引对应的链数组中获取条目。将该值与我们试图定位的符号的哈希值进行比较（忽略第0位）。

   2.d If the hash value matches, get the name offset of the symbol and using it as the NUL-terminated name.

    如果哈希值匹配，获取符号的名称偏移，并将其作为以NUL结尾的名称使用。

   2.e Compare the symbol name with the relocation name.

    将符号名称与需要重定位的名称进行比较。

   2.f If the names match, compare the version names as well. This only has to happen if both, the reference and the definition, are versioned. It requires a string comparison, too. If the version name matches or no such comparison is performed, we found the definition we are looking for.

    如果名称匹配，还需要比较版本名称。只有在引用和定义都带有版本时才需要进行此操作。这也需要进行字符串比较。如果版本名称匹配或不需要进行此类比较，我们就找到了需要的定义。

   2.g If the definition does not match and the value loaded from the hash bucket does not have bit 0 set, continue with the next entry in the hash bucket array.

    如果定义不匹配，并且从哈希桶加载的值的第 0 位未被设置，继续查找哈希桶数组中的下一个条目。

   2.h If bit 0 is set there are no further entry in the hash chain we proceed with the next object in the lookup scope.

    如果第 0 位被设置，表示哈希链中没有更多条目，我们将继续在查找范围内的下一个对象中进行。

3. If there is no further object in the lookup scope the lookup failed.

    如果查找范围内没有更多对象，查找失败。

```cpp
auto lookup_symbol(symbol_name, lookup_scope) {
    // 1. Determine the hash value for the symbol name.
    auto hash_value = compute_hash(symbol_name);

    // 2. In the first/next object in the lookup scope:
    for (obj : lookup_scope) {
        // 2.a The hash value is used to determine whether an entry with the
        // given hash value is present at all. This is done with a 2-bit Bloom
        // filter. If the filter indicates there is no such definition the next 
        // object in the lookup scope is searched.
        if (!bloom_filter_check(obj.bloom_filter, hash_value)) {
            continue;
        }

        // 2.b Determine the hash bucket for the symbol using the hash value and
        // the hash table size in the object. The value is a symbol index.
        auto bucket_index = determine_hash_bucket(hash_value, obj.nbuckets);
        auto symbol_index = obj.buckets[bucket_index];

        if (symbol_index < 0) {
            continue;
        }

        // 2.c Get the entry from the chain array corresponding to the symbol
        // index. Compare the value with the hash value of the symbol we are 
        // trying to locate. Ignore bit 0.
        while (symbol_index > 0) {
            auto entry_hash = obj.chain[symbol_index];
            if ((entry_hash >> 1) == (hash_value >> 1)) {
                // 2.d If the hash value matches, get the name offset of the 
                // symbol and using it as the NUL-terminated name.
                auto symbol_name_in_obj = get_symbol_name(obj, symbol_index);

                // 2.e Compare the symbol name with the relocation name.
                if (symbol_name_in_obj == symbol_name) {
                    // 2.f If the names match, compare the version names as well.
                    // This only has to happen if both, the reference and the 
                    // definition, are versioned. It requires a string 
                    // comparison, too. If the version name matches or no such 
                    // comparison is performed, we found the definition we are 
                    // looking for.
                    if (has_version(symbol_name) && has_version(symbol_name_in_obj)) {
                        if (!compare_versions(symbol_name, symbol_name_in_obj)) {
                            symbol_index = obj.chain[symbol_index];
                            continue;  // 版本不匹配，继续查找下一个条目
                        }
                    }

                    // 版本匹配或不需要版本比较，找到符号定义
                    return symbol_name_in_obj;
                }
            }

            // 2.g If the definition does not match, retry with the next element
            // in the chain for the hash bucket.
            if ((entry_hash & 1) == 0) {
                symbol_index = obj.chain[symbol_index];
            } else {
                break; // 2.h If bit 0 is set there are no further entries in the
                       // hash chain, we proceed with the next object in the lookup scope.
            }
        }
    }

    // 3. If there is no further object in the lookup scope, the lookup failed.
    return nullptr;
}

```

This new process seems more complicated. Not only is this not really the case, it is also much faster. The number of times we actually have to compare strings is reduced significantly. The Bloom filter alone usually filters out $80\%$ or more (in many cases $90+\%$) of all lookups. I.e., even in the case the hash chains are long no work is done since the Bloom filter helps to determine that there will be no match. This is done with one single memory access.

这个新过程看起来更复杂。但事实并非如此，它实际上也更快。我们实际上需要进行字符串比较的次数显著减少。单独的布隆过滤器通常可以过滤掉 80% 以上（在许多情况下是 90+%）的所有查找。也就是说，即使在哈希链很长的情况下，由于布隆过滤器能帮助确定不会有匹配结果，因此也不会产生任何工作。这只需一次内存访问即可完成。

Second, comparing the hash value with that of the symbol table entry prevents yet more string comparisons. Each hash chain can contain entries with different hash value and this simple word comparison can filter out a lot of duplicates. There are rarely two entries with the same hash value in a hash chain which means that an unsuccessful string comparison is rare. The probability for this is also increased by using a different hash function than the original ELF specification dictates. The new function is much better in spreading the values out over the value range of 32-bit values.

其次，通过比较哈希值与符号表条目的哈希值，可以大大减少字符串比较的次数。每个哈希链中包含的条目具有不同的哈希值，这种简单的比较可以有效地过滤掉重复项。在一个哈希链中，很少会有两个条目具有相同的哈希值，因此不成功的字符串比较是非常罕见的。通过使用不同于原始 ELF 规范的哈希函数，可以进一步降低字符串比较的概率。新的哈希函数在32位值范围内的分布更均匀，减少了哈希碰撞的概率，使哈希链中的条目分布更为合理，从而提高查找效率。

The hash chain array is organized to have all entries for the same hash bucket follow each other. There is no linked list and therefore the cache utilization is much better.

哈希链数组的组织方式是，相同哈希桶的所有条目彼此紧密跟随，没有采用链表结构，因此缓存的利用效率得到了显著提高。

Only if the Bloom filter and the hash function test succeed do we access the symbol table itself. All symbol table entries for a hash chain are consecutive, too, so in case we need to access more than one entry the CPU cache prefetching will help here, too.

只有当布隆过滤器和哈希函数测试都成功时，我们才会访问符号表本身。同一个哈希链上的所有符号表条目也是连续排列的，因此如果需要访问多个条目，CPU缓存的预取功能在这里也将发挥作用。

One last change over the old format is that the hash table only contains a few, necessary records for undefined symbol. The majority of undefined symbols do not have to appear in the hash table. This in some cases significantly reduces the possibility of hash collisions and it certainly increases the Bloom filter success rate and reduces the average hash chain length. The result are significant speed-ups of 50% or more in code which cannot depend on pre-linking (prelinking is always faster).

最后一个改进是，新的哈希表格式仅包含少数必要的未定义符号记录，大多数未定义符号无需出现在哈希表中。这一改动显著减少了哈希碰撞的概率，提高了布隆过滤器的成功率，并缩短了平均哈希链的长度。这样做的结果是，在不能依赖预链接（预链接总是更快）的代码中，速度明显提高 $50\%$，甚至更多。

This does not mean, though, that the optimization techniques described in the previous section are irrelevant. They still should very much be applied. Using the new hash table implementation just means that not optimizing the exported and referenced symbols will not have a big effect on performance as it used to have.

然而，这并不意味着上一节描述的优化技术就变得无关紧要了。这些优化措施仍然非常适用。采用新的哈希表实现意味着，我不就不需要关注导出和引用的符号的优化了，因为现在它们至少不会像以前那样对性能产生大影响。

> The new hash table format was introduced in Fedora Core 6. The entire OS, with a few deliberate exceptions, is created without the compatibility hash table by using --hash-style=gnu. This means the binaries cannot be used on systems without support for the new hash table format in the dynamic linker. Since this is never a goal for any of the OS releases making this decision was a nobrainer. The result is that all binaries are smaller than with the second set of hash tables and in many cases even smaller than binaries using only the old format.
>
> 新的哈希表格式在Fedora Core 6中引入。除了少数特意的例外，整个操作系统都使用`--hash-style=gnu`创建，因此没有旧格式的兼容性哈希表。这意味着这些二进制文件不能在不支持新格式的系统上运行。由于这从来都不是任何操作系统版本的目标，因此做出这个决定是轻而易举的。其结果是，所有的二进制文件都比包含第二组哈希表时更小，而且在许多情况下甚至比仅使用旧格式的二进制文件还要小。

Going back to the OpenOffice.org example, we can make some estimates about the speedup. If the Bloom filter is able to filter out a low 80% of all lookups and the probability of duplicates hash values is a high 15% we only have to actually compare on average 72 × 0.2 × 0.15 × 1.1931 = 2.58 strings. This is an improvement of a factor of 33. Adding to this improved memory handling and respect for the CPU cache we have even higher gains. In real world examples we can reduce the lookup costs so that programs start up 50% faster or more.

回到OpenOffice.org 的例子，我们可以对加速做一些估算。如果布隆过滤器能够过滤掉至少80%的所有查找，而重复哈希值的概率高达 15%，我们只需要平均比较 $72 × 0.2 × 0.15 × 1.1931 = 2.58$ 个字符串。这是 33 倍的改进。加上改进的内存处理和对CPU缓存的尊重，我们获得了更高的收益。在现实世界的例子中，我们可以减少查找成本，使得程序启动速度提升 50% 或更多。

### 1.5.4 Lookup Scope

The lookup scope has so far been described as an ordered list of most loaded object. While this is correct it has also been intentionally vague. It is now time to explain the lookup scope in more detail.

到目前为止，查找范围被描述为已加载对象的有序列表。虽然这种描述是准确的，但它同时也故意保持了一定的不明确性。现在是时候更详细地阐释查找范围了。

The lookup scope consists in fact of up to three parts. The main part is the global lookup scope. It initially consists of the executable itself and all its dependencies. The dependencies are added in breadth-first order. That means first the dependencies of the executable are added in the order of their `DT_NEEDED` entries in the executable’s dynamic section. Then the dependencies of the first dependency are added in the same fashion. DSOs already loaded are skipped; they do not appear more than once on the list. The process continues recursively and it will stop at some point since there are only a limited number of DSOs available. The exact number of DSOs loaded this way can vary widely. Some executables depend on only two DSOs, others on 200.

查找范围实际上由三个部分组成。其中最主要的部分是全局查找范围，它最初包括了可执行文件本身及其所有的依赖项。在这其中，依赖项是按照广度优先顺序添加的。也就是说，首先根据可执行文件的动态段中的 `DT_NEEDED` 条目的顺序添加可执行文件的依赖项。然后，按照相同的方式添加每个依赖项的后续依赖项。在这期间，已经加载的动态共享对象（DSO）会被忽略，它们在列表中不会重复出现。这个过程是递归进行的并在加载完所有依赖项后便停止。由于每个可执行文件的依赖关系不同，这就导致 DSO 的数量可能会有很大差异。一些可执行文件只依赖于少数几个DSO，而另一些则可能依赖于多达 200 个 DSO 。

If an executable has the DF_SYMBOLIC flag set (see section 2.2.7) the object with the reference is added in front of the global lookup scope. Note, only the object with the reference itself is added in front, not its dependencies. The effects and reasons for this will be explained later.

作为第二部分的查找范：如果可执行文件设置了 DF_SYMBOLIC 标志（参见 2.2.7节），则引用符号的对象将被添加到全局查找范围的最前面。注意，只有这个引用符号的对象本身被添加到前面，而不是它的依赖项。这些效果和原因将在后面详细解释。 ^c678ee

A more complicated modification of the lookup scope happens when DSOs are loaded dynamically using dlopen. If a DSO is dynamically loaded it brings in its own set of dependencies which might have to be searched. These objects, starting with the one which was requested in the dlopen call, are appended to the lookup scope if the object with the reference is among those objects which have been loaded by dlopen. That means, those objects are not added to the global lookup scope and they are not searched for normal lookups. This third part of the lookup scope, we will call it local lookup scope, is therefore dependent on the object which has the reference.

当使用 dlopen 动态加载 DSO 时，查找范围会发生更复杂的变化。动态加载的 DSO 会带来属于自己的一组依赖项，这些依赖可能需要被搜索。比如说：通过 dlopen 加载的对象内部包含了其他对象的符号引用，则这个对象和它引入的所有依赖项将被加入到一个专门的查找范围中。这意味着，这些对象不会被添加到全局查找范围，并且不会在正常查找中被搜索。因此，这第三部分的查找范围，我们将其称为局部查找范围，因此依赖于具有引用的对象。

The behavior of dlopen can be changed, though. If the function gets passed the RTLD_GLOBAL flag, the loaded object and all the dependencies are added to the global scope. This is usually a very bad idea. The dynamically added objects can be removed and when this happens the lookups of all other objects is influenced. The entire global lookup scope is searched before the dynamically loaded object and its dependencies so that definitions would be found first in the global lookup scope object before definitions in the local lookup scope. If the dynamic linker does the lookup as part of a relocation this additional dependency is usually taken care of automatically, but this cannot be arranged if the user looks up symbols in the lookup scope with dlsym.

尽管如此，`dlopen` 的行为是可以通过传递 `RTLD_GLOBAL` 标志来改变的，这会将加载的对象及其所有依赖项添加到全局查找范围中。这通常不是一个好的做法。因为动态添加的对象可能会被移除，这会影响到其他对象的符号查找，可能会产生运行时的错误。全局查找范围会在动态加载的对象及其依赖项之前被搜索，所以符号会先在全局查找范围内找到，而不是在本地查找范围内。如果动态链接器在重定位时进行查找，这种额外的依赖关系通常会被自动处理，然而，如果用户尝试通过 `dlsym` 在查找范围内手动查找符号，则无法保证自动处理这种情况，即可能无法找到相应的符号。 ^118818

And usually there is no reason to use RTLD_GLOBAL. For reasons explained later it is always highly advised to create dependencies with all the DSOs necessary to resolve all references. RTLD_GLOBAL is often used to provide implementations which are not available at link time of a DSO. Since this should be avoided the need for this flag should be minimal. Even if the programmer has to jump through some hoops to work around the issues which are solved by RTLD_GLOBAL it is worth it. The pain of debugging and working around problems introduced by adding objects to the global lookup scope is much bigger.

通常没有必要使用 `RTLD_GLOBAL` 标志。出于后面将要解释的原因，我们强烈建议在编写程序时明确创建与所有需要解析符号的 DSO 的依赖关系。`RTLD_GLOBAL` 常被用来在链接时提供某些不可用的实现，但这种做法应尽量避免，因此对该标志的需求应该降到最低。即使工程师们需要花费额外的一些功夫来补充上 `RTLD_GLOBAL` 能够解决的问题，这种做法长期看来更加稳妥，因为相比之下，全局查找范围添加对象可能引起的问题其复杂度和调试显得更加的困难。

The dynamic linker in the GNU C library knows since September 2004 one more extension. This extension helps to deal with situations where multiple definitions of symbols with the same name are not compatible and therefore cannot be interposed and expected to work. This is usally a sign of design failures on the side of the people who wrote the DSOs with the conflicting definitions and also failure on the side of the application writer who depends on these incompatible DSOs. We assume here that an application `app` is linked with a DSO `libone.so` which defines a symbol `duplicate` and that it dynamically loads a DSO `libdynamic.so` which depends on another DSO `libtwo.so` which also defines a symbol `duplicate`. When the application starts it might have a global scope like this:

自 2004 年 9 月以来，GNU C 库中的动态链接器增加了一项扩展，帮助处理同名符号多重定义且不兼容的情况。这通常是因为 DSO 编写者在设计上有缺陷，以及应用程序开发者依赖这些不兼容的 DSO。在这里，我们假设一个名为 `app` 的应用程序链接到一个名为 `libone.so` 的 DSO，它定义了一个名为 `duplicate` 的符号，并且动态加载了一个名为 `libdynamic.so` 的 DSO，该 DSO 依赖于另一个名为 `libtwo.so` 的 DSO，并且其也定义了一个名为 `duplicate` 的符号。当应用程序启动时，它可能具有如下的全局范围： ^ee4c43

```
app → libone.so → libdl.so → libc.so
```

If now libtwo.so is loaded, the additional local scope could be like this:

如果此时加载了 libtwo.so，那么一份额外的本地范围可能如下所示：

```
libdynamic.so → libtwo.so → libc.so
```

This local scope is searched after the global scope, possibly with the exception of `libdynamic.so` which is searched first for lookups in this very same DSO if the `DF_DYNAMIC` flag is used. But what happens if the symbol duplicate is required in `libdynamic.so`? After all we said so far the result is always: the definition in `libone.so` is found since libtwo.so is only in the local scope which is searched after the global scope. If the two definitions are incompatible the program is in trouble.

本地查找范围是在全局查找范围之后进行搜索的，但如果 [[#^c678ee |DF_SYMBOLIC]] 标志被使用，那么 libdynamic.so 会首先在自己的范围内进行查找。然而，如果 libdynamic.so 需要使用符号 duplicate，结果会如何呢？按照之前的描述，答案是：libone.so 中的定义会被找到。这是因为 libtwo.so 只在本地范围内，而本地范围是在全局范围之后进行搜索的。如果这两个定义不兼容，程序就会遇到问题。

This can be changed with a recent enough GNU C library by ORing RTLD DEEPBIND to the flag word passed as the second parameter to dlopen. If this happens, the dynamic linker will search the local scope before the global scope for all objects which have been loaded by the call to dlopen. For our example this means the search order changes for all lookups in the newly loaded DSOs libdynamic.so and libtwo.so, but not for libc.so since this DSO has already been loaded. For the two affected DSOs a reference to duplicate will now find the definition in libtwo.so. In all other DSOs the definition in libone.so would be found.

在较新的 GNU C 库中，可以通过在 dlopen 调用时传递 `RTLD_DEEPBIND` 标志来改变这种情况。这样，动态链接器会优先在本地范围内搜索由 dlopen 加载的所有对象，然后再搜索全局范围。在我们的例子中，这意味着新加载的 libdynamic.so 和 libtwo.so 的查找顺序会改变，但不会影响已经加载的 libc.so。因此，对于 libdynamic.so 和 libtwo.so，符号 duplicate 的引用将首先找到 libtwo.so 中的定义，而在其他 DSO 中，将找到 libone.so 中的定义。

While this might sound like a good solution for handling compatibility problems this feature should only be used if it cannot be avoided. There are several reasons for this:

虽然 `RTLD_DEEPBIND` 标志似乎是解决[[#^ee4c43|兼容性]]问题的好方法，但只有在无法避免的情况下才应使用。原因如下： ^f5069c

- The change in the scope affects all symbols and all the DSOs which are loaded. Some symbols might have to be interposed by deﬁnitions in the global scope which now will not happen.

    范围的改变会影响所有符号和加载的所有 DSO。有些符号可能需要在全局范围内定义，但是现在无法实现。

- Already loaded DSOs are not affected which could cause unconsistent results depending on whether the DSO is already loaded (it might be dynamically loaded, so there is even a race condition).

    已经加载的动态共享库（DSOs）不会受到新的加载行为的影响，这可能会导致程序的行为不一致。这是因为某个 DSO 是否已经加载（尤其是动态加载的情况下）会影响符号查找的结果。由于这些 DSO 可能是动态加载的，因此在并发环境下可能会出现竞争条件（race condition），即多个进程或线程可能会争夺加载同一个 DSO，导致未预期的行为。

- LD_PRELOAD is ineffective for lookups in the dynamically loaded objects since the preloaded objects are part of the global scope, having been added right after the executable. Therefore they are looked at only after the local scope.

    对于动态加载的对象，`LD_PRELOAD` 无效，因为预加载的对象在全局范围内，且在可执行文件之后添加。因此，在查找符号时，这些符号会在局部范围查找之前被查找。

- Applications might expect that local deﬁnitions are always preferred over other deﬁnitions.  This (and the previous point) is already partly already a problem with the use of DF_SYMBOLIC but since this ﬂag should not be used either, the arguments are still valid.

    应用程序通常希望局部范围内的符号定义优先于全局范围内的定义。然而，使用 DF_SYMBOLIC 标志时已经出现了部分问题。尽管不推荐使用 DF_SYMBOLIC 标志，但上述问题和论点依然成立。

- If any of the implicitly loaded DSOs is loaded explicitly afterward, its lookup scope will change.

    如果一个动态共享库（DSO）最初是通过依赖关系隐式加载的（即作为其他库的依赖被加载），而之后又被显式地再次加载（比如通过 `dlopen` 函数），那么这个库的符号查找范围将会改变。具体来说，显式加载会改变该库的查找优先级和查找顺序，从而影响程序在运行时对符号的解析方式。这可能导致程序行为的变化，尤其是在符号冲突或重复定义的情况下。

- Lastly, the ﬂag is not portable.

    最后，该标志是不可移植的

The RTLD_DEEPBIND ﬂag should really only be used as a last resort. Fixing the application to not depend on the ﬂag’s functionality is the much better solution.

RTLD_DEEPBIND ﬂag 只能在万不得已的情况下使用。修复应用程序，使其不依赖于 ﬂag 的功能，才是更好的解决方案。

### 1.5.5 GOT and PLT

The Global Offset Table (GOT) and Procedure Linkage Table (PLT) are the two data structures central to the ELF runtime. We will introduce now the reasons why they are used and what consequences arise from that.

全局偏移表（GOT）和过程链接表（PLT）是 ELF 文件格式运行时的重要数据结构。接下来我们将解释它们的用途以及它们对程序运行的影响。

Relocations are created for source constructs like

我们通过下面的源结构来描述重定位的过程。

```c
extern int foo;

extern int bar (int);

int call_bar (void)  {
  return bar(foo);
}
```

The call to bar requires two relocations: one to load the value of foo and another one to ﬁnd the address of bar. If the code would be generated knowing the addresses of the variable and the function the assembler instructions would directly load from or jump to the address. For IA- 32 the code would look like this:

调用 `bar` 函数时，需要进行两次重定位：一次是加载变量 `foo` 的值，另一次是找到函数 `bar` 的地址。如果在生成代码时已经知道这些变量和函数的地址，汇编指令将直接加载或跳转到这些地址。对于 IA-32 架构，代码如下所示：

```asm
pushl foo
call bar
```

This would encode the addresses of foo and bar as part of the instruction in the text segment. If the address is only known to the dynamic linker the text segment would have to be modiﬁed at runtime. According to what we learned above this must be avoided. 

这会将 `foo` 和 `bar` 的地址编码到文本段中的指令里。然而，如果这些地址只有在运行时由动态链接器（dynamic linker）才能确定，那么在程序执行时就需要修改文本段中的这些指令，以更新这些地址。根据我们之前的讨论，这是必须避免的。

Therefore the code generated for DSOs, i.e., when using -fpic or -fPIC, looks like this:

因此，当为动态共享对象（DSOs）生成代码时，例如使用 `-fpic` 或 `-fPIC`，代码看起来如下：

```asm
movl foo@GOT(%ebx), %eax

pushl (%eax)

call bar@PLT
```

The address of the variable foo is now not part of the instruction. Instead it is loaded from the GOT. The address of the location in the GOT relative to the PIC register value (%ebx) is known at linktime. Therefore the text segment does not have to be changed, only the GOT

通过这样的实现，变量 `foo` 的地址不再直接嵌入在指令中，而是从全局偏移表（GOT）中加载。在链接时，GOT 中的位置地址相对于 PIC 寄存器值（%ebx）是已知的。因此，不需要修改文本段，只需修改 GOT。

The situation for the function call is similar. The function bar is not called directly. Instead control is transferred to a stub for bar in the PLT (indicated by bar@PLT). For IA-32 the PLT itself does not have to be modiﬁed and can be placed in a readonly segment, each entry is 16 bytes in size. Only the GOT is modiﬁed and each entry consists of 4 bytes. The structure of the PLT in an IA-32 DSO looks like this:

函数调用的情况也类似。函数 `bar` 不是直接调用的，而是通过过程链接表（PLT）中的一个存根进行控制转移（即 `bar@PLT`）。对于 IA-32 架构来说，PLT 本身无需修改，可以放在只读段中，每个条目大小为 16 字节。只需修改 GOT，每个条目占 4 字节。IA-32 动态共享对象（DSO）中的 PLT 结构如下：

```asm
.PLT0: pushl 4(%ebx)
       jmp *8(%ebx)
       nop; nop
       nop; nop
       
.PLT1: jmp *name1@GOT(%ebx)
       pushl $offset1
       jmp .PLT0@PC
       
.PLT2: jmp *name2@GOT(%ebx)
       pushl $offset2
       jmp .PLT0@PC
```

This shows three entries, there are as many as needed, all having the same size. The ﬁrst entry, labeled with .PLT0, is special. It is used internally as we will see. All the following entries belong to exactly one function symbol. The ﬁrst instruction is an indirect jump where the address is taken from a slot in the GOT. Each PLT entry has one GOT slot. At startup time the dynamic linker ﬁlls the GOT slot with the address pointing to the second instruction of the appropriate PLT entry.  I.e., when the PLT entry is used for the ﬁrst time the jump ends at the following pushl instruction.  The value pushed on the stack is also speciﬁc to the PLT slot and it is the offset of the relocation entry for the function which should be called. Then control is transferred to the special ﬁrst PLT entry which pushes some more values on the stack and ﬁnally jumps into the dynamic linker.  The dynamic linker has do make sure that the third GOT slot (offset 8) contains the address of the entry point in the dynamic linker.  Once the dynamic linker has determined the address of the function it stores the result in the GOT entry which was used in the jmp instruction at the beginning of the PLT entry before jumping to the found function. This has the effect that all future uses of the PLT entry will not go through the dynamic linker, but will instead directly transfer to the function. The overhead for all but the ﬁrst call is therefore “only” one indirect jump.

这个例子展示了三个 PLT 条目，数量可以根据需要增加，每个条目大小相同。第一个条目 `.PLT0` 是特殊的，用于内部处理，后文会详细解释。所有后续条目都对应一个具体的函数符号。第一条指令是一个间接跳转，跳转地址来自于 GOT 的一个槽。每个 PLT 条目都有一个对应的 GOT 槽。在启动时，动态链接器会将 GOT 槽填充为指向相应 PLT 条目第二条指令的地址。也就是说，当 PLT 条目第一次被调用时，跳转会进入后面的 `pushl` 指令。压入堆栈的值与 PLT 槽相关，是要调用的函数在重定位表中的偏移量。然后，控制权会转移到特殊的第一个 PLT 条目，该条目会在堆栈上压入更多的值，最终跳转到动态链接器。动态链接器需要确保第三个 GOT 槽（偏移量为 8）包含动态链接器的入口地址（`jmp *8(%ebx)`）。一旦动态链接器确定了函数地址，它会将地址存储在用于初始跳转的 GOT 条目中（`jmp *name@GOT(%ebx)`），然后跳转到目标函数。这意味着除了第一次调用外，所有后续调用只需一次间接跳转，从而减少了开销。

The PLT stub is always used if the function is not guaranteed to bedeﬁned in the object which references it. Please note that a simple deﬁnition in the object with the reference is not enough to avoid the PLT entry. Looking at the symbol lookup process it should be clear that the definition could be found in another object (interposition) in which case the PLT is needed. We will later explain exactly when and how to avoid PLT entries.

即使在引用它的对象中已经定义了符号，也有可能使用 PLT。这是因为在符号查找过程中，符号可能会在其他对象中被找到，即："符号插入"（interposition）。也就是说，符号的定义可能会被其他对象中的同名符号覆盖。在这种情况下，即使引用对象中有符号的定义，我们也不得不交到运行时期间去完成，即程序仍然会通过 PLT 来调用这个符号，以确保能够正确处理符号插入。

How exactly the GOT and PLTis structured is architecture-speciﬁc, speciﬁed in the respective psABI. What was said here about IA-32 is in some form applicable to some other architectures but not for all. For instance, while the PLT on IA-32 is readonly it must be writable for other architectures since instead of indirect jumps using GOT values the PLT entries are modiﬁed directly. A reader might think that the designers of the IA-32 ABI made a mistake by requiring a indirect, and therefore slower, call instead of a direct call. This is no mistake, though. Having a writable and executable segment is a huge security problem since attackers can simply write arbitrary code into the PLT and take over the program. We can anyhow summarize the costs of using GOT and PLT like this:

GOT 和 PLT 的具体结构是根据不同的体系结构而定的，由相应的 psABI 规范规定。这里关于 IA-32 的描述在某些方面也适用于其他一些体系结构，但并非全部。例如，虽然 IA-32 上的 PLT 是只读的，但对于其他体系结构，PLT 条目必须是可写的，因为它们直接修改条目，而不是使用 GOT 值进行间接跳转。读者可能会认为 IA-32 ABI 的设计者要求使用间接调用而不是直接调用是一种错误，因为间接调用较慢。但实际上，这样设计是为了安全考虑。如果段既可写又可执行，攻击者可以轻易地将任意代码写入 PLT 并控制程序。我们可以总结一下使用 GOT 和 PLT 的成本：

- every use of a global variable which is exported uses a GOT entry and loads the variable values indirectly; 

    每次访问导出的全局变量时，都会通过 GOT 条目来间接加载变量的值；

- each function which is called (as opposed to refer- enced as a variable) which is not guaranteed to be deﬁned in the calling object requires a PLT entry. The function call is performed indirectly by transferring control ﬁrst to the code in the PLT entry which in turn calls the function. 

    每次调用的函数（而不是作为变量引用）如果不能保证在调用对象中定义，则需要使用一个 PLT 条目。函数调用通过间接方式执行，首先控制权转移到 PLT 条目的代码，然后再调用实际的函数；

- for some architectures each PLT entry requires at least one GOT entry.

    对于某些架构，每个 PLT 条目至少需要一个对应的 GOT 条目。

Avoiding a jump through the PLT therefore removes on IA-32 16 bytes of text and 4 bytes of data. Avoiding the GOT when accessing a global variable saves 4 bytes of data and one load instruction (i.e., at least 3 bytes of code and cycles during the execution).  In addition each GOT entry has a relocation associated with the costs described above.

在 IA-32 架构中，避免通过 PLT 跳转可以减少 16 字节的代码和 4 字节的数据。访问全局变量时避免使用 GOT 可以节省 4 字节的数据和一条加载指令（至少 3 字节的代码和执行时间）。此外，每个 GOT 条目都需要重定位操作，增加了前面提到的成本。

### **1.5.6** Running **the** **Constructors**

Once therelocations are performed the DSOs and the ap- plication code can actually be used. But there is one more thing to do: optionally the DSOs and the application must be initialized. The author of the code can deﬁne for each object a number of initialization functions which are run before the DSO is used by other code. To perform the ini- tialization the functions can use code from the own object and all the dependencies. To make this work the dynamic linker must make sure the objects are initialized in the correct order, i.e., the dependencies of an object must be initialized before the object.

在完成重定位之后，动态共享对象（DSOs）和应用程序代码就可以实际运行了。但还需要进行一项操作：初始化 DSOs 和应用程序（如果有需要）。开发者可以为每个对象定义多个初始化函数，这些函数会在 DSO 被其他代码使用之前执行。为了完成初始化，这些函数可以调用自身对象的代码和所有依赖的代码。为了确保初始化过程正确进行，动态链接器必须保证对象按照正确的顺序初始化，也就是说，一个对象的依赖项必须先于该对象初始化。

To guarantee that the dynamic linker has to perform a topological sort in the list of objects.  This sorting is no linear process. Like all sorting algorithms the run-time is at least O(nlog n) and since this is actually a topological sort the value is even higher. And what is more: since the order at startup need not be the same as the order at shutdown (when ﬁnalizers have to be run) the whole process has to be repeated.

为了确保初始化顺序正确，动态链接器必须对对象列表进行拓扑排序。这种排序不是线性的。与所有排序算法一样，运行时间至少是  $O(nlog_n)$，由于是拓扑排序，实际运行时间可能更长。此外，启动时的初始化顺序与关闭时的终结顺序可能不同（关闭时需要运行终结函数），因此，动态链接器需要再次进行拓扑排序，以确定正确的终结顺序。

> 针对于 DSO 依赖关系的梳理以保证以一条正确的初始化顺序去执行，这是动态链接器保证的。然而针对于在代码中声明的拥有静态存储期的变量的初始化顺序，在编译器中是未定义的。这里我们要区别开来二者之间的关系。

So we have again a cost factor which is directly depending on the number of objects involved.  Reducing the number helps a bit even though the actual costs are normally much less than that of the relocation process.

因此，涉及对象的数量直接影响成本。虽然实际成本通常比重定位过程低很多，但减少对象数量仍能有所帮助。

At this point it is useful to look at the way to correctly write constructors and destructors for DSOs.  Some systems had the convention that exported functions named  _init and  _fini are automatically picked as constructor and destructor respectively.  This convention is still followed by GNU ld and using functions with these names on a Linux system will indeed cause the functions used in these capacities. But this is totally, 100% wrong!

在这里，我们需要了解如何正确编写 DSOs（动态共享对象）的构造函数和析构函数。某些系统有一个约定，即如果一个函数命名为 `_init` 或 `_fini`，它们会自动被当作 DSO 的构造函数和析构函数。这种约定在 GNU 链接器（GNU ld）中仍然适用，在 Linux 系统上使用这些名称的函数确实会被识别为构造函数和析构函数。但这种做法是完全错误的！

By using these functions the programmer overwrites whatever initialization and destruction functionality the system itself is using.   The result is a DSO which is not  fully initialized and this sooner or later leads to a catastrophy.  The correct way of adding constructors and destructors is by marking functions with the constructor  and destructor function attribute respectively.

使用这些函数会让程序员覆盖系统本身的初始化和销毁功能（在某些系统中，这样的函数会有特殊的用途）。这样会导致 DSO 未完全初始化，最终引发严重问题。正确的做法是通过构造函数和析构函数属性来标记这些函数。

```c
__attribute((constructor))
void init_function(void) {
  ...
}

__attribute((destructor))
void fini_function(void) {
  ...
}
```

These functions should not be exported either (see sections 2.2.2 and 2.2.3) but this is just an optimization. With the functions deﬁned like this the runtime will arrange that they are called at the right time, after performing whatever initialization is necessary before.

这些函数（DSO 的构造函数和析构函数）不应该被声明（参见第 2.2.2 和 2.2.3 节），但这只是一个优化建议。通过这种方式定义函数，运行时系统会在完成所有必要的初始化操作之后，确保在适当的时间调用它们。

## 1.6 Summary of the Costs of ELF

We have now discussed the startup process and how it is affected by the form of the binaries. We will now summarize the various factors so that we later on can determine the beneﬁts of an optimization more easily.

我们已经讨论了启动过程以及二进制文件形式对启动过程的影响。现在，我们将总结各种因素，以便之后能够更容易地评估优化的好处。

- Code Size:

    代码大小：

    As everywhere, a reduced size for code with the same semantics often means higher efﬁciency and performance.  Smaller ELF binaries needless memory at run-time.

    和其他地方一样，在语义相同的情况下，代码大小的减少通常意味着更高的效率和性能。较小的 ELF 二进制文件在运行时需要更少的内存。

    In general the compiler will always generate the best code possible and we do not cover this further. But it must be known that every DSO includes a certain overhead in data and code. Therefore fewer DSOs means smaller text.

    通常，编译器会尽可能生成最佳代码，我们不会进一步讨论这个问题。但必须知道，每个 DSO 都包含一定的数据和代码开销。因此，DSO 数量越少，代码体积越小。

- Number of Objects:

    对象的数量：

    The fact that a smaller number of objects containing the same functionality is beneﬁcial has been mentioned in several places:

    减少包含相同功能的对象数量是有益的，这一点在多个地方都有提到：

    - Fewer objects are loaded at run-time.  This directly translates to fewer system call. In the GNU dynamic linker implementation loading a DSO requires at least 8 system calls, all of them can be potentially quite expensive.

      运行时加载的对象越少，系统调用次数越少。在 GNU 动态链接器中，加载一个 DSO 至少需要 8 次系统调用，这些调用可能非常昂贵。

    - Related, the application and the dependencies with additional dependencies must record the names of the dependencies. This is not a terribly high cost but certainly can sum up if there are many dozens of dependencies.

      此外，应用程序及其依赖项需要记录所有依赖项的名称。虽然这不是很高的成本，但如果有几十个依赖项，成本会累积。

    - The lookup scope grows.  This is one of the dominating factors in cost equation for the re- locations.

       查找范围扩大，这是重定位成本的主要因素之一。

    - More objects means more symbol tables which in turn normally means more duplication. Undeﬁned references are not collapsed into one and handling of multiple deﬁnitions have to be sorted out by the dynamic linker.

      更多的对象意味着更多的符号表，这通常会导致更多的符号重复。相同符号的多个定义不会自动合并为一个，动态链接器必须处理这些重复的定义。

      Moreover, symbols are often exported from a DSO to be used in another one. This would not have to happen if the DSOs would be merged.

      此外，符号通常从一个 DSO 导出供另一个 DSO 使用。如果将 DSOs 合并，这种情况就不会发生。

    - The sorting of initializers/ﬁnalizers is more complicated.

      初始化器/终结器的排序变得更加复杂。

    - In general does the dynamic linker have some overhead for each loaded DSO per process. Everytime a new DSO is requested the list of already loaded DSOs must be searched which can be quite time consuming since DSOs can have many aliases.

       通常，动态链接器在每个进程中加载每个 DSO 都会有一些开销。每次请求加载新的 DSO 时，都必须搜索已经加载的 DSO 列表，这个过程可能非常耗时，因为 DSOs 可能有多个别名。

- Number of Symbols:

     符号的数量：

     The number of exported and undeﬁned symbols determines the size of the dynamic symbol table, the hash table, and the average hash table chain length. The normal symbol table is not used at runtime and it is therefore not necessary to strip a binary of it. It has no impact on performance.

    导出符号和未定义符号的数量决定了动态符号表、哈希表以及哈希链的平均长度。区别于动态符号表，普通符号表在运行时不使用，因此没有必要从二进制文件中删除它，这对性能没有影响。

     Additionally, fewer exported symbols means fewer chances for conﬂicts when using pre-linking (not covered further).

     此外，减少导出的符号可以减少预链接时发生冲突的机会（此处不再详细讨论）。

- Length of Symbol Strings:

     符号字符串的长度：

     Long symbol lengths cause often unnecessary costs. A successful lookup of a   symbol must match the whole string and comparing dozens or hundreds of characters takes time. Unsuccessful lookups suffer if common preﬁxes are long as in the new C++ mangling scheme.  In any case do long symbol names cause large string tables which must be present at run-time and thereby is adding costs in load time and in use of address   space which is an issue for 32-bit machines.

    长符号名称通常会导致不必要的开销。成功查找符号时，需要匹配整个字符串，比较几十或几百个字符会消耗时间。如果符号有很长的公共前缀（如新的 C++ 名字修饰方案），未成功查找时也会受到影响。无论如何，长符号名称会导致字符串表变大，这些表在运行时必须存在，因此会增加加载时间和地址空间的使用成本，这对于 32 位机器来说尤其问题严重。

- Number of Relocations:

    重定位的数量：

     Processing relocations constitute the majority of work during start and therefore any reduction is directly noticeable.

    处理重定位是启动过程中的主要工作，因此减少重定位的数量会显著提高启动速度。

- Kind of Relocations:

  重定位类型：
  
     The kind of relocations which are needed is important, too, since processing a relative relocation is much less expensive than a normal relocation.  Also, relocations against text segments must be avoided.

     重定位的类型非常重要，因为处理相对重定位的开销比普通重定位[[#^a6c493|小得多]]。此外，应避免对代码段的重定位。

- Placement of Code and Data:

     代码和数据的放置：

     All executable code should be placed in readonly memory and the compiler normally makes sure this is done correctly. When creating data objects it is mostly up to the user to make sure it is placed in the correct segment. Ideally data is also readonly but this works only for constants. The second best choice is a zero-initialized variable which does not have to be initialized from ﬁle content. The rest has to go into the data segment.

    所有可执行代码都应放置在只读内存中，编译器通常会确保正确执行这一点。在创建数据对象时，用户通常需要确保其放置在正确的段中。理想情况下，数据应是只读的，但这仅适用于常量。次佳选择是零初始化变量，这些变量不需要从文件内容初始化。其他数据必须放置在数据段中。

In the following we will not cover the ﬁrst two points given here.  It is up to the developer of the DSO to decide about this. There are no small additional changes to make the DSO behave better, these are fundamental design decisions. We have voiced an opinion here, whether it is has any effect remains to be seen.

接下来我们不会讨论前面提到的这两点，因为这些是 DSO 开发人员需要做出的基本设计决策。这些决策不是通过一些小的改动就能提升 DSO 性能的。我们在这里表达了我们的看法，但是否有效还有待验证。

## 1.7 Measuring **ld.so** Performance


To perform the optimizations it is useful to quantify the effect of the optimizations. Fortunately it is very easy to do this with glibc’s dynamic linker. Using the LD_DEBUG environment variable it can be instructed to dump in- formation related to the startup performance.  Figure 8 shows an example invocation, of the echo program in this case.

为了进行优化，量化优化效果是非常重要的。幸运的是，使用 glibc 的动态链接器可以很容易做到这一点。通过设置 LD_DEBUG 环境变量，可以让动态链接器输出与启动性能相关的信息。图 8 显示了一个调用 echo 程序的示例。

```text
$ env LD_DEBUG=statistics /bin/echo '+++ some text +++'
...:
...:   run-time linker statistics:
...:     total startup time in dynamic loader: 748696 clock cycles
...:               time needed for relocation: 378004 clock cycles (50.4%)
...:                    number of relocations: 133
...:         number of relocations from cache: 5
...:              time needed to load objects: 193372 clock cycles (25.8%)
+++ some text +++
...:
...:   run-time linker statistics:
...:              final number of relocations: 188
...:   final number of relocations from cache: 5
```
<center>
<div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 8: Gather Startup Statistics</div>
</center>

The output of the dynamic linker is divided in two parts. The part before the program’s output is printed right be- fore the dynamic linker turns over control to the application after having performed all the work we described in this section. The second part, a summary, is printed after the application terminated (normally). The actual format might vary for different architectures. It includes the timing information only on architectures which pro- vide easy access to a CPU cycle counter register (modern IA-32, IA-64, x86-64, Alpha in the moment).  For other architectures these lines are simply missing.

动态链接器的输出分为两部分。第一部分在程序输出之前打印，即在动态链接器完成所有初始化工作并将控制权移交给应用程序之前。第二部分是总结信息，在应用程序正常结束后打印。实际的输出格式可能因架构不同而有所变化。只有在可以轻松访问 CPU 周期计数寄存器的架构上（如现代的 IA-32、IA-64、x86-64、Alpha）才包含计时信息。对于其他架构，这些信息将缺失。

The timing information provides absolute values for the total time spend during startup in the dynamic linker, the time needed to perform relocations, and the time spend in the kernel to load/map binaries.  In this example the relocation processing dominates the startup costs with more than 50%. There is a lot of potential for optimizations here.  The unit used to measure the time is CPU cycles. This means that the values cannot even be com- pared across different implementations of the same architecture. E.g., the measurement for a Pentium III and a Pentium 4 machine will be quite different. But the measurements are perfectly suitable to measure improve- ments on one machine which is what we are interested here.

计时信息提供了动态链接器在启动过程中所花费的总时间、执行重定位所需的时间以及内核加载/映射二进制文件的时间。在这个例子中，重定位处理占据了启动成本的主要部分，超过了50%。这表明这里有很大的优化潜力。时间的测量单位是 CPU 周期，这意味着不同处理器之间的值不可直接比较。例如，Pentium III 和 Pentium 4 的测量结果会有很大差异。但这些测量结果非常适合评估同一台机器上的性能改进，这是我们关心的重点。

Since relocations play such a vital part of the startup performance some information on the number of relocations is printed.  In the example a total of 133 relocations are performed, from the dynamic linker, the C library, and the executable itself.  Of these 5 relocations could be served from the relocation cache. This is an optimization implemented in the dynamic linker to handle the case of multiple relocations against the same symbol more efﬁcient. After the program itself terminated the same information is printed again.  The total number of relocations here is higher since the execution of the application code caused a number, 55 to be exact, of run-time relocations to be performed.

由于重定位在启动性能中起着关键作用，因此会打印一些关于重定位数量的信息。在这个例子中，总共执行了 133 次重定位，包括动态链接器、C 库和可执行文件本身的重定位。其中 5 次重定位可以从重定位缓存中处理。这是动态链接器中的一种优化，用于更高效地处理对同一符号的多次重定位。程序结束后，会再次打印相同的信息。此时的总重定位次数更高，因为应用程序代码的执行引发了一些运行时重定位，总计 55 次。

The number of relocations which are processed is stable across successive runs of the program. The time measurements not. Even in a single-user mode with no other programs running there would be differences since the cache and main memory has to be accessed. It is therefore necessary to average the run-time over multiple runs.

处理的重定位次数在程序的连续运行中是稳定的。而时间测量则会有所不同。即使在没有其他程序运行的单用户模式下也会有差异，因为需要访问缓存和主存。因此，有必要多次运行来计算平均运行时间。

It is obviously also possible to count the relocations without running the program.  Running `readelf  -d` on the binary shows the dynamic section in which the DT_RELSZ, DT_RELENT, DT_RELCOUNT, and DT_PLTRELSZ entries are interesting. They allow computing the number of normal and relative relocations as well as the number of PLT entries. If one does not want to do this by hand the `relinfo` script in [[#A Counting Relocations|appendix A]] can be used.

显然，也可以在不运行程序的情况下计算重定位次数。通过运行 `readelf -d` 命令，可以查看二进制文件的动态部分，其中 DT_RELSZ、DT_RELENT、DT_RELCOUNT 和 DT_PLTRELSZ 条目是关键。它们可以帮助计算普通重定位、相对重定位的数量以及 PLT 条目的数量。如果不想手动计算，可以使用 [[#A Counting Relocations|附录A]] 中的 `relinfo` 的脚本。

# 2 Optimizations for DSOs

In this section we describe various optimizations based on C or C++ variables or functions. The choice of variable or function, unless explicitly said, is made deliberately since many of the implementations apply to the one or the other. But there are some architectures where functions are handled like variables. This is mainly the case for embedded RISC architectures like SH-3 and SH-4 which have limitations in the addressing modes they pro- vide which make it impossible to implement the function handling as for other architectures.  In most cases it is no problem to apply the optimizations for variables and functions at the same time. This is what in fact should be done all the time to achieve best performance across all architectures.

在本节中，我们描述了如何对 C 或 C++ 的变量和函数进行优化。变量或函数的选择（除非特别说明）是经过深思熟虑的，因为许多优化对两者都适用。但在某些架构中，例如嵌入式 RISC 架构 SH-3 和 SH-4，由于它们在寻址模式上的限制，函数被当作变量处理。在大多数情况下，变量和函数的优化可以一起进行。这实际上是为了在所有架构上实现最佳性能。

The most important recommendation is to always use -fpic or -fPIC when generating code which ends up in DSOs. This applies to data as well as code. Code which is not compiled this way almost certainly will contain text relocations. For these there is no excuse. Text relocations requires extra work to apply in the dynamic linker.  And argumentation saying that the code is not shared because no other process uses the DSO is invalid. In this case it is not useful to use a DSO in the ﬁrst place; the code should just be added to the application code.

最重要的建议是，在生成将包含在 DSOs 中的代码时始终使用 -fpic 或 -fPIC。这不仅适用于代码，也适用于数据。未以这种方式编译的代码几乎肯定会包含文本重定位。这些重定位需要动态链接器额外处理，没有任何合理理由这样做。因为没有其他进程使用 DSO，所以代码不是共享的说法是无效的。在这种情况下，使用 DSO 本身就没有意义；这些代码应该直接添加到应用程序代码中。

> [!tip]
>文本重定位（Text Relocation），也称为代码重定位，是指在运行时（由动态链接器）需要修改可执行文件或共享库的代码段（通常是只读的文本段）中的地址指令，以确保程序能够在加载后的内存地址上正确运行。这通常涉及修改指令中的绝对地址，以适应程序实际加载的内存位置。
>
>
>文本重定位并不是一个很好的设计，我们应当尽量避免它的出现：首先，代码段通常是只读的，这样可以防止恶意代码修改。如果需要进行文本重定位，就必须使代码段变为可写，这增加了被攻击的风险；其次，进行文本重定位会增加程序启动时的开销，因为在程序加载到内存后，需要额外的步骤来修改代码段中的地址指令；最后，编写和维护包含文本重定位的代码更复杂，因为需要确保所有绝对地址在加载时都能正确更新。
>

^855784

Some people try to argue that the use of -fpic/-fPIC on some architectures has too many disadvantages. This is mainly brought forward in argumentations about IA- 32.  Here the use of %ebx as the PIC register deprives the compiler of one of the precious registers it could use for optimization.  But this is really not that much of a problem.  First, not having %ebx available was never a big penalty. Second, in modern compilers (e.g., gcc after release 3.1) the handling of the PIC register is much more ﬂexible.  It is not always necessary to use %ebx which can help eliminating unnecessary copy operations.  And third, by providing the compiler with more information as explained later in this section a lot of the overhead in PIC can be removed. This all combined will lead to overhead which is in most situations not noticeable.

有些人认为在某些架构上使用 -fpic/-fPIC 有很多缺点，尤其是在 IA-32 架构上。他们认为，使用 %ebx 作为 PIC 寄存器会让编译器失去一个重要的优化寄存器。但实际上，这并不是个大问题。首先，缺少 %ebx 并不是一个很大的惩罚。其次，在现代编译器（例如 gcc 3.1 之后的版本）中，PIC 寄存器的处理更加灵活，不一定总是需要使用 %ebx，从而减少不必要的复制操作。最后，通过向编译器提供更多信息（如本节后面解释的），可以消除许多 PIC 的开销。这些因素结合起来，使得在大多数情况下，这些开销几乎不可察觉。

When gcc is used, the options -fpic/-fPIC also tell the  compiler that a number of optimizations which are possible for the executable cannot be performed. This has to  do with symbol lookups and cutting it short.  Since the  compiler can assume the executable to be the ﬁrst object  in the lookup scope it knows that all references of global  symbols known to be deﬁned in the executable are re-  solved locally.  Access to locally deﬁned variable could  be done directly, without using indirect access through  the GOT. This is not true for DSOs:  the DSOs can be  later in the lookup scope and earlier objects might be interposed.  It is therefore mandatory to compile all code  which can potentially end up in a DSO with -fpic/-fPIC since otherwise the DSO might not work correctly. There  is no compiler option to separate this optimization from the generation of position-independent code.

使用 gcc 时，-fpic/-fPIC 选项还告诉编译器，一些适用于可执行文件的优化不能用于 DSO。这涉及符号查找并简化了处理。编译器假定可执行文件是查找范围中的第一个对象，因此可执行文件中定义的所有全局符号引用都能本地解析，访问本地定义的变量可以直接进行，无需通过 GOT 间接访问。而对于 DSOs，情况则不同：DSOs 可能在查找范围的后面，必须考虑（查找范围的）前面已经加载的其他对象。因此，所有可能包含在 DSO 中的代码必须用 -fpic/-fPIC 编译，否则 DSO 可能无法正常工作。没有编译器选项可以让编译器在生成位置无关代码（用于确保 DSOs 正确解析符号）的同时，继续执行这种用于可执行文件的符号查找优化。

Which of the two options, -fpic or -fPIC, have to be used must be decided on a case-by-case basis. For some architectures there is no difference at all and people tend to be careless about the use. For most RISC there is a big difference. As an example, this is the code gcc generates for SPARC to read a global variable global when using -fpic:

使用 -fpic 还是 -fPIC 取决于具体情况。对于某些架构，这两者没有区别，因此人们不太在意其使用。但对于大多数 RISC 架构，区别很大。以下是 gcc 在 SPARC 上使用 -fpic 读取全局变量 global 时生成的代码示例：

```
sethi %hi(_GLOBAL_OFFSET_TABLE_-4), %l7

call .LLGETPC0

add %l7,%lo(_GLOBAL_OFFSET_TABLE_+4),%l7

ld [%l7+global],%g1

ld [%g1],%g1
```

And this is the code sequence if -fPIC is used:

这是使用 -fPIC 时的代码序列：

```
sethi %hi(_GLOBAL_OFFSET_TABLE_-4),%l7

call .LLGETPC0

add %l7,%lo(_GLOBAL_OFFSET_TABLE_+4),%l7

sethi %hi(global),%g1

or %g1,%lo(global),%g1

ld [%l7+%g1],%g1

ld [%g1],%g1
```

In both cases %l7 is loaded with the address of the GOT ﬁrst.   Then the  GOT is accessed to get the address of global. While in the -fpic case one instruction is suf- ﬁcient, three instructions are needed in the -fPIC case. The -fpic option tells the compiler that the size of the GOT does not exceed an architecture-speciﬁc value (8kB in case of SPARC). If only that many GOT entries can be present the offset from the base of the GOT can be encoded in the instruction itself, i.e., in the ld instruc- tion of the ﬁrst code sequence above.  If -fPIC is used no such limit exists and so the compiler has to be pes- simistic and generate code which can deal with offsets of any size. The difference in the number of instructions in this example correctly suggests that the -fpic should be used at all times unless it is absolutely necessary to use -fPIC. The linker will failand write out a message when this point is reached and one only has to recompile the code.

在这两种情况下，%l7 都首先加载 GOT 的地址。然后，通过访问 GOT 来获取全局变量 global 的地址。使用 -fpic 时，一条指令就足够了，而使用 -fPIC 时则需要三条指令。-fpic 选项告诉编译器，GOT 的大小不会超过特定的架构值（例如在 SPARC 中为 8kB）。如果 GOT 条目数量在这个限制内，偏移量可以直接编码在指令中，即在上面第一段代码序列中的 ld 指令中。如果使用 -fPIC，则没有这种限制，因此编译器必须生成可以处理任意大小偏移量的代码。这个例子中的指令数量差异正确地表明，除非绝对必要，否则应始终使用 -fpic。当链接器达到限制时，它会失败并输出一条消息，此时只需重新编译代码即可。

When writing assembler code by hand it is easy to miss cases where position independent code sequences must be used.  The non-PIC sequences look and actually are simpler and more natural. Therefore it is extremely im- portant to in these case to check whether the DSO is marked to contain text relocations.  This is easy enough to do:

在手写汇编代码时，容易忽略需要使用位置无关代码的情形。非位置无关代码的序列看起来更简单、更自然。因此，在这种情况下，检查 DSO 是否标记为包含文本重定位是非常重要的。这可以轻松实现：

```
readelf -d _binary_ | grep TEXTREL
```

If this produces any output text relocations are present and one better starts looking what causes them.

如果有任何输出，就会出现文本重定位，最好开始查找导致文本重定位的原因。

## 2.1 Data Definitions

Variables can be deﬁned in C and C++ in several different ways. Basically there are three kinds of deﬁnitions:

在 C 和 C++ 中，变量可以有几种不同的定义方式。主要有三种类型：

- Common

    公共的

    Common variables are more widely used [FORTRAN](https://en.wikipedia.org/wiki/Fortran) but they got used in C and C++ as well to work around mistakes of programmers. Since in the early days people used to drop the extern keyword from variable deﬁnitions, in the same way it is possible to drop it from function declaration, the compiler often has multiple deﬁnitions of the same variable in different ﬁles. To help the poor and clueless programmer the C/C++ compiler normally generates common variables for uninitialized deﬁnitions such as

    公共变量最早在 [FORTRAN](https://en.wikipedia.org/wiki/Fortran) 中广泛使用，但在 C 和 C++ 中也被用来处理程序员的错误。因为早期人们习惯省略变量定义中的 extern 关键字，就像省略函数声明中的 extern 关键字一样，编译器通常会在不同文件中遇到同一个变量的多个定义。为了帮助那些迷茫的程序员，C/C++ 编译器通常会为未初始化的定义生成公共变量，例如：

  ```
  int foo;
  ```

    For common variables there can be more than one deﬁnition and they all get uniﬁed into one location in the output ﬁle. Common variables are always initialized with zero. This means their value does not have to be stored in an ELF ﬁle.  Instead the ﬁle size of a segment is chosen smaller than the memory size as described in [[#1.4 Startup In The Kernel|1.4]]

    对于公共变量，可以存在多个定义，它们在输出文件中会被统一处理到一个位置。公共变量总是被初始化为零，这意味着它们的值不需要存储在 ELF 文件中。因此，段的文件大小比内存大小更小，如 [[#1.4 Startup In The Kernel|1.4]] 中所描述的那样。

- Uninitialized

    未初始化的

    If the programmer uses the compiler command line option `-fno-common` the generated code will contain uninitialized variables instead of common variables if a variable deﬁnition has no initializer. Alternatively, individual variables can be marked like this:

    如果程序员使用编译器命令行选项 `-fno-common`，那么对于没有初始化值的变量定义，编译器会生成未初始化的变量，而不是公共变量。或者，可以单独标记变量，如下所示：

    ```
    int foo attribute((nocommon));
    ```

    The result at runtime is the same as for common variable, no value is stored in the ﬁle. But the representation in the object ﬁle is different and it allows the linker to ﬁnd multiple deﬁnitions and ﬂag them as errors. Another difference is that it is possible to deﬁne aliases, i.e., alternative names, for uninitialized variables while this is not possible for common variables.

    未初始化变量在运行时的行为与公共变量相同，即它们的值不会存储在文件中。但在目标文件中的表示方式不同。这使得链接器能够找到多个定义并将其标记为错误（公共变量是允许合并重复的定义）。另一个区别是，可以为未初始化变量定义别名（即替代名称），而这对公共变量是不可能的。

    With recent gcc versions there is another method to create uninitialized variables. Variables initialized with zero are stored this way. Earlier gcc versions stored them as initialized variables which took up space in the ﬁle. This is a bit cumbersome for variables with structured types. So, sticking with the per-variable attribute is probably the best way.

    在最新版本的 gcc 中，还有另一种方法可以创建未初始化变量。用零初始化的变量会以这种方式存储。早期版本的 gcc 将它们存储为已初始化变量，这会占用文件中的空间。对于结构化类型的变量，这有点麻烦。因此，使用每个变量的属性可能是最好的方法。

- Initialized

    已初始化

    The variable is deﬁned and initialized to a programmer-deﬁned value. In C:

    变量被定义并初始化为程序员指定的值。在 C 语言中：

    ```
    int foo = 42;
    ```

    In this case the initialization value is stored in the ﬁle.  As described in the previous case initializations with zero are treated special by some compilers.

    在这种情况下，初始化值会存储在文件中。 如前所述，一些编译器会对零初始化进行特殊处理。

Normally there is not much the user has to do to create optimal ELF ﬁles. The compiler will take care of avoiding the initializers. To achieve the best results even with old compilers it is desirable to avoid explicit initializations with zero if possible. This creates normally common variables but if combined with gcc’s `-fno-common` ﬂag the same reports about multiple deﬁnitions one would get for initialized variables can be seen.

通常，用户不需要做太多工作就能生成优化的 ELF 文件。编译器会自动避免使用初始化器。即使使用旧版本的编译器，为了获得最佳效果，也应尽量避免显式地将变量初始化为零。这通常会生成公共变量，但如果结合使用 gcc 的 `-fno-common` 选项，编译器会像处理已初始化变量那样报告未初始化变量的多重定义错误。

There is one thing the programmer is responsible for. As an example look at the following code:

有一件事是程序员需要特别注意的。请看以下代码：

```c
bool is_empty = true;

char s[10];

const char *get_s(void) {
  return is_empty ? NULL : s;
}
```

The function `get_s` uses the boolean variable is empty to decide what to do.  If the variable has its initial value the variable `s` is not used. The initialization value of is empty is stored in the ﬁle since the initialize is non-zero. But the semantics of is empty is chosen arbitrarily. There is no requirement for that. The code could instead be rewritten as:

函数 `get_s` 使用布尔变量 `is_empty` 来决定要执行什么操作。如果变量具有初始值，则变量 `s` 不会被使用。因为 `is_empty` 的初始值是非零的，所以该值会存储在文件中。但 `is_empty` 的初始值是随意选择的，没有特定的要求。因此，这段代码可以重写为：

```c
bool is_empty = false;

char s[10];

const char *get_s(void) {
  return !is_empty ? s : NULL;
}
```

Now the semantics of the control variable is reversed. It is initialized with false which is guaranteed to have the numeric value zero. The test in the function `get_s` has to be changed as well but the resulting code is not less or more efﬁcient than the old code.

现在控制变量的语义被反转了。它被初始化为 `false`，这确保了数值为零。在函数 `get_s` 中的条件判断也需要修改，但修改后的代码效率与原来的代码相同。

By simple transformations like that it is often possible to avoid creating initialized variables and instead using common or uninitialized variables. This saves disk space and eventually improves startup times. The transformation is not limited to boolean values. It is sometimes possible to do it for variables which can take on more than two values, especially enumeration values. When deﬁning `enum`s one should always put the value, which is most often used as initializer, ﬁrst in the `enum` deﬁnition. I.e.

通过这样的简单转换，通常可以避免创建已初始化变量，而使用公共变量或未初始化变量。这节省了磁盘空间，并最终提高了启动时间。此转换不仅限于布尔值，有时也可以用于具有多个值的变量，尤其是枚举类型。在定义 `enum` 时，应始终将最常用的初始值放在 `enum` 定义的第一位（第一位的枚举值通常初始化为 0）。例如：

```c
enum { val1, val2, val3 } ;
```

should be rewritten as

应该重写为

```c
enum { val3, val1, val2 } ;
```

if `val3` is the value most often used for initializations. To summarize, it is always preferable to add variables as uninitialized or initialized with zero as opposed to as initialized with a value other than zero.

如果 `val3` 是最常用于初始化的值。总的来说，最好将变量设为未初始化或初始化为零，而不要用非零值进行初始化。

##  2.2 Export Control

When creating a DSO from a collection of object files the  dynamic symbol table will by default contain all the symbols which are globally visible in the object files.In most cases this set is far too large. Only the symbols which are actually part of the ABI should be exported.Failing to restrict the set of exported symbols are numerous drawbacks:

在从一组目标文件创建 DSO 时，动态符号表默认会包含目标文件中所有全局可见的符号。然而，这些符号通常过多。只有实际属于 ABI 的符号才应该被导出。不限制导出符号集会带来许多问题：

- Users of the DSO could use interfaces which they  are not supposed to.This is problematic in revisions of the DSO which are meant to be binary compatible. The correct assumption of the DSO developer is that interfaces, which are not part of the ABI, can be changed arbitrarily. But there are  always users who claim to know better or do not care about rules.

    DSO 的用户可能会使用他们不应该使用的接口。这在需要保持二进制兼容性的 DSO 版本中是个问题。DSO 开发人员通常假设非 ABI 的接口可以随意更改，但总有一些用户会违反或忽视这些规则。

- According to the ELF lookup rules all symbols in the dynamic symbol table can be interposed(unless the visibility of the symbol is restricted). I.e., definitions from other objects can be used.This means that local references cannot be bound at link time. If it is known or intended that the local definition should always be used the symbol in the reference must not be exported or the visibility must be restricted.

    根据 ELF 查找规则，动态符号表中的所有符号都可能被其他对象的定义替换（除非限制了符号的可见性）。这意味着本地引用在链接时无法确定。如果希望始终使用本地定义的符号，那么这些符号不应被导出，或者其可见性应受到限制。

- The dynamic symbol table and its string table are available at run-time and therefore must be loaded. This can require a significant amount of memory, even though it is read-only. One might think that the size is not much of an issue but if one examines the length of the mangled names of C++ variables or functions,it becomes obvious that this is not the case. In addition we have the run-time costs of larger symbol tables which we discussed in the previous section

    动态符号表及其字符串表在运行时必须加载，这可能需要大量内存，即使它们是只读的。虽然有人认为大小不是问题，但如果检查 C++ 变量或函数的修饰名称长度，就会发现情况并非如此。此外，较大符号表的运行时成本也是一个问题，如我们在前一节讨论的那样。

We will now present a number of possible solutions for the problem of exported interfaces.Some of them solve the same problem in slightly different ways.We will say which method should be preferred.The programmer has to make sure that whatever is used is available on the target system.

现在，我们将介绍一些解决导出接口问题的可行方案，其中一些方案以略有不同的方式解决了相同的问题，我们将说明应优先选择哪种方法。程序员必须确保所使用的方案在目标系统中可用。

In the discussions of the various methods we will use one example:

在讨论各种方法时，我们将使用一个例子：

```c
int last;

int next(void) {
  return ++last;
}

int index(int scale) {
  return next() << scale;
}
```

Compiled on a IA-32 Linux machine a DSO with only this code(plus startup code etc)contains seven relocations, two of which are relative, and four PLT entries (use the relinfo script). We will see how we can improve on this. Four of the normal and both relative relocations as well as three PLT entries are introduced by the additional code used by the linker to create the DSO. The actual example code creates only one normal relocation for last and one PLT entry for next. To increment and read the variable last in next the compiler generates code like

在一台 IA-32 Linux 机器上编译的包含该代码（以及启动代码等）的 DSO 会有七个重定位记录，其中两个是相对重定位，另外四个是 PLT 条目（可以使用 relinfo 脚本查看）。我们将探讨如何改进这一点。四个普通重定位、两个相对重定位以及三个是由链接器在创建 DSO 时附加的代码引入的 PLT 条目。实际的示例代码仅为 `last` 变量创建了一个普通重定位，为 `next` 函数创建了一个 PLT 条目。为了在 `next` 函数中递增和读取 `last` 变量，编译器生成了如下代码。

```asm
movl last@GOT(%ebx), %edx
movl (%edx), %eax
incl %eax
movl %eax, (%edx)
```

and the call of `next` is compiled to

而 `next` 的调用被编译为

```asm
call next@PLT
```

These code fragments were explained in section [[#1.5.5 GOT and PLT|1.5.5]]

这些代码片段已在[[#1.5.5 GOT 和 PLT|1.5.5]] 节中说明。

### 2.2.1 Use static

The easiest way to not export a variable or function is to define it with file-local scope. In C and C++ this is done by defining it with static, in C++ additionally using  anonymous namespaces. This is for many people obvious but unfortunately not for all. Many consider adding static as optional. This is true when considering only the C semantics of the code.

避免导出变量或函数的最简单方法是将它们定义为文件本地范围。在 C 和 C++ 中，这可以通过 `static` 关键字来实现；在 C++ 中，还可以使用匿名命名空间。虽然这对许多人来说是显而易见的，但遗憾的是，并非所有人都理解这一点。许多人认为添加 `static` 是可选的，这在仅考虑 C 语言语义时确实如此。（然而，这种观点忽略了减少符号导出的重要性，即便在一个文件中，减少符号导出也是可以显著提高动态链接器的效率。）

If in our example neither last or next are needed out- side the file we can change the source to:

如果在我们的示例中，`last` 和 `next` 都不需要在文件外使用，我们可以将源代码改为：

```c
static int last;

static int next (void) { 
  return ++last; 
}

int index (int scale) {
  return next () <<scale;
}
```

Compiled in the same way as before we see that all the relocations introduced by  our example code vanished. L.e., we are left with six relocations and three PLT entries. The code to access last now looks like this:

按照同样的方法编译后，我们发现示例代码中引入的所有重定位都消失了。也就是说，我们只剩下六个重定位和三个 PLT 条目。现在最后访问的代码是这样的

```asm
movl last@GOTOFF(%ebx), %eax
incl %eax
movl %eax, last@GOTOFF(%ebx)
```

The code improved by avoiding the step which loads the address of the variable from the GOT. Instead, both memory accesses directly address the variable  in memory.At link-time the variable location has a fixed offset from the PIC   register, indicated symbolically by last@GOTOFF. By adding the value to the PIC register value we get the address of last. Since the value is known at link-time this construct does not need a relocation at run-time.

通过避免从 GOT 中加载变量地址的步骤，代码得到了改进。取而代之的是，两次内存访问都直接寻址内存中的变量。在链接时，变量位置与 PIC 寄存器有一个固定偏移，用 last@GOTOFF 符号表示。将该值与 PIC 寄存器的值相加，就得到了最后的地址。由于该值在链接时已知，因此该结构在运行时无需重新定位。

The situation is similar for the call to next. The IA-32 architecture, like many others, knows a PC-relative addressing mode for jumps and calls. Therefore the compiler can generate a simple jump instruction

对于调用 `next` 的情况也是类似的。IA-32 架构与许多其他架构一样，支持用于跳转调用的 PC 相对寻址模式。因此，编译器可以生成一个简单的跳转指令。

```asm
call next
```

and the assembler generates a PC-relative call. The difference between the address of the instruction following the call and the address of next is constant at link-time and therefore also does not need any relocation. Another advantage is  that, in the case of IA-32, the PIC register does not have to be set up before the jump. If the compiler wouldn't know the jump target is in the same DSO the PIC register would have to be set up. Other architectures have  similar requirements.

汇编器生成了一个基于 PC 相对寻址的调用。调用指令后的位置与 `next` 地址之间的差值在链接时是固定的，因此不需要重定位。另一个优点是，在 IA-32 架构中，PIC 寄存器在跳转前不需要设置。如果编译器知道跳转目标（如 `next` 函数）在同一个 DSO 中，它可以省略设置 PIC 寄存器的步骤，因为目标地址在链接时已经确定，不需要在运行时重新计算。如果编译器不知道跳转目标是否在同一个 DSO 中，则必须设置 PIC 寄存器以确保能够正确计算目标地址。其他架构在处理类似跳转和调用时，也有类似的要求。

The generated code is optimal. The compiler might even consider inlining some code if it finds that this is beneficial. It is always advised that the programmer places the various variable and function definitions in the same file as the references and then define the referenced objects as static. When generating the production binaries it might even be desirable to merge as many input files as possible together to mark as many objects as possible static. Unless one is comfortable  with one giant file there is a limit on how many functions can be grouped together. It is not necessary to continue the process ad infinitum since there are other  ways to achieve the same result(minus inlining).

生成的代码是最优的。如果编译器发现内联某些代码有益，它甚至可能会这样做。建议程序员将变量和函数定义放在与引用它们的相同文件中，并将被引用的对象定义为 `static`。在生成生产二进制文件时，甚至可以将尽可能多的输入文件合并在一起，以标记尽可能多的静态对象。除非你能接受一个非常大的文件，否则能组合在一起的函数数量是有限的。没有必要无限制地继续这个过程，因为还有其他方法可以实现相同的效果（除了内联）。

### 2.2.2 Define Global Visibility

The next best thing to using static is to explicitly define the visibility of objects in the DSO. The generic ELF ABI defines visibility of symbols. The specification defines four classes of which here only two are of interest. `STV_DEFAULT` denotes the normal visibility. The symbol is exported and can be interposed.The other interesting class is denoted by `STV_HIDDEN`. Symbols marked like this are not exported from the DSO and therefore cannot be used from other objects. There are a number of different methods to define visibility.

除了使用 `static` 关键字之外，另一种次优的做法是明确指定 DSO 中符号的可见性。ELF ABI 通用规范定义了符号的可见性。该规范定义了四种可见性类别，但我们只关心其中的两种。`STV_DEFAULT` 表示符号具有正常的可见性，这些符号是导出的，并且可以被其他对象覆盖。另一种有趣的可见性是 `STV_HIDDEN`，标记为这种可见性的符号不会从 DSO 中导出，因此不能被其他对象引用。定义可见性的方法有多种。

Starting with version 4.0, gcc knows about a the command line option `-fvisibility`.  It takes a parameter and the valid forms are these:

从 4.0 版开始，gcc 支持命令行选项 `-fvisibility`。 它需要一个参数，有效形式如下： ^3c87c1

```
-fvisibility=default
-fvisibility=hidden
-fvisibility=internal
-fvisibility=protected
```

Only the first two should ever be used. The default is unsurprisingly default since this is the behavior of the compiler before the introduction of this option. When `-fvisibility=hidden` is specified gcc changes the default visibility of all defined symbols which have no explicit assignment of visibility: all symbols are defined with STV HIDDEN unless specified otherwise. This option has to be used with caution since unless the DSO is prepared by having all APIs marked as having default visibility, the generated DSO will not have a single exported symbol. This is usually not what is wanted.

只有前两种（STV_DEFAULT 和 STV_HIDDEN）应该被使用。默认的可见性自然是默认的，因为这是在引入该选项之前编译器的行为。当使用 `-fvisibility=hidden` 时，gcc 会更改所有未明确指定可见性的符号的默认可见性：所有符号都被定义为 STV_HIDDEN（除非另有说明）。使用此选项时需要谨慎，因为如果 DSO 的所有 API 没有被标记为具有默认可见性（可导出），那么生成的 DSO 将不会有任何导出的符号。这通常不是我们所期望的。

In general it is the preference of the author which decides whether `-fvisibility=hidden` should be used. If it is not used, symbols which are not to be exported need to be marked in one way or another. The next section will go into details. In case the option is used all exported functions need to be declared as having visibility default which usually means the header files are significantly uglified. On the other hand it means that no symbol can accidentally be exported because an appropriate declaration etc is missing. In some situations this can prevent bad surprises.

一般来说，是否使用 `-fvisibility=hidden` 由作者的偏好决定。如果不使用此选项，需要以某种方式标记不需要导出的符号。下一节将详细讨论这些方法。如果使用了此选项，所有导出的函数都需要被声明为具有默认可见性，这通常会使头文件变得非常复杂。然而，这也意味着不会因为缺少适当的声明而意外导出符号。在某些情况下，这可以防止不良的意外发生。

### 2.2.3 Define Per-Symbol Visibility

Instead of changing the default visibility the programmer can choose to define to hide individual symbols. Or, if the default visibility is hidden, make specific symbols exportable by setting the visibility to default.

程序员可以选择隐藏特定的符号，而不是更改所有符号的默认可见性。亦或是说，如果默认可见性设置为隐藏，可以通过将特定符号的可见性设置为默认来使其可导出。

Since the C language does not provide mechanisms to define the visibility of a function or variable gcc resorts once more to using attributes:

因为 C 语言本身不具备定义函数或变量可见性的机制，所以 gcc 采用属性来实现这一点：

```c
int last __attribute__((visibility ("hidden")));

int __attribute__((visibility ("hidden"))) next (void) {
  return ++last;
}

int index (int scale) {
  return next () << scale;
}
```

This defines the variable last and the function next as hidden. All the object files which make up the DSO which contains this definition can use these symbols. I.e., while static restricts the visibility of a symbol to the file it is defined in, the hidden attribute limits the visibility to the DSO the definition ends up in. In the example above the definitions are marked. This does not cause any harm but it is in any case necessary to mark the declaration. In fact it is more important that the declarations are marked appropriately since it is mainly the code generated for in a reference that is influenced by the attribute.

这种属性声明可将变量 `last` 和函数 `next` 定义为隐藏。构成包含此定义的 DSO 的所有目标文件都可以使用这些符号。换句话说，`static` 将符号的可见性限制在定义它的文件内，而隐藏属性则将可见性限制在包含该定义的整个 DSO 内。在上面的示例中，定义已被标记。这不会造成任何损害，但无论如何都有必要对声明进行标记。事实上，适当标记声明更为重要，因为受属性影响的主要是在引用中生成的代码（编译器在生成引用这些符号的代码时，会根据声明中的可见性属性来处理符号的可见性。如果声明没有正确标记可见性属性，引用这些符号的代码可能会出现问题。）。

Instead of adding an visibility attribute to each declaration or definition, it is possible to change the default temporarily for all definitions and declarations the compiler sees at this time. This is mainly useful in header files since it reduces changes to a minimum but can also be useful for definitions. This compiler feature was also in troduced in gcc 4.0 and is implemented using a pragma:

与其为每个声明或定义添加可见性属性，不如临时更改编译器此时看到的所有定义和声明的默认值。这主要适用于头文件，因为它可以将更改减少到最低限度，但也适用于定义。这一编译器功能也在 gcc 4.0 中引入，并通过一个 pragma 来实现：

```c
#pragma GCC visibility push(hidden)

int last;

int next (void) {
  return ++last;
}

#pragma GCC visibility pop

int index (int scale) {
  return next () << scale;
}
```

As in the example using the attributes, last and next are both defined with hidden visibility while index is defined with default visibility (assuming this is the default currently in use). As the pragma syntax suggests, it is possible to nest the pragmas with the expected result.

在使用属性的例子中，`last` 和 `next` 都被定义为隐藏可见性，而 `index` 被定义为默认可见性（假设这是当前的默认值）。按照 `pragma` 语法，可以嵌套 `pragma` 指令并获得预期的结果。

In case the `-fvisibility=hidden` command line option is used, individual symbols can be marked as exportable by using the same syntax as presented in this section, except with default in place of hidden. In fact, the names of all four visibilities are allowed in the attribute or pragma.

如果使用了 `-fvisibility=hidden` 命令行选项，可以使用与本节相同的语法将单个符号标记为可导出，只需将 `hidden` 替换为 `default`。实际上，属性或 `pragma` 中允许使用所有[[#^3c87c1|]]。

Beside telling the backend of the compiler to emit code to flag the symbol as hidden, changing the visibility has an other purpose: it allows the compiler to assume the definition is local. This means the addressing of variables and function can happen as if the definitions would be locally defined in the file as static. Therefore the same code sequences we have seen in the previous section can be generated. Using the hidden visibility attribute is therefore almost completely equivalent to using static; the only difference is that the compiler cannot automatically inline the function since it need not see the definition.

除了告诉编译器后端生成标记符号为隐藏的代码之外，更改可见性还有另一个重要作用：它允许编译器将定义视为本地定义。这意味着变量和函数的寻址方式可以像它们在文件中被定义为 `static` 一样。因此，编译器可以生成与前一节中看到的相同代码序列。因此，使用隐藏可见性属性几乎完全等同于使用 `static`；唯一的区别是编译器不能自动内联函数，因为它允许符号在多个文件（构成同一个 DSO 的多个文件）中被引用，这使得编译器不确定所有引用是否都在同一个文件中。

We can now refine the rule for using static: merge source files and mark as many functions static as far as one feels comfortable. In any case merge the files which contain functions which potentially can be inlined. In all other cases mark functions (the declarations) which are not to be exported from the DSO as hidden.

现在，我们可以完善使用静态的规则：合并源文件，并尽可能多地将函数标记为静态。在任何情况下，合并包含有可能被内联的函数的文件。在所有其他情况下，将不需要从 DSO 导出的函数（声明）标记为隐藏。

（这段话的主要意思是，建议开发者通过合并源文件和适当地使用 `static` 和隐藏可见性属性来优化代码。对于可能被内联的函数，建议将它们所在的文件合并在一起。对于不需要导出的函数，建议将其声明标记为隐藏。这些方法可以帮助减少符号的数量，优化代码的性能和安全性。）

Note that the linker will not add hidden symbols to the dynamic symbol table. I.e., even though the symbol tables of the object files contain hidden symbols they will disappear automatically. By maximizing the number of hidden declarations we therefore reduce the size of the symbol table to the minimum.

请注意，链接器不会将隐藏符号添加到动态符号表中（static 关键字也同理，但是它会出现在局部符号表当中）。也就是说，尽管目标文件的符号表中包含隐藏符号，这些符号在链接过程中会自动被移除。因此，通过增加隐藏声明的数量，可以最大限度地减少符号表的大小。

The generic ELF ABI defines another visibility mode: protected. In this scheme references to symbols defined in the same object are always satisfied locally. But the symbols are still available outside the DSO. This sounds like an ideal mechanism to optimize DSO by avoiding the use of exported symbols (see section 2.2.7) but it isn’t. Processing references to protected symbols is even more expensive than normal lookup. The problem is a requirement in the ISO C standard. The standard requires that function pointers, pointing to the same function, can be compared for equality. This rule would be violated with a fast and simple-minded implementation of the protected visibility. Assume an application which references a protected function in a DSO. Also in the DSO is another function which references said function. The pointer in the application points to the PLT entry for the function in the application’s PLT. If a protected symbol lookup would simply return the address of the function inside the DSO the addresses would differ.

通用 ELF ABI 还定义了另一种可见性模式：protected。在这种模式下，同一对象中定义的符号的引用总是本地解决的，但这些符号在 DSO 之外仍然可见。这似乎是通过减少导出符号来优化 DSO 的理想方法（见第 2.2.7 节），但实际上并不是。处理对受保护符号的引用比普通查找更耗时。问题在于 ISO C 标准的一项要求。标准要求指向同一函数的函数指针可以进行相等性比较。如果受保护可见性的实现过于简单和快速，这一规则就会被违反。假设一个应用程序引用了 DSO 中的一个受保护函数，同时 DSO 内还有另一个函数也引用了这个函数。在应用程序中该函数的引用其指针指向的是应用程序 PLT 中的一个条目。如果受保护符号直接返回 DSO 内函数的实际地址，这两个地址将不一致。

In programming environments without this requirement on function pointers the use of the protected visibility would be useful and fast. But since there usually is only one implementation of the dynamic linker on the system and this implementation has to handle C programs as well, the use of protected is highly discouraged.

在没有函数指针相等性要求的编程环境中，使用 protected 可见性会非常有用且快速。但是，由于系统上通常只有一个动态链接器，并且这个链接器通常是按照 ISO C 标准去处理应用程序，故不建议使用 protected 可见性，因为这会增加处理复杂度和性能开销。

There are some exceptions to these rules. It is possible to create ELF binaries with non-standard lookup scopes. The simplest example is the use of DF SYMBOLIC (or of DT SYMBOLIC in old-style ELF binaries, see page 25). In these cases the programmer decided to create a non-standard binary and therefore accepts the fact that the rules of the ISO C standard do not apply.

然而，这些规则也有例外情况。可以创建具有非标准查找范围的 ELF 二进制文件。最简单的例子是使用 DF_SYMBOLIC（或者在旧式 ELF 二进制文件中使用 DT_SYMBOLIC，参见第 25 页）。在这种情况之下，既然程序员选择了使用非标准定义的 ELF 文件，那么也就是说，他原意承担由于标准不适导致出现未定义行为的事实。

### 2.2.4 Define Visibility for C++ Classes

For C++ code we can use the attributes as well but they have to be used very carefully. Normal function or variable definitions can be handled as in C. The extra name mangling performed has no influence on the visibility. The story is different when it comes to classes. The symbols and code created for class definitions are member functions and static data or function members. These variables and functions can easily be declared as hidden but one has to be careful. First an example of the syntax.

对于 C++ 代码，我们同样可以使用这些属性，但需要特别小心。普通函数或变量的定义可以像在 C 语言中那样处理，额外的名称混淆对可见性没有影响。不过，涉及到类时，情况就不同了。经类定义所创建的符号或代码包括了成员函数、静态数据和静态成员函数。它们可以轻松地被声明为隐藏，但必须小心处理，首先来看一个语法示例。

```c
class foo {
  static int u __attribute__ ((visibility ("hidden")));
  
  int a;

public:
  foo(int b = 1);
  
  void offset(int n);

  int val() const __attribute__ ((visibility ("hidden")));
};

int foo::u __attribute__ ((visibility ("hidden")));

foo::foo (int b) : a (b) { }

void foo::offset (int n) { u = n; }

int __attribute__ ((visibility ("hidden")))

foo::val () const { return a + u; }
```

In this example code the static data member u and the member function val are defined as hidden. The symbols cannot be accessed outside the DSO the definitions appear in. Please note that this is an additional restriction on top of the C++ access rules. For the member functions one way around the problem is to instantiate the class in more than one DSO. This is usually causing no problems and "only" adds to code bloat. Things are getting more interesting when static data members or static local variables in member functions are used. In this case there must be exactly one definition used (please note: “used”, not “present”). To obey this rule it is either necessary to not restrict the export of the static data member of member function from the DSO or to make sure all accesses of the data or function are made in the DSO with the definitions. If multiple definitions are present it is very easy to make mistakes when hiding static data members or the member functions with static variables since the generated code has no way of knowing that there are multiple definitions of the variables. This leads to very hard to debug bugs.

在这个示例代码中，静态数据成员 `u` 和成员函数 `val` 被定义为隐藏，这意味着这些符号无法在定义它们的 DSO 之外访问。这是一种比 C++ 访问规则更严格的限制。对于成员函数，一种解决方法是，在有需要的外部对象中单独定义被隐藏的成员并限于当前对象中使用，这通常不会导致问题，然而这种做法仅只是一种冗余。当涉及静态数据成员或成员函数中的静态局部变量时，问题就变得复杂了。静态成员在类范围内是唯一的，并且在所有对象实例之间共享。因此，如果多个 DSO 中存在相同静态成员的定义，可能会存在竞态条件和运行时错误。在这种情况下，必须确保只有一个定义被使用（请注意，是 "使用"，而不是 "存在"）。为了遵守这一规则，要么去除静态数据成员或成员函数的 `hidden` 定义，从而允许它们在 DSO 之外访问（这样就不需要采用前述冗余的方案了），要么确保所有对数据或函数的访问都在定义它们的 DSO 中进行。如果存在多个定义，在隐藏静态数据成员或具有静态变量的成员函数时容易出现错误，因为生成的代码无法知道有多个定义，这会导致非常难以调试的错误。

In the example code above the static data member u is declared hidden. All users of the member must be defined in the same DSO. C++ access rules restrict access only to member functions, regardless of where they are defined. To make sure all users are defined in the DSO with the definition of u it is usually necessary to avoid inline functions which access the hidden data since the inline generated code can be placed in any DSO which contains code using the class definition. The member function offset is a prime example of a function which should be inlined but since it accesses u it cannot be done. Instead offset is exported as an interface from the DSO which contains the definition of u.

在上述示例代码中，静态数据成员 `u` 被声明为隐藏。这意味着所有访问该成员的代码必须在同一个 DSO 内。C++ 的访问规则仅对成员函数的访问进行限制，而不管它们在哪里定义。为了确保所有访问 `u` 的代码都在同一个 DSO 内，通常需要避免内联函数，因为内联生成的代码可以被放置在任何包含该类定义的 DSO 中。成员函数 `offset` 是一个本应内联的函数的典型示例，但由于它访问了 `u`，所以不能这样做。相反，`offset` 被作为接口从定义了 `u` 的 DSO 中导出。

If a member function is marked as hidden, as val is in the example, it cannot be called from outside the DSO. Note that in the example the compiler allows global access to the member function since it is defined as a public member. The linker, not the compiler, will complain if this member function is used outside the DSO with the instantiation. Inexperienced or not fully informed users might interpret this problem as a lack of instantiation which then leads to problems due to multiple definitions. 

如果成员函数被标记为隐藏，如示例中的 `val`，则该函数不能在 DSO 之外被调用。请注意，示例中编译器允许全局访问成员函数，因为它被定义为公共成员。如果在 DSO 外部使用此成员函数，链接器会发出警告（不是编译器）。没有经验或了解不全面的用户可能会误认为这是由于缺少实例化，从而导致多个定义的问题。

Because these problems are so hard to debug it is essential to get the compiler involved in making sure the user follows the necessary rules. The C++ type system is rich enough to help if the implementor puts some additional effort in it. The key is to mimic the actual symbol access as closely as possible with the class definition. For this reason the class definitions of the example above should actually look like this:

由于这些问题非常难以调试，因此必须让编译器参与进来，以确保用户遵循必要的规则。C++ 的类型系统非常丰富，如果实现者愿意付出一些额外的努力，就可以利用它来帮助解决这些问题。关键是要在类定义中尽可能真实地模拟实际的符号访问。为此，上述示例的类定义实际上应该如下所示：

```c
class foo {
  static int u __attribute__ ((visibility ("hidden")));

  int a;
public:
  foo(int b = 1);

  void offset(int n);

  int val() const __attribute__ ((visibility ("hidden")));
};

class foo_ext : protected foo {
public:
  foo_ext (int b = 1) : foo (b) {}
  
  void offset (int n) { 
    return foo::offset(n);
  }
};
```

The class foo is regarded as a private class, not to be used outside the DSO with the instantiation. The public interface would be the class foo ext. It provides access to the two public interfaces of the underlying class. As long as the users of the DSO containing the definitions respect the requirement that only foo_ext can be used there is no way for the compiler not noticing accesses to foo::u and foo::val outside the DSO containing the definitions.

我们将类 foo 被视为私有类，它不应在包含定义它的 DSO 之外使用。公共接口将是类 foo_ext，它提供对底层类两个公共接口的访问。只要使用包含这些定义的 DSO 的用户遵守只使用 `foo_ext` 的要求，编译器就能够确保对 `foo::u` 和 `foo::val` 的访问仅限于定义所在的 DSO 内部，不会在 DSO 外部访问这些成员。这是一种依赖程序员自律的约定，而不是编译器强制执行的规则。

Template class and functions are not different. The syntax is the same. Non-inline function definitions get yet again less readable but that is something which can be mostly hidden with a few macros.

模板类和函数在处理方式上是相同的。虽然模板中的外部函数定义（在类的作用域之外）可能不太容易阅读，但大多数情况下可以通过使用一些宏来简化和隐藏这些复杂性。

```cpp
template<class T>
class a {
  T u;

public:
  a (T a = 0);

  T r () const __attribute__ ((visibility ("hidden")));
};

template<class T> a<T>::a (T a) { 
  u = a; 
}

template<class T> T __attribute__ ((visibility ("hidden"))) a<T>::r () const { 
  return u; 
}
```

For templatized classes the problems of making sure that if necessary only one definition is used is even harder to fix due to the various approaches to instantiation. 

对于模板化的类，由于实例化的方法多种多样，确保必要时只允许使用唯一的一种特化就显得非常困难。

One sort of function which can safely be kept local and not exported are inline function, either defined in the class definition or separately. Each compilation unit must have its own set of all the used inline functions. And all the functions from all the DSOs and the executable better be the same and are therefore interchangeable. It is possible to mark all inline functions explicitly as hidden but this is a lot of work. Since version 4.0 gcc knows about the option -fvisibility-inlines-hidden which does just what is wanted. If this option is used a referenced inline function is assumed to be hidden and an out-of-line copy of the function is marked with STV HIDDEN. I.e., if the function is not inlined the separate function created is not exported. This is a quite frequent situation at times since not all functions the programmer thinks should be inlined are eligible according to the compiler’s analysis. This option is usable in almost all situations. Only if the functions in the different DSOs can be different or if the code depends on exactly one copy of the function ever being used (e.g., if the function address is expected to be the same) should this option be avoided. If a C++ class is used only for the implementation and not used in any interface of a DSO using the code, then it would be possible to mark each member function and static data element as hidden. This is cumbersome, error prone, and incomplete, though. There is perhaps a large number of members which need to be marked and when a new member is added it is easy to forget about adding the necessary attributes. The incompleteness stems from the fact that the C++ compiler automatically generates a few members functions such are constructors and destructors. These member functions would not be affected by the attributes.

有一种函数可以安全地保留在本地而不导出，那就是内联函数，它可以在类中定义，也可以单独定义。当一个内联函数在多个编译单元中使用时，每个编译单元都需要包含这个内联函数的定义。为了确保系统的稳定性和一致性，所有动态共享对象（DSO）和可执行文件中的这些内联函数的定义应该是相同的，这样它们就可以互换使用（即这些分布在不同编译单元的内联函数其行为是一致的）。可以显式地将所有内联函数标记为隐藏，但这需要很多工作。自 gcc 4.0 版本起，引入的 -fvisibility-inlines-hidden 编译选项可以帮助自动完成这一任务。如果使用此选项，调用了使用 inline 关键字声明但未实际内联的函数符号将默认视为隐藏。即，编译器会生成该函数的独立副本，并将其标记为 STV_HIDDEN 以让其不会被导出。这种情况很常见，因为并不是所有程序员所编写的内联函数都能够符合编译器的内联标准。这个选项几乎在所有情况下都适用。如果同一个函数在不同的动态共享对象（DSO）中有不同的实现，或者需要确保函数在整个程序中只有一个唯一的副本（例如，当代码依赖于函数地址相同的时候）时，才应避免使用此选项。

如果一个 C++ 类中无需对外暴露任何接口，那么可以将每个成员函数和静态数据元素标记为隐藏。然而，这样做很繁琐，容易出错且不完全。需要标记的成员可能很多，当添加新成员时，很容易忘记添加必要的属性。此外，C++ 编译器会自动生成一些成员函数，如构造函数和析构函数，这些函数不会受到属性的影响。

The solution to these problems is to explicitly determine the visibility of the entire class. Since version 4.0 does gcc have support for this. There are two ways to achieve the goal. First, the already mentioned pragma can be used.

可以通过明确整个类的可见性来解决手动标记类成员繁琐且容易出错的问题。从 gcc 4.0 版本开始，已经支持这种功能。实现这一目标的方法有两种，其中之一是使用之前提到的 pragma 指令。

```cpp
#pragma GCC visibility push(hidden)

class foo {
...
};

#pragma GCC visibility pop
```

All member functions and static data members of foo are automatically defined as hidden. This extends even to implicitly generated functions and operators if necessary.

foo 类的所有成员函数和静态数据成员都会被自动设定为隐藏。这甚至包括编译器自动生成的函数和操作符。

The second possibility is to use yet another extension in gcc 4.0. It is possible to mark a function as hidden when it is defined. The syntax is this:

第二种方法是利用 gcc 4.0 中的另一个扩展功能，可以在定义函数时直接将其标记为隐藏。其语法如下：

```cpp
class __attribute ((visibility ("hidden"))) foo {
...
};
```

Just as with the pragma, all defined functions are defined as hidden symbols. Explicitly using attributes should be preferred since the effect of the pragmas is not always obvious. If the push and pop lines are far enough from each other a programmer might accidentally add a new declaration in the range even though the visibility of this new declaration is not meant to be affected. Both, the pragma and the class attribute, should only be used in internal headers. In the headers which are used to expose the API of the DSO it makes no sense to have them since the whole point is to hide the implementation details. This means it is always a good idea to differentiate between internal and external header files.

与 pragma 一样，所有定义的函数都会被隐藏。需要注意的是，显式使用属性更可取，因为 pragma 的效果有时不明显。如果 push 和 pop 行相隔太远，程序员可能会在这段范围内意外添加一个本应不被隐藏的声明，从而错误的影响其可见性。无论是 pragma 还是类属性，都应该只在内部头文件中使用。在用于暴露 DSO API 的头文件中使用它们没有意义，因为我们的目的是为了隐藏实现细节。因此，在项目架构中区分内部和外部头文件是个不错的做法。

Defining entire classes with hidden visibility has some problems which cannot be modeled with sophisticated class layout or moving the definition in private headers. For exception handling the compiler generates data structures (typeinfo symbols) which are also marked according to the visibility attribute used. If an object of this type is thrown the catch operation has to look for the typeinfo information. If that information is in a different DSO the search will be unsuccessful and the program will terminate. All classes which are used in exception handling and where the throw and catch are not both guaranteed to reside in the DSO with the defi-nition must be declared with default visibility. Individual members can still be marked with an visibility attribute but since the typeinfo data is synthesized by the compiler on command there is no way for the programmer to overwrite a hidden visibility attribute associated with the class. 

用隐藏可见性定义整个类会引发一些问题，这些问题无法通过复杂的类布局或将定义移到私有头文件中来解决。对于异常处理，编译器会生成数据结构（typeinfo 符号），这些符号的可见性也会受到属性设置的影响。如果抛出的对象类型的 typeinfo 信息位于不同的 DSO 中，catch 操作将无法找到该信息，导致程序终止。因此，所有用于异常处理的类，如果不能保证 throw 和 catch 都在同一个 DSO 中，就必须使用默认可见性。尽管可以为单个成员设置可见性属性，但由于 typeinfo 数据是由编译器生成的，程序员无法更改类的隐藏可见性属性。

The use of the most restrictive visibility possible can be of big benefit for C++ code. Each inline function which is (also) available as a stand-alone function, every syn-thesized function are variable has a symbol associated which is by default exported. For templatized classes this is even worse, since each instantiated class can bring is many more symbols. It is best to design the code right away so that the visibility attributes can be applied whenever possible. Compatibility with older compilers can easily be achieved by using macros.

为 C++ 代码应用最严格的可见性设置能够带来显著的好处。每个可作为独立函数存在的内联函数、每个自动生成的函数和变量，都会有一个默认导出的符号。对于模板类来说，情况更为复杂，因为每个实例化都会引入更多符号。因此，从一开始就设计好代码，以便随时应用可见性属性是最好的做法。通过使用宏，还可以轻松确保与旧版编译器的兼容性。

### 2.2.5 Use Export Maps

If for one reason or another none of the previous two solutions are applicable the next best possibility is to instruct the linker to do something. Only the GNU and Solaris linker are known to support this, at least with the syntax presented here. Using export maps is not only useful for the purpose discussed here. When discussing maintenance of APIs and ABIs in chapter 3 the same kind of input file is used. This does not mean the previous two methods should not be preferred. Instead, export (and symbol) maps can and should always be used in addition to the other methods described.

如果出于某种原因前两种解决方案都不可行，最好的替代方法是指示链接器执行某些操作。目前已知只有 GNU 和 Solaris 链接器支持这种操作，至少是用这里介绍的语法。使用导出映射（export maps）不仅在当前讨论的场景中有用，在第 3 章讨论 API 和 ABI 维护时也会用到相同类型的输入文件。不过，这并不意味着前两种方法不应该优先考虑。相反，导出映射应该始终作为其他方法的补充一起使用。

The concept of export maps is to tell the linker explicitly which symbols to export from the generated object. Every symbol can belong to one of two classes: exported or not exported. Symbols can be listed individually, glob-expressions can be used, or the special `*` catch-all glob expression can be used. The latter only once. The symbol map file for our example code could look like this:

导出映射的概念是明确指定链接器应从生成的对象文件中导出哪些符号。每个符号可以分为两类：导出和不导出。可以单独列出符号，使用通配符表达式，或者使用特殊的 `*` 通配符表达式，但后者只能使用一次。以下是我们示例代码的符号映射文件示例：

```c
{
  global: index;
  local: *;
};
```

This tells the linker that the symbol index is to be exported and all others (matched by `*`) are local. We could have listed last and next explicitly in the local: list but it is generally advised to always use the catch-all case to mark all not explicitly mentioned symbols as local. This avoids surprises by allowing access only to the symbols explicitly mentions. Otherwise there would also be the problem with symbols which are matched neither by the global: nor by the local:, resulting in unspecified behavior. Another unspecified behavior is if a name appears in both lists or is matched using globbing in both lists. 

这段代码告诉链接器要导出 `index` 符号，而所有其他符号（由 `*` 匹配的）都设置为本地符号。尽管我们可以在 local: 列表中显式列出 `last` 和 `next`，但通常建议使用通配符 `*` 来标记所有未明确提到的符号为本地符号。这种做法可以避免意外情况，只允许访问明确指定的符号。否则，既不匹配 global: 也不匹配 local: 的符号会导致行为未定义。如果一个符号同时出现在两个列表中或被两个列表中的通配符匹配到，也会导致未定义行为。

To generate a DSO with this method the user has to pass the name of the map file with the `--version-script` option of the linker. The name of the option suggests that the scripts can be used for more. We will get back to this when we discuss ABIs in the next chapter.

要使用这种方法生成 DSO，用户需要在链接器的 `--version-script` 选项中指定映射文件的名称。简单插入一段：通过选项名称可知这些脚本有更多用途。这一部分我们将在下一章讨论 ABI 时进一步探讨这一点。

```
$ gcc -shared -o foo.so foo.c -fPIC -Wl,--version-script=foo.map
```

The file `foo.map` is supposed to contain text like the one shown above.

文件 `foo.map` 应包含类似上面展示的内容。

It is of course also possible to use export maps with C++ code. One has two options in this case: explicitly name the symbols using their mangled names, or rely on pattern matching for the mangled names. Using the mangled names is straight-forwarded. Just use the identifiers as in the C examples. Using the demangled names require support in the linker. Assume a file defining the following functions:

导出映射也可以用于 C++ 代码。在这种情况下，有两种方法：显式使用符号的混淆名称，或依赖模式匹配修饰名称。使用混淆名称的方法很简单，只需像在 C 示例中那样使用标识符。而使用去混淆名称则需要链接器的支持。假设有一个文件定义了以下函数：

```c
int foo (int a) { ... }
int bar (int a) { ... }
struct baz {
  baz (int);
  int r () const;
  int s (int);
};
```

A DSO containing definitions for all these functions and members should only export the function foo and the destructor(s) of `baz` and `baz::s`. An export map to achieve this could look like this:

```
{
  global:
    extern "C++" {
      foo*;
      baz::baz*;
      baz::s*
  };
  local: *;
};
```

The use of `extern "C++"` tells the linker to match the following patterns with demangled C++ names. The first entry `foo*` matches the first global function in the example. The second entry matches the constructor(s) of `baz` and the third entry matches the function `baz::s`. Note that patterns are used in all cases. This is necessary since `foo`, `baz::baz`, and `baz::s` are not the complete names. The function parameter are also encoded in the mangled name and must be matched. It is not possible to match complete demangled C++ names since the current linker implementation refuses to allow non-alphanumeric characters. Using pattern might have unwanted effects. If there is another member function in baz starting with the letter 's' it will also be exported. And one last oddity should be mentioned: currently the linker requires that there is no semicolon after the last entry in the C++ block.

Using export maps seems like a very desirable solution. The sources do not have to be made less readable using attribute declarations or eventually pragmas. All the knowledge of the ABI is kept locally in the export map file. But this process has one fundamental problem: exactly because the sources are not modified the final code is not optimal. The linker is used only after the compiler already did its work and the once generated code cannot be optimized significantly.

In our running example the compiler must generate the code for the `next` function under the worst case scenario assumption that the variable `last` is exported. This means the code sequence using `@GOTOFF` which was mentioned before cannot be generated. Instead the normal two instruction sequence using `@GOT` must be generated.

This is what the linker will see when it gets instructed to hide the symbol `last`. The linker will not touch the actual code. Code relaxation here would require substantial analysis of the following code which is in theory possible but not implemented. But the linker will not generate the normal `R_386_GLOB_DAT` relocation either. Since the symbol is not exported no interposition is allowed. The position of the local definition relative to the start of the DSO is known and so the linker will generate a relative relocation.

For function calls the result is often as good as it gets. The code generated by the compiler for a PC-relative jump and a jump through the PLT is identical. It is just the code which is called (the target function versus the code in the PLT) which makes the difference. The code is only not optimal in one case: if the function call is the only reason the PIC register is loaded. For a call to a local function this is not necessary and loading the PIC is just a waste of time and code.

To summarize, for variables the use of symbol maps creates larger and less efficient code, adds an entry in the GOT, and adds a relative relocation. For functions the generated code sometimes contains unnecessary loads of the PIC. One normal relocation is converted into a relative relocation and one PLT entry is removed. This is one relative relocation worse than the previous methods. These deficiencies are the reason why it is much preferable to tell the compiler what is going on since after the compiler finished its work certain decisions cannot be reverted anymore.

# 4. Appendix

## A Counting Relocations

The following script computes the number of normal and relative relocations as well as the number of PLT entries present in a binary.  If an appropriate readelf implementation is used it can also be used to look at all ﬁles in an archive. If prelink is available and used, the script also tries to provide information about how often the DSO is used. This gives the user some idea how much “damage” an ill-written DSO causes.

```perl
#! /usr/bin/perl

# ubuntu 22.04; g++-11

eval "exec /usr/bin/perl -S $0 @ARGV"
  if 0;

# Copyright (C) 2000, 2001, 2002, 2003, 2004, 2005 Ulrich Drepper
# Written by Ulrich Drepper <drepper@gmail.com>, 2000.

# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software Foundation,
# Inc., 59 Temple Place - Suite 330, Boston, MA 02111-1307, USA. */

for ($cnt = 0; $cnt <= $#ARGV; ++$cnt) {
  $relent = 0;
  $relsz = 0;
  $relcount = 0;
  $pltrelsz = 0;
  $extplt = 0;
  $users = 0;
  
  open(READLINK, "readlink -f $ARGV[$cnt] |") || die "cannot open readlink";
  while (<READLINK>) {
    chop;
    $fullpath = $_;
  }
  close(READLINK);

  open(READELF, "eu-readelf -d $ARGV[$cnt] |") || die "cannot open $ARGV[$cnt]";
  while (<READELF>) {
    chop;
    if (/.*RELAENT\s+([0-9]+).*/) {
      $relent = $1 + 0;
    } elsif (/.*RELASZ\s+([0-9]+).*/) {
      $relsz = $1 + 0;
    } elsif (/.*RELACOUNT\s+([0-9]+).*/) {
      $relcount = $1 + 0;
    } elsif (/.*PLTRELSZ\s+([0-9]+).*/) {
      $pltrelsz = $1 + 0;
    }
  }
  close(READELF);

  open(READELF, "eu-readelf -r $ARGV[$cnt] |") || die "cannot open $ARGV[$cnt]";
  while (<READELF>) {
    chop;
    if (/.*JUMP_SLOT\s+0x[0-9a-fA-F]+.*/) {
      ++$extplt;
    }
  }
  close(READELF);

  if (open(PRELINK, "prelink -p 2>/dev/null | fgrep \"$fullpath\" |")) {
    while (<PRELINK>) {
      ++$users;
    }

    close(PRELINK);
  } else {
    $users = -1;
  }

  printf("%s: %d relocations, %d relative (%d%%), %d PLT entries, %d for local syms (%d%%)",
    $ARGV[$cnt], 
    $relent == 0 ? 0 : $relsz / $relent, $relcount,
    $relent == 0 ? 0 : ($relcount * 100) / ($relsz / $relent),
    $relent == 0 ? 0 : $pltrelsz / $relent,
    $relent == 0 ? 0 : $pltrelsz / $relent - $extplt,
    $relent == 0 ? 0 : ((($pltrelsz / $relent - $extplt) * 100) / ($pltrelsz / $relent)));

  if ($users  >=  0) {
    printf(", %d users", $users);
  }

  printf("\n");
}
```