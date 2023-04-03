# git restore

这命令用指定的 source 来覆盖工作目录的文件或者 stage 里面的文件（索引指向的 blob 对象）。

### 指定要覆盖哪里

不带 --staged， 就是覆盖工作目录里面的文件。

带 --staged，就是覆盖 stage 里面的文件。

    -W, --worktree, -S, --staged

也可以用 -W 显式指定要覆盖工作目录的文件，也可以同时用 -WS 同时覆盖工作目录和 stage（有 -S 默认 source 就是 HEAD）。

### 指定 source

在不指定 source 的情况下， 带 --staged 默认就用 HEAD 覆盖 stage 里面的文件，不带 --staged 默认就用 stage 覆盖工作目录里面的文件。

    -s <tree>, --source=<tree>

也可以用 -s 显式指定 source。这里 tree 通常是 commit， branch，或者是 tag。

### 举例

下面的一系列操作，就是切换到 master 分支，然后把 Makefile 返回到前两个版本；删除 hello.c 文件，然后复制 stage 里面的版本到工作目录：

    $ git switch master
    $ git restore --source master~2 Makefile  (1)
    $ rm -f hello.c
    $ git restore hello.c                     (2)

如果要恢复所有 C 代码文件，可以用通配符：

    $ git restore '*.c'

注意单引号。另外尽管 hello.c 已经从工作目录中删除了，一样能恢复，因为通配符匹配的是 stage 里面的条目，而不是工作目录。

恢复当前目录的所有文件：

    $ git restore .

或者恢复整个工作目录文件：

    $ git restore :/

用 HEAD 里面的版本覆盖 stage 里某个文件（这和 git-reset 作用一样）：

    $ git restore --staged hello.c

也可以同时覆盖 stage 和工作目录（这和 git-checkout 作用一样）：

    $ git restore --source=HEAD --staged --worktree hello.c

或者用缩写形式，只是阅读性不强：

    $ git restore -s@ -SW hello.c

### 还有许多关于 merge 方面的用法以后再写