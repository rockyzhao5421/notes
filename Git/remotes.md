# git clone 

    git clone <url> 

就是把远程 repository copy 一份到本地某一个文件夹内：

    $ git clone https://github.com/libgit2/libgit2
    Cloning into 'libgit2'...
    remote: Enumerating objects: 121125, done.
    remote: Counting objects: 100% (121125/121125), done.
    remote: Compressing objects: 100% (33188/33188), done.
    remote: Total 121125 (delta 86165), reused 120851 (delta 85928), pack-reused 0
    Receiving objects: 100% (121125/121125), 62.37 MiB | 2.60 MiB/s, done.
    Resolving deltas: 100% (86165/86165), done.
    $ ls
    gitCommandStudy  gitStudy  libgit2  React  ReactNative

默认是创建一个 libgit2 的文件夹。


如果要指定文件夹名，多加一个参数就行了：

    $ git clone https://github.com/libgit2/libgit2 libgit_special
    Cloning into 'libgit_special'...
    remote: Enumerating objects: 121125, done.
    remote: Counting objects: 100% (121125/121125), done.
    remote: Compressing objects: 100% (33188/33188), done.
    remote: Total 121125 (delta 86165), reused 120851 (delta 85928), pack-reused 0
    Receiving objects: 100% (121125/121125), 62.37 MiB | 5.83 MiB/s, done.
    Resolving deltas: 100% (86165/86165), done.
    $ ls
    gitCommandStudy  gitStudy  libgit2  libgit_special  React  ReactNative

# 查看 remotes

    git remote

是查看一个本地 repository 配置的 remote repository，比如我们刚 clone 的 libgit2:

    $ cd libgit2
    $ git remote
    origin

origin 是 Git 给 remotes 的默认短名，短名是为了方便使用。

加上 -v 就可以查看每个 remote 对应的 URL，这个 URL 就是你 pull/push 时候实际使用的。

如果一个本地 repository 有多个 remotes，会全部列出来：

    $ git remote -v
    bakkdoor  https://github.com/bakkdoor/grit (fetch)
    bakkdoor  https://github.com/bakkdoor/grit (push)
    cho45     https://github.com/cho45/grit (fetch)
    cho45     https://github.com/cho45/grit (push)
    defunkt   https://github.com/defunkt/grit (fetch)
    defunkt   https://github.com/defunkt/grit (push)
    koke      git://github.com/koke/grit.git (fetch)
    koke      git://github.com/koke/grit.git (push)
    origin    git@github.com:mojombo/grit.git (fetch)
    origin    git@github.com:mojombo/grit.git (push)

这样的话，我们可以从它们中的任意一个 pull，在有权限的情况下，也可以往任意一个或者多个 push。

# 添加 remote

    git remote add <shortname> <url>

给 libgit2 加一个 remote repository：

    $ git remote add pb https://github.com/paulboone/ticgit
    $ git remote -v
    origin	https://github.com/libgit2/libgit2 (fetch)
    origin	https://github.com/libgit2/libgit2 (push)
    pb	https://github.com/paulboone/ticgit (fetch)
    pb	https://github.com/paulboone/ticgit (push)

现在在命令行执行命令的时候，就可以用 pb 了，就不用打出整个 URL 了：

    $ git fetch pb
    remote: Enumerating objects: 634, done.
    remote: Total 634 (delta 0), reused 0 (delta 0), pack-reused 634
    Receiving objects: 100% (634/634), 88.93 KiB | 929.00 KiB/s, done.
    Resolving deltas: 100% (261/261), done.
    From https://github.com/paulboone/ticgit
    * [new branch]          master     -> pb/master
    * [new branch]          ticgit     -> pb/ticgit

# fetch / pull

    $ git fetch <remote>

这个命令就是去 remote 获取所有本地没有的数据，执行命令之后本地就获取了 remote 的**所有分支**。

如果你 clone 了一个 repository，clone 命令默认把这个 remote 命名为 origin，所以执行
```git fetch origin``` 会把 clone 之后（或者上次 fetch 之后）remote 上所有的更改都
拉下来。

有一点非常重要，fetch 命令只会把 remote 的数据拉到本地 repository，它并不会和你当前分支
合并，你只能自己手动合并。pull 命令就不一样了，它拉取你当前分支对应的远程分支之后，会自动合并到当前分支。

默认情况下， clone 命令自动把本地的 master 分支设置成跟踪 remote 的 master 分支（其实是跟踪 remote 的默认分支，这里假设默认分支是 master）


# push

    git push <remote> <local branch>:<remote branch>

如果本地分支名和远程分支名相同，则可以省略 "```:<remote branch>```" :

    git push <remote> <local branch>

如果你要 push 当前分支到远程的同名分支，连```<local branch>```都可以省略：

    git push <remote>

如果你只配置了一个 remote， 上面 push 当前分支到远程同名分支的情况，连 remote 都可以不要：

    git push

例如：你想把本地的 master 分支 push 到 origin，运行下面命令就行了：

    $ git push origin master


但是前提是你有这个 remote 的写权限，并且在这期间没有其他人 push 过，如果有人 push 过，你必须先把他的更新拉下来并且合并。

# 查看 remote 详情

    git remote show <remote>

查看某个 remote 的详细信息：

1. 列出本地分支会 push 到远程哪个分支。
2. 列出本地哪些分支在 pull 的时候可以和它跟踪的分支合并。
3. 列出本地没有，远程有的分支。
4. 列出本地有，远程已经删除的分支。



        $ git remote show origin
        * remote origin
        Fetch URL: https://github.com/libgit2/libgit2
        Push  URL: https://github.com/libgit2/libgit2
        HEAD branch: main
        Remote branches:
            bindings/libgit2sharp/022_1                    tracked
            brianmario/attr-from-tree 
        ……

        Local branch configured for 'git pull':
            main merges with remote main
        Local ref configured for 'git push':
            main pushes to main (local out of date)

# 删除 / 重命名 remote

### 重命名

就是改 remote 的短名，例如把 pb 改为 paul:

    [rocky@rocky-pc libgit2]$ git remote rename pb paul
    Renaming remote references: 100% (2/2), done.
    [rocky@rocky-pc libgit2]$ git remote
    origin
    paul
    [rocky@rocky-pc libgit2]$ 

值得注意的是，原来所有跟踪 pb/xxx 的分支都会改为 paul/xxx，例如原来的 pb/master 现在就改成了
paul/master。

### 删除

    git remote remove <remote> or git remote rm <remote>

一旦用这种方式删除，对应正在跟踪这个 remote 分支都会删除，针对这个 remote 的那些配置也会删除。