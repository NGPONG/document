# Git
---

<br/>

## 关于Git介绍

##### 1. Git的诞生
`Linux` 之父 `Linus` 当时为了解决社区代码版本控制的问题，使用 C 语言花费了大概两周时间所编写出来的分布式版本控制系统，这套系统就是 `Git`
##### 2. 分布式控制系统的优点
- 分布式版本控制系统根本没有中央服务器，每个人的电脑上都是一个完整的版本库，这样，你工作的时候，就不需要联网了，因为版本库就在你自己的电脑上  
> 虽然是可以通过Git来达到”分布式版本控制系统没有中央服务器“这一概念，但在实际的开发过程当中使用Git的时候都会有一台电脑充当一个中央服务器，它就是 [GitHub](https://github.com/ "Go!")
- 分布式版本控制系统的安全性性能相较于集中式版本控制系统，其安全性能要高得多，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，随便从其他人那里复制一个就可以了
##### 3. 分布式管理系统的工作方式

<br/>

![0.jpg](https://i.loli.net/2019/07/12/5d27f53875c0148819.jpg)

<br/>

##### 4. Git的工作方式

<br/>

![bg2015120901.png](https://i.loli.net/2019/07/12/5d283cd23a90693337.png)

<br/>

## Git的使用方式

##### 1. 如何安装Git
在Windows系统上使用Git，可以直接从Git的官方网站[下载安装程序](https://git-scm.com/downloads)

##### 2. 使用前的初始化
当我们安装完成Git后，我们可以在任一文件夹下鼠标邮件，点击`Git Bush Here`即可开始操作Git
- 创建一个文件夹：
`$ mkdir Directory Name`
- 定位到当前文件夹：
`$ cd Directory Name`
- 初始化Git，构建Repository：
`$ git init`  

##### 3. 基础配置
- 设置提交人：
`$ git config --global user.name "Your Name"`
- 设置提交人的Email地址：
`$ git config --global user.email "email@example.com"`
- 设置Remote Repository的地址：
`$ git remote set-url origin Remote Repository URL`
- 设置Git使用SS代理http协议：
`$ git config --global http.proxy http://127.0.0.1:1080`
- 设置Git使用SS代理https协议：
`$ git config --global https.proxy https://127.0.0.1:1080`
- 取消http协议的代理设置：
`$ git config --global --unset http.proxy`
- 取消https协议的代理设置：
`$ git config --global --unset https.proxy`
- 配置用户名和密码的存储方式：
  - cache：登陆凭证存放在内存中，15分钟后清楚
  - store：登陆凭证用明文的形式存放在磁盘中，永不过期  
  
  `$ git config --global credential.helper store/cache`

##### 4. 提交
- 添加文件至本地暂存区：
  - 可以指定具体的文件名，也可以通过 `.` 来指定当前工作区下的所有文件都提交至暂存区中

  `$ git add FileName/.`
- 把暂存区的文件提交至本机的Repository：
`$ git commit -m "Commit Message"`

##### 5. 复位
- 复位工作区文件/暂存区/本机Reposttory至指定的版本
  - `$ git reset --hard [commitId]`：复位工作区文件、暂存区数据和本机Repository的版本指针到本机Repository的最新版本或指定`commitId`的版本
  - `$ git reset [commitId]`：复位暂存区数据和本机Repository的版本指针到本机Repository的最新版本或指定`commitId`的版本
  - `$ git reset --keep [commitId]`：复位本机Respository的版本指针到本机Repository的最新版本或指定`commitId`的版本


##### 6. 差异和检查
- 检查当前工作区的状态
  - 查看当前工作区哪些文件被修改了但是还未`add`至暂存区
  - 查看当前工作区哪些文件已经`add`了但是还未`commit`到本机的Repository
  - 查看当前工作区哪些文件已经`commit`了但是还未合并到远程Repository的分支下
  
  `$ git status`
  ![微信截图_20190712154521.png](https://i.loli.net/2019/07/12/5d283ab06bcd044382.png)
- 检查差异
    - 查看当前工作区与暂存区之间的差异
    `$ git diff`
    - 查看暂存区与本机Repostiory之间的差异
    `$ git diff --cached`
    - 查看工作区与本机Repostiory最新`commit`的差异
    `$ git diff HEAD`
    - 查看工作区与本机Repostiory指定版本`commit`的差异
    `$ git diff VersionId`
- 显示版本日志
    - `$ git log`：显示已经`commit`到本机Repostiory的完整版的日志格式
    - `$ git log --pretty=oneline`：显示已经`commit`到本机Repostiory的简化版的日志格式
- 显示历史命令记录
`$ git reflog`

##### 7. Git中的一些概念
- 在Git中除了可以使用版本号来指定版本外，还可以通过一些特殊的符号来表示
    - `HEAD`：表示最新的版本
    - `HEAD^`：表示最新版本的上一个版本，如果想指定在最新版本之上的某个具体的版本，也可以通过多个`^`来实现，例如 `HEAD^`/`HEAD^^`/`HEAD^^^`
    - `HEAD~num`：表示最新版本之上多少个版本，例如我想表示最新版本的上100个版本则为 `HEAD~100`
