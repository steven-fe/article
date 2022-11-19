# Git 撤销commit

当某个功能开发完毕之后，需要及时提交commit到当前分支。为了降低代码丢失的风险，需要在恰当的时候把当前分支推送到远程分支。然而，在某些场景下（比如：向分支提交了错误的代码），需要从分支中撤销某个commit的提交。 面对这种实际需求，Git提供了两个命令：`git revert`和`git reset`。下面详细介绍这两个命令的使用以及区别。

## git revert

命令：`git revert <commit id>`

创建revert commit，其内容为反转某个要撤销commit所引入的更改，向当前分支添加该commit。

原理：不是真正地撤销某个commit，而是利用互补原理，反转该commit所引入的更改。

### 使用场景 - 撤销某个非merge commit的commit

初始commit log：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')---B5(B5)
```
初始文件index.js：
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
B5 // you, 6 hours ago ∙ B5
```

> 撤销commit id为B5的commit，删除所提交的代码“B5”

找到提交代码“B5”的commit的commit id，然后执行revert命令：
```
git revert B5
```

执行命令之后，会自动弹出编辑revert commit message：
```
Revert "B5"

This reverts commit B5.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Your branch and 'origin/master' have diverged,
# and have 5 and 8 different commits each, respectively.
#   (use "git pull" to merge the remote branch into yours)
#
# Changes to be committed:
# modified:   index.js
```

保存&退出revert commit message之后，revert commit就会自动添加在当前分支的末端。

结果：在当前分支添加revert commit，“B5”代码被删除。

此时commit log被改为：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')---B5(B5)---B6(Revert 'B5')
```
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

### 使用场景 - 撤销某个merge commit

初始commit log：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')---B5(B5)
```
初始文件index.js：
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
B5 // you, 6 hours ago ∙ B5
```

> 撤销commit id为B4`的commit，删除所提交的代码“B2”、“B3”、“B4”

找到merge commit的commit id，然后执行revert命令：
```
git revert B4`
```

执行命令之后，打印出错误的提示：
```
error: commit B4` is a merge but no -m option was given.
fatal: revert failed
```

通过查询[相关文档](https://git-scm.com/docs/git-revert#Documentation/git-revert.txt--mparent-number)了解到：无法直接revert merge commit；因为merge commit的父级commit有多个，Git无法判断要使用哪个父级commit（请观察上面的“初始commit log”，commit B4\`有两个父级commit：1. commit - B1，2. commit - B4）；使用命令`git revert <commit id> -m <parent number>`，决定反转到哪个父级commit；parent number是自然数，以1开始，具体的parent number与commit id映射关系，可以查看merge commit - log中的merge字段。

通过命令`git log <commit id>`，查看merge commit的具体信息：
```
git log B4`

commit B4`
Merge: B1 B4
Author: ......
Date:   ......

    Merge branch 'fix'

......

```

通过上面的log信息，可以看出：parnet number为1的分支，其commit id为B1；parent number为2的分支，其commit id为B4。<br />
因为需要“撤销commit id为B4\`的commit，删除commit id为B4\`的commit所提交的代码‘B2’、‘B3’、‘B4’”，所以需要让merge commit反转回到commit B1，执行命令：
```
git revert B4` -m 1
```
如果遇到合并冲突，则需要手动解决。提交revert commit message之后，revert commit就会自动添加在当前分支的末端。

结果：在当前分支添加revert commit，删除commit id为B4`的commit所提交的代码“B2“、“B3“、“B4“。

此时commit log被改为：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')---B5(B5)---B6(Revert "Merge branch 'fix'")
```
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B5 // you, 6 hours ago ∙ B5
```

### 使用场景 - 撤销多个commit

初始commit log：
```
B0(B0)---B1(B1)---B2(B2)---B3(B3)---B4(B4)---B5(B5)
```

#### 多个连续的commit

```
git revert -n startCommitId^..endCommitId
```

> 将[startCommitId, endCommitId]这个区间的commit id都撤销。

#### 多个不连续的commit

```
git revert -n commitId1
git revert -n commitId5
git revert -n commitId9
...
```

> 将commit id1、commit id5、commit id9这几个commit都撤销。

## git reset

命令：`git reset --hard <commit id>`

将当前分支的HEAD指向为`<commit id>`，`<commit id>`之后的commit都从git log中移除，已达到撤销commit的目的。

> 命令中的`--hard`参数，表明：将强制恢复到指定`<commit id>`时的状态。本地工作区、暂存区的修改都将被删除。

### 使用场景 - 撤销某个commit

初始commit log：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')---B5(B5)
```
初始文件index.js：
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
B5 // you, 6 hours ago ∙ B5
```

> 撤销commit id为B5，删除commit id为B5的commit所提交的代码“B5”

找到提交代码“B5”的commit的上一级commit的commit id，然后执行reset命令：
```
git reset --hard B4
```

执行上面的命令之后，Git自动将HEAD恢复到commit - B4，并且移除了commit - B5

此时commit log被改为：
```
            B2(B2)---B3(B3)---B4(B4)
           /                    \
B0(B0)---B1(B1)-----------------B4`(Merge branch 'fix')
```
```
B0 // you, 1 hours ago ∙ B0
B1 // you, 2 hours ago ∙ B1
B2 // you, 3 hours ago ∙ B2
B3 // you, 4 hours ago ∙ B3
B4 // you, 5 hours ago ∙ B4
```

## 总结：

git revert 和 get reset，这两个命令都可以达到撤销某个commit，删除该commit所提交更改的目的，但原理不一样：

- git revert通过反转的操作，达到目的；优点是，不改变commit log history，通过commit log可以清晰地看到撤销记录。
- git reset通过移除commit，达到目的；优点是，彻底删除某个commit；缺点是，改变了commit log history，不太适合在公共分支上进行操作，除非能承担git reset之后的风险。
