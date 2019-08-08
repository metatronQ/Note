## 1.创建版本库
* **初始化git仓库：**git init  
* **添加文件到git仓库：**  
1.使用git add < file>,可反复使用添加多个文件，git add .代表添加全部文件  
2.git commit -m"本次提交" 提交到本地仓库  

## 2.时光穿梭机：  
* git status ：掌握仓库当前状态，告诉文件是否被修改或提交
* git diff < file>：察看difference、修改记录  
* cat < file>：文本形式显示文件内容
### 版本回退：
* git log：察看历史记录，加上--pretty=oneline 可只显示commit id  
* git reset --hard HEAD^:HEAD表示当前版本，HEAD^表示上个版本，以此类推，HEAD~100表示往上100个版本
* git reset --hard < commit id 前几位> :跳到指定版本  
* git reflog:记录每一次命令（可由此得到未来版本commit id)  

### 工作区和暂存区：  
* 工作区：电脑里能看到的目录；
>* 版本库Repository：隐藏目录.git  
>> * 暂存区：stage/index 

git add 将文件添加到暂存区  
git commit 将暂存区的所有内容提交到当前分支  

### 管理修改：  
* 如果修改不用git add到暂存区，那么久不会加入到commit中  

### 撤销修改：  
* git checkout -- < file>：在add 之前撤销修改（--前后都有空格）  
* git reset HEAD < file>:add后把暂存区的修改撤销回工作区，即撤销add操作
* commit之后可用reset进行版本回退操作；

### 删除文件：git rm 等于 手删+add
* 删除：git rm < file>:删除文件，之后git commit，此后发现误删只能版本回退
* 误删：rm之后  
1.git reset HEAD < file>:退回工作区  
2.git checkout -- < file>:用版本库的版本替换工作区的版本。
* **未添加到版本库就被删除的文件是无法恢复的**  

## 3.远程仓库： 
**1.添加远程库：**  

* *关联一个远程库：*git remote add origin git@server-name:path/name.git(git remote add origin git@github.com:metatronQ/MyAndroidDemo.git)
* 删除远程连接：删除远程连接：git remote rm origin

* *关联后：*使用命令git push -u origin master第一次推送master分支的所有内容
* *此后每次提交：*git push origin master 推送最新修改

**2.从远程库克隆：**  
* *克隆一个仓库：*git clone 仓库地址  

## 4.分支管理：

### 创建与合并分支：  
* 创建并切换分支：git checkout -b < name> = git branch name + git checkout name
* 查看分支：git branch
* 合并name分支到当前分支：git merge < name>
* 删除分支：git branch -d < name>  

###解决冲突：  
* git log --graph --pretty=oneline --abbrev-commit可以看到分支合并图
* git status 可以告诉我们冲突的文件
* 可用cat name 查看冲突的内容
###分支管理策略  
* 禁用Fast forword的普通合并：git merge --no-ff -m "描述" < name>    
可用git log --graph --pretty=oneline --abbrev-commit查看分支历史  

### Bug分支:并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？  

* git stash:将当前工作现场（chenfu）隐藏起来
* 在要修改bug的分支上（master）创建临时分支（bugtest）来修复bug
* 完成合并（master）并删除bug分支
* 切换回工作现场(chenfu)
* git stash list:查看工作现场(name)
* git stash apply(恢复) + git stash drop(删除)/git stash pop:恢复并删除工作现场
* git stash apply name：指定恢复工作现场  

### Feature分支:每添加一个新功能，就在（chenfu）上新建一个feature分支，完成后，合并并删除该分支  
* git branch -D < name>:丢弃一个没有被合并过的分支  

### 多人协作： 

* 多人协作的工作模式通常是这样：

> 首先，可以试图用git push origin <branch-name>推送自己的修改；

> 如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；

> 如果合并有冲突，则解决冲突，并在本地提交；

> 没有冲突或者解决掉冲突后，再用git push origin <branch-name>推送就能成功！

> 如果git pull提示no tracking information，则说明本地分支和远程分支的链接关系没有创建，用命令git branch --set-upstream-to <branch-name> origin/<branch-name>。

* git remote -v:查看远程库信息

### Rebase  
* git rebase:把本地为push的分叉提交历史整理成直线

## 5.标签管理： 
### 创建标签:
* git tag < tagname>:在要打标签的分支上打上标签；
* git tag:查看所有标签 
* git tag < tagname> < commit id>：对某个提交打标签
* git show < tagname>:查看标签信息
* git tag -a < tagname> -m"说明文字" ：指定标签信息

### 操作标签: 

* git tag -d < tagname>:删除标签
* git push origin < tagname>:推送某个标签到远程
* git push origin --tags:推送全部到远程
* 删除远程标签：
> * git tag -d < tagname>
> * git push origin :refs/tags/< tagname>

