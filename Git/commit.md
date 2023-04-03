# git commit

该命令把 state 里面的内提交到 repository 中， 还可以同过 -m 设置 comment。提交之后，HEAD 就会自动指向它。

下面内容都可以 commit：

### 通过 add 加入到 stage 的内容。



### 通过 git rm 把某一个或者多个文件从工作目录和 stage 同时删除的改动。

先把 file1 删除：

    $ git rm file1
    rm 'file1'
    $ git status
    On branch master
    Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
        deleted:    file1

    $ ls
    subDir

顺便练习一下 git restore 命令把状态恢复，现在 stage 里也没 file1 了，所以从 stage 恢复是不行的：

    git restore file1
    error: pathspec 'file1' did not match any file(s) known to git

之前有 commit，那么就从 HEAD 恢复试试：

    $ git restore -s@ -SW file1
    $ ls
    file1  subDir
    $ git status
    On branch master
    nothing to commit, working tree clean

现在工作目录、stage、repository 都是一致的了，所以是 clear 状态。

### 把要 commit 的文件罗列在参数列表中

当然不能有 --interactive 或者 --pathc，这种方式的话就会忽略掉 staged 中的变化，只 commit 当前这些文件的内容
，statge 里面已经 add 的更改和工作目录还没 add 的更改都会提交（相当于用了 -a），当然这些文件一定是已经 tracked 的文件。 其它没有罗列的文件在 stage 里面的更改就不会提交。

例如，现在 file1 一个改动在 stage，一个在工作目录；stage里面还有 subfile1 的更改：

    $ git status
    On branch master
    Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
        modified:   file1
        modified:   subDir/subfile1

    Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git restore <file>..." to discard changes in working directory)
        modified:   file1

只提交 file1 后，file1 两处的更改都提交了，但是 subfile1 还在 stage 里面：

    $ git commit file1 -m 'fifth commit'
    [master 1e01639] fifth commit
    1 file changed, 2 insertions(+)
    [rocky@rocky-pc gitCommandStudy]$ git status
    On branch master
    Changes to be committed:
    (use "git restore --staged <file>..." to unstage)
        modified:   subDir/subfile1

### -a

这个选项告诉 commit 命令，自动 add 文件，然后再 commit。该命令只对已经 track 的文件有效，对新增的文件无效。


### options

**```-C <commit>, --reuse-message=<commit>```**

就是在一个已存在的 commit 上追加提交，comment、timestamp、作者什么的都不变。

**```-c <commit>, --reedit-message=<commit>```**

和上面大写 C 差不多的作用，只是用打开一个编辑器让你有机会改一下 comment。

**```--amend ```**

用新的 commit 替换当前分支的 head，comment、 timestamp、 作者什么的都不变。