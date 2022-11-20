# Git 工具 - 贮藏

当在项目中开发一些需求，会新增文件/代码、删除文件/代码，这些改动首先存在于工作区，部分改动会添加到暂存区。所有的东西都进入了混乱的状态，如果这时需要切换到另一个分支或者同步远程分支去做一些其他需求（例如，临时修复线上bug），就相当麻烦。一般地做法，可能是先commit这些中间状态的代码，后期再恢复到该分支继续开发，但这样做会额外新增一个无意义的commit，而且所有的改动都被添加到了commit里了，与离开分支时的多种实际状态不相符。幸好，Git提供了临时贮藏的命令`git stash`，接下来就详细介绍一下：

命令`git stash`作用：将当前分支中的不想commit的改动内容，保存至stash堆栈中；后续可以在某个分支上恢复出stash堆栈中的改动内容。

## 实际应用

### 初始化状态

文件index.js；原先有代码“B0”和“B1”；添加了代码“B2”，并且添加到staging area；添加了代码“B3”，停留在working directory：
```
B0 // local repository
B1 // local repository
B2 // staging area
B3 // working directory
```

文件index_copy.js；是新增文件，并且有代码“B0”、“B1”、“B2”、“B3”，都在working directory：
```
B0 // working directory
B1 // working directory
B2 // working directory
B3 // working directory
```

### 保存当前状态

通过命令`git stash`，贮藏当前状态：
```
git stash
```

通过命令`git status`，查看当前状态：
```
git status

On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index_copy.js

nothing added to commit but untracked files present (use "git add" to track)
```

通过的上面的状态信息可以看出，文件“index_copy.js”未被贮藏。

> 默认情况下，stash只会贮藏被Git跟踪的文件的代码修改；不会贮藏未被Git跟踪的文件。使用命令`git stash -a`，可以将所有的文件都贮藏，包括未追踪的文件、被git忽略的文件；使用命令`git stash -u`，可以将未追踪的文件进行贮藏，但排除被git忽略的文件。因为项目开发中，node_modules目录中有大量的文件，在开发过程中可能会有修改，但这些文件并非期望被贮藏，所以建议使用命令`git stash -u`。

通过命令`git stash -u`，贮藏当前状态下所有的改动：
```
git stash -u

Saved working directory and index state WIP on master: 1a38246 B1
```

通过命令`git stash list`，可以展示出stash堆栈列表中所有的贮藏信息：
```
git stash list
stash@{0}: WIP on master: 1a38246 B1
```

通过上面的stash list可以看出，贮藏的信息仅包含branch name、last commit id、last commit message信息。可以通过命令`git stash -u -m <message>`，为该条贮藏添加注释message：
```
git stash -u -m "temporary storage"
Saved working directory and index state On master: temporary storage

git stash list
stash@{0}: On master: temporary storage
```

此时再通过命令`git status`，查看当前状态：
```
git status

On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

通过上面信息得知：没有东西需要commit，工作树也是干净的。

再查看文件index.js，只有commit的代码：
```
B0 // local repository
B1 // local repository
```

文件index_copy.js被删除。

命令执行结果，符合预期。

### 查看stash信息

通过命令`git stash list`，可以看到每个stash的信息；stash list采用先进后出的顺序，最后stash的改动，位于第一位：
```
git stash list

stash@{0}: On master: storage untrack file and ignored file
stash@{1}: On master: storage track file
```

通过命令`git stash show`，可以查看stash list中最新保存的stash与当前目录的概要差异：
```
git stash show

 index.js | 1 +
 1 file changed, 1 insertion(+)
```

通过命令`git stash show -p`，可以查看stash list中最新保存的stash与当前目录的详细差异：
```
git stash show -p

diff --git a/index.js b/index.js
index d6ff485..481c108 100644
--- a/index.js
+++ b/index.js
@@ -1,2 +1,3 @@
 B0
 B1
+B4
```

通过命令`git stash show stash@{n}`，可以查看stash list中指定序号为n的stash与当前目录的概要差异：
```
git stash show stash@{1}

 index.js | 2 ++
 1 file changed, 2 insertions(+)
```

通过命令`git stash show stash@{n} -p`，可以查看stash list中指定序号为n的stash与当前目录的详细差异：
```
git stash show stash@{1}

diff --git a/index.js b/index.js
index d6ff485..f189f43 100644
--- a/index.js
+++ b/index.js
@@ -1,2 +1,4 @@
 B0
 B1
+B2
+B3
```

### 应用stash存储的内容

stash存储的内容，可以恢复到任何分支，不仅仅是保存stash时的分支。

命令`git stash apply`，将堆栈中栈顶位置的stash的内容应用到当前目录。

命令`git stash apply <stash>`，将堆栈中指定stash的内容应用到当前目录。

命令`git stash pop`与`git stash apply`的区别在于：在应用到目录之后，会立即从stash list中删除stash。

#### 恢复保存stash时状态

在分支master上应用[保存stash时的状态](#保存当前状态)。

通过命令`git stash apply`，恢复状态：
```
git stash apply

On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index_copy.js

no changes added to commit (use "git add" and/or "git commit -a")
```

通过对比[保存stash时的状态](#保存当前状态)，可以看出区别：文件index.js中的代码“B2”，本应该位于staging area，但却位于working directory。

> 命令`git stash ( pop | apply )`只能将贮藏的内容恢复到working directory；通过添加参数“--index”，可以将内容恢复到staging area。

通过命令`git stash apply --index`，完全恢复到[保存stash时的状态](#保存当前状态)。
```
git stash apply --index

On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   index.js

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   index.js

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        index_copy.js
```

通过上面的信息可以看到，当前的状态已经完全恢复。

### 删除stash

命令`git stash drop`，从stash list中删除堆栈中栈顶位置的stash。

命令`git stash drop <stash>`，从stash list中删指定stash。

命令`git stash clear`，将stash list清空。

## 总结

命令`git stash`高效地解决了临时存储开发状态的需求。

新增：`git stash`，只针对Git跟踪的文件；`git stash -u`，可以贮藏未跟踪的文件的改动；`git stash -a`，可以贮藏所有文件的改动，包括被Git忽略的文件改动；参数`-m <message>`，可以为stash添加注释message。

查看：`git stash list`，查看stash list；`git stash show <options> <stash>`，可以查看指定stash与当前目录的差异。

应用：`git stash ( pop | apply ) --index <stash>`，可以将指定stash应用到当前目录。

删除：`git stash drop <stash>`、`git stash clear`，可以删除指定的stash，或者清除整个stash list。

## 参考

[git-stash](https://git-scm.com/docs/git-stash)
