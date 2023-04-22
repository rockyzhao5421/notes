# 段落

Markdown 是以空行分割段落的，也就是说你在一段文字前后加上一个空行就行成了段落。
所以你想在文字开头加上tab是没用的，它不会缩进。

# 换行

想换行？ 敲回车是没用的，在行尾打两个或两个以上的空格，再来一个回车就可以了

# 标题

就是以 # 开头，后面跟标题名称，中间有空格。一共六个级别，根据 # 的个数区分，一个级别最高，六个级别最低。

# 列表

#### 无序列表使用 *, + 或 - 

    *   Red
    *   Green
    *   Blue

#### 有序列表使用数字，后面在加一个英文的点：

    1.  Bird
    2.  McHale
    3.  Parish

#### 列表项可以包含多个段落，每个项目下的其它段落都必须缩进 4 个空格或是 1 个 Tab：

1. sdfasd fasdfasdfasd fasdfa sdfasd fasdfasd fasdfas dfasdfa sdfasdf asdfa sdf as dfa sd fas dfa sdf asdf asd  fa sdf asd fa  sdfasdf

    asdfasd asdf asdf asdf

    asdfasddf

2. sdfa asdf adsfad asdfas asdf asdf  asdfa asdf asdfasdf asd asd asdfa asdf asdf asdf ad asd asdf asdf asdf
 

#### 如果要在列表项目内放进引用，那 > 就需要缩进：

*   A list item with a blockquote:

    > This is a blockquote
    > inside a list item.

#### 如果要放代码区块的话，该区块就需要缩进两次，也就是 8 个空格或是 2 个制表符：

*   一列表项包含一个列表区块：

        <代码写在这>

#### 在行首出现数字-句点-空白，要避免这样的状况，你可以在句点前面加上反斜杠。

    1986\. What a great season.

#### 列表嵌套

只用在子列表前面加一个 Tab 就行了。

# 区块引用 Blockquotes

其格式就是: '> 文字'，中间有空格。

#### 可以每行前面都加 > :

    > This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
    > consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
    > Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
    > 
    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
    > id sem consectetuer libero luctus adipiscing.

#### 也可以只在段落的第一行加 > :

    > This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
    consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
    Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

    > Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
    id sem consectetuer libero luctus adipiscing.

#### 还可以区块套区块，只要根据层次加上不同数量的 > :

    > This is the first level of quoting.
    >
    > > This is nested blockquote.
    >
    > Back to the first level.

#### 也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等相互嵌套 ：

    > ## 这是一个标题。
    > 
    > 1.   这是第一行列表项。
    > 2.   这是第二行列表项。
    > 
    > 给出一些例子代码：
    > 
    >     return shell_exec("echo $input | $markdown_script");


# 代码块
每行都加上四个空格或者一个 Tab 就会显示为代码块，一个代码区块会一直持续到没有缩进的那一行（或是文件结尾）。

另外，这些缩进只是起到标记作用，是不会显示出来的。

在代码区块里面， & 、 < 和 > 会自动转成 HTML 实体，这样的方式让你非常容易使用 Markdown 插入范例用的 HTML 原始码，只需要复制贴上，再加上缩进就可以了，剩下的 Markdown 都会帮你处理，例如：

    <div class="footer">
        &copy; 2004 Foo Corporation
    </div>

如果要标记一小段行内代码，你可以用反引号把它包起来（`），例如：

    Use the `printf()` function.

# 分隔线
你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西，除了空格。

    * * *

    ***

    *****

    - - -

    ---------------------------------------

# 强调

被星号或者下划线包裹的内容会被标记为强调，被两个星号或者两个下划线包裹的内容会被标记为强强调；

经过测试星号、下划线和标记内容之间**不能有空格**，下划线和其它非标记内容之间**必须有空格**，所以为了简单期间推荐用星号：

    *single asterisks*

    _single underscores_

    **double asterisks**

    __double underscores__

# 链接

    This is [an example](http://example.com/ "Title") inline link.

方括号里面放要显示的内容，小括号里面是链接地址，当鼠标放链接上时双引号里面的内容会悬浮显示。

如果你是要链接到同样主机的资源，你可以使用相对路径：

    See my [About](/about/) page for details.   

也可以这样直接把链接地址放尖括号里面：

    <http://example.com/>
显示效果：

<http://example.com/>

# 图片

    ![Alt text](/path/to/img.jpg)

    ![Alt text](/path/to/img.jpg "Optional title")

感叹号开头，方括号放要显示的文字，小括号放图片地址，双引号方提示文字。

# 表格

用三个或以上的 “-” 来分割标题行和其它普通行， 用 "|" 来分割列。每列上 "-" 个数不必相同。

    | Syntax      | Description |
    | ----------- | ----------- |
    | Header      | Title       |
    | Paragraph   | Text        |

显示效果：

| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |

#### 对齐

通过在标题行中的连字符的左侧，右侧或两侧添加冒号（:），将列中的文本左对齐，右对齐或中心对齐：

    | Syntax      | Description | Test Text     |
    | :---        |    :----:   |          ---: |
    | Header      | Title       | Here's this   |
    | Paragraph   | Text        | And more      |

显示效果：

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

也可以用 [Markdown Tables Generator](https://www.tablesgenerator.com/markdown_tables)在线图形界面做表，然后生成 Markdown 文本格式，最后复制黏贴就可以了。

# 颜色

直接用 HTML ：

    <span style="color:orangered;">橙色</span>

<span style="color:orangered;">橙色</span>

# 删除线
    ~~世界是平坦的。~~ 我们现在知道世界是圆的。

~~世界是平坦的。~~ 我们现在知道世界是圆的。

[官网](https://markdown.com.cn/basic-syntax/)