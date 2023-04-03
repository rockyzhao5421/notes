# React Fiber

### React Fiber 的由来

当 React 要加载或者更新组件树时，会做很多事，比如调用各个组件的生命周期函数，计算和比对React DOM 树，最后更新 React DOM 树。

在推出 Fiber 之前这些事情是同步的，也就是说只要一个加载或者更新过程开始，那React就一鼓作气运行到底，中途绝不停歇。面对庞大的组件树，这个机制就会阻碍用户的操作。

所以 Facebook 在 React 16 推出 Fiber 来解决这个问题。Fiber 的作用就是把更新任务分片，每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。

### React Fiber 对生命周期方法的影响

因为一个更新过程可能被打断，所以React Fiber 把一个更新过程被分为两个阶段：第一个阶段Render Phase 和第二阶段 Commit Phase。

在第一阶段 Render Phase，这个阶段就是做好准备再调用```render()```生成元素 ，这个阶段是可以被打断的；但是到了第二阶段Commit Phase，那就一鼓作气把 React DOM 更新完，绝不会被打断。

这两个阶段大部分工作都是React Fiber做，和我们相关的也就是生命周期函数。

以```render()``` 函数为界，第一阶段可能会调用下面这些生命周期函数，说是“可能会调用”是因为不同生命周期调用的函数不同。

    getDerivedStateFromProps
    shouldComponentUpdate
    render


下面这些生命周期函数则会在第二阶段调用。

    componentDidMount
    componentDidUpdate
    componentWillUnmount

**在一个更新过程中**，render 阶段的生命周期函数可能会调多次，第二阶段的生命周期函数只会被调一次。

# 生命周期函数

## 概述
组件有三个状态，进入每个状态会回调一些生命周期方法。

### Mounting

当一个组件实例被创建、插入到 React DOM 时，会按顺序调用下面几个方法：

    constructor()
    static getDerivedStateFromProps()
    render()
    componentDidMount()

### Updating

```props```和 ```state```的改变会引起组件重新渲染。当一个组件被重新渲染时会按顺序调用下面几个方法：

    static getDerivedStateFromProps()
    shouldComponentUpdate()
    render()
    getSnapshotBeforeUpdate()
    componentDidUpdate()

### Unmounting

当组件要从 React DOM 中移除时，会调用这个方法：

    componentWillUnmount()

## 详解

### render()

render() 方法是 class 组件中唯一必须实现的方法。

render() 函数应该为纯函数，这意味着在不修改组件 state 的情况下，每次调用时都返回相同的结果，并且它不会直接与浏览器交互。

如需与浏览器进行交互，请在 componentDidMount() 或其他生命周期方法中执行你的操作。保持 render() 为纯函数，可以使组件更容易实现。

如果 ```shouldComponentUpdate()``` 返回 false，则不会调用 ```render()```。

### constructor(props)

通常，在 React 中，构造函数仅用于以下两种情况：

1. 通过给 this.state 赋值对象来初始化内部 state。
2. 为事件处理函数绑定实例

否则，则不需要为 React 组件实现构造函数。


如果实现了它，首先第一句就是要调 ```super(props)```，否则，this.props 在构造函数中可能会出现未定义的 bug。

不要在构造函数中引入任何副作用或订阅，这些操作应该放在 componentDidMount 中。

**不要把 ```props``` 复制给 state**，这是一个常见的错误：

    constructor(props) {
        super(props);
        // Don't do this!
        this.state = { color: props.color };
    }

这样做没有必要，因为你要读这个值的话可以直接用 ```this.props```；并且这样做的话，即使更新了```props```也不会让对应的 ```state``` 更新。

**只有在你刻意忽略 prop 更新的情况下使用**。此时，应将 prop 重命名为 initialColor 或 defaultColor

[参考这篇博客](https://react.docschina.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)，以了解出现 state 依赖 props 的情况该如何处理。

### componentDidMount()

它是 mounting 状态的最后一个回调函数。组件挂载到 React DOM 树后，立马会调用它。所以这个方法的执行应该是在 ``` render()``` 之后，在浏览器把 DOM 显示出来之前。

一个组件只会挂载一次，因此该函数也只会调一次。

所以，一般请求数据、订阅等操作就是在这个回调函数里发起。依赖 React DOM 的一些初始化动作也可以放这里。

可以在这个函数里面直接调用 ```setState()```，这会导致再一次调 ```render （）```，但此渲染会发生在浏览器更新屏幕之前。如此保证了即使在 ```render()``` 两次调用的情况下，用户也不会看到中间状态。尽量不要这样用，会引发性能问题。除非你的渲染依赖 React DOM。

### componentDidUpdate()

    componentDidUpdate(prevProps, prevState, snapshot)

这个方法其实是和 ```componentDidMount()``` 差不多，都是在 ```render()``` 之后、DOM 显示出来之前被调用，只不过一个是挂载后的第一次 ```render()```之后，一个是更新导致 ```render()``` 之后。第一次挂载后并不会调用这个方法。

更新不像挂载，这个方法可能会被多次调用。

所以你如果在它里面调 ``` setState()```， 必须加条件判断，不然会造成死循环。

如果组件实现了 ```getSnapshotBeforeUpdate()``` ，则它的返回值将作为 ```componentDidUpdate()``` 的第三个参数 “snapshot” 参数传递。否则此参数将为 ```undefined```。

如果 ```shouldComponentUpdate()``` 返回值为 false，则不会调用 componentDidUpdate()。

### componentWillUnmount()

```componentWillUnmount()``` 会在组件卸载、销毁之前立即调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 ```componentDidMount()``` 中创建的订阅等。

```componentWillUnmount()``` 中不应调用 setState()，因为该组件将永远不会重新渲染。组件实例卸载后，永远不会再挂载它。

### shouldComponentUpdate()

    shouldComponentUpdate(nextProps, nextState)

看名字就知道，这个函数的返回值能决定是否 Update 组件，首次挂载那次的 ```render()```调用之前不会受这个函数影响，```forceUpdate()``` 也会跳过这个函数。

它只会在有新的 ```props```或者新的 ```state```来后，也就是 updating 状态中的 ```getDerivedStateFromProps(props, state)``` 和 ```render()```方法之间调用。

提供这个方法的目的就是为了性能。但是有了更好的替代品 ```PureComponent```，这个替代品会对 ```props``` 和 ```state``` 进行浅层比较，并减少了跳过必要更新的可能性。

如果你要手动实现这个方法，就要手动比较前后的 ```props```和```state```了。要注意的是即使返回false，也不会阻止子组件在 state 更改时重新渲染。

另外在这个函数中，不要妄想手动对 ```props```、```state```就行深层比较，或者动用 ```JSON.stringify()```进行字符串比较，会影响性能。

目前这个函数返回 false , ```render()``` 和 ```componentDidUpdate```就不会调了，将来这个函数可能会只有提示功能，所以不建议用了。

### static getDerivedStateFromProps()

    static getDerivedStateFromProps(props, state)

这个函数会在```render()```之前调用，如果是 update，它会在```shouldComponentUpdate()```之前。不管是挂载或者是 update，***每次调```render()```之前必定调这个函数***。

它会返回一个对象来更新```state```。如果返回 ```null ```那就不更新 ```state```, 返回 ```null```等同于没覆写这个函数。

设计这个函数的目的，就是处理那些 ```state```是从```props```来的，```state```每次更新都要要依赖```props```的变化。

看这个例子：

    class Header extends React.Component {
        constructor(props) {
            super(props);
            this.state = {favoritesite: "runoob"};
        }
        static getDerivedStateFromProps(props, state) {
            return {favoritesite: props.favsite };
        }
        changeSite = () => {
            this.setState({favoritesite: "google"});
        }
        render() {
            return (
                <div>
                    <h1>我喜欢的网站是 {this.state.favoritesite}</h1>
                    <button type="button" onClick={this.changeSite}>修改网站名</button>
                </div>
            );
        }
    }
    
    ReactDOM.render(<Header favsite="taobao"/>, document.getElementById('root'))

永远显示的都是 taobao，因为每次 ```render()```前，```state```都会被这个函数改为 taobao。

官方不推荐使用这个函数，[参考篇博客](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。

[还有这篇](https://www.jianshu.com/p/50fe3fb9f7c3)，值得思考。

### getSnapshotBeforeUpdate()

    getSnapshotBeforeUpdate(prevProps, prevState)

这个函数在最近一次渲染提交至 DOM 树之前执行，此时 DOM 树还未改变，我们可以在这里获取 DOM 改变前的信息，例如：更新前 DOM 的滚动位置。

它接收两个参数，分别是：```prevProps、prevState```，上一个状态的 ```props``` 和上一个状态的 ```state```。它的返回值将会传递给 ```componentDidUpdate``` 的第三个参数。

    class ScrollingList extends React.Component {
        constructor(props) {
            super(props);
            this.listRef = React.createRef();
        }

        getSnapshotBeforeUpdate(prevProps, prevState) {
            // 我们是否在 list 中添加新的 items ？
            // 捕获滚动​​位置以便我们稍后调整滚动位置。
            if (prevProps.list.length < this.props.list.length) {
                const list = this.listRef.current;
                return list.scrollHeight - list.scrollTop;
            }
            return null;
        }

        componentDidUpdate(prevProps, prevState, snapshot) {
            // 如果我们 snapshot 有值，说明我们刚刚添加了新的 items，
            // 调整滚动位置使得这些新 items 不会将旧的 items 推出视图。
            //（这里的 snapshot 是 getSnapshotBeforeUpdate 的返回值）
            if (snapshot !== null) {
                const list = this.listRef.current;
                list.scrollTop = list.scrollHeight - snapshot;
            }
        }

        render() {
            return (
            <div ref={this.listRef}>{/* ...contents... */}</div>
            );
        }
    }