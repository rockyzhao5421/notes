在 JSX 表达式中，开始标签和结束标签之间的内容就是children，它是一个特殊的 props：```props.children```

    const element = {
        type: 'MyButton',
        props: {
            color: 'blue',
            shadowSize: '2',
            children: 'Click Me'
        }
    };


# 传 children 的方式：

给一个组件传 props.children 有很多方式：

### 传字符串作为 children

    <MyComponent>Hello world!</MyComponent>

JSX 会移除行首尾的空格以及空行。与标签相邻的空行均会被删除，文本字符串之间的新行会被压缩为一个空格。因此以下的几种方式都是等价的：

    <div>Hello World</div>

    <div>
        Hello World
    </div>

    <div>
        Hello
        World
    </div>

    <div>

        Hello World
    </div>

### 传元素作为 children

    <MyContainer>
        <MyFirstComponent />
        <MySecondComponent />
    </MyContainer>

### 传 JS 表达式作为 children

JavaScript 表达式可以被包裹在 {} 中作为 children。例如，以下表达式是等价的：

    <MyComponent>foo</MyComponent>
    <MyComponent>{'foo'}</MyComponent>

这对于展示任意长度的列表非常有用。例如，渲染 HTML 列表：

    function Item(props) {
        return <li>{props.message}</li>;
    }

    function TodoList() {
        const todos = ['finish doc', 'submit pr', 'nag dan to review'];
        return (
            <ul>
                {todos.map((message) => <Item key={message} message={message} />)}
            </ul>
        );
    }

JavaScript 表达式也可以和其他类型的 children 组合。这种做法可以方便地替代模板字符串：

    function Hello(props) {
        return <div>Hello {props.addressee}!</div>;
    }

### 传函数作为 children

上面几种传的都是用来显示的。 ```props.children``` 和其它 props 一样可以传任意类型的数据， 不仅仅是 React 渲染类型。

例如，如果你有一个自定义组件，你可以把回调函数作为 props.children 进行传递：

    // 调用子元素回调 numTimes 次，来重复生成组件
    function Repeat(props) {
        let items = [];
        for (let i = 0; i < props.numTimes; i++) {
            items.push(props.children(i));
        }
        return <div>{items}</div>;
    }

    function ListOfTenThings() {
        return (
            <Repeat numTimes={10}>
                //这里传入一个回调函数作为 props.children
                {(index) => <div key={index}>This is item {index} in the list</div>}
            </Repeat>
        );
    }

# 传递的两种写法

要记得 ```children``` 也是 ```props```，所以它不但可以像上面那样用 JSX 节点的写法，还可以通过元素的属性列表传：

    <MyContainer children= { <MyComponent /> }/>


# boolean、undefined 和 null 会被忽略

以下的 JSX 表达式渲染结果相同：

    <div />

    <div></div>

    <div>{false}</div>

    <div>{null}</div>

    <div>{undefined}</div>

    <div>{true}</div>

这有助于依据特定条件来渲染其他的 React 元素。例如，在以下 JSX 中，仅当 showHeader 为 true 时，才会渲染 ```<Header />``` 组件：

    <div>
        {showHeader && <Header />}
        <Content />
    </div>

# children 数据类型

1. 如果当前组件没有子节点，它就是 undefined；
2. 如果有一个子节点，数据类型是 object；
3. 如果有多个子节点，数据类型就是 array;// 有待求证

# children 工具箱-React.Children

    React.Children.map(children, function[(thisArg)])

和数组的 map 方法类似：遍历 children, 把每个 child 传入第二个参数并执行，返回一个新的数组。会把 ```this``` 指向这个函数的参数 thisArg。

如果子节点为 ​null​ 或是 ​undefined​，则此方法将返回 ​null​ 或是 ​undefined​，而不会返回数组。

根据源码，如果 children 中的 child 还有 child，这个方法会递归调用确保深层的 child 也会被处理到。但是根据官方文档，它只会处理直接 child。这五个方法除 only 方法其它四个都有这个疑问。

[参考这里](https://juejin.cn/post/6844904133531549709)
[还有这里](https://juejin.cn/post/6869920189916381198)

    React.Children.forEach(children, function[(thisArg)])

同 map , 只是不返回数组。

    React.Children.only(children)

返回 children 中 child 的个数，等于 call map/forEach 回调函数的次数。

    React.Children.only(children)

验证 children 中只有一个 child（React 元素），并返回这个 child，否则抛出异常。

这里注意如果不是 React 元素也会抛出异常。

    React.Children.toArray(children)

把 children 转换为数组，你可以在 ```render()``` 中用它对一个 child collection 进行操作。如果你想把 ```props.children``` 传给子组件前对它进行排序或者 ```slice```操作，也可以用这个方法。


