**高阶组件是参数为组件，返回值为新组件的函数。**

假设有一个 CommentList 组件，从外部订阅评论数据，用以渲染评论列表：

    class CommentList extends React.Component {
        constructor(props) {
            super(props);
            this.handleChange = this.handleChange.bind(this);
            this.state = {
                // 假设 "DataSource" 是个全局范围内的数据源变量
                comments: DataSource.getComments()
            };
        }

        componentDidMount() {
            // 订阅更改
            DataSource.addChangeListener(this.handleChange);
        }

        componentWillUnmount() {
            // 清除订阅
            DataSource.removeChangeListener(this.handleChange);
        }

        handleChange() {
            // 当数据源更新时，更新组件状态
            this.setState({
            comments: DataSource.getComments()
            });
        }

        render() {
            return (
                <div>
                    {this.state.comments.map((comment) => (
                        <Comment comment={comment} key={comment.id} />
                    ))}
                </div>
            );
        }
    }

稍后，编写了一个组件，从外部订阅一篇博客数据，然后显示出来：

    class BlogPost extends React.Component {
        constructor(props) {
            super(props);
            this.handleChange = this.handleChange.bind(this);
            this.state = {
                blogPost: DataSource.getBlogPost(props.id)
            };
        }

        componentDidMount() {
            DataSource.addChangeListener(this.handleChange);
        }

        componentWillUnmount() {
            DataSource.removeChangeListener(this.handleChange);
        }

        handleChange() {
            this.setState({
                blogPost: DataSource.getBlogPost(this.props.id)
            });
        }

        render() {
            return <TextBlock text={this.state.blogPost} />;
        }
    }

这两个组件只有：
1. 数据源不同，一个是评论列表数据源，一个是单个博客数据源。
2. ```render()``` 的输出不同。

其它方面都相同：
1. 构造函数中初始化 ```state```，里面放上初始化的数据。
2. 挂载的时候在数据源里面注册一个监听器，数据源有更新就会触发监听器。
3. 监听器被触发后就去拿新的数据，并更新到 ```state``` 里面。
4. 卸载的时候注销监听器。

为了代码复用，我们需要把这些相同的地方抽象出来，放在一个地方，然后可以共享给其它组件复用。

我们把抽象出来代码放在一个组件里面，那么怎么共享给其它组件呢？HOC 采用的是装饰者模式，就像 Java 中的 ```BufferedInputStream``` 就是一个装饰器，哪个 Stream 想要有 buffer 功能，就用```BufferedInputStream``` 包一下就行了，因为他们实现了同一个接口，因为装饰者要可以完全取代被装饰者。这里我们抽象出来的组件也是一个装饰器，就像 ```BufferedInputStream``` ,哪个组件需要这个抽象功能，就用这个组件包一下就行了，这样这个抽象组件的功能就得到了复用。

我们定义一个函数来创建这个抽象组件。为什么要用函数？ 因为只有通过函数我们才可以把抽象组件的创建延迟到函数调用的时候，才有机会把会变化的东西——主要是被包装的组件作为参数传进去。

具体到这个例子：

    // This function takes a component...
    function withSubscription(WrappedComponent, selectData) {
        // ...and returns another component...
        return class extends React.Component {
            constructor(props) {
                super(props);
                this.handleChange = this.handleChange.bind(this);
                //这是组件生命周期里面第一个回调，所以组件一开始就拿到了当前最新的数据
                this.state = {
                    data: selectData(DataSource, props)
                };
            }

            componentDidMount() {
                // 挂载后监听数据源的更新
                DataSource.addChangeListener(this.handleChange);
            }

            componentWillUnmount() {
                DataSource.removeChangeListener(this.handleChange);
            }

            handleChange() {
                //数据源有更新就会同步到```state```，从而 re-render 更新 UI
                this.setState({
                    data: selectData(DataSource, this.props)
                });
            }

            render() {
                // ... and renders the wrapped component with the fresh data!
                // Notice that we pass through any additional props
                return <WrappedComponent data={this.state.data} {...this.props} />;
            }
        };
    }

使用的时候：

    const CommentListWithSubscription = withSubscription(
        CommentList,
        (DataSource) => DataSource.getComments()
    );

    const BlogPostWithSubscription = withSubscription(
        BlogPost,
        (DataSource, props) => DataSource.getBlogPost(props.id)
    );
# 参数
第一个参数是被包装的组件，每一个 HOC 必定要这个参数，第二个参数在这是一个获取数据的方法。这两个参数就把该例中变化的东西参数化了。

要记得 HOC 就是一个函数。除了第一个参数，其余参数都是根据实际情况来定的。比如这个例子里面的 ```DataSource``` ，因为是写死到包装组件里面的，如果有组件需要订阅功能，但是数据源不是 ```DataSource``` ,就不能用这个 HOC，你可以把这个数据源作为参数传入，使这个 HOC 能适用于更多的数据源订阅。所以参数的本质就是要把变化的东西参数化、可配置化，从而尽可能的把包装者和被包装隔离开来，这样你的 HOC 才能更广泛的被复用。

但是，很多时候不能抽象过度。其实官方给的这个例子，把握就很好，它没有把数据源也参数化。你参数越多，这 HOC 适用的范围越广，但是也表明了这个 HOC 的用途越模糊，越难以理解；参数越多，也表明能重用的代码越少。


# props

我们再看看 props，在这个例子中新组件没有增加新的 props，也没有删除（不给某个 prop 传值就相当于删除了）被包装组件的 props，只是给其中的 ```props.data``` 提供了数据源，这就是包装的目的——为被包装组件提供订阅功能。任意组件，只要有 ```props.data``` ，都可以通过这个 HOC 包装后获得订阅 ```DataSource``` 的功能。

我们装饰的目的就是，在被装饰者的行为之前或者之后加上装饰者自己的行为。具体到 React，我们只能在”之前“加上自己的行为，因为组件 output 之后的元素是不可更改的。你可以在包装组件上加 props 、也可以劫持被包装组件的 props、也可以劫持被包装组件的渲染（条件渲染、在被包装组件上再套个边框……），总之为了达到目的你可以为所欲为。

HOC 为了扩展功能增加的 props 不应该传给被包裹组件，而其它和自己功能无关的都要原封不动地传给被包裹组件。在这个例子中，没有新增 props，只是特殊处理了 ```props.data```。


总之，包装者只负责数据获取，不管最终如何使用。被包装者只管使用数据，不管数据从何而来。

**HOC 只是一个函数，它返回的容器组件才是重点，重用的代码就在这个容器组件内。用函数的目的只是为了接受参数，并把容器组件的实例化延后而已。**

# 不要修改被包装组件

不要修改被包装组件，而是要用一个容器组件包裹它，采用组合的方式来组合容器组件和被包裹的功能。记住 HOC 的目的是 “重用”，“增强”，而不是 “修改”。

# 透传和增强无关的 props

HOC 为组件添加特性。自身不应该大幅改变约定。HOC 返回的组件与原组件应保持类似的接口。所以，HOC 应该透传与自身无关的 props。大多数 HOC 都应该包含一个类似于下面的 render 方法：

        render() {
            // 把 HOC 扩展的 props 提取出来，这些 props 是自己用的不用传给被包装的组件。
            const { extraProp, ...passThroughProps } = this.props;

            // Inject props into the wrapped component. These are usually state values or
            // instance methods. 我觉得这些是 HOC 处理好的一些数据，也就是它给予的增强功能。
            const injectedProp = someStateOrInstanceMethod;

            // Pass props to wrapped component
            return (
                <WrappedComponent
                    injectedProp={injectedProp}
                    {...passThroughProps}
                />
            );
        }

# HOC 可以最大化可组合性

使用 HOC 可以让组件更容易组合，以 redux 的 ```connect``` 为例 :

    // React Redux's `connect`
    const ConnectedComment = connect(commentSelector, commentActions)(CommentList);

对柯里化、偏函数不熟悉的话可能不太明白这是个啥，我们把它拆开写：

    // connect is a function that returns another function
    const enhance = connect(commentListSelector, commentListActions);
    // The returned function is a HOC, which returns a component that is connected
    // to the Redux store
    const ConnectedComment = enhance(CommentList);

这样就可以看出，```connect``` 是一个高阶函数，它返回的是一个 HOC，这个 HOC 只有一个参数——被包裹的组件。

这种单参数的 HOC 有一个显著的特点：输入与输出一样，或者说都是组件。这种输入与输出一样的函数很容易相互组合，这和 Linux 的管道一个道理。

```connect``` 返回的 HOC 很容易和其它的 HOC 再次组合。例如这里的 ```CommentList``` 就可以是一个 HOC。

# 为了方便调试，加上 DisplayName

和普通组件一样，HOC 返回的容器组件也会显示在 React Developer Tools 中，为了方便调试，最好给这个容器组件的  ```DisplayName``` 赋值。让人一看就知道它是 HOC 返回的容器组件。

举例来说，如果 HOC 叫 ```withSubscription```，被包裹的组件叫 ```CommentList```，那么对应的容器组件的 ```DisplayName``` 的值应该是 ```WithSubscription(CommentList)```：

    function withSubscription(WrappedComponent) {
        class WithSubscription extends React.Component {/* ... */}
        WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
        return WithSubscription;
    }

    function getDisplayName(WrappedComponent) {
        return WrappedComponent.displayName || WrappedComponent.name || 'Component';
    }

# 注意事项

### 不要在 render 方法中使用 HOC

React 的 diff 算法是先直接比较两个组件是否为同一个的，如果 ```render()``` 返回的组件和上一次 ```render()``` 返回是同一个，那么就用 diff 算法来递归更新既有的树，否则就直接用新的组件替换旧的组件。

所以，你如果在 ```render()``` 中使用了 HOC，那么每次 ```render()``` 被调用时，HOC 都会创建一个新的容器组件，前一个容器组件就会被完全卸载掉，用新的代替：

    render() {
        // A new version of EnhancedComponent is created on every render
        // EnhancedComponent1 !== EnhancedComponent2
        const EnhancedComponent = enhance(MyComponent);
        // That causes the entire subtree to unmount/remount each time!
        return <EnhancedComponent />;
    }

这不仅仅是性能问题 - 重新挂载组件会导致该组件及其所有子组件的状态丢失。

解决的办法就是在外面调用 HOC，然后在 ```render()``` 中直接使用它返回的新组件就行了。如果真需要动态创建容器组件，那值能在生命周期函数中进行。

# 务必复制静态方法

这个很好理解，静态方法是属于类的，不属于实例。容器组件里面包的是原始组件的实例，所以容器组件不会有原始组件里面的静态方法。所以 copy 给容器组件就行了：

    function enhance(WrappedComponent) {
        class Enhance extends React.Component {/*...*/}
        // 必须准确知道应该拷贝哪些方法 :(
        Enhance.staticMethod = WrappedComponent.staticMethod;
        return Enhance;
    }


# Refs 不会被传递

这个在 Refs 那篇笔记里已经说过了。