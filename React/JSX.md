# 概念
    const element = <h1>Hello, world!</h1>;
上面的变量声明，等号右边的部分就是 JSX 语法。它是 JS 的扩展，具有 JS 的全部功能。
我们建议在 React 中用它来写 UI。



## JSX 是语法糖
在 Bable 编译之后，JSX 会被转换为 ```React.createElement()```调用。

以下两种示例代码完全等效：

    const element = (
        <h1 className="greeting">
            Hello, world!
        </h1>
    );

    const element = React.createElement(
        'h1',
        {className: 'greeting'},
        'Hello, world!'
    );

```React.createElement() ```会预先执行一些检查，以帮助你编写无错代码，但实际上它创建了一个这样的对象：

    // 注意：这是简化过的结构
    const element = {
        type: 'h1',
        props: {
            className: 'greeting',
            children: 'Hello, world!'
        }
    };

**这些对象被称为 “React 元素”**。你可以把它们当成 描述 UI 的对象。React 通过读取这些对象，然后使用它们来构建 DOM 以及保持随时更新。

所以 JSX 语句其实就是 JS 表达式， 表达式都会有值，这里的值就是一个对象。

既然是一个值，那你就可以在 if 语句和 for 循环的代码块中使用 JSX，也可以将 JSX 赋值给变量，把 JSX 当作参数传入，以及从函数中返回 JSX：

    function getGreeting(user) {
        if (user) {
            return <h1>Hello, {formatName(user)}!</h1>;
        }
        return <h1>Hello, Stranger.</h1>;
    }

同样的原因， 可以在 JSX 中嵌入其它 JS 表达式。

## 在 JSX 中嵌入表达式
    const name = 'Josh Perez';
    const element = <h1>Hello, {name}</h1>;

    ReactDOM.render(
        element,
        document.getElementById('root')
    );

在 JSX 语法中，你可以在大括号内放置任何有效的 JavaScript 表达式。

    function formatName(user) {
        return user.firstName + ' ' + user.lastName;
    }

    const user = {
        firstName: 'Harper',
        lastName: 'Perez'
    };

    const element = (
        <h1>
            Hello, {formatName(user)}!
        </h1>
    );

    ReactDOM.render(
        element,
        document.getElementById('root')
    );

## JSX 属性
你可以通过使用引号，来将属性值指定为字符串字面量：

    const element = <div tabIndex="0"></div>;

也可以使用大括号，来在属性值中插入一个 JavaScript 表达式：

    const element = <img src={user.avatarUrl}></img>;