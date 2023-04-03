# 概念

render props 是一个组件的```props```，它的值是一个函数，这个函数返回的是 React 元素，这个组件通过这个渲染函数抽象了它要显示的内容，从而使它的行为可以被复用。

所以，**render props，其实质是指有 render props 的组件**。

# 来历

上面已经说了，render props 的目的是复用，那组件本身不是已经复用的手段了吗？
组件的复用只能复用组件的 UI，不能复用它的```state```、行为给其它组件。

例如下面这个显示鼠标坐标的组件：

    class MouseTracker extends React.Component {
        constructor(props) {
            super(props);
            this.handleMouseMove = this.handleMouseMove.bind(this);
            this.state = { x: 0, y: 0 };
        }

        handleMouseMove(event) {
            this.setState({
                x: event.clientX,
                y: event.clientY
            });
        }

        render() {
            return (
                <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
                    <h1>Move the mouse around!</h1>
                    <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
                </div>
            );
        }
    }

这个组件就是跟踪鼠标的坐标，然后记录在 ```state``` 里面，进而显示出来。

那么问题来了：我们怎么在其它组价里面复用这个行为呢？换句话说，如果另一个组件也需要跟踪鼠标坐标，但是又不是把坐标显示出来，而是另有它用呢？我们如何封装这个行为才能共享给这个组件呢？

例如，另一个组件 ``` <Cat/>``` ，这个组件显示一个猫的图片，Mouse到哪里这个猫追到哪里。

很容易想到的方法是：把 ```<Cat/>``` 直接放在上面的组件的 ```render()``` 里面不就行了？这样 ```<Cat/>``` 不就可以访问到 ```state``` 里面的坐标了吗？

    class Cat extends React.Component {
        render() {
            const mouse = this.props.mouse;
            return (
                <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
            );
        }
    }

    class MouseWithCat extends React.Component {
        constructor(props) {
            super(props);
            this.handleMouseMove = this.handleMouseMove.bind(this);
            this.state = { x: 0, y: 0 };
        }

        handleMouseMove(event) {
            this.setState({
                x: event.clientX,
                y: event.clientY
            });
        }

        render() {
            return (
                <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

                    {/*
                    We could just swap out the <p> for a <Cat> here ... but then
                    we would need to create a separate <MouseWithSomethingElse>
                    component every time we need to use it, so <MouseWithCat>
                    isn't really reusable yet.
                    */}
                    <Cat mouse={this.state} />
                </div>
            );
        }
    }

    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>
                    <MouseWithCat />
                </div>
            );
        }
    }

不错，这样确实可以，但是也只是实现了功能而已，并没有真正达到复用的效果。正如注释中所说，当坐标的作用不同时，你不得不创建各种各样的 ```MouseWithXXX``` 组件，里面的代码还是要一遍遍的复制粘贴。

那我把 ```<Cat/>``` 作为 ```props.children``` 传入行不行？ 好，改一下：


    class Mouse extends React.Component {
       ……

        render() {
            return (
                <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>
                   { this.props.children}
                </div>
            );
        }
    }

    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>
                    <MouseWith>
                        <Cat mouse = ?/>
                    </MouseWith>
                </div>
            );
        }
    }

看到没？这样做的话又访问不了坐标，代码走到 ```{ this.props.children}``` 的时候 ```children``` 已经是元素了，已经不是函数(组件)调用了，元素一旦创建就不能改变，所以不能在这里再把坐标传给 ```{ this.props.children}``` 了。

所以，要想把坐标传到 ```<Cat/>```，必须把它的调用延后，延后到 ```Mouse``` 里面。那只能把它的调用封装到一个函数里面，这个函数的参数就是 ```<Cat/>``` 需要的坐标，再把函数通过 ```props``` 传到 ```Mouse```里面，然后再在 ```Mouse``` 里面通过调用这个函数来调用 ```<Cat/>``` 来创建元素就可以读到坐标了：

    class Cat extends React.Component {
        render() {
            const mouse = this.props.mouse;
            return (
                <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
            );
        }
    }

    class Mouse extends React.Component {
        constructor(props) {
            super(props);
            this.handleMouseMove = this.handleMouseMove.bind(this);
            this.state = { x: 0, y: 0 };
        }

        handleMouseMove(event) {
            this.setState({
                x: event.clientX,
                y: event.clientY
            });
        }

        render() {
            return (
                <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

                    {/*
                    Instead of providing a static representation of what <Mouse> renders,
                    use the `render` prop to dynamically determine what to render.
                    */}
                    {this.props.render(this.state)}
                </div>
            );
        }
    }

    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>
                    <Mouse render={mouse => (
                    <Cat mouse={mouse} />
                    )}/>
                </div>
            );
        }
    }

这就是 render props 的来历：一个有 render props 的组件，有了这个 ```props``` 就可以动态决定它要渲染什么，而不是写死在它的 ```render()``` 函数内。

**render prop 是一个用于告知组件需要渲染什么内容的函数 prop**。

# 设计

至此我们可以看出，这个 render prop 只是代码复用的一个非常简单的设计。在面向对象里面，它甚至算不上 23 中设计模式中的一种，它只是一个面向对象的设计原则：**依赖倒置原则**, 要依赖抽象，不要依赖具体。这里的 render prop 就是渲染的抽象。

类似 Java 里面的回调设计，只不过 java 里面的回调是回调 listener 接口里面某一个函数，那是因为 Java 不能把函数作为参数传递，只能依托类来曲线救国；而 JS 的函数是一个变量，更简单，直接通过 ```props``` 传递就行了。

所以说， 重点是这个组件有一个 ```props``` 能接受外面传入一个能告诉它需要渲染什么的函数，这个 ```props``` 的名字并不一定要叫 render。甚至你可以直接用 ```props.children``` 来传递这个函数：

    <Mouse children={mouse => (
        <p>The mouse position is {mouse.x}, {mouse.y}</p>
    )}/>

或者这样写：

    <Mouse>
        {mouse => (
            <p>The mouse position is {mouse.x}, {mouse.y}</p>
        )}
    </Mouse>


# 利用 Render Props 写 HOC

把 Render Props 改成 HOC 的重用方式很简单，直接把被包裹的组件作为 Render Props 的输出元素就行了：

    // If you really want a HOC for some reason, you can easily
    // create one using a regular component with a render prop!
    function withMouse(Component) {
        return class extends React.Component {
            render() {
                return (
                    <Mouse render={mouse => (
                        <Component {...this.props} mouse={mouse} />
                    )}/>
                );
            }
        }
    }

这里说到了 HOC，就说说自己对它们的理解：

Render Props 不会创建新的组件，它是把可以重用功能写在一个组件里，包括可以重用的 UI 元素，只是让你可以动态决定这个组件里面一个或者多个输出元素。就像一个固定好的模版，留好了窟窿，你只用用你想要的东西填这写窟窿就行了，其它的地方你也改不了。

HOC 是创建了新的组件，是把重用的代码写在一个容器组件里，哪个组件想具有容器组件里面的功能，就用这个容器装饰一下，就成了一个全新的组件。实质就是装饰者模式的运行，运用的是组合思想

从最终的作用上来说，它们两个是相同的，都是为了复用。

# Render Props 与 React.PureComponent

如果你 render props 组件是继承自 ```React.PureComponent```，那就要小心了。如果你在调用组件的 ```render()``` 里面创建(定义)那个 render props 函数，就会使 PureComponent 的作用荡然无存。

我们还用上面的例子稍作修改：

    class Mouse extends React.PureComponent {
        // Same implementation as above...
    }

    class MouseTracker extends React.Component {
        render() {
            return (
                <div>
                    <h1>Move the mouse around!</h1>

                    {/*
                    This is bad! The value of the `render` prop will
                    be different on each render.
                    */}
                    <Mouse render={mouse => (
                        <Cat mouse={mouse} />
                    )}/>
                </div>
            );
        }
    }

因为每次这个 ```MouseTracker``` 渲染都会重新创建一个函数传给 ```Mouse```， 让它每次都接收到新的 ```props```。

解决办法很简单，把函数定义在 ```render()``` 外面：

    class MouseTracker extends React.Component {
        // Defined as an instance method, `this.renderTheCat` always
        // refers to *same* function when we use it in render
        renderTheCat(mouse) {
            return <Cat mouse={mouse} />;
        }

        render() {
            return (
            <div>
                <h1>Move the mouse around!</h1>
                <Mouse render={this.renderTheCat} />
            </div>
            );
        }
    }

In cases where you cannot define the prop statically (e.g. because you need to close over the component’s props and/or state) <Mouse> should extend React.Component instead.

这句不理解


# 总结

render props 是一种代码复用的设计，和其它复用方式相比它有特有的优势：

**最初**，单纯用组件的方式，定义一个组件可以到处调用并传入不同的 ```props```，渲染稍微不同的 UI 。这中方式可以简单认为是一个 UI 函数，只能复用 UI。

**然后**，我们通过直接 JSX 标签组合不同的组件，组成更复杂的组件，从而达到组件被复用的效果:
   
    function Header(props) {
        return (
            <UserInfo/>
                <UserImage/>
                <UserName/>
            </UserInfo>
        );
    }

这是第一种方式的延伸，这种复用方式是我们用的最多的方式。

+ 优点是：语义清楚，一看新组成的组件名字 Header 就知道作用是什么；父组件的数据、状态也可以通过 ```props``` 共享给子组件。

+ 缺点是：组件之前的关系是静态的(hardcode)，每每需要一个新的 UI，你就要定义一个新的组件。

**接着**，为了解决简单组和方式的缺点，又发展出把子组件通过父组件的 props 传递进去，达到父组件被复用的效果，典型的方式就是利用 ```props.children```。

+ 优点是：可以动态决定父组件要渲染的内容，同时不用再定义新的组件出来，因为传不同的子组件给父组件就可以了。
+ 缺点是：
  1. 语义不明确。例如：你觉得每个页面的边距都是相同的，为了不在没个页面都写这个边距的代码，你决定才用这种方式定义一个 PageContainer 的组件，然后具体每个页面按自己要显示的内容传对应的 ```props.children``` 进去。到最后你的代码看起来就会是这样的：
  
            <>   
                <PageContainer>
                    <View1/>
                </PageContainer>

                <PageContainer>
                    <View2/>
                </PageContainer>  

                <PageContainer>
                    <View2/>
                </PageContainer>
            </> 
        那怎么判断要不要用这种方式呢，我觉得当你的父组件的名字要起的很抽象的时候，就不要用这种方式，像上面的 PageContainer。如果名字是 ConfirmDialog，这种就比较具体的名字就可以。

   2. 父组件的数据不能共享给子组件，就如上面的 ```View1``` 就引用不到 ```PageContainer``` 里面的 ```props```、```state```，因为运行时传到父组件之后，已经是子组件的 output 了，不是函数了。

所以这种方式只能用在需要小范围内复用父组件，并且不需要共享父组件的数据给子组件的情况。

**最后**，为了解决上述不能共享数据的问题，推出了 render props，它和上面那种方式相比，唯一解决的问题就是父组件可以共享数据给子组件，语义不明的缺点同样存在。




