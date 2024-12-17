# Git 合并分支

在日常开发的工作中，只要使用git工具管理代码，就绕不开分支合并这一操作。面对分支的合并，发现几乎所有的人都会使用`git merge`命令。当合并结果并非期望时，手动修改代码，创建一个新的commit，并且提交到远程仓库。简单粗暴，行之有效。但几乎很少有人会关心合并分支过程中的细节，因为这对代码在生产环境的运行没有任何帮助。最终导致commit log信息乱七八糟，commit log成为了摆设，其他人很难从commit log层面管理项目的代码。本文将详细讲解合并分支中常用的`git merge`命令以及不经常用的`git rebase`命令。

## git merge

日常开发中，使用 `git merge <branch>`，会遇到三种场景：

### 1. 快进 - 无冲突

`被合并分支`是`当前分支`的直接后继，则合并命令直接把`当前分支的最新commit` `向前移动` 为 `被合并分支的最新commit`。

假设分支master有三个commit：B0、B1、B2。

```
B0---B1---B2(master)
```

<br />因为需要紧急修复线上问题，于是新建了分支fix，并且在分支fix上提交了两个commit：B3、B4。

```
            B3---B4(fix)
           /
B0---B1---B2(master)
```

<br />当在分支fix上的工作结束，切换到分支master，然后把分支fix合并到分支master：

```
git merge fix

Updating 4f6316b..2c975d7
Fast-forward
 footer.js | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 footer.js
```

由于分支fix最新commit（B4）是分支master最新commit（B2）的后继，因此Git会直接将指针向前移动（由B2移动到B4）。换句话说，当试图合并两个分支时，如果顺着一个分支走下去能够达到另一个分支，那么Git在合并两者的时候，只会简单的将指针向前推进（指针右移），因为这种情况下的合并操作没有需要解决的分歧————这就叫做“快进”（fast-forward）。

<br />合并结果如下：

```
B0---B1---B2---B3---B4(master)
```

<br />合并分支fix之后，已经不再需要该分支，可以删除该分支了。

```
git branch -d fix
```

> tips: 当不想使用快进合并，想让每次合并都有一个merge commit，可以使用`git merge <branch> --no-ff`。

### 2. 非“快进”，修改不同文件。（无冲突）

`被合并分支`并非`当前分支`的直接后继，无法“快进合并”，只能做“三方合并”，并且把合并的结果生成一个merge commit添加在`当前分支`上。

假设分支master有三个commit：B0、B1、B2。

```
B0---B1---B2(master)
```

<br />因为需要紧急修复线上问题，于是新建了分支fix，并且在分支fix上提交了两个commit（与分支master无代码冲突）：B3、B4。

```
            B3---B4(fix)
           /
B0---B1---B2(master)
```

<br />当在分支fix上开发时，另一位开发人员开发了新功能，并且已经合并入分支master，添加了新的commit（与分支master、分支fix都无代码冲突）：B5。

```
            B3---B4(fix)
           /
B0---B1---B2---B5(master)
```

<br />当在分支fix上的工作结束，切换到分支master，然后把分支fix合并到分支master：

```
git merge fix

Merge made by the 'ort' strategy.
 footer.js | 2 ++
 1 file changed, 2 insertions(+)
 create mode 100644 footer.js
```

<br />由于分支master最新commit B5不是分支fix最新commit B4的直接祖先，所以Git无法做“快进”（fast-forward）合并。
<br />Git会使用两个分支的最新commit（B4和B5）以及这两个分支的最新公共commit（B2），做一个简单的三方合并。并且把合并结果在分支master上创建一个新的commit（B6）。commit B3和commit B4的修改内容都体现在commit B6。

```
            B3----B4(fix)
           /        \
B0---B1---B2---B5---B6(master)
```

<br />合并完分支fix之后，已经不再需要该分支，可以删除该分支了。

```
git branch -d fix
```

### 3. 非“快进”，修改相同文件。（有冲突）

`被合并分支`并非`当前分支`的直接后继，无法“快进合并”，只能做“三方合并”。并且当“三方合并”时，产生了分支合并冲突，手动解决冲突，把合并的结果生成在merge commit上，添加在`当前分支`。

假设分支master有三个commit：B0、B1、B2。

```
B0---B1---B2(master)
```

<br />因为需要紧急修复线上问题，于是新建了分支fix，并且在分支fix上提交了两个commit：B3，B4。

```
            B3---B4(fix)
           /
B0---B1---B2(master)
```

<br />当在分支fix上开发时，另一位开发人员开发了新功能，并且已经合并入分支master，添加新的commit（与分支fix有代码冲突）：B5。

```
            B3---B4(fix)
           /
B0---B1---B2---B5(master)
```

<br />当在分支fix上的工作结束，切换到分支master，然后把分支fix合并到分支master：

```
git merge fix

Auto-merging footer.js
CONFLICT (add/add): Merge conflict in footer.js
Automatic merge failed; fix conflicts and then commit the result.
```

由于分支master最新commit B5与分支fix最新commit B4存在修改同一个文件的同一处代码，所以Git无法自动合并，需要开发者自己解决冲突并且提交结果。

打开有冲突的文件，Git已经标记出代码冲突：

```
<<<<<<< HEAD
B5
=======
B3
B4
>>>>>>> fix
```

手动整理该部分代码，解决冲突。

```
B3
B4
B5
```

添加该文件到暂存区，随后添加到分支master。

```
git add footer.js
git commit -m 'B6'
```

合并结果如下：

```
            B3----B4(fix)
           /        \
B0---B1---B2---B5---B6(master)
```

<br />合并分支fix之后，已经不再需要该分支，可以删除该分支了。

```
git branch -d fix
```

### squash

`git merge <branch> --squash`，用于将`被合并分支`上的若干差异commit整合为一条commit添加在`当前分支`上。

> 使用场景：当合并到当前分支后新增的多个commit意义不大，整合为一条commit更好。

假设分支master有三个commit：B0、B1、B2。

```
B0---B1---B2(master)
```

<br />因为需要紧急修复线上问题，于是新建了分支fix，并且在分支fix上提交了两个commit：B3，B4。

```
            B3---B4(fix)
           /
B0---B1---B2(master)
```

<br />当在分支fix上的工作结束，切换到分支master，然后把分支fix合并到分支master：

```
git merge fix --squash

Updating 98b9a62..044e72b
Fast-forward
Squash commit -- not updating HEAD
 index.js | 2 ++
 1 file changed, 2 insertions(+)
```

如果存在合并冲突，则手动解决冲突。当无合并冲突时，手动编写commit message，并且提交该squash commit之当前分支。

```
git commit -m 'B5'
```

合并结果如下：

```
B0---B1---B2---B5(master)
```

<br />合并分支fix之后，已经不再需要该分支，可以删除该分支了。

```
git branch -d fix
```

## git rebase

`git rebase`，重新整理commit提交记录。该命令主要有两个应用的维度：1. 分支维度；2. commit维度，下面详细来看看。

### git rebase \<branch\>

命令：`git rebase <branch>`

`git rebase <branch>`，以分支为维度，重新整理commit提交记录：在`当前分支`，以`被合并分支`为基底，重新应用`被合并分支`与`当前分支`之间的差异commit。

> 使用场景：本地分支合并远程分支。

#### 命令的具体执行步骤：

1. 找到当前分支（local branch）与被合并分支（remote branch）之间的最新公共祖先commit（B2）。

```
            B5(remote branch)
           /
B0---B1---B2---B3---B4(local branch)
```

2. 在当前分支中，从commit B2之后的commit B3开始到最新的commit B4都取消掉，并且把这些提交记录临时保存为补丁（patch）。

```
B0---B1---B2(local branch)
```

3. 因为当前分支是被合并分支的直接后继，所以直接把当前分支更新为被合并分支。最后把patch应用到当前分支上，得到commit B3\`和commit B4\`（因为commit id的相对位置发生变化，所以需要更换新的commit id）。

```
B0---B1---B2---B5(local branch)

B0---B1---B2---B5---B3`---B4`(local branch)
```

> tips: 当合并有冲突时，需要先解决冲突、git add、git commit，然后再执行`git rebase --continue`继续rebase流程。

通过对比[git merge](#2-非快进修改不同文件无冲突)：

```
            B3----B4(fix)
           /        \
B0---B1---B2---B5---B6(master)
```

可以看出：git rebase合并结果未产生多余的merge commit，就好像直接在合并分支上提交commit一样。

> 警告使用：不要轻易在公共分支上进行rebase。因为rebase会修改公共部分的commit history，可能给其他开发者造成困扰，除非能控制公共分支rebase之后的风险！

### git rebase -i \<commit id\>

命令：`git rebase -i <COMMIT ID>`，进入交互模式。可以修改COMMIT之后的所有commit信息（不包括COMMIT）。

`git rebase -i <commit id>`，以commit为维度，重新整理commit提交记录，其中包括：调整commit提交顺序、合并某些commit、删除某些commit、修改commit message、修改commit提交的代码等等。

> 使用场景：以单个commit为维度，修改commit。

执行该命令后，会使用vim打开交互模式的配置界面；该配置界面会按提交commit的时间顺序，从上到下依次列出COMMIT之后的所有commit。<br />
编辑完成之后，保存该配置，Git将会按照该配置中的commit命令顺序，从上至下，依次执行。

```
pick 1a38246 B1
pick 4f6316b B2
pick 98b9a62 B5

# 重定基底 e7e7452..98b9a62 到 e7e7452 （3个commit）
#
# 命令:
# p, pick <commit> = 使用该commit，保持现状，不做任何修改
#
# r, reword <commit> = 使用该commit，但可以编辑commit message
#
# e, edit <commit>（该命令可以使大型提交拆分为较小的提交，或者删除在提交中所做的错误更改）
#   1. 使用该commit，但rebase会暂停，等待编辑代码；
#   2. 编辑完代码之后，使用git add命令添加文件之暂存区；
#   3. 在暂存区，commit message为既有commit message，如果需要修改，可以使用git commit --amend；
#   4. 如有其他commit需要添加，可以使用git add、git commit命令添加commit；
#   5. 确认修改完毕之后，使用git rebase --continue，继续rebase。
#
# s, squash <commit>
#   1. 使用该commit，但会将commit的代码改动合并到上一个非squash的commit中；
#   2. 被合并的commit，将会被移除；
#   2. 提供编辑squash commit message的机会。
#
# f, fixup [-C | -c] <commit>
#   1. 和“squash”功能类似；
#   2. 但不会提供编辑squash commit message的机会；
#   3. squash commit message为合并的commit的commit message。
#
# x, exec <command> = 使用 shell 执行命令
#
# b, break = 在该commit处暂停，使用“git rebase --continue”命令继续rebase
#
# d, drop <commit> = 删除该commit
#
# l, label <label> = 为当前 HEAD 命名
#
# t, reset <label> = 重设 HEAD 到该标记
#
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified); use -c <commit> to reword the commit message
#
# 可以对这些commit行进行重新排序，commit行将从上至下依次执行。
#
# 如果删除某一行的commit，该commit将在git log中移除。
#
# 然而，如果删除所有的commit行，rebase命令将会终止执行。
```

#### 使用场景 - 修改commit message

初始commit log：

```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)
```

初始文件index.js：

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

> 修改commit id为B2的commit message为“reword B2”。

找到commit id为B2的上一个commit id B1，进入rebase的交互模式：

```
git rebase -i B1
```

```
pick B2 B2
pick B3 B3
pick B4 B4

# Rebase B1..B4 onto B1 (3 commands)
#
# ......
#
```

将`pick B2 B2`改为`r B2 B2`：

```
r B2 B2
pick B3 B3
pick B4 B4

# Rebase B1..B4 onto B1 (3 commands)
#
# ......
#
```

保存修改后的配置之后，自动弹出编辑B2 commit message的配置：

```
B2

# Please enter the commit message for your changes. Lines starting
# ......
#
```

将commit message`B2`改为`reword B2`，保存并且退出：

```
reword B2

# Please enter the commit message for your changes. Lines starting
# ......
#
```

结果：原先commit B2的message被改写为“reword B2”，commit id被替换为“B2`”。为了保持commit id的唯一性，发生变化的commit之后的commit，其commit id会自动更换。

此时commit log被改为：

```
B0(B0)---B1(B1)---B2`(reword B2)---B3`(B3)---B4`(B4)
```

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ reword B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

#### 使用场景 - 编辑commit

初始commit log：

```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)
```

初始文件index.js：

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

> 修改commit id - B2：将添加的代码由“B2”改为“B2.5”、commit message改为“edit B2”；然后复制index.js，文件名为"index_copy.js"，commit message为“copy index.js”。

找到commit id为B2的上一个commit id B1，进入rebase的交互模式：

```
git rebase -i B1
```

```
pick B2 B2
pick B3 B3
pick B4 B4

# Rebase B1..B4 onto B1 (3 commands)
#
# ......
#
```

将`pick B2 B2`改为`e B2 B2`：

```
e B2 B2
pick B3 B3
pick B4 B4

# Rebase B1..B4 onto B1 (3 commands)
#
# ......
#
```

保存修改后的配置之后，分支最新的commit id暂停在B2，等待编辑commit id - B2：

```
B0(B0)---B1(B1)---B2(B2)
```

编辑index.js，将“B2”修改为“B2.5”之后，保存：

```
B0
B1
B2.5
```

使用命令`git add index.js`，将index.js保存至暂存区。

此时，暂存区的commit message仍为“B2”；使用命令`git commit --amend`编辑commit message：

将

```
B2

# Please enter the commit message for your changes. Lines starting
#
# ....
#
```

修改为

```
edit B2

# Please enter the commit message for your changes. Lines starting
#
# ....
#
```

复制index.js，重命名为“index_copy.js”；并且添加至暂存区，编辑commit message，最后提交commit：

```
git add index_copy.js

git commit -m 'add index_copy.js'
```

当编辑完成时，使用命令`git rebase --continue`，继续rebase。在后面rebase的过程中，如果遇到合并冲突，则需要手动解决。

结果：commit B2添加的代码“B2”被改写为“B2.5”，commit B2的message被改写为“edit B2”，commit id被替换为“B2`”；添加了新文件“index_copy.js”，其commit message为“add index_copy.js”。为了保持commit id的唯一性，发生变化的commit之后的commit，其commit id会自动更换。

此时commit log被改为：

```
B0(B0)---B1(B1)---B2`(edit B2)---B2a`(add index_copy.js)---B3`(B3)---B4`(B4)
```

```
B0   // you, 1 hours ago ∙ B0
B1   // you, 2 hours ago ∙ B1
B2.5 // you, 3 hours ago ∙ edit B2
B3   // you, 4 hours ago ∙ B3
B4   // you, 5 hours ago ∙ B4
```

#### 使用场景 - 合并若干commit

初始commit log：

```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)
```

初始文件index.js：

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

> 将commit id为B1、B2、B3的三个commit合并为一个，并且将commit message编辑为“squash B1, B2, B3”。

找到commit id为B1的上一个commit id B0，进入rebase的交互模式：

```
git rebase -i B0
```

```
pick B1 B1
pick B2 B2
pick B3 B3
pick B4 B4

# Rebase B0..B4 onto B0 (4 commands)
#
# ......
#
```

将`pick B2 B2`改为`s B2 B2`，`pick B3 B3`改为`s B3 B3`：

```
pick B1 B1
s B2 B2
s B3 B3
pick B4 B4

# Rebase B0..B4 onto B1 (4 commands)
#
# ......
#
```

保存修改后的配置之后，自动弹出编辑squash B1、B2、B3 commit message的界面：

```
# This is a combination of 3 commits.
# This is the 1st commit message:

B1

# This is the commit message #2:

B2

# This is the commit message #3:

B3

# Please enter the commit message for your changes. Lines starting
#
# ......
#
```

编辑commit message，输入“squash B1, B2, B3”，保存并且退出：

```
# This is a combination of 3 commits.
# This is the 1st commit message:

squash B1, B2, B3

# This is the commit message #2:

# B2

# This is the commit message #3:

# B3

# Please enter the commit message for your changes. Lines starting
#
# ......
#
```

结果： commit B2和commit B3的代码改动被合并进commit B1，然后移除了这两个commit；commit B1的commit message被改写为“squash B1, B2, B3”，commit B1的 commit id被替换为“B1`”。为了保持commit id的唯一性，发生变化的commit之后的commit，其commit id会自动更换。

此时commit log被改为：

```
B0(B0)---B1(squash B1, B2, B3)---B4`(B4)
```

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ squash B1, B2, B3
B2 // you, 2 hours ago ∙ squash B1, B2, B3
B3 // you, 2 hours ago ∙ squash B1, B2, B3
B4 // you, 5 hours ago ∙ B4
```

#### 使用场景 - 删除若干commit

初始commit log：

```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)
```

初始文件index.js：

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

> 将commit id为B1、B3的commit移除。

找到commit id为B1的上一个commit id B0，进入rebase的交互模式：

```
git rebase -i B0
```

```
pick B1 B1
pick B2 B2
pick B3 B3
pick B4 B4

# Rebase B0..B4 onto B0 (4 commands)
#
# ......
#
```

将`pick B1 B1`改为`d B1 B1`，`pick B3 B3`改为`d B3 B3`：

```
d B1 B1
pick B2 B2
d B3 B3
pick B4 B4

# Rebase B0..B4 onto B1 (4 commands)
#
# ......
#
```

当保存修改后的配置，Git将自动从上至下执行rebase命令。如果遇到合并冲突，则需要手动解决。

结果：commit B1和commit B3被移除了。为了保持commit id的唯一性，发生变化的commit之后的commit，其commit id会自动更换。

此时commit log被改为：

```
B0(B0)---B2`(B2)---B4`(B4)
```

```
B0 // you, 1 hours ago ∙ B0
B2 // you, 2 hours ago ∙ B2
B4 // you, 5 hours ago ∙ B4
```

#### 使用场景 - 更改commit提交顺序

初始commit log：

```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)
```

初始文件index.js：

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

> 将commit id为B2、B3的两个commit提交顺序相互对调。

找到commit id为B2的上一个commit id B1，进入rebase的交互模式：

```
git rebase -i B1
```

```
pick B2 B2
pick B3 B3
pick B4 B4

# Rebase B1..B4 onto B1 (4 commands)
#
# ......
#
```

将`pick B2 B2`和`pick B3 B3`对调行位置：

```
pick B3 B3
pick B2 B2
pick B4 B4

# Rebase B1..B4 onto B1 (4 commands)
#
# ......
#
```

当保存修改后的配置，Git将自动从上至下执行rebase命令，所以先提交commit B3，然后再提交commit B2。如果遇到合并冲突，则需要手动解决。

结果：commit B3在前，commit B2在后。为了保持commit id的唯一性，发生变化的commit之后的commit，其commit id会自动更换。

此时commit log被改为：

```
B0(B0)---B3`(B3)---B2`(B2)---B4`(B4)
```

```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

## 总结

合并分支分为两大命令：`git merge` 和 `git rebase`。

`git merge`，不会修改即有commit，只会新增commit：

- 当被合并分支为合并分支的直接后继时，采用快进合并（fast-forward），效果相当于在合并分支上提交这些差异的commit。
- 当被合并分支不是合并分支的直接后继时，采用三方比较合并，产生一个额外的merge commit；冲突合并记载在merge commit。
- 当需要产生一个额外的merge commit时，使用命令`git merge <branch> --no-ff`。
- `git merge <branch> --squash`，将多个commit压缩为一个commit，提交到合并分支上。

`git rebase`，会修改即有commit，分为两个修改维度：

- 分支维度 - 在`当前分支`，以`被合并分支`为基底，重新应用`被合并分支`与`当前分支`之间的差异commit。
- commit维度 - 以单个commit为对象，对commit进行全方面的修改。
