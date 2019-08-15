[TOC]

# GIT学习记录

## 配置github与gitlib两个账号

```bash
ssh-keygen -t rsa -C "your_name@github.com" #生成github的rsa
#将 rsa与rsa.pub放到user/yourname/.ssh下
ssh-keygen -t rsa -C "your_name@gitlab.com" #生成gitlab的rsa，文件名称与刚才的github区别
# 同样将两个文件放到.ssh文件夹下
```

```bash
#在 .ssh文件夹下创建config文件，名称就叫config
# gitlab
Host as-gitlab.cisco.com
    HostName as-gitlab.cisco.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab_id-rsa
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa
```

当然最后一步就是用文本编辑打开.pub文件，将里面的公钥分别放到github与gitlab中。至此两个都能够使用了。

## 基本操作

### git init：初始化仓库

- 新建文件夹test，
- 进入test，右键 `Git Bash Here`
- 键入 `git init`
- 在test文件夹中会增加一个.git的子目录，该文件夹中保存着管理当前目录内容所需的仓库数据
- 我们将test文件夹称为附属于该仓库的工作树

### git status：查看仓库状态

工作树与仓库被操作之后，可以使用`git status` 查看当前状态。

- 当没有任何变化时会返回如下信息

```bash
$ git status
On branch master

No commits yet

nothing to commit (create/copy files and use "git add" to track)
# 在master分支下，没有任何东西可以提交
```

- 创建一个README.md，之后再运行 `git status`

```bash
$ touch README.md
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        README.md

nothing added to commit but untracked files present (use "git add" to track)
# 显示新加的README.md文件，无记录，就是没有被git仓库管理
```

### git add ：向缓存区中添加文件

刚才创建了README.md文件，现在我们把它加到git仓库中

```bash
$ git add README.md # 如果文件特别多可以使用 git add .(是空格.)，但是不建议这样做
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   README.md
# 显示等待着提交的改变
```

### git commit 保存仓库的历史记录

下面我将我们新建的文件提交到git的历史记录中，

```bash
$ git commit -m 'First Commit'
[master (root-commit) 9f129ba] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
# -m 参数可以输入一个简短的备注，如果比较长，可以不加-m 将会进入编辑模式
# 编辑模式与vi的操作一样，按insert输入文字，:wq保存退出即可
# 在编辑模式下如果不输入comment直接退出，将会自动取消提交
```

提交之后我们再查看仓库状态

```bash
$ git status
On branch master
nothing to commit, working tree clean
```

<font color="red">注：补充一个命令 `git commit -am`</font>，可以起到 `git add `+ `git commit -m`两个命令的效果

### git log 查看提交日志

git log可以查看以往的仓库中的提交的日志包括可以查看什么人在什么时候进行了提交或合并。

```bash
$ git log
commit 274487c29eef98e8eaba26ec207075c396445a89 (HEAD -> master)
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:07:33 2019 +0800

    First Commit
# 显示简述信息
$ git log --pretty=short # = 左右两边不要有空格，否则会报错
commit 274487c29eef98e8eaba26ec207075c396445a89 (HEAD -> master)
Author: KanePD <18842634041@163.com>

    First Commit
#显示某个文件的log信息
$ git log README.md
commit 274487c29eef98e8eaba26ec207075c396445a89 (HEAD -> master)
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:07:33 2019 +0800

    First Commit
#显示文件的改动
$ git log -p
commit 274487c29eef98e8eaba26ec207075c396445a89 (HEAD -> master)
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:07:33 2019 +0800

    First Commit

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..e69de29
#显示某个文件的改动
$ git log -p README.md
```

注：如果想要退出log模式直接按一个q就可以

### git diff ： 查看工作树、暂存区、最新提交的差别

- 工作树与暂存区的差别

```bash
# 我们更改 README.md增加 `# GIT`保存后键入 `git diff`
$ git diff
diff --git a/README.md b/README.md
index e69de29..a25b2f0 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1,2 @@
+# GIT
+
# 只显示了工作树与最新提交的差别
$ git add .# 我们将这次提交放入暂存区
# 继续修改README.md 加入 `# GIT2`
$ git diff
$ git diff
#-----------------------------------
diff --git a/README.md b/README.md
index a25b2f0..35b4006 100644
--- a/README.md
+++ b/README.md
@@ -1,2 +1,4 @@
 # GIT

+# GIT2
+
#-----------------------------------
# 可以看到工作树、暂缓去的差别都出来了
```

- 工作树、暂存区与最新提交的差别

```bash
$ git diff HEAD
diff --git a/README.md b/README.md
index e69de29..35b4006 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1,4 @@
+# GIT
+
+# GIT2
+
```

注：书上说这个是工作树与最新提交的差别，但是我自己尝试的时候上面的#GIT 是暂存区的东西，所以这个指令应该是无论是git 记录上（暂存区）没记录上（工作树）的内容与最新提交的差别。

- 将修改内容提交

```bash
$ git add .
# 此时修改的两个内容都在暂存区，我又执行了一次 git diff HEAD 发现与上面的结果相同
$ git commit -m 'Add index'
```

- 再次查看 `git log`

```bash
$ git log
commit 72838f5f7c1943b51900c0f40a54ea28e1daf479 (HEAD -> master)
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:58:11 2019 +0800

    Add index

commit 274487c29eef98e8eaba26ec207075c396445a89
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:07:33 2019 +0800

    First Commit
```

### 至此基本操作已经完成了

## 关于分支（Branch）的操作

### git branch ：显示分支一览表

```bash
$ git branch -a
* master
# 有*【星号】的branch为当前使用的branch
```

### git checkout -b ：创建、切换分支

- 创建、切换分支

```bash
$ git checkout -b feature-A master
Switched to a new branch 'feature-A'
# 注书上介绍的时候没有后面的master
# 最后面的master的意思是基于master创建分支 feature-A
# 如果不加这个部分的参数，会基于当前操作的branch创建分支
# ---------------------------------------------
# 上面的指令其实是做了如下两个指令
$ git branch feature-A #先创建
$ git checkout feature-A#再切换
# 此时查看当前分支一览
$ git branch
* feature-A
  master
# 发现当前为 feature-A
```

<font color="red">注：这里有一个问题，书上创建的feature-A分支中的README.md中什么都没有，而我创建的时候里面拥有了master的数据了，我尝试了确实是如果不加后面的参数的话，是以master为基础创建的。我在项目中也尝试了，确实是基于当前分支创建的。所以书上应该是有问题的。</font>

- 对当前分支进行一些操作

```bash
# 修改READM.md 增加 `- feature-A`
$ git add README.md
$ git commit -m 'Add feature-A'
# 在这之后 这一行就在分支feature-A中的README.md中有了
#-------------------------------
# 我们切换回master分支
$ git checkout master
Switched to branch 'master'
$ cat README.md
# GIT

# GIT2
# 发现在master 的README.md中并没有`- feature-A`这一行
```

- 切换回上一个分支

```bash
# 这是一个简便的操作
git checkout -
```

- 在这里介绍几个概念

  注：不代表以下概念是并行的，可能没有关联性

  - 培育分支

  我们创建好feature-A之后，一直对分支feature-A进行操作、修改，提交也是提交到feature-A这个分支中。像这样不断地对一个分支进行提交的操作，我们成为“培育分支”

  - 特性分支

  除了完成该分支要完成的工作之外，不再做其他任何的代码修改。在这个分支上只实现单一特性。如果这个分支提交后，发现了bug ，也要重新创建分支，进行修改。

  - 主干分支

  主干分支是特性分支的原点，也是合并的终点。一般以master做主干分支

### git merge ： 合并分支

我们要做的工作是将 feature-A branch的东西合并到master

```bash
# 切换到master分支
$ git checkout master
Switched to branch 'master'
# 执行合并命令
# 书上的命令是git merge --no --ff feature-A，但是--no已经没有了 --ff 是默认的
$ git merge feature-A
Updating 72838f5..4aff578
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
# 书上说会进入编辑模式，但是此处我并没有进入编辑模式
# 可使用 git merge -e进入编辑模式
# -------------------
# 既然branch feature-A的东西已经merge到 master上了 就可以删除掉了。
# 当然在删除之前，最好先推到远端，
# 由于这个分支再下一部分还需要使用，我没有真正的执行删除命令
$ git branch -d feature-A
```

### git log --graph：以图表的方式查看分支

书上说里面可以看到合并的操作，但是我并没有看见。我后来又创建了一个分支，还是看不到，只能看到如下信息

**注：**后来明白了，我再合并feature-A到master时没有冲突并没有做合并这个操作，所以没有

```bash
$ git log --graph
* commit dfa1d86e8c850a020a1edfe866095d8f89204048 (HEAD -> feature-A, test)
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 17:17:39 2019 +0800
|
|     test-1
|
* commit 0c74e08b9672eac7a36582d4abf5d3face77c83e
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 17:16:58 2019 +0800
|
|     test-1
|
* commit 6f28d3388bbde22f273769a14596c8413f070116 (master)
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 17:07:29 2019 +0800
|
|     Twice change for feature-A
|
* commit 4aff578e78688112e9c5d2a42af7da90da3310b5
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 15:13:06 2019 +0800
|
|     Add feature-A
* commit 72838f5f7c1943b51900c0f40a54ea28e1daf479
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 14:58:11 2019 +0800
|
|     Add index
|
* commit 274487c29eef98e8eaba26ec207075c396445a89
  Author: KanePD <18842634041@163.com>
  Date:   Thu Mar 14 14:07:33 2019 +0800

      First Commit
# 可以看到 从第一次提交到后来的提交，
# 可以看到master拥有哪些commit
# 可以看到feature 与 test分别拥有哪些commit
# merge信息没有看到，不知道为什么
# 我们可以输入 git reflog 查看当前的操作日志
$ git reflog
dfa1d86 (test) HEAD@{2}: merge test: Fast-forward
6f28d33 (master) HEAD@{3}: checkout: moving from master to feature-A
6f28d33 (master) HEAD@{4}: checkout: moving from test to master
dfa1d86 (test) HEAD@{5}: commit: test-1
0c74e08 HEAD@{6}: commit: test-1
6f28d33 (master) HEAD@{7}: checkout: moving from master to test
6f28d33 (master) HEAD@{8}: checkout: moving from feature-A to master
6f28d33 (master) HEAD@{9}: checkout: moving from master to feature-A
6f28d33 (master) HEAD@{10}: merge feature-A: Fast-forward (no commit created; -m option ignored)
4aff578 HEAD@{11}: checkout: moving from feature-A to master
6f28d33 (master) HEAD@{12}: commit: Twice change for feature-A
4aff578 HEAD@{13}: checkout: moving from master to feature-A
4aff578 HEAD@{14}: checkout: moving from feature-A to master
4aff578 HEAD@{15}: checkout: moving from master to feature-A
4aff578 HEAD@{16}: merge feature-A: Fast-forward
72838f5 (HEAD -> fix-B, feature-A) HEAD@{17}: checkout: moving from feature-A to master
4aff578 HEAD@{18}: checkout: moving from master to feature-A

```

## 更改已经提交的操作

### git reset ： 回溯历史版本

我们回到feature-A分支之前，创建一个新的分支fix-B。从上面的`git log`的信息可以查看出feature-A分支之前的哈希值是`72838f5f7c1943b51900c0f40a54ea28e1daf479`

```bash
$ git reset --hard 72838f5f7c1943b51900c0f40a54ea28e1daf479
HEAD is now at 72838f5 Add index
# 我们再输入git log
$ git log
commit 72838f5f7c1943b51900c0f40a54ea28e1daf479 (HEAD -> feature-A)
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:58:11 2019 +0800

    Add index

commit 274487c29eef98e8eaba26ec207075c396445a89
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 14:07:33 2019 +0800

    First Commit
# 发现只剩这两次提交了
# 创建新的分支 fix-B
$ git checkout -b fix-B
Switched to a new branch 'fix-B'
# 更改README.md
vi README.md
$ cat README.md
# GIT

# GIT2
- fix-B
# 发现只多了fix-B feature-A与test的branch的东西都不在
$ git add .
$ git commit -m 'fix-B'
# 下面我要将fix-B分支合并到master上
####################一定要先切换到master分支上，否则合并不上
$ git checkout master
# 我们同样输入 git reflog 查到合并feature-A的哈希值为6f28d33
# 进行合并fix-B
$ git reset --hard 6f28d33
# 会报冲突，需要解决冲突，整个状态如下
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.

kanewang@KANEWANG-NGTKD MINGW64 /c/Workflow/GitHub/test (master|MERGING)
# 这个时候我们可以 git merge --abort来取消这个合并
# 我们此时打开README.md
vi README.md   #内容如下
# GIT

# GIT2
<<<<<<< HEAD

- feature-A
- feature-A-1
- feature-A-2
- test-1
=======
- fix-B
>>>>>>> fix-B
# 我们两个都要，删除掉多余的行并保存
# 之后 重新 git add 与 git commit
```

这次有了合并之后我们再看一下`git log --graph`的功能，终于有效果了。

```bash
$ git log --graph
*   commit 217ecb7bff4b945f6a8bbb9da7098463db178d62 (HEAD -> master)
|\  Merge: dfa1d86 e38c3ed
| | Author: KanePD <18842634041@163.com>
| | Date:   Thu Mar 14 21:43:05 2019 +0800
| |
| |     merge fix-B to master
| |
| * commit e38c3ede8cfe2b668ea61893691802b5a257be68 (fix-B)
| | Author: KanePD <18842634041@163.com>
| | Date:   Thu Mar 14 21:33:23 2019 +0800
| |
| |     Fix B
| |
* | commit dfa1d86e8c850a020a1edfe866095d8f89204048
| | Author: KanePD <18842634041@163.com>
| | Date:   Thu Mar 14 17:17:39 2019 +0800
| |
| |     test-1
| |
* | commit 0c74e08b9672eac7a36582d4abf5d3face77c83e
| | Author: KanePD <18842634041@163.com>
| | Date:   Thu Mar 14 17:16:58 2019 +0800
| |
| |     test-1
| |
* | commit 6f28d3388bbde22f273769a14596c8413f070116
| | Author: KanePD <18842634041@163.com>
| | Date:   Thu Mar 14 17:07:29 2019 +0800
| |
| |     Twice change for feature-A
| |
* | commit 4aff578e78688112e9c5d2a42af7da90da3310b5
|/  Author: KanePD <18842634041@163.com>
|   Date:   Thu Mar 14 15:13:06 2019 +0800
|
|       Add feature-A
|
* commit 72838f5f7c1943b51900c0f40a54ea28e1daf479 (feature-A)
| Author: KanePD <18842634041@163.com>
| Date:   Thu Mar 14 14:58:11 2019 +0800
|
|     Add index
|
* commit 274487c29eef98e8eaba26ec207075c396445a89
  Author: KanePD <18842634041@163.com>
  Date:   Thu Mar 14 14:07:33 2019 +0800

      First Commit
```

### git commit --amend ：修改提交信息

有时候提交的时候会出现一些小的纰漏，那么我们该怎么办呢

```bash
# 执行这个命令编辑器就会启动
$ git commit --amend
# 复制不了了 上图
```

图片

```bash
# 将merge fix-B to master 修改为 merge fix-B to master amend
# 保存并退出
[test1 3c1840a] merge fix-B to master amend
 Date: Thu Mar 14 21:43:05 2019 +0800
# git log 查看当前commit
$ git log
commit 3c1840aa293fc14c32dec47ec656e559512542f9 (HEAD -> test1)
Merge: dfa1d86 e38c3ed
Author: KanePD <18842634041@163.com>
Date:   Thu Mar 14 21:43:05 2019 +0800

    merge fix-B to master amend
# 信息已经更改
```

### git rebase -i ：压缩历史

有些时候如果发现了提交之后有错误， 我们再提交一个版本将错误更改了，但是就会有两次提交记录下来。我们可以将这两次提交合并。

```bash
# 创建一个新的branch feature-C
$ git checkout -b feature-C
# 我们再README.md中增加一个错误的单次
# faeture-C 留着以后哦更改
# 使用 git commit -am  代替git add 与git commit -m
$ git commit -am 'feature-C wrong spell'
# 这时发现了错误，我们再去修改它，并再提交一个版本
$ git commit -am 'feature-C correct wrong spell'
# 我们查看一下 git log，只列出feature-C的两个提交
$ git log
commit 24e5a4cbd1bb8bcec6cc93bbde0c755d38925268 (HEAD -> feature-C)
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 10:04:02 2019 +0800

    feature-C correct wrong spell

commit 5c8787c8bb5605c22d5a641e0ed45610f24ccc56
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 10:01:39 2019 +0800

    feature-C wrong spell
# 下面我们rebase一下 
# HEAD~2的意思是，拿最新的两个commit出来
$ git rebase -i HEAD~2
```

上图

```bash
# 将feature-C correct wrong spell前面的pick 改成fixup
# 将会把 feature-C correct wrong spell合并到 feature-C wrong spell上
# 我们在查看git log
$ git log
commit 5c8787c8bb5605c22d5a641e0ed45610f24ccc56
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 10:01:39 2019 +0800

    feature-C wrong spell
# 发现已经没有 feature-C correct wrong spell 的commit了 我们成功的清除了这个历史
```

上面的方法是书上的，下面组内一位大哥教的也可以实现上面的功能

```bash
# 下面我运用另一个方法将 testtest2合并到testtest1上，
$ git log -p
commit faeeba645a121e4b97dfa321cecab8d20b90fcf0 (HEAD -> feature-C)
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 11:25:37 2019 +0800

    testtest3

diff --git a/README.md b/README.md
index 479269c..bc5359c 100644
--- a/README.md
+++ b/README.md
@@ -16,3 +16,4 @@
 - test3:wq!
 - testtest1
 - testtest2
+- testtest3

commit 94c5e65dd4afbe17d042272208dbc574d949c818
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 11:25:25 2019 +0800

    testtest2

diff --git a/README.md b/README.md
index cf9e2be..479269c 100644
--- a/README.md
+++ b/README.md
@@ -15,3 +15,4 @@
 - test2
 - test3:wq!
 - testtest1
+- testtest2

commit 39249bfa6ed8cf878ccff11347f84138db6274ba
Author: KanePD <18842634041@163.com>
Date:   Fri Mar 15 11:25:06 2019 +0800

    testtest1

diff --git a/README.md b/README.md
index dc87501..cf9e2be 100644
--- a/README.md
+++ b/README.md
@@ -14,4 +14,4 @@
 - test1
 - test2
 - test3:wq!
-
+- testtest1

# 查看三个commit 每个commit 增加一行我要吧testtest2的commit合并到testtest1上
$ git rebase -i HEAD~3#进入rebase编辑模式
# 将testtest2前面的pick 修改为 edit 保存退出
# 此时会显示停在testtest2的commit上
$ git reset HEAD~1 #此处HEAD是reset到前两个commit点上 也是testtest1那个位置
#我们reset到test1的节点上,会提示未声明的改动
Unstaged changes after reset:
M       README.md
$ git add .将这部分的改动加进入
$ git commit --amend #进入修改commit界面，想要改commit的备注 可以修改，不修改的话直接退出
$ git rebase --continue #退出后 continue rebase
```

最后将feature-C合并到master上

```bash
$ git checkout master
Switched to branch 'master'
$ git merge feature-C
Updating 217ecb7..2b82757
Fast-forward
 README.md      | 10 +++++++++-
 "\357\274\201" | 18 ++++++++++++++++++
 2 files changed, 27 insertions(+), 1 deletion(-)
 create mode 100644 "\357\274\201"
```

## 将本地分支推送到远端分支上

### git remote  : 远程仓库操作

我们到github上找到 .git的ssh链接。我的是`git@github.com:KanePD/Self-Document.git`

```bash
$ git remote add kane git@github.com:KanePD/Self-Document.git
# 前面的kane是别名
$ git remote -v # 查看本地的远程仓库都有什么
$ git remote -v
kane    git@github.com:KanePD/Self-Document.git (fetch)
kane    git@github.com:KanePD/Self-Document.git (push)
```

### git push： 推送到远端仓库

 本地有了远端仓库之后，直接推送到远端仓库

```bash
$ git push kane master#将我本地的代码推到远端的master上
```

下面推送一个新的分支

```bash
# 如果远端没有这个分支，会在远端创建一个分支
$  git checkout -b feature-D
Switched to a new branch 'feature-D'
$ git push kane feature-D
Counting objects: 40, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (25/25), done.
Writing objects: 100% (40/40), 3.29 KiB | 54.00 KiB/s, done.
Total 40 (delta 7), reused 0 (delta 0)
remote: Resolving deltas: 100% (7/7), done.
remote:
remote: Create a pull request for 'feature-D' on GitHub by visiting:
remote:      https://github.com/KanePD/Self-Document/pull/new/feature-D
remote:
To github.com:KanePD/Self-Document.git
 * [new branch]      feature-D -> feature-D
# 由于我的本地master 与这个仓库的master不是一个东西，我没有真正的推master上去
```

远端的feature-D分支没有什么用，我们在本地删除它

```bash
$ git push kane --delete feature-D
To github.com:KanePD/Self-Document.git
 - [deleted]         feature-D
# 重新访问 github 远端的feature-D分支已经被删除了
```

## 从远程仓库获取

### git clone ：获取远程仓库

将远端的仓库的某个分支clone下来。

```bash
$ git clone git@github.com:KanePD/Self-Document.git
# git clone 后我们会默认处于master分支下，同事系统将会自动将origin 设置成远程仓库的标识。也就是说，当前本地仓库的 master 分支与 GitHub 端远程仓库（origin）的 master 分支在内容上是完全相同的
# 如果我们想拉取别的分支
# 我们首先查看远端的所有分支
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/test
# 我们将远端的test分支拉取下来
$ git checkout -b test origin/test
Switched to a new branch 'test'
Branch 'test' set up to track remote branch 'test' from 'origin'.
# 我们再查看一下，test分支已经拉取下来
$ git branch -a
  master
* test
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/test
```

### git pull ： 获取最新的远程仓库分支

```bash
$ git pull origin test
From github.com:KanePD/Self-Document
 * branch            test       -> FETCH_HEAD
Already up to date.
# 介绍一下 git pull --rebase
# git pull 的操作是git fetch+git merge
# git pull --rebase 的操作是 git fetch+git rebase
# 如果在拉取的情况下有冲突，使用git pull的话 会创建一个合并的commit
# 但是使用git pull --rebase之后就只是将自己的修改追加到远端的修改后面，不会创建新的commit
```

## fork 别人的分支进行开发

我fork一个第三方插件，额外说明一下，这个插件是验证BGP数据的，很好用的噢。

- 如下图，点击fork。点击后会等待一会儿。这个操作是将别人的仓库fork到我们自己的github上
- 如下图，成功将刚才的仓库，fork到我们的github中
- 我们将刚才fork的仓库，clone到本地

```bash
$ git clone git@github.com:KanePD/rpki-validator.git
```

- 拉取下来之后，在本地修改，推送到自己的远端的仓库里面去，
- 有修改之后，我们新建`pull request`，将自己仓库的master分支，合并到开源项目的master分支上即可 

## 一个IMBA的指令 gitk

直接上图，可以清楚的看到每个节点，节点之间的关系等。