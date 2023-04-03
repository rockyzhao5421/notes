# git rm

用于把文件从 stage 和 工作目录删除，或者只从 stage 删除。支持删除单个文件、多个文件、文件夹（加上 -r）

### 从 state 和 工作目录删除

    git rm file1

要删除的文件必须符合两个条件：

1. 文件必须和分支最新快照一致。
2. 不能有更改在 stage

比如现在我们删除 file1:

    $ git rm file1
    error: the following file has changes staged in the index:
        file1
    (use --cached to keep the file, or -f to force removal)

### 只删除 stage 里面的记录

    git rm --cached file1

要删除的文件要么和工作目录里面文件一致，要么和分支最新快照一致才能删除。


