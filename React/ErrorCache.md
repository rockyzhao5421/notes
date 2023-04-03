React 用一个特殊的组件来捕获**子组件**没有被捕获的异常，这个特殊的组件就是 error boundary。

要想让一个类组件成为 error boundary 组件，只用实现 ```static getDerivedStateFromError() ， componentDidCatch()``` 这两个函数中的一个，或者两个都实现就行了。这个组件能捕获**它下面的子组件**所有未捕获的 JS 异常。

有一点要强调 error boundary 组件只是用来捕获它的子组件抛出的未捕获的异常，**不要用它来控制你的流程**。

# static getDerivedStateFromError()

    static getDerivedStateFromError(error)

当一个子组件抛出一个异常后，就会调用这个函数，并且把这个异常作为参数传进来。这个函数会有一个返回值，这个返回值是用来更新```state```的，然后你可以在```render()```函数里根据```state```来显示一个错误页面。

    class ErrorBoundary extends React.Component {
        constructor(props) {
            super(props);
            this.state = { hasError: false };
        }

        static getDerivedStateFromError(error) {
            // Update state so the next render will show the fallback UI.
            return { hasError: true };
        }

        render() {
            if (this.state.hasError) {
                // You can render any custom fallback UI
                return <h1>Something went wrong.</h1>;
            }
            return this.props.children;
        }
    }

注意：这个函数是在 render phase 调用的，所以里面不要有副作用的操作。这中情况要用```componentDidCatch()```。

# componentDidCatch()

    componentDidCatch(error, info)

这个函数和上一个基本上一样，只不过它是在 commit phase 调用的，所以它里面可以有副作用的操作，比如，可以在里面写 log 。

他有两个参数：
1. error 抛出的异常.
2. info  包含错误堆栈的 info 对象 

