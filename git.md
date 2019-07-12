## GitHub克隆仓库到本地

克隆项目有两个选项

1. HTTPS

   需要每次对此操作时输入用户名密码

2. SSH

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712120604473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNjA1OTY4,size_16,color_FFFFFF,t_70)

   

   只需要第一次输入，使用SSH Key，没有权限就是没有SSH Key，需要在本机生成一个和GitHub关联的SSH Key

   [生成地址](https://help.github.com/en/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

   生成后，可以看到生成的随机证书保存在 /c/Users/weiao/.ssh/下面

   ![1562904813294](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1562904813294.png)

   生成SSH Key后就需要把它添加到GitHub的SSH Key里去

   [在GitHub进行绑定](https://help.github.com/en/articles/adding-a-new-ssh-key-to-your-github-account)

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712122429432.png)

   此命令可以将生成的SSH Key进行拷贝

   然后在GitHub的Setting中配置SSH Key就OK了

进行克隆就会成功了。

## Git常用操作

### 清除屏幕

` $ git cls`

### 克隆

` $ git clone xxx [name]`

### Git工作流程

![img](https://www.runoob.com/wp-content/uploads/2015/02/git-process.png)

### Git的一些基本概念

![img](https://www.runoob.com/wp-content/uploads/2015/02/1352126739_7909.jpg)

* 可以通过git init 初始化一个git工作空间，git工作空间就是可以看到的文件夹

* 可以通过git add [.] /[filename]将在工作空间添加的文件加入暂存区
* 可以通过commit提交到远端
* 可以通过reset回退到指定版本库

### 设置用户名和邮箱

`$ git config --global user.name "xxx"`

`$ git config --global user.email "xxxx"`

### 查看状态

`$ git status                                                                    `

### 查看版本

`$ git --version                                                                 `

### 提交

`$ git commit -m "xxx"                                                         `

### 提交日志

`$ git log                                                                       `

 --oneline 选项来查看历史记录的简洁的版本

可以用 --reverse 参数来逆向显示所有日志

```
git log --reverse --oneline
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712132254297.png)

commit 后的提交序列号，可以使用序列号进行版本回退

### 显示提交信息

`$ git show 序列号                             `

![1562909215590](C:\Users\weiao\AppData\Roaming\Typora\typora-user-images\1562909215590.png)

### 版本回退到暂存区

`$ git reset  序列号 `

## Git多人协作

一个主资源，可能会被多个人使用，他们同时提交相同的选项时，只会有第一个才会提交上去，其余的会显示发生冲突（conflict）必须先解决冲突才能进行提交

### 分支管理

[为什么分支，分支的好处](https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424)

#### 创建分支

```
git branch (branchname)
```

#### 切换分支

```
git checkout (branchname)
```

#### 创建并切换

```
git checkout -b (branchname)
```

#### 合并分支

将当前分支合并到主分支上去，一般合并完就将它删除

```
git merge xxx
```

#### 列出分支

```
git branch
```

#### 删除分支

```
git branch -d (branchname)
```

### 提交冲突及解决

发生冲突的时候会在master中出现，需要手动的对这些冲突进行解决，直接进入冲突文件，删除掉冲突文件。







