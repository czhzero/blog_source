---
title: Git 基础命令操作指引
date: 2016-12-26 20:05:34
categories: 开发工具
tags: Git
---

Git是一个分布式版本管理系统，是为了更好地管理Linux内核开发而创立的。Git与Svn相比，不光有远程代码库，也有本地代码库，方便多人协作。

另外，Git中分支的活用，也极大的方便了开发人员开发过程中的多版本同时开发。

![](http://o7y1sf21i.bkt.clouddn.com/blog/016/Git-start3.png)

<!-- more -->

# Git 初始化项目

## Create a new repository 

```
	git clone git@*******/test.git
	cd test
	touch README.md
	git add README.md
	git commit -m "add README"
	git push -u origin master
	
```


## Existing folder or Git repository

```
    cd existing_folder
    git init
    git remote add origin git@@*******/test.git
    git add .
    git commit
    
```


# Git 参数配置

## config 配置文件位置

```
All User : /etc/gitconfig
Current User : ~/.gitconfig

```

## config 查看所有配置信息

```
git config --list

```


## config 设置基本用户信息

```
git config --global user.name "xxx"
git config --global user.email "xxxx@163.com"

```

增加 `--global` 表示该设置对当前系统登录用户生效，配置完成后，可以在`~/.gitconfig`中查看结果。否则，表示只对当前git项目生效，在项目的`.git/config`中查看结果。

## config 设置命令别名

```
$ git config --global alias.co checkout      //git co 代替 git checkout 命令
$ git config --global alias.ci commit	     //git ci 代替 git commit 命令
$ git config --global alias.br branch        //git br 代替 git branch 命令

```

别名设置除了可以设置单个命令的缩写外，还可以结合options进行设置。

```
//git unstage 代替 git reset HEAD
$ git config --global alias.unstage 'reset HEAD'  
   
//git last 代替 git log -l
$ git config --global alias.last 'log -1' 
           
//git lg 代替 复杂的git log效果
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"          		
```

别名配置好了之后，如果通过`vi ~/.gitconfig` 查看结果如下，修改配置文件，则可以修改别名设置效果。

```
...

[user]
        name = 你的姓名
        email = 你的邮箱

...

[alias]
      st = status
      ci = commit
      br = branch
      co = checkout
      df = diff
      cp = cherry-pick
      rb = "rebase -i HEAD~10"
      lg = "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

```



# Git 基础命令

## git status

查看git仓库文件状态, git文件状态如下所示。

![git文件状态](http://o7y1sf21i.bkt.clouddn.com/blog/016/Git-start6.png)


```
unstaged - git仓库中没有此文件的相关记录 
modified - git仓库中有这个文件的记录，且此文件当前有改动 
staged - 变更的文件被暂存,没有提交到仓库 (git add命令之后)
commited - 文件被提交到本地git仓库 (git commit之后)
    
```

## git log
  
```
git log             #查看历史commit记录
git log -1          #查看最近一个commit记录
  
```

## git add


```
  git add file/fileDir         #modified或unstaged的文件转换为staged状态
  git add -A/--all             #modified或unstaged的所有文件转换为staged状态
  
```

## git rm
  
```
  git rm test.java             #从git仓库删除test.java文件，同时删除本地文件
  git rm  --cache test.java    #从git仓库删除test.java文件，不删除本地文件
```


  
## git commit

```
  git commit -m "xxx comment"    #staged状态转换为commited状态
  
```


## git diff 

```
  git diff                     #查看本次修改的所有文件的改动
  git diff file-path           #查看本次修改的指定文件有哪些改动

```


## git pull

```
  git pull [remote-name branch-name]  #获取远程服务器数据,并同时与本地数据合并(可能会出现冲突)
  
```


## git fetch

```
git fetch [remote-name]     #(获取仓库的所有更新，但是不自动合并当前分支)
git fetch -p                #(获取仓库的所有更新，且自动删除不需要的分支，更新最新的分支下来)

```

## git push 

```
  git push [remote-name branch-name]         #上传数据到远程分支，若出现冲突，则上传失败。
  git push --force [remote-name branch-name] #强制上传数据到远程分支**，(覆盖远程数据，慎用)**
  
```


## git stash

可用来暂存当前正在进行的工作，比commit简单，而且可以跨分支合并
	

```
git stash                    #暂存更改的内容
git stash save "comment"     #暂存更改的内容,带注释
git stash list               #显示所有暂存的内容 
	
git show stash@{0}           #显示指定stash的更改内容，类似于git diff 
	
git stash apply stash@{1}    #应用指定编号的stash, 但不从stash-list中清除
git stash pop                #应用栈顶的stash,并从stash-list清除该记录
git stash clear              #清除stash-list
	
git stash --help             #获取更多的信息
	
```

## git tag


```
git tag v1.01								        #增加新的tag
git tag -a v1.01 -m "Relase version 1.01"    #增加新的tag -a 表示标签名称 -m 表示标签注释                      
git tag [-l]                                 # 查看所有标签
git push origin -tags                        # 推送tag到远程分支
git tag -d v1.01	                            # 删除本地标签
git push origin :refs/tags/v1.01             # 删除远程标签

```

tag建立好了后，可以当做 commit-id使用，git reset的时候直接调到tag。


# Git 分支

## git branch
  

```
  git branch                  #查看本地分支列表
  git branch branch-name      #创建新分支
  git branch -r               #查看远程分支列表
  git branch -a               #查看所有分支
  git branch -d branch-name   #删除指定分支
  git branch -D branch-name   #强制删除指定分支
```


## git checkout 
  

```
  git checkout branch-name                         #切换到指定分支 (切换前记得先commit或者stash)
  git checkout -b new-branch-name                  #基于当前分支创建一个新分支
  git checkout -b branch-name origin/branch-name   #获取远程分支到本地 
  git checkout file-name                           #获取远程最新文件
  git checkout -- .                                #放弃本地所有的modify文件改动
```



## add remote branch

```
  1.git checkout branch-name              #切换到要上传的分支
  2.git remote add origin branch-name     #添加远程分支
  3.git push origin branch-name            #推送数据到远程分支 (自动创建分支，step2可省去) 
   
```



##  delete remote branch

```
  git push origin :remote-branch         #删除指定远程分支
  
```



# Git 分支合并

## git merge

```
  1. git merge branch-name                 #合并指定分支内容到当前分支
  2. git add -A                            #手动解决冲突后,重新暂存
  3. git commit -m "xxx 解决冲突"           #再次提交
```


有些情况下我们需要优先选择使用 `git merge --squash branch-name`
--squash选项的含义是：本地文件内容与不使用该选项的合并结果相同，
但是不提交、不移动HEAD，因此需要一条额外的commit命令。
其效果相当于将another分支上的多个commit合并成一个，放在当前分支上，原来的commit历史则没有拿过来。
判断是否使用--squash选项最根本的标准是，待合并分支上的历史是否有意义。
如果在开发分支上提交非常随意，甚至写成微博体，那么一定要使用--squash选项。

## git rebase

使用rebase合并代码可以使提交的历史记录显得更简洁

```
1. git rebase target-branch-name         #合并目标分支内容到当前所在分支
2. git add -A                            #手动解决冲突后, 重新暂存
3. git rebase -- abort                   #放弃rebase操作
4. git rebase -- continue                #解决冲突后，继续rebase操作

```

暂时取消刚才的合并 (只能回退一个合并的commit)

```
git reset --hard HEAD~

```


## git cherry-pick 

挑选指定的commit-id,一个一个合并，合并速度慢了点，但是合并效果最好。 

```
  1. git cherry-pick commit-id         #合并指定commit到当前分支
  2. git add -A                        #手动解决冲突后, 重新暂存
  3. git cherry-pick -- continue       #继续刚才的pick操作，这一步容易忘记
  4. git cherry-pick -- abort          #放弃cherry-pick操作
```



## 解决自动合并失败的冲突

由于在同一行进行了修改，合并时候，产生了冲突。内容格式如下显示，然后根据实际需要解决冲突。

冲突解决完成后，先使用 `git add -A` ，然后根据合并命令，执行`git cherry-pick --continue` 
或者 `git rebase --continue`命令。

```
<<<<<<< HEAD

冲突内容一

=======

冲突内容二

>>>>>>> commit-id

```

  
# Git 修改与撤销

## git commit --amend

```
git commit --amend   #编辑工具会显示最近一次提交的提交消息，用来修改消息

```

## git reset

```
	git reset HEAD file          #将文件从staged转换为unstaged
	git reset HEAD .             #将所有文件从staged转换为unstaged
	git reset --mixed commit-id  #默认方式，回退到指定个版本，只保留源码，回退commit和index信息
	
	git reset --soft commit-id   #回退到某个版本，只回退了commit的信息，
	                             #不会恢复到index file级。如果还要提交，直接commit即可   
	
	git reset --hard commit-id   #彻底回退到某个版本，本地的源码也会变为上一个版本的内容
```

## git revert 

git revert 是生成一个新的提交来撤销某次提交，此次提交之前的commit都会被保留
git reset 是回到某次提交，提交及之前的commit都会被保留，但是此次之后的修改都会被退回到暂存区

具体一个例子，假设有三个commit， 

```
commit3: add test3.md
commit2: add test2.md
commit1: add test1.md

```
当执行`git revert HEAD~1`时， commit2被撤销了, 运行`git log`:

```
revert "commit2":this reverts commit 5fe21s2...
commit3: add test3.md
commit2: add test2.md
commit1: add test1.md

```

运行`git status`， 没有任何暂存。

如果换做执行`git reset --soft(默认) HEAD~1`后，运行`git log`:

```
commit2: add test2.md
commit1: add test1.md

```

运行`git status`， 其中 `test3.md` 处于暂存区，准备提交。

再如果换做执行`git reset --hard HEAD~1`后,

显示：`HEAD is now at commit2`，运行`git log`

```
commit2: add test2.md
commit1: add test1.md

```
运行`git status`， 没有任何暂存。

另外：

`git revert <commit-id>` 是撤消该commit，作为一个新的commit。


  
## git rebase


```
(1) git rebase -i commit-id   #在vi编辑器中，显示commit-id到最新的所有commit记录
(2) git rebase -i HEAD~10     #在vi编辑器中，显示最新10条commit记录
(3) 在vi编辑器中, 修改commit内容, wq保存
(4) git add -A                #解决冲突后，暂存文件
(5) git rebase --continue     #完成未完成rebase操作
(6) git rebase --abort        #完成未完成rebase操作

```
	
假定vim编辑器的初始内容如下:
	
```
pick 8e5da5f test-comment-1 
pick 627a433 test-comment-2 
pick 627a433 test-comment-3 
pick dcc8310 test-comment-4 
	
```
	
git常见的几种操作方式，都是通过vim编辑器来实现的。

- Scene1 ： 修改commit的message内容

修改vim内容结果如下，然后保存。
   
```	
# 修改commit内容
pick 8e5da5f test-comment-1 
pick 627a433 test-comment-2 
r 627a433 修改后的内容 
pick dcc8310 test-comment-4 
	
```
	
- Scene2 ：删除指定commit的提交内容
		
修改vim内容结果如下，然后保存。

```
# commit-id删除 
pick 8e5da5f test-comment-1 
pick 627a433 test-comment-2 
pick dcc8310 test-comment-4 
	
```
	
- Scene3 ：调换指定commit的提交顺序

修改vim内容结果如下，然后保存。若有冲突，先解决冲突，
然后使用`git add -A`添加，再使用`git rebase --continue`继续。
	
```
# 调换
pick 8e5da5f test-comment-1 
pick 627a433 test-comment-3 
pick 627a433 test-comment-2 
pick dcc8310 test-comment-4 
	
```
	
- Scene4 ：合并指定的两个commit的内容

修改vim内容结果如下，然后保存, 若有冲突，先解决冲突，
然后使用`git add -A`添加，再使用`git rebase --continue`继续。

```
# commit-id 3 与 4合并
pick 8e5da5f test-comment-1 
pick 627a433 test-comment-2 
pick 627a433 test-comment-3 
s dcc8310 test-comment-4 
	
```


## git reflog

  显示所有过去commit历史，可使用`git reset --hard`命令跳转到指定commit-id位置，
  用来恢复某些的误操作。
	

