# 概念
元素是构成 React 应用的最小单元，一般通过JSX语法创建，描述我们在屏幕上能够看到的内容。
    
JSX其实是 ```React.createElement()``` 的语法糖，所以 Reac 元素实际上是一个简单的JavaScript对象。

    const element = <h1>Hello, world</h1>;

你说上面等号右边是 React Element 也对，你说由 ```React.createElement()``` 创建的 Plain Object 也对，从这个方法的名字上就可以看出它返回的就是元素。

像 ```<h1/>```、```<MyComponent/>``` 这些 JSX 标签，有些文档也称之为元素，其实指的是运行时它们的返回值，它们只是对生成元素的函数（组件）的调用(还记得 JSX 标签是谁的语法糖不？)，它们的返回值才是元素。

# 分类
## DOM 类型元素
DOM类型的元素使用像h1、div、p等DOM节点创建React 元素，上面的例子创建的就是 DOM 类型元素。

DOM类型元素通常在创建后被直接渲染，因为 DOM 类型元素和页面的DOM节点直接对应，所以 React 知道该如何进行渲染（直接翻译成真实 DOM 节点就行了）。

## 组件类型元素
组件类型的元素使用 React 组件创建 React 元素，如下所示，Welcome就是一个用户自定义的组件，而element是一个组件类型元素：

    function Welcome(props){
        return <h1>Hello,{props.name}</h1>;
    }

    const element=<Welcome name="Sara" />;
    ReactDOM.render(element,document.getElementById('root'))

由于组件类型元素是由自定义组件创建的，所以它和真实 DOM 节点点不是一一对应的，React 就无法直接翻译成真实的 DOM 节点，这时就需要组件自身提供 React 能够识别的 DOM 节点信息。

# 函数组件
其本质就是 JavaScript 函数，它接受任意的入参（即“props”），并返回用于描述页面展示内容的React元素。

    function Welcome(props) {
        return <h1>Hello, {props.name}</h1>;
    }

React 通过组件的思想，将界面拆分成一个个可以复用的模块（通过传入不同的 props），每一个模块就是一个 React 组件。一个 React 应用由若干组件组合而成，一个复杂组件也可以由若干简单组件组合而成。

React 组件和 React 元素关系密切，React 组件最核心的作用是返回 React 元素。

# 渲染 React Element
假设你的 HTML 页面某一个地方有一个 ```<div>``` 节点：

    <div id="root"></div>
我们称之为 root DOM 节点，因为它里面的所有内容都会被 React DOM 管理。

要把一个 React 元素渲染到这个 root DOM 节点下，首先要调用 ``` ReactDOM.createRoot（） ``` 并传入这个节点：

    const root = createRoot(
        document.getElementById('root')
    );

创建一个 **React root**，这个 React root 就是管理 React DOM tree 的地方。


所以接下来我们只用把要渲染的元素传入 ```render()``` 方法就行了， ```render()``` 负责把 React DOM 翻译成 HTML 并同步到真实的 DOM 中，是渲染的入口：

    const element = <h1>Hello, world</h1>;
    root.render(element);


react 元素是一个不可改变对象，所以它一旦创建后是不允许改变的，包括更改它的子元素和属性这些都是不允许的。 一个元素就像动画里的一帧，代表某一个时刻 UI 的样子。

所以到现在为止，要更改 UI 你只能再创建一个新的 React 元素传入到```render()``` 方法里进行渲染。

第二次调用 ```render()``` 就会触发 React DOM diff 算法，比较前后两个 React DOM tree，只把不同的部分更新到真实的 DOM。

实际上，大多数 React 应用只会调用一次 ```root.render()```，还不知道怎么做到这点的，估计是 React 自己在 state/props更新的时候调用了 ```root.render()```。

从 APP 到一个个组件，组件再创建出一个一个元素，最终这些元素实际上描述的一个虚拟 DOM，就是 React DOM Tree。 

React 的整个渲染机制就是调用 ```render（）``` 函数构建出整个 React DOM Tree。

当 state/props 改变时再调用 ```render()``` 函数生成一个新的 React DOM Tree。构造出新的 React Dom tree 跟原来的 React Dom Tree用 Diff 算法进行比较，找到需要更新的地方批量改动，再把改动的地方同步到真实的DOM上，由于这样做就减少了对Dom的频繁操作，从而提升的性能。



