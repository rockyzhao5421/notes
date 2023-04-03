
先准备一个文件夹并初始化一个 repository 作为练习场所：

    $ ls -R .
    .:
    file1  subDir

    ./subDir:
    subfile1

外面的文件 file1 和 子文件夹里面的 subfile1 里面都只有一个字符串 'line1'

先看看 stage 初始状态：

    $ git status
    On branch master

    No commits yet

    Untracked files:
    (use "git add <file>..." to include in what will be committed)
        file1
        subDir/

    nothing added to commit but untracked files present (use "git add" to track)

看看 .git 目录：

    $ ls -l .git
    total 32
    drwxr-xr-x 2 rocky rocky 4096  3月 7日 21:02 branches
    -rw-r--r-- 1 rocky rocky   92  3月 7日 21:02 config
    -rw-r--r-- 1 rocky rocky   73  3月 7日 21:02 description
    -rw-r--r-- 1 rocky rocky   23  3月 7日 21:02 HEAD
    drwxr-xr-x 2 rocky rocky 4096  3月 7日 21:02 hooks
    drwxr-xr-x 2 rocky rocky 4096  3月 7日 21:02 info
    drwxr-xr-x 4 rocky rocky 4096  3月 7日 21:02 objects
    drwxr-xr-x 4 rocky rocky 4096  3月 7日 21:02 refs

    $ ls .git/refs/heads/
    $ 


没有 index 文件， heads 目录下也是空的，没有 master branch 对应的文件。

# git add

add 命令用于把当前时间点工作目录下的更改加到 stage。

    add parameter <path>

好，下面用 add 命令把所有文件加入 stage：

    $ git add .

    $ git status
    On branch master

    No commits yet

    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
        new file:   file1
        new file:   subDir/subfile1

可以看到新增文件也被加入了，所以默认是 -A。

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


# git commit

该命令把 state 里面的内提交到 repository 中， 还可以同过 -m 设置 comment。提交之后，HEAD 就会自动指向它。

我们现在就提交：

    git commit -m 'the first commit'
    [master (root-commit) 2e0913c] the first commit
    2 files changed, 3 insertions(+)
    create mode 100644 file1
    create mode 100644 subDir/subfile1

现在看看 status：

    $ git status
    On branch master
    nothing to commit, working tree clean

看看 branch 的信息，HEAD 指向 master

    $ cat .git/HEAD 
    ref: refs/heads/master

heades 文件夹下也多了 master 分支的文件，master 分支也指向上一个 commit。

    $ ls -l .git/refs/heads/
    total 4
    -rw-r--r-- 1 rocky rocky 41  3月19日 16:56 master

    $ cat .git/refs/heads/master 
    2e0913ce1c853b9945b57c0fde6dfa3d88fce58f


