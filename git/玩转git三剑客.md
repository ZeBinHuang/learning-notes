## Git基础

### 安装git

查看安装是否成功：在命令行中输入git version



### 最小配置

配置用户信息

```
git config --global user.name 'hzb'
git config --global user.email '648164883@qq.com'
```

config的三个作用域

```
git config --local      #local只对某个仓库有效
git config --global		#global对当前用户所有仓库有效
git config --system		#system对系统所有登录的用户有效
```

显示config的配置，加--list

```
git config --list --local
git config --list --global
git config --list --system
```

清除

```
git config --unset --global user.name
```

优先级

```
local > global > system
```



### 建git仓库

两种场景：

1、把已有的代码纳入git管理

```
cd 项目代码所在的文件夹
git init
```

2、新建的项目直接用git管理

```
cd 某个文件夹
git init your_project  #会在当前路径下创建和项目名称同名的文件夹
cd your_project
```

创建第一个仓库并配置local用户信息

```
git init git_learning
git config --local user.name 'hzb'
git config --local user.email '648164883@qq.com'
```



### 认识工作区和暂存区

往仓库里添加文件

```
工作目录 git add files-> 暂存区 git commit-> 版本历史
```

修改后的文件可以直接使用 git add -u

查看暂存区文件状态：git status



 ### 修改文件名

步骤：

```
mv readme readme.md
git add readme.md
git rm readme
以上三步可以用一步代替：git mv readme readme.md
```



### 查看版本演变历史

```
git log 
	--n4 
	--oneline
	--all
	--graph
git branch -av  #查看所有分支
git checkout -b temp #新建分支
git checkout master  #切换到master分支
git help --web log #打开帮助文档的网页
gitk  #图形界面
```



### .git目录

```
HEAD:当前指向的分支
config:保存有用户信息、关联的仓库等等信息


```



### 三个对象的关系

```
commit
tree:文件夹
blob:文件

git cat-file -p hash值  #查看内容
git cat-file -t hash值  #查看类型
```



### 分离头指针的注意事项

一个commit没有和某一个分支绑定在一起，会导致这个commit可能会被git当作垃圾清理

所以分离头指针时进行了commit之后，如果要切换到某一个分支，为了防止丢失，要将分离头指针情况下的commit绑定到具体的分支上

分离头指针的使用情况：临时想基于某个commit做变更，试试新方案是否可行，就可以采用分离头指针的方式



## 在本地独自使用Git时的常见场景

### 删除不需要的分支

```
git branch -d 分支名
用-d进行删除之前，会先判断当前分支是否有合并到其他分支上，若没有，会有提示信息
git branch -D 分支名
用-D进行强制删除
git merge 分支名  #将分支名合并到当前分支
```



### 修改最新commit的message

```
git commit --amend
```



### 修改老旧commit的message

```
git rebase -i 要修改的commit的父commit
```



### 把连续多个commit整理成一个

```
git rebase -i 要合并的commit中第一个祖先commit的父commit
```



### 把间隔的几个commit整理成一个

```
git rebase -i 要合并的commit中第一个祖先commit的父commit
```



### 比较HEAD和暂存区所含文件的差异

```
git diff --cached
git diff --staged
```



### 比较暂存区和工作区所含文件的差异

```
git diff  #默认比较暂存区和工作区
git diff -- 文件名   #只比较某一个文件
```



### 让暂存区恢复成和HEAD一样

```
git reset HEAD  #丢弃暂存区中所有文件的更改
git diff --cached  #查看是否恢复成功
git reset --hard  #工作区和暂存区的内容都变回HEAD对应的内容
```



### 让工作区的文件恢复成和暂存区一样

```
git checkout 文件名 
```



### 取消暂存区中部分文件的更改

```
git reset HEAD 文件名
```



### 消除最近几次的提交

```
git reset --hard 祖先commit的id  #会同时把工作区和暂存区的内容也消除，慎用！
```



### 查看不同提交的指定文件的差异

```
git diff 分支1 分支2 -- 文件名
```



### 正确删除文件的做法

```
git rm 文件名
```



### 开发中临时加塞紧急任务

```
git stash   #将当前工作区的内容存起来
git stash list   #查看存起来的堆栈信息
git stash apply  #取出栈顶的内容，但stash中的内容没有删除
git stash pop	 #取出栈顶的内容，同时删除stash中的内容
```



### 指定不需要Git管理的文件

```
创建.gitignore文件，在里面指定不需要管理的文件
```



### Git仓库备份到本地

```
#备份仓库
git clone --bare file:///c/Users/64816/Desktop/java-study/git/101-GitRunner/git_learning/.git zhineng.git
#原仓库和备份仓库关联起来
git remote add zhineng file:///c/Users/64816/Desktop/java-study/git/101-GitRunner/zhineng.git
#在原仓库建立一个新分支
git checkout -b temp
#将新分支push到备份仓库
git push --set-upstream zhineng temp
```



## Git与GitHub的简单同步

### 注册一个GitHub账号



### 配置公私钥



### 在GitHub上创建个人仓库



### 把本地仓库同步到GitHub

fast-forward:远端分支是本地分支的祖先

```
#默认clone或add的名字是origin
git remote add github git@github.com:ZeBinHuang/git_learning.git
git push github --all
#远端仓库中有我们本地仓库中没有的东西，需要先fetch远端仓库的东西
git fetch github master
#将本地仓库和远端仓库的分支变成fast forward关系
git merge --allow-unrelated-histories github/master
#push之后远端分支的指针会变成和本地指针的指向相同
git push github master
```



## 多人协作

### 不同人在同一分支修改不同文件

```
#在远端仓库创建分支，本地克隆远端仓库后，需要基于远端仓库创建的分支在本地自己创建这个分支，然后自动切换到这个分支
git checkout -b 分支名 远端仓库的分支

```



### 不同人在同一分支修改同一文件的不同区域

```
#每次操作前最好先和远端仓库保持同步
git pull
#其中一个人修改文件后push到远端仓库，这时另一个人不知道文件已修改，当要将修改文件push到远端时，被拒绝
#这时两个人基于同一文件的修改导致了远端分支和本地分支不是fast-forward关系
#所以决解方法是先将远端分支拉取到本地，再将本地修改push到远端
```



### 不同人在同一分支修改同一文件的同一区域

```
#出现冲突需要自行决定如何修改，确认修改好后再进行commit
```



### 同时修改了文件名和文件内容

```
#一个人修改了文件名commit之后push到远端，另一个人修改了文件内容commit之后要push到远端被拒绝
#这是需要先pull远端的到本地，git会智能的修改了本地的文件名，同时保留我们修改的文件内容
```



### 不加入版本控制的文件

```
#如果远端有我们错提交上去的不想加入版本控制的文件，可以用下面命令将远端上面这个文件删除,本地的文件保留
git rm --cached 文件名
#这个命令会把该文件加入到.gitignore，如果没有加入可以手动加入
```



### Git集成使用禁忌

```
#禁止向集成分支执行 git pull -f
#禁止向集成分支执行变更历史的操作
```





















