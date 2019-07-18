# GIT学习

## 一、安装git

在windows上安装完git后

```cmd
-- 需要配置用户名和邮箱
git config --global user.name "liukc"
git config --global user.email "912396604@qq.com"
```

`git config`命令的`--global`参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址。



## 二、创建版本库

版本库又名仓库，英文名**repository**，可以理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”

1. 先创建一个目录

   ```git
   mkdir note
   cd note
   pwd
   ```

   `pwd`命令用于显示当前目录。

   <span style="color:red">使用Windows系统，为了避免遇到各种莫名其妙的问题，请确保目录名（包括父目录）不包含中文。</span>

2. 通过 `git init` 命令把这个目录变成 git 能管理的库

   ```git
   git init
   ```

   <span style="color:red">使用windows注意：</span>

   千万不要使用Windows自带的**记事本**编辑任何文本文件。原因是Microsoft开发记事本的团队使用了一个非常弱智的行为来保存UTF-8编码的文件，他们自作聪明地在每个文件开头添加了0xefbbbf（十六进制）的字符，你会遇到很多不可思议的问题，比如，网页第一行可能会显示一个“?”，明明正确的程序一编译就报语法错误，等等，都是由记事本的弱智行为带来的

3. 添加文件到库里

   ```git
   // git add 文件名
   git add readme.md
   ```

4. 把文件commit（提交）到库中

   ```git
   git commit -m "add readme"
   ```

   `git commit`命令，`-m`后面输入的是本次提交的说明，可以输入任意内容，最好是有意义的，方便历史记录里方便地找到改动记录。



### 小结

初始化一个Git仓库，使用`git init`命令。

添加文件到Git仓库，分两步：

1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
2. 使用命令`git commit -m <message>`，完成。



## 三、时光穿梭机

`git status` 命令：

`git status`命令可以让我们时刻掌握仓库当前的状态

如：我们在刚才的readme.md 修改下内容

```git
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   "GIT\345\255\246\344\271\240.md"
        modified:   readme.md

no changes added to commit (use "git add" and/or "git commit -a")
```

`git diff`命令：

`git diff`就是查看与上一个版本的不同之处

### 1. 版本回退

`git log`命令：

`git log`命令显示从最近到最远的提交日志，如果嫌输出信息太多，可以加上`--pretty=oneline`参数：

```git
$ git log --pretty=oneline
e52a1e330f1d66f6592f182f82301d72483e8080 (HEAD -> master) update version3.0
b3734681b84d38e5458261abc99be4fcf8d5932a update version2.0
b8d3a64936e5d913cef82539aa61c72a942efead update version
0ca127785e961037224e579b414a1dd6145a0b23 add readme
```

`git reset`命令：

首先，Git必须知道当前版本是哪个版本，在Git中，用`HEAD`表示当前版本，也就是最新的提交`1094adb...`（注意我的提交ID和你的肯定不一样），上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

`git reset`命令用于回退版本

```git
$ git reset --hard HEAD^
HEAD is now at b373468 update version2.0
```

当想回到未来的版本，只要拥有未来版本的版本号，就能回去

```git
$ git reset --hard e52a1e330f1d66f6592f182f82301d72483e8080
HEAD is now at e52a1e3 update version3.0
```

版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。

`git reflog`命令：

`git reflog`用来记录你的每一次命令

```git
$ git reflog
e52a1e3 (HEAD -> master) HEAD@{0}: reset: moving to e52a1e330f1d66f6592f182f82301d72483e8080
b373468 HEAD@{1}: reset: moving to HEAD^
e52a1e3 (HEAD -> master) HEAD@{2}: commit: update version3.0
b373468 HEAD@{3}: commit: update version2.0
b8d3a64 HEAD@{4}: commit: update version
0ca1277 HEAD@{5}: commit (initial): add readme
```

#### 小结

- `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
- 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
- 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

### 2. 工作区和暂存区

把文件往Git版本库里添加的时候，是分两步执行的：

第一步是用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支。

一旦提交后，如果你又没有对工作区做任何修改，那么工作区就是“干净”的

### 3. 管理修改

Git跟踪并管理的是修改，而非文件。

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中

### 4. 撤销修改

`git checkout -- file`命令：

`git checkout -- file`可以丢弃工作区的修改

```git
$ git checkout -- readme.txt

```

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

`git reset HEAD <file>`命令：

`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区

```git
$ git reset HEAD readme.txt
Unstaged changes after reset:
M       readme.txt
```

#### 小结

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

### 5. 删除文件

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ rm test.txt

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   test.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    test.txt
```

如果版本库没有删除，可以用`git checkout --<file>`恢复。

`git rm`命令：

`git rm`删掉版本库的文件	

```git
$ git rm test.txt
rm 'test.txt'
```

#### 小结	

命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失**最近一次提交后你修改的内容**。

## 四、远程仓库

第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

```git
$ ssh-keygen -t rsa -C "912396604@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/LENOVO/.ssh/id_rsa):
Created directory '/c/Users/LENOVO/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/LENOVO/.ssh/id_rsa.
Your public key has been saved in /c/Users/LENOVO/.ssh/id_rsa.pub.
...
```

第2步：登陆GitHub，打开“Account settings”，“SSH Keys”页面：

然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容

### 1. 添加远程仓库

首先，登陆GitHub，然后，在右上角找到“Create a new repo”按钮，创建一个新的仓库

根据GitHub提示，输入

```git
$ git remote add origin https://github.com/liukc/learnGit.git

$ git push -u origin master
Enumerating objects: 21, done.
Counting objects: 100% (21/21), done.
Delta compression using up to 8 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (21/21), 1.63 KiB | 334.00 KiB/s, done.
Total 21 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), done.
To https://github.com/liukc/learnGit.git
 * [new branch]      master -> master

```

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

现在就可以直接推送了

```git
$ git push origin master
```

把本地`master`分支的最新修改推送至GitHub	

### 2. 从远程库克隆

`git clone`命令：

`git clone`克隆一个本地库

```git
$ git clone https://github.com/liukc/learnGit.git
Cloning into 'learnGit'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
```

## 五、分支管理

分支在实际中有什么用呢？假设你准备开发一个新功能，但是需要两周才能完成，第一周你写了50%的代码，如果立刻提交，由于代码还没写完，不完整的代码库会导致别人不能干活了。如果等代码全部写完再一次提交，又存在丢失每天进度的巨大风险。

现在有了分支，就不用怕了。你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，直到开发完毕后，再一次性合并到原来的分支上，这样，既安全，又不影响别人工作。

### 1. 创建和合并分支

`git checkout`命令加上`-b`参数表示创建并切换

```git
$ git checkout -b dev
Switched to a new branch 'dev'
M       README.md
```

它相当于两条命令：

```
$ git branch dev
$ git checkout dev
Switched to branch 'dev'
```

`git branch`命令查看当前分支：

```git
$ git branch
* dev
  master
```

`git branch`命令会列出所有分支，当前分支前面会标一个`*`号。

然后，我们就可以在`dev`分支上正常提交

```git
$ git commit -m "branch test"
[dev 43f075e] branch test
 1 file changed, 5 insertions(+), 1 deletion(-)
```

切换回主分支：

```git
$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
```

把`dev`分支的工作成果合并到`master`分支上：

```git
$ git merge dev
Updating 37692d2..43f075e
Fast-forward
 README.md | 6 +++++-
 1 file changed, 5 insertions(+), 1 deletion(-)
```

删除`dev`分支：

```git
$ git branch -d dev
Deleted branch dev (was 43f075e).
```

#### 小结

Git鼓励大量使用分支：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`

创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

### 2. 解决冲突

当暂存区两个分支的内容存在非子集内容时，就会产生冲突，要手动解决冲突后才能合并

```git
$ git merge dev
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

```git
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

解决冲突后提交：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test/git_clone/learnGit (master|MERGING)
$ git add readme.txt 
$ git commit -m "conflict fixed"
[master c720463] conflict fixed
```

带参数的`git log`也可以看到分支的合并情况：

```git
$ git log --graph --pretty=oneline --abbrev-commit
*   c720463 (HEAD -> master) conflict fixed
|\
| * 80a0ad8 (dev) branch commit
* | 7ddde68 main branch commit
|/
* 43f075e (origin/master, origin/HEAD) branch test
* 37692d2 Initial commit
```

### 3. 分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。



`--no-ff`参数，表示禁用`Fast forward`：

```git
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

```git
$ git log --graph --pretty=oneline --abbrev-commit
*   681c364 (HEAD -> master) merge with no-ff
|\
| * 195e4f7 (dev) add merge
|/
*   c720463 conflict fixed
|\
| * 80a0ad8 branch commit
* | 7ddde68 main branch commit
|/
* 43f075e (origin/master, origin/HEAD) branch test
* 37692d2 Initial commit
```

#### 分支策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

#### 小结

Git分支十分强大，在团队开发中应该充分应用。

合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

### 4. BUG 分支

有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。

Git提供了一个`stash`功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作

```git
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```

工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```git
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了

可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的stash，用命令：

```git
$ git stash apply stash@{0}
```

#### 小结

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场。

### 5. feature分支

软件开发中，总有无穷无尽的新的功能要不断添加进来。

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。

#### 小结

开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。

### 6. 多人协作

用`git remote`查看远程库的信息

用`git remote -v`显示更详细的信息：

#### 推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```git
$ git push origin dev
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

- `master`分支是主分支，因此要时刻与远程同步；
- `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！

#### 抓取分支

要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，用这个命令创建本地`dev`分支：

```git
$ git checkout -b dev origin/dev
```

多人协作的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

#### 小结

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

### 7. rebase

 多人在同一个分支上协作时，很容易出现冲突。即使没有冲突，后push的童鞋不得不先pull，在本地合并，然后才能push成功。

每次合并再push后，分支变成了这样：

```
$ git log --graph --pretty=oneline --abbrev-commit
* d1be385 (HEAD -> master, origin/master) init hello
*   e5e69f1 Merge branch 'dev'
|\  
| *   57c53ab (origin/dev, dev) fix env conflict
| |\  
| | * 7a5e5dd add env
| * | 7bd91f1 add new env
| |/  
* |   12a631b merged bug fix 101
|\ \  
| * | 4c805e2 fix bug 101
|/ /  
* |   e1e9c68 merge with no-ff
|\ \  
| |/  
| * f52c633 add merge
|/  
*   cf810e4 conflict fixed
```

总之看上去很乱，看看怎么把分叉的提交变成直线。

在和远程分支同步后，我们对`hello.py`这个文件做了两次提交。用`git log`命令看看：

```
$ git log --graph --pretty=oneline --abbrev-commit
* 582d922 (HEAD -> master) add author
* 8875536 add comment
* d1be385 (origin/master) init hello
*   e5e69f1 Merge branch 'dev'
|\  
| *   57c53ab (origin/dev, dev) fix env conflict
| |\  
| | * 7a5e5dd add env
| * | 7bd91f1 add new env
...
```

注意到Git用`(HEAD -> master)`和`(origin/master)`标识出当前分支的HEAD和远程origin的位置分别是`582d922 add author`和`d1be385 init hello`，本地分支比远程分支快两个提交。

现在我们尝试推送本地分支：

```
$ git push origin master
To github.com:michaelliao/learngit.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'git@github.com:michaelliao/learngit.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

很不幸，失败了，这说明有人先于我们推送了远程分支。按照经验，先pull一下：

```
$ git pull
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
Unpacking objects: 100% (3/3), done.
From github.com:michaelliao/learngit
   d1be385..f005ed4  master     -> origin/master
 * [new tag]         v1.0       -> v1.0
Auto-merging hello.py
Merge made by the 'recursive' strategy.
 hello.py | 1 +
 1 file changed, 1 insertion(+)
```

再用`git status`看看状态：

```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 3 commits.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

加上刚才合并的提交，现在我们本地分支比远程分支超前3个提交。

用`git log`看看：

```
$ git log --graph --pretty=oneline --abbrev-commit
*   e0ea545 (HEAD -> master) Merge branch 'master' of github.com:michaelliao/learngit
|\  
| * f005ed4 (origin/master) set exit=1
* | 582d922 add author
* | 8875536 add comment
|/  
* d1be385 init hello
...
```

对强迫症童鞋来说，现在事情有点不对头，提交历史分叉了。如果现在把本地分支push到远程，有没有问题？

有！

什么问题？

不好看！

有没有解决方法？

有！

这个时候，rebase就派上了用场。我们输入命令`git rebase`试试：

```
$ git rebase
First, rewinding head to replay your work on top of it...
Applying: add comment
Using index info to reconstruct a base tree...
M	hello.py
Falling back to patching base and 3-way merge...
Auto-merging hello.py
Applying: add author
Using index info to reconstruct a base tree...
M	hello.py
Falling back to patching base and 3-way merge...
Auto-merging hello.py
```

输出了一大堆操作，到底是啥效果？再用`git log`看看：

```
$ git log --graph --pretty=oneline --abbrev-commit
* 7e61ed4 (HEAD -> master) add author
* 3611cfe add comment
* f005ed4 (origin/master) set exit=1
* d1be385 init hello
...
```

原本分叉的提交现在变成一条直线了！这种神奇的操作是怎么实现的？其实原理非常简单。我们注意观察，发现Git把我们本地的提交“挪动”了位置，放到了`f005ed4 (origin/master) set exit=1`之后，这样，整个提交历史就成了一条直线。rebase操作前后，最终的提交内容是一致的，但是，我们本地的commit修改内容已经变化了，它们的修改不再基于`d1be385 init hello`，而是基于`f005ed4 (origin/master) set exit=1`，但最后的提交`7e61ed4`内容是一致的。

这就是rebase操作的特点：把分叉的提交历史“整理”成一条直线，看上去更直观。缺点是本地的分叉提交已经被修改过了。

最后，通过push操作把本地分支推送到远程：

```
Mac:~/learngit michael$ git push origin master
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 576 bytes | 576.00 KiB/s, done.
Total 6 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
To github.com:michaelliao/learngit.git
   f005ed4..7e61ed4  master -> master
```

再用`git log`看看效果：

```
$ git log --graph --pretty=oneline --abbrev-commit
* 7e61ed4 (HEAD -> master, origin/master) add author
* 3611cfe add comment
* f005ed4 set exit=1
* d1be385 init hello
...
```

远程分支的提交历史也是一条直线。



#### 小结

- rebase操作可以把本地未push的分叉提交历史整理成直线；
- rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

## 六、标签管理 

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

### 1. 创建标签

敲命令`git tag <name>`就可以打一个新标签：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag v1.0

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag
v1.0
```

默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？

方法是找到历史提交的commit id，然后打上就可以了：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git log --pretty=oneline --abbrev-commit
cd6d2d9 (HEAD -> master, tag: v1.0, origin/master) add test.txt
952930f update version4.1
ea0486b update version 4.1
e52a1e3 update version3.0
b373468 update version2.0
b8d3a64 update version
0ca1277 add readme

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag v0.9 952930f

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag
v0.9
v1.0

```

可以用`git show <tagname>`查看标签信息：

```git

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git show v1.0
commit cd6d2d94ad8a5f5ba692b7d8fdea824bbb40f66b (HEAD -> master, tag: v1.0, origin/master)
Author: liukc <912396604@qq.com>
Date:   Wed Jul 17 09:25:52 2019 +0800

    add test.txt

diff --git a/test.txt b/test.txt
new file mode 100644
index 0000000..e553405
--- /dev/null
+++ b/test.txt
@@ -0,0 +1 @@
+测试git 的删除功能
\ No newline at end of file
```

创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字:

```git
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

#### 小结

- 命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；
- 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；
- 命令`git tag`可以查看所有标签。

### 2. 操作标签

#### 删除标签

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag
v0.9
v1.0

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag -d v0.9
Deleted tag 'v0.9' (was 952930f)

LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag
v1.0
```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

#### 推送标签到远程

如果要推送某个标签到远程，使用命令`git push origin <tagname>`：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git push origin v1.0
Enumerating objects: 21, done.
Counting objects: 100% (21/21), done.
Delta compression using up to 8 threads
Compressing objects: 100% (11/11), done.
Writing objects: 100% (21/21), 1.63 KiB | 238.00 KiB/s, done.
Total 21 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), done.
To https://github.com/liukc/learnGit.git
 * [new tag]         v1.0 -> v1.0
```

一次性推送全部尚未推送到远程的本地标签：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git push origin --tags
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/liukc/learnGit.git
 * [new tag]         v0.8 -> v0.8
 * [new tag]         v0.9 -> v0.9
```

如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git tag -d v0.8
Deleted tag 'v0.8' (was ea0486b)
```

然后删除掉远程的标签：

```git
LENOVO@DESKTOP-HKPQ94M MINGW64 /f/git_test (master)
$ git push origin :refs/tags/v0.8
To https://github.com/liukc/learnGit.git
 - [deleted]         v0.8
```

#### 小结

- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

## 七、使用GitHub

在GitHub上，利用Git极其强大的克隆和分支功能，可以自由参与各种开源项目。

比如人气极高的bootstrap项目，这是一个非常强大的CSS框架，可以访问它的项目主页https://github.com/twbs/bootstrap，点“Fork”就在自己的账号下克隆了一个bootstrap仓库，然后，从自己的账号下clone：

```git
git clone git@github.com:liukc/bootstrap.git
```

一定要从自己的账号下clone仓库，这样才能推送修改。如果从bootstrap的作者的仓库地址`git@github.com:twbs/bootstrap.git`克隆，因为没有权限，将不能推送修改。

Bootstrap的官方仓库`twbs/bootstrap`、在GitHub上克隆的仓库`my/bootstrap`，以及你自己克隆到本地电脑的仓库，他们的关系就像下图显示的那样：

```ascii
┌─ GitHub ────────────────────────────────────┐
│                                             │
│ ┌─────────────────┐     ┌─────────────────┐ │
│ │ twbs/bootstrap  │────>│  my/bootstrap   │ │
│ └─────────────────┘     └─────────────────┘ │
│                                  ▲          │
└──────────────────────────────────┼──────────┘
                                   ▼
                          ┌─────────────────┐
                          │ local/bootstrap │
                          └─────────────────┘
```

如果想修复bootstrap的一个bug，或者新增一个功能，立刻就可以开始干活，干完后，往自己的仓库推送。

如果你希望bootstrap的官方库能接受你的修改，你就可以在GitHub上发起一个pull request。当然，对方是否接受你的pull request就不一定了。

如果你没能力修改bootstrap，但又想要试一把pull request，那就Fork一下我的仓库：https://github.com/michaelliao/learngit，创建一个`your-github-id.txt`的文本文件，写点自己学习Git的心得，然后推送一个pull request给我，我会视心情而定是否接受。

#### 小结

- 在GitHub上，可以任意Fork开源仓库；
- 自己拥有Fork后的仓库的读写权限；
- 可以推送pull request给官方仓库来贡献代码。

## 八、自定义git

让Git显示颜色，会让命令输出看起来更醒目：

```git
$ git config --global color.ui true
```

#### 忽略特殊文件

在Git工作区的根目录下创建一个特殊的`.gitignore`文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。

不需要从头写`.gitignore`文件，GitHub已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore

忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

有些时候，你想添加一个文件到Git，但发现添加不了，原因是这个文件被`.gitignore`忽略了

可以用`-f`强制添加到Git：

```git
$ git add -f App.class
```

需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

```git
$ git check-ignore -v App.class
.gitignore:3:*.class	App.class
```

#### 小结

- 忽略某些文件时，需要编写`.gitignore`；
- `.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理！