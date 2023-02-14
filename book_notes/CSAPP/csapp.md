# *Computer Systems: A Programmer's Perspective*


<br/>


## Chapter <font color = "red">1</font> A Tour of Computer Systems

### 了解系统硬件的组成

---

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="https://raw.githubusercontent.com/NGPONG/Blog/master/img/2023-02-14-00-18-51.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">近期Intel系统产品族的硬件组成模型</div>
</center>

<br/>

#### 1. 总线 $(bus)$

穿整个系统的是一组电子管道，称作总线 $(bus)$，它携带字节信息并负责在各个组件之间传递，通常总线被设计成传送 **定长** 的字节块，也就是字 $(word)$

> **字通常描述一个字节数，也可称为 字长，它是一个基本的系统参数，在各个系统中都不尽相同，现在的大多数机器字长要么是4个字节(32bit)， 要么是8个字节(64bit)**

#### 2. I/O设备 $(I/O \ Devices)$

I/O(输入/输出)设备是系统与外部世界的联系通道

在