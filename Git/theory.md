
# 原理 

### 集中 VS 分布

SVN 是集中式管理，repository 在服务器上，client 拉下来的只是快照。

Git 是分布式管理，从技术上讲，没有 server 和 client 区分，每台电脑上都有一个完整的 repository，你也可以把代码 push 到其它任何一台电脑上。

这样做好处很多。本地有完整的 repository，首先很多操作都可以在本地完成，比如查看 history，比较代码；每个 repository 都是一个完整的备份，也降低了数据丢失的风险。

### diff VS 快照

SVN 保存的是：基础文件 + 一系列的修改；

Git 保存的是：所有文件的快照；

每次提交更新，Git 对那个时间点的全部文件制作一个快照并保存这个快照的索引。如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个快照流。

### Checksum

在Git 中，所有数据在存储前都会用数据内容生成一个哈希值，这样其它地方就可以用它校验数据的完整性、一致性。这意味着不可能在 Git 不知情时更改任何文件内容或目录内容，在传送过程中丢失信息或损坏文件，Git 也能发现。

Git 中使用这种哈希值的情况很多，经常看到这种哈希值。 实际上，Git 数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

### 三种状态

+ modified
+ staged
+ committed

对应的有三个区域

+ Working Directory
+ Staging Area 暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 .git 目录中。 有时候也被称作‘索引’，不过一般说法还是叫暂存区域
+ Repository(.git/)

基本的 Git 工作流程如下：

1. 在工作目录中修改文件，这时候就是 modified 状态。
2. 然后将修改过文件的快照加入暂存区域，这时候就是 staged 状态
3. 最后将暂存区域的快照提交到本地 repository，就是 committed 状态。

# 配置

git 的配置文件有三个，优先级从低到高：

    /etc/gitconfig //系统级
    ~/.gitconfig //用户级
    某一个 project/.git/config //项目级

可以直接修改对应的文件，也可以使用 config 命令来设置：

    $ git config --global user.name "rocky"
    $ git config --global user.email rockykzhao@foxmail.com

+ --system 对应系统级配置文件
+ --global 对应用户级配置文件
+ --local  对应项目级配置文件
  
查看配置信息：

    git config -l

看帮助手册有三种方式：

    $ git help <verb>
    $ git <verb> --help
    $ man git-<verb>

例如，看 config 命令的手册：

    $ git help config

# 创建 repository

什么是 repository？ 就是一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

创建一个 repository 非常简单，创建一个空目录，然后进入这个目录运行：

    git init

git init命令只做一件事，就是在项目根目录下创建一个.git子目录，用来保存版本信息。

    $ ls -l .git
    total 32
    drwxr-xr-x 2 rocky rocky 4096  2月12日 18:08 branches
    -rw-r--r-- 1 rocky rocky   92  2月12日 18:08 config
    -rw-r--r-- 1 rocky rocky   73  2月12日 18:08 description
    -rw-r--r-- 1 rocky rocky   23  2月12日 18:08 HEAD
    drwxr-xr-x 2 rocky rocky 4096  2月12日 18:08 hooks
    drwxr-xr-x 2 rocky rocky 4096  2月12日 18:08 info
    drwxr-xr-x 4 rocky rocky 4096  2月12日 18:08 objects
    drwxr-xr-x 4 rocky rocky 4096  2月12日 18:08 refs

# 对象模型

在这个本地 repository 中存储了 git 的所有对象，git 有四种： blob、tree、commit 和 tag。它们的名字都是用 SHA-1 对其内容进行哈希，然后用这个哈希值命名。可以用 git cat-file -t 查看对象类型，用 git cat-file -p 查看每个对象的内容。

### blob 

blob 在 git 中是最常用的对象，它用来存储文件数据，它里面只有文件内容，没有其它数据。先看一个列子：

    $ touch test
    $ git hash-object -w test
    e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
    $ 

上面我创建一个空文件 test，然后用 ```hash-object``` 命令把它压缩成一个 blob 对象（二进制文件）存储到 objects 目录下，这个命令还用文件内容计算除哈希值，用于命名：

    $ ls -R .git/objects

    .git/objects/e6:
    9de29bb2d1d6434b8b29ae775ad8c2e48c5391

可以看到，.git/objects下面多了一个子目录，目录名是哈希值的前2个字符，该子目录下面有一个文件，文件名是哈希值的后38个字符。

用 ```git show``` 命令查看一下文件内容，是空的：

    $ git show e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
    $

往里面写一些内容:

    $ echo 'hello world' > test

再次保存成 blob 对象到 objects 目录：

    $ git hash-object -w test
    3b18e512dba79e4c8300dd08aeb37f8e728b8dad

上面代码可以看到，随着内容改变，test 的哈希值已经变了。同时，新文件.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad 也已经生成了。现在可以看到文件内容了。

    $ git show 3b18e512dba79e4c8300dd08aeb37f8e728b8dad
    hello world

文件保存到 object，还需要通知 Git 哪些文件发生了变动。所有变动的文件，Git 都记录在一个文件中，叫做"暂存区"（英文叫做 index 或者 stage）：

```git update-index```命令用于在暂存区记录一个发生变动的文件:

    $ git update-index --add --cacheinfo 100644 \
    3b18e512dba79e4c8300dd08aeb37f8e728b8dad test

再看一下 .git 下面多了一个 index 文件，没错这个文件就是所谓的 stage，就是一个文件：

    $ ls -l .git
    total 36
    drwxr-xr-x 2 rocky rocky 4096  2月26日 16:13 branches
    -rw-r--r-- 1 rocky rocky   92  2月26日 16:13 config
    -rw-r--r-- 1 rocky rocky   73  2月26日 16:13 description
    -rw-r--r-- 1 rocky rocky   23  2月26日 16:13 HEAD
    drwxr-xr-x 2 rocky rocky 4096  2月26日 16:13 hooks
    -rw-r--r-- 1 rocky rocky  104  2月26日 16:44 index
    drwxr-xr-x 2 rocky rocky 4096  2月26日 16:13 info
    drwxr-xr-x 6 rocky rocky 4096  2月26日 16:30 objects
    drwxr-xr-x 4 rocky rocky 4096  2月26日 16:13 refs
    $ 

```git ls-files```命令可以查看这个文件的内容：

    $ git ls-files --stage
    100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0	test

这四列分别是：访问权限、对象名、stage number 和 目录/文件名。其中最重要的是这个对象名，git 就是通过这个名字来读取 objects 目录里面的文件内容。

保存对象、在暂存区记录对象的索引，git 把这两个步骤合并为一个命令  ```git add```, 执行 ```add``` 命令就相当于执行了这两个步骤。

再 test 里面加一行内容：

    $ echo 'second line'>>test
    $ git add test

多了一个 blob 对象：

    $ ls -R .git/objects
    .git/objects:

    // 第一次修改生成的对象
    .git/objects/3b:
    18e512dba79e4c8300dd08aeb37f8e728b8dad

    //空 test 对应的对象
    .git/objects/e6:
    9de29bb2d1d6434b8b29ae775ad8c2e48c5391

    // 最后一次修改生成的 blob 对象
    .git/objects/f0:
    e9ea9cdcd1c7d022373365088a65e94c5ab13e

但是 index 中的索引只保留最新的一条：

    $ git ls-files -s
    100644 f0e9ea9cdcd1c7d022373365088a65e94c5ab13e 0	test

同一个文件，多次修改， objects 目录可以同时存在多个 blob 文件，但是同一个文件在 index 文件中只会保留最新的一条索引。

### tree, commit 

为了描述更清楚，我们在 gitStudy 下再建立一个子文件夹 subdir，并在里面创建一个新的文件 subfile1 并加到 stage:

    $ mkdir subdir
    $ touch subdir/subfile1
    $ echo 'first line'>subdir/subfile1 
    $ git add subdir/subfile1 

再看看 index 的内容， subfile1 已经加到 stage：

    $ git ls-files -s
    100644 08fe2720d8e3fe3a5f81fbb289bc4c7a522f13da 0	subdir/subfile1
    100644 f0e9ea9cdcd1c7d022373365088a65e94c5ab13e 0	test

不用再看 objects 目录也已经保存了对应的 blob 对象。

但是到目前为止，objects 里面只是保存单个文件的 blob 对象，blob 对象只是保存了文件的内容，里面并没有文件的路径信息， 即目录结构，index 文件才有这些信息。

```git write-tree```：这个命令就是根据当前 index 文件中的文件路径，生成一些相应的 tree对象，并保存到 objects 目录下， tree 里面就记录了目录结构。

    rocky@rocky-pc gitStudy]$ git write-tree
    79c830489e29e52b09bcd88837e6af96574f17c4

在看看 objects 里面的变化：

    $ ls -R .git/objects
    .git/objects:

    .git/objects/08:  // subfile1 对应的对象
    fe2720d8e3fe3a5f81fbb289bc4c7a522f13da

    .git/objects/2d:  // 对应 subdir 的 tree 对象
    f4a4d613eb9c5cc64cbaa7738e45c6254dacc5

    .git/objects/3b:  // 第一次修改生成的对象
    18e512dba79e4c8300dd08aeb37f8e728b8dad

    .git/objects/79:  // 对应根目录的 tree 对象
    c830489e29e52b09bcd88837e6af96574f17c4

    .git/objects/e6: //空 test 对应的对象
    9de29bb2d1d6434b8b29ae775ad8c2e48c5391

    .git/objects/f0: // 最后一次修改 test 生成的 blob 对象
    e9ea9cdcd1c7d022373365088a65e94c5ab13e

多了两个对象，它们的类型都是 tree：

    $ git cat-file -t 2df4a4d613eb9c5cc64cbaa7738e45c6254dacc5
    tree
    $ git cat-file -t 79c830489e29e52b09bcd88837e6af96574f17c4
    tree

在看看内容：（git ls-tree 也可以查看 tree 对象）

    $ git cat-file -p 2df4a4d613eb9c5cc64cbaa7738e45c6254dacc5
    100644 blob 08fe2720d8e3fe3a5f81fbb289bc4c7a522f13da	subfile1

    $ git cat-file -p 79c830489e29e52b09bcd88837e6af96574f17c4
    040000 tree 2df4a4d613eb9c5cc64cbaa7738e45c6254dacc5	subdir
    100644 blob f0e9ea9cdcd1c7d022373365088a65e94c5ab13e	test

很容易理解，每个 tree 都对应一个目录，第一个对应 subdir, 第二个对应根目录。每个tree 对象里面都有它所包含的文件的索引，所以这些 tree 对象也完整的描述了要提交的内容。我们上面执行 ```git write-tree``` 命令在命令行输出的就是根目录的 tree 对象。

所以 一个tree对象有一串指向 blob 对象或是其它 tree 对象的指针，它用来表示内容之间的目录层次关系。它里面并没有 blob 对象， blob 保存在 objects 中。

终于说到 commit 了。前面说过 git 是快照流，其历史就是由一串快照组成； commit 命令其实就是用来生成快照的。

```git commit-tree```：命令用于生成快照：

    $ echo "first commit" | git commit-tree 79c830489e29e52b09bcd88837e6af96574f17c4
    463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae

输出了一个新的对象，它的类型为 commit：

    $ git  cat-file -t 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    commit

该对象也保存在 objects 文件夹里面：

    $ ls -R .git/objects/
    .git/objects/:

    .git/objects/08:
    fe2720d8e3fe3a5f81fbb289bc4c7a522f13da

    .git/objects/2d:
    f4a4d613eb9c5cc64cbaa7738e45c6254dacc5

    .git/objects/3b:
    18e512dba79e4c8300dd08aeb37f8e728b8dad

    .git/objects/46: // commit 对象
    3bc07cf12e3ed9c5f2f3194d995a21f6edc7ae

    .git/objects/79:
    c830489e29e52b09bcd88837e6af96574f17c4

    .git/objects/e6:
    9de29bb2d1d6434b8b29ae775ad8c2e48c5391

    .git/objects/f0:
    e9ea9cdcd1c7d022373365088a65e94c5ab13e

可以看到之前每一步生成的 blob 对象都还在 objects 中。

+ a tree: The SHA1 name of a tree object (as defined below), representing the contents of a directory at a certain point in time.
  
+ parent(s): The SHA1 name of some number of commits which represent the immediately previous step(s) in the history of the project. The example above has one parent; merge commits may have more than one. A commit with no parents is called a "root" commit, and represents the initial revision of a project. Each project must have at least one root. A project can also have multiple roots, though that isn't common (or necessarily a good idea).
  
+ an author: The name of the person responsible for this change, together with its date.
  
+ a committer: The name of the person who actually created the commit, with the date it was done. This may be different from the author; for example, if the author wrote a patch and emailed it to another person who used the patch to create the commit.

+  a comment describing this commit.

看看这个对象的内容：


    $ git  cat-file -p 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    tree 79c830489e29e52b09bcd88837e6af96574f17c4
    author rocky <rockykzhao@foxmail.com> 1677596531 +0800
    committer rocky <rockykzhao@foxmail.com> 1677596531 +0800

    first commit

用 ```git log``` 也可以看：

    $ git log 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    commit 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    Author: rocky <rockykzhao@foxmail.com>
    Date:   Tue Feb 28 23:02:11 2023 +0800

        first commit


commit 对象是对一次提交的描述，实质的内容还是保存在 objects 目录中。git commit-tree 命令真是名不副实，它并不是把内容复制到另外一个地方，它只是生成 commit 对象而已；所以所谓快照就是 commit 对象，然后分支指向快照就形成了 history。

Git 提供了git commit命令，简化提交操作。保存进暂存区以后，只要 git commit 一个命令，就同时提交目录结构和说明，生成快照，并移动 HEAD 指向这个快照。

此外，还有两个命令也很有用。

```git checkout```命令用于切换到某个快照。

    git checkout 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae

```git show```命令用于展示某个快照的所有代码变动。

    git show 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae



### tag

后面再补上

### branch

我们已经提交了，现在用 ``` git log ``` 看看提交记录吧：


    $  git log
    fatal: your current branch 'master' does not have any commits yet

提示说当前分支 master 上没有任何提交，原来 git log 命令只显示当前分支的变动，虽然我们前面已经提交了快照，但是还没有记录这个快照属于哪个分支，也就是说这个快照还没指定给 master， 你用 git log 看的就是 master 分支，所以什么都没有。

所谓**分支就是指向某个快照的指针**，分支名就是指针名。哈希值是无法记忆的，分支使得用户可以为快照起别名。而且，**分支会自动更新，如果当前分支有新的快照，指针就会自动指向它**。比如，master 分支就是有一个叫做 master 指针，它指向的快照就是 master 分支的当前快照。

用户可以对任意快照新建指针。比如，新建一个 fix-typo 分支，就是创建一个叫做 fix-typo 的指针，指向某个快照。所以，Git 新建分支特别容易，成本极低。

Git 有一个特殊指针HEAD， 总是指向当前分支的最近一次快照。另外，Git 还提供简写方式，HEAD^指向 HEAD的前一个快照（父节点），HEAD~6则是HEAD之前的第6个快照。

每一个分支指针都是一个文本文件，保存在.git/refs/heads/目录，该文件的内容就是它所指向的快照的二进制对象名（哈希值）。

    [rocky@rocky-pc gitStudy]$ ls -l .git/refs/heads/
    total 0

但是这里现在是空的，是因为 master 上面还没有任何提交。

还有一个疑问， 前面不是已经提交了吗？为什么 stage 里面还有内容：

    $ git ls-files -s
    100644 08fe2720d8e3fe3a5f81fbb289bc4c7a522f13da 0	subdir/subfile1
    100644 f0e9ea9cdcd1c7d022373365088a65e94c5ab13e 0	test


为了演示 parent，我们再进行一次提交：

    $ echo 'hello world again' > test 
    $ git hash-object -w test
    c90c5155ccd6661aed956510f5bd57828eec9ddb
    git update-index test
    $ git write-tree
    f4c14b9498c855a6bcc5a24d40b3ea7896e97755
    $ echo 'second commit'|git commit-tree f4c14b9498c855a6bcc5a24d40b3ea7896e97755 -p 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    93d86520f3ed513112c71f4035f9a5853703aca8

git commit-tree的-p参数用来指定父节点，也就是本次快照所基于的快照。

现在，我们把本次快照的哈希值，写入.git/refs/heads/master文件，这样就使得master指针指向这个快照:

    $ echo 93d86520f3ed513112c71f4035f9a5853703aca8>.git/refs/heads/master

现在再看看 stage：

    $ git ls-files -s
    100644 08fe2720d8e3fe3a5f81fbb289bc4c7a522f13da 0	subdir/subfile1
    100644 c90c5155ccd6661aed956510f5bd57828eec9ddb 0	test

还是有内容，这个是一个疑问，先留这里吧。

在看看status：

    $ git status
    On branch master
    nothing to commit, working tree clean

这里显示倒是正常，难道提交后只是在 index 的相应条目加了标记？

再看看log,  现在能看到了：

    $ git log
    commit 93d86520f3ed513112c71f4035f9a5853703aca8 (HEAD -> master)
    Author: rocky <rockykzhao@foxmail.com>
    Date:   Mon Mar 6 21:24:30 2023 +0800

        second commit

    commit 463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
    Author: rocky <rockykzhao@foxmail.com>
    Date:   Tue Feb 28 23:02:11 2023 +0800

        first commit


git log的运行过程是这样的：

1. 查找HEAD指针对应的分支，本例是master
2. 找到master指针指向的快照，本例是93d86520f3ed513112c71f4035f9a5853703aca8
3. 找到父节点（前一个快照）463bc07cf12e3ed9c5f2f3194d995a21f6edc7ae
4. 以此类推，显示当前分支的所有快照    

最后，补充一点。前面说过，分支指针是动态的。原因在于，下面三个命令会自动改写分支指针。

+ git commit：当前分支指针移向新创建的快照。
+ git pull：当前分支与远程分支合并后，指针指向新创建的快照。
+ git reset [commit_sha]：当前分支指针重置为指定快照。


参考链接：

https://www.ruanyifeng.com/blog/2018/10/git-internals.html

http://shafiul.github.io/gitbook/

