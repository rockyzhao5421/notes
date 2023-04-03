# git add

add 命令用于把当前时间点工作目录下的更改加到 stage。

    add parameter <path>

### path

这里的 path 可以是文件，也可以是文件夹, 文件夹的话是包括子目录的。

可以给文件名加通配符，也可以指定文件夹(例如：```add subdir/file1``` )，指定文件夹的时候，就是把该文件夹下的所有文件更改作为一个整体添加到 stage，包括更新、新增 和删除，旧版本的 git 会忽略删除的文件，对于新版本的 git，如果想只添加更新和新增文件，忽略删除文件就要用 ```--no-all``` 。

这个命令只是把**当时**的 blob 对象的名字加入 index 文件，所以每次 commit 之前，如果有任何更改，都要再次 add。在 commit 之前可以多次执行该命令。

默认情况下，add 命令不会加被忽略的文件。如果在 add 命令后面显式指定一个被忽略的文件，add 就会失败，并列出忽略文件列表；如果用通配符匹配到忽略文件，或者文件夹中包含了忽略文件，那么这些被忽略的文件就会被直接忽略掉，add 并不会失败，也不会打印出忽略列表，如果想加忽略文件就要用 ```-f```。

如果没有 path, 表示当前路径 . 。

### parameter

-u: 是把已经跟踪文件的更新加入 stage。

-A：是把已经跟踪文件和没有跟踪的文件（新增文件）的变更都加入 stage。


好，下面用 add 命令把所有文件加入 stage：

    $ git add .

    $ git status
    On branch master

    No commits yet

    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
        new file:   file1
        new file:   subDir/subfile1

可以看到新增文件被加入了，所以默认是 -A。

    $ ls -l .git
    -rw-r--r-- 1 rocky rocky  184  3月12日 21:34 index

可以看到多了 index 文件。

再看看 index 文件的内容：

    $ git ls-files -s
    100644 a29bdeb434d874c9b1d8969c40c42161b03fafdc 0	file1
    100644 a29bdeb434d874c9b1d8969c40c42161b03fafdc 0	subDir/subfile1

所有变动的文件——实际就两个文件，都已记录在案了。

$ ls .git/refs/heads/
$ 

现在没有任何分支。