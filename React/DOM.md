# 概念
DOM 是 Document Object Model：
+ Document
    > 是指标记型文档，比如用 HTML 编写的文档。
+ Object
    > 是指某种语言的对象实例，例如 JS 对象。
+ Model
    > 是一个规范或者说是标准，这里是指对所有 Document 的共有的特征进行总结，进行形成规范、标准。

所以，总的来说 DOM 是一个规范，是独立于平台、浏览器和语言的。

根据这个规范或者说是标准，浏览器先把标记型文档解析、转化、封装成内存中的对象树 (DOM Tree)，然后再把这个树渲染出来；同时也可以用脚本 (JS) 来操作这些 DOM 对象，处理完 DOM 浏览器接着渲染。

Dom 技术使得用户页面可以动态地变化，如可以动态地显示或隐藏一个元素，改变它们的属性，增加一个元素等，Dom 技术使得页面的交互性大大地增强。

# 节点

(浏览器) 解析标记型文档的过程, 就是把文档的所有内容，按照所在的层级，转换成一个 DOM Tree，文档中的标记就成了 DOM Tree 中的一个个对应的节点 （Node）。

节点的类型有七种：
+ Document：整个文档树的顶层节点
+ DocumentType：doctype标签（比如<!DOCTYPE html>）
+ Element：网页的各种HTML标签（比如<body>、<a>等）
+ Attr：网页元素的属性（比如class="right"）
+ Text：标签之间或标签包含的文本
+ Comment：注释
+ DocumentFragment：文档的片段

浏览器提供一个原生的节点对象Node，上面这七种节点都继承了Node，因此具有一些共同的属性和方法。

# 节点树
浏览器原生提供document节点，代表整个文档。就是整个 DOM Tree 的最顶层的节点。
接下来有两个子节点：第一个是文档类型节点（<!doctype html>），第二个是 HTML 网页的顶层容器标签<html>。后者构成了树结构的根节点（root node），其他 HTML 标签节点都是它的下级节点。