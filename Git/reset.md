# git reset

主要有下面两种用法，两种用法中 tree-ish , commit 默认都是 HEAD

### ```git reset [<tree-ish>] [--] <pathspec>…​```

用 tree-ish 覆盖 pathspec 在 **stage** 里面的记录。


### ```git reset [<mode>] [<commit>]```

这种形式**一定会**用 commit 覆盖 当前分支的 head，可选操作是同时覆盖 stage 、工作目录，可选操作依赖于 mode。

但是要记得 commit 默认值是 HEAD，所以在不提供 commit 的情况下，这个中形式就变成了，用 HEAD 去覆盖 stage 和工作
目录了。


mode 默认是 mixed

**--soft**

不动 stage 和工作目录。

**--mixed** 

reset stage, 不动工作目录。

mixed 又是默认值，所以 ``` git reset ``` 就是擦除 stage 而已。

**--hard**

reset stage 和工作目录。注意没有 track 的会被删除

**--merge**

reset stage， 并且更新工作目录中那些  HEAD 和 commit 不同的文件，但是会保留工作目录中那些和 stage 不同的文件。
但是如果有文件的 stage 和 commit 不同，又有改动没有 add，就不能 --merge（这一句就不太懂了，后面再看吧）。

也就是说已经提交的会被擦除( reset 前 HEAD 指向最后的提交)， 但是还没 add 的更改会保留。


### 举例