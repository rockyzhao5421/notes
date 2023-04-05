Git 的分支，其实本质上仅仅是指向 commit 对象的可变指针，分支名就是指针名。

每一个指针都是一个文本文件，保存在.git/refs/heads/目录，例如 Notes 这个项目
只有一个 master 分支：

    [rocky@manjaro Notes]$ ls -l .git/refs/heads
    total 4
    -rw-r--r-- 1 rocky rocky 41  4月 3日 23:15 master
    
    
该文件的内容就是它所指向的快照的二进制对象名（哈希值）

    $ cat .git/refs/heads/master
    c102ba1069675e2090324bcff217bced03d471bc

# 创建分支

    $ git branch testing

这命令会创建一个分支，并指向当前分支指向的 commit 对象，也就是和你当前分支指向同一个 commit 对象。

    [rocky@manjaro Notes]$ git branch testing
    [rocky@manjaro Notes]$ git branch
    * master
    testing

这个命令只是创建分支，并不切换分支，所以现在当前分支还是 master。

# 切换分支

    $ git checkout testing

该命令让 HEAD 指向指定的分支：

    [rocky@manjaro Notes]$ git checkout testing
    Switched to branch 'testing'
    [rocky@manjaro Notes]$ git branch
    master
    * testing

![HEAD points to the current branch testing](branch-head-to-testing.png)


现在当前分支是 testing，如果再有提交， testing 会自动指向新的 commit，master 就不会了：

![](branch-master-stop.png)

需要注意的是 ```git log``` 默认情况下只会显示当前分支的 log，如果查其它分支的 log 就要加上
分支名： ```git log teting```，或者加上 ```--all``` 查所有分支的 log。

我们切回 master：

    [rocky@manjaro Notes]$ git checkout master
    Switched to branch 'master'
    Your branch is up to date with 'origin/master'.
    [rocky@manjaro Notes]$ git branch
    * master
    testing

checkout 的时候，该命令做了两件事情，第一是 HEAD 指向 master；第二是把工作目录里面的文件恢复成 master 分支所指向
快照的内容。

在切换分支时，一定要注意你工作目录里的文件会被改变。 如果是切换到一个较旧的分支，你的工作目录会恢复到该分支最后一次提交时的样子。 如果 Git 不能干净利落地完成这个任务，它将禁止切换分支。

如果我们在 master 和 testing 都有提交，那么现在就有了分叉：

![](branch-master-testing-is-diff.png)


# 创建新分支的同时切换过去

通常我们会在创建一个新分支后立即切换过去，这可以用 git checkout -b <newbranchname> 一条命令搞定。
