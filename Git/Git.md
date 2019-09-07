# Git

<br/>

### 关于Git介绍

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

---

<br/>

### Git的使用方式

##### 1. 如何安装Git
在Windows系统上使用Git，可以直接从Git的官方网站[下载安装程序](https://git-scm.com/downloads)

##### 2. 使用前的初始化
当我们安装完成Git后，我们可以在任一文件夹下鼠标邮件，点击`Git Bush Here`即可开始操作Git
- 创建一个文件夹：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ mkdir [Directory Name]`
- 定位到当前文件夹：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ cd [Directory Name]`
- 初始化Git，构建本机的Repository：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git init`  

##### 3. 基础配置
- 设置提交人：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global user.name "Your Name"`
- 设置提交人的Email地址：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global user.email "email@example.com"`

- 设置Git使用SS代理http协议：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global http.proxy http://127.0.0.1:1080`
- 设置Git使用SS代理https协议：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global https.proxy https://127.0.0.1:1080`
- 取消http协议的代理设置：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global --unset http.proxy`
- 取消https协议的代理设置：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global --unset https.proxy`
- 配置用户名和密码的存储方式：
  - cache：登陆凭证存放在内存中，15分钟后清楚
  - store：登陆凭证用明文的形式存放在磁盘中，永不过期  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git config --global credential.helper store/cache`

##### 4. 远程
- 添加Repository相关联的Remote
  - ShortName：远程仓库命名，可自定义，默认为 `origin` ，如果自定义命名则每次进行远程同步操作的时候都需要指定自己自定义的Remote的命名
  - Url：Remote的地址

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git remote add [ShortName] [Url]`
- 设置Remote关联地址

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git remote set-url origin [Url]`
- 把Remote克隆到本机上，下载一个项目和它的整个历史代码

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git clone [Url]`
- 查看Repository所指定的Remote

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git remote -v`
- 查看指定ShortName的Remote的具体信息

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git remote show [ShortName]`

##### 5. 分支
- 查看Repository所存在的分支，和 `HEAD` 目前所指向的分支

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git branch`
- 为当前Repository创建一个分支

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git branch [branchName]`
- 把`HEAD`指针的指向切换到目标分支，并把工作区的文件内容同步成目标分支上目前所处理的内容

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout [branchName]`
- 创建并切换到该分支，为上面两个步骤的一种简写形式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout -b [branchName]`
- 合并指定分支所 `commit` 的内容到当前分支上
    - 在合并分支的过程当中，可能会遇到无法合并的问题，如下：
        ```
        Auto-merging readme.txt
        CONFLICT (content): Merge conflict in readme.txt
        Automatic merge failed; fix conflicts and then commit the result.
        ```
    - 出现这种情况的原因是因为当前分支和所合并的分支出现了冲突合并的情况，这时候我们输入 `$ git status` 进行状态查看的时候可以看到所谓造成冲突的一些文件，Git会把合并失败的结果修改在文件内部，需要我们<span style="color:red;">手动的在文件中整合需要合并的内容</span>，然后在提交至暂存区、 `commit` 至本地分支上才能够和并成功
    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git merge [branchName]`
- 删除指定分支

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git branch -d [branchName]`



##### 6. 提交
- 添加文件至本地暂存区
  - 可以指定具体的文件名，也可以通过 `.` 来指定当前工作区下的所有文件都提交至暂存区中

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git add [FileName/.]`
- 把暂存区的文件提交至Repository的`HEAD`所指向的分支上

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git commit -m "Commit Message"`
- 把本地分支 `BranchName` 的内容推送到远程仓库上去

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git push [RepositoryName] [BranchName]`
##### 7. 删除
- 删除本地暂存区当中的指定文件并把删除结果映射至工作目录中
    - 执行的时候会优先先检查暂存区中是否包含此文件，如果此文件还未 `add` 至暂存区中则无法删除
    - 在做完上面所说的检查后还会检查当前分支的信息，如果当前所删除的文件没有往当前分支上进行过任何 `commit` 则无法删除
    
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git rm [FileName]`
- 删除本地暂存区的所有文件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git rm --cached -r .`
- 删除本地暂存区的指定文件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git rm --cached [FileName]`


##### 8. 复位
- 复位工作区文件、暂存区数据和 `HEAD` 指针指向到到当前分支的 `commitId` 的版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git reset --hard [commitId]`
- 复位暂存区数据和 `HEAD` 指针指向到到当前分支的 `commitId` 的版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git reset [commitId]`
- 复位 `HEAD` 指针指向到到当前分支的 `commitId` 的版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git reset --keep [commitId]`

##### 9. 撤销
- 把暂存区中的指定文件恢复到工作区中：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout [filename]`
- 把暂存区中的所有文件恢复到工作区中：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout [.]`
- 把本地分支指定版本的所有文件恢复至工作区和暂存区当中

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout [commitId] [fileName/.]`

##### 10. 差异和检查
- 检查当前工作区的状态
  - 查看当前工作区哪些文件被修改了但是还未 `add` 至暂存区
  - 查看当前工作区哪些文件已经 `add` 了但是还未 `commit` 到当前分支上
  - 查看当前工作区哪些文件已经 `commit` 了但是还未合并到Remote的分支下
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git status`
  
 ![微信截图_20190712154521.png](https://i.loli.net/2019/07/12/5d283ab06bcd044382.png)
  
- 检查差异
Git的差异检测是反向检测的，什么意思呢，比如说我们工作区新加了一个文件，但是没有加入到暂存区，这时候检查不同的时候是不会记录的，除非我们把这个文件`add`至暂存区后，我们再手动的删除了工作区中的这个文件，才会检测到不同，简而言之，<span style="color:red;font-weight:bold;">检测不同的机制是以上一级为准的，我上一级没有的自然不会检查你下一级的这个文件，但是我上一级有的但是下一级没有，则会成功的检测出差异</span>
    - 查看当前工作区与暂存区之间的差异
    
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git diff`
    - 查看工作区与当前分支指定版本的差异
    
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git diff [VersionId]`
    - 查看暂存区与当前分支最新`commit`的版本之间的差异
    
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git diff --cached`
    - 查看工作区与当前分支最新`commit`的版本之间的的差异
    
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git diff HEAD`
    - 检查工作区/暂存区与当前分支存在差异但还未`commit`的文件
    
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git checkout --`
- 显示版本日志
    - 显示已经`commit`到当前分支的完整版的日志格式

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git log`
    - 显示已经`commit`到当前分支的简化版的日志格式

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git log --pretty=oneline`
- 显示历史命令记录

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git reflog`
- 显示分支合并记录
    - 显示完整版本的分支合并记录

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git log --graph`
    - 显示简化版本的分支合并记录

    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$ git log --graph --pretty=oneline --abbrev-commit`

##### 11. Git中的一些概念
> 7.1 本机Git工作区的构成
![bg2015120901.png](https://i.loli.net/2019/09/07/fqKXWDecx5mF2Sh.png)
- 工作区 `Work Space` ：当前所操作Git的文件夹就是一个工作区
- 版本库 `Repository` ：在工作区中有一个隐藏目录`.git`，在我们使用初始化操作 `$ git init` 的时候就会为我们构建这么一个目录，这个目录就是 <span style="color:red;font-weight:bold;">本机</span> 的一个仓库
- 远程仓库 `Remote` ：Repository的最终指向则为远程仓库，Remote为Repository提供了 `pull` ，Repository为Remote提供了 `push` ，简而言之，远程仓库就是我们所同步并保存在 `GitHub` 上的代码仓库
- 暂存区 `index` ：在版本库中有一个暂存区，用于暂存我们所 `add` 的数据，暂存区中的数据等待我们 `commit` 到当前分支上
- 分支 `branch` ：暂存区中所 `commit` 的数据就是提交至本机当前所操作的branch身上，我们可以为Repository构建多个branch，不同的branch间所操作的版本和代码互不影响，它们之间只同步在新建当前分支前所对应的之前的版本，在此之后的版本互相对应着自己的工作版本
<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img src = "https://i.loli.net/2019/07/23/5d3678d37bb2910714.png" width = "60%" height = "60%">
<br/>
- `HEAD` ：HEAD是一个指针，它指向了Repository所操作的branch，更具体的来说它是指向了当前branch上的版本号记录(<span style="color:red">每一次提交到当前分支上的修改都会为当前分支生成一个最新的、相对应的版本号记录</span>)，在 `Git` 中，我们可以通过HEAD关键字来表示当前分支上的版本信息
    - `HEAD`：表示当前分支的最新的版本
    - `HEAD^`：当前指针上一个版本，如果想表示其上一个版本可以通过关键字`^`来实现，当然我们可以指定多个 `^` ，例如：`HEAD^`/`HEAD^^`/`HEAD^^^`
    - `HEAD~num`：表示最新版本之上多少个版本，例如我想表示最新版本的上100个版本则为 `HEAD~100`
