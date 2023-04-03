# JSX 是语法糖
JSX 是 ```React.createElement(component, props, ...children)``` 函数的语法糖。

JSX 代码：

    <MyButton color="blue" shadowSize={2}>
        Click Me
    </MyButton>

会被编译为：

    React.createElement(
        MyButton,
        {color: 'blue', shadowSize: 2},
        'Click Me'
    )

执行后(```render()```方法执行后)会生成下面对象：

    // 注意：这是简化过的结构
    const element = {
        type: 'MyButton',
        props: {
            color: 'blue',
            shadowSize: '2',
            children: 'Click Me'
        }
    };
# 指定元素的类型

JSX 的标签名决定元素的类型

**小写：**
以小写字母开头的元素代表一个 HTML 内置组件，比如 ```<div>``` 或者 ```<span>``` 会生成相应的字符串 'div' 或者 'span' 传递给 ```React.createElement（作为参数）```。

**大写：**
大写字母开头的元素则对应着在 JavaScript 引入或自定义的组件，如 ```<Foo />``` 会编译为 ```React.createElement(Foo)```。 这个标签会被编译成指向叫这个名字的变量，所以如果你要用 ```<Foo/>```， ```Foo``` 必须在作用域内。

例如下面代码：

    import React from 'react';
    import CustomButton from './CustomButton';

    function WarningButton() {
        // return React.createElement(CustomButton, {color: 'red'}, null);
        return <CustomButton color="red" />;
    }

虽然代码中没有直接引用变量 ```React``` 和 ```CustomButton```，但是依然要导入。因为 JSX 是 ```React.createElement()``` 的语法糖，所以要引入 ```React```; 又因为```<CustomButton/>``` 会被编译为指向变量(也就是组件) 的引用，所以要也导入。

# 自定义组件必须以大写字母开头

如果你确实需要一个以小写字母开头的组件，则在 JSX 中使用它之前，必须将它赋值给一个大写字母开头的变量。

# 在运行时选择类型

其实是在运行时选择组件/元素。

不能用一个表达式做一个元素的名字。如果你想通过通用表达式来（动态）决定元素类型，你需要首先将它赋值给大写字母开头的变量。这通常用于根据 prop 来渲染不同组件的情况下:

    import React from 'react';
    import { PhotoStory, VideoStory } from './stories';

    const components = {
        photo: PhotoStory,
        video: VideoStory
    };

    function Story(props) {
        // 错误！JSX 类型不能是一个表达式。
        return <components[props.storyType] story={props.story} />;
    }

要解决这个问题, 需要首先将类型赋值给一个大写字母开头的变量：

    import React from 'react';
    import { PhotoStory, VideoStory } from './stories';

    const components = {
        photo: PhotoStory,
        video: VideoStory
    };

    function Story(props) {
        // 正确！JSX 类型可以是大写字母开头的变量。
        const SpecificStory = components[props.storyType];
        return <SpecificStory story={props.story} />;
    }

# JSX 中的 props

### 表达式作为 ```prop``` 的值
  
        <MyComponent foo={1 + 2 + 3 + 4} />

### if 语句以及 for 循环不是 JavaScript 表达式，所以不能在 JSX 中直接使用。但是，你可以用在 JSX 以外的代码中。比如：
  
        function NumberDescriber(props) {
            let description;
            if (props.number % 2 == 0) {
                description = <strong>even</strong>;
            } else {
                description = <i>odd</i>;
            }
            return <div>{props.number} is an {description} number</div>;
        }

### 字符串字面量作为 prop 的值，下面两个 JSX 表达式是等价的：
  
        <MyComponent message="hello world" />
        <MyComponent message={'hello world'} /> 

### boolean 类型的 ```prop```的默认值是 ```true```。 下面代码是等价的：
  
        <MyTextBox autocomplete />
        <MyTextBox autocomplete={true} />

    >   不建议不传递 value 给 prop，因为这可能与 ES6 对象简写混淆，{foo} 是 {foo: foo} 的简写，而不是 {foo: true}。这样实现只是为了保持和 HTML 中标签属性的行为一致。

### 属性展开
  
    如果你已经有了一个 props 对象，你可以使用展开运算符 ... 来在 JSX 中传递整个 props 对象。以下两个组件是等价的：

        function App1() {
            return <Greeting firstName="Ben" lastName="Hector" />;
        }

        function App2() {
            const props = {firstName: 'Ben', lastName: 'Hector'};
            return <Greeting {...props} />;
        }
    你还可以选择只保留当前组件需要接收的 props，并使用展开运算符将其他 props 传递下去。

        const Button = props => {
            const { kind, ...other } = props;
            const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
            return <button className={className} {...other} />;
        };

        const App = () => {
            return (
                <div>
                <Button kind="primary" onClick={() => console.log("clicked!")}>
                    Hello World!
                </Button>
                </div>
            );
        };
    在上述例子中，kind 的 prop 会被安全的保留，它将不会被传递给 DOM 中的 ```<button>``` 元素。 所有其他的 props 会通过 ...other 对象传递，使得这个组件的应用可以非常灵活。你可以看到它传递了一个 onClick 和 children 属性。

    属性展开在某些情况下很有用，但是也很容易将不必要的 props 传递给不相关的组件，或者将无效的 HTML 属性传递给 DOM。我们建议谨慎的使用该语法。

### 不要更改 props

无论是函数组件还是类组件，都不要去修改其 ```props```。力求做到组件为纯函数：

+ 除了参数输入，函数内不要有任何输入（不能引用全局变量）。
+ 除了返回值输出，不能有其它任何输出（不能有副作用）。


正确示例：

    function sum(a, b) {
        return a + b;
    }

错误示例：

    function withdraw(account, amount) {
        account.total -= amount;
    }

