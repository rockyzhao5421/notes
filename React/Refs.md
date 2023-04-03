React 的数据流向是从上到下的，父组件只能通过 ```props``` 和 child 交互。要修改 child 只能通过修改 ```props``` 来重新渲染。

但有时候我们需要直接调用**组件实例**或者 DOM 的方法，例如：

+ 获取输入框的焦点、文本选择、播放。
+ 动画
+ 集成第三方库

这时候就需要 Refs 出场了。

# Refs
Refs 是一个对象，它只有一个属性 ```current``` , ```current``` 用来指向实例。为什么要把实际的引用放在对象的属性上？大概是想在一个组件的生命周期内持有同一个 Refs 对象，至于为什么要持有同一个对象？原因还不知道。


# React.createRef()

通过这个函数可以创建的 Refs, 创建后可以把 Refs 通过 ```render()``` 中 React 元素的 ```ref``` 传递给元素。这里要说明的是 ```ref={Refs}``` 并不是一个 prop，它和 ```key``` 一样是 React  的一个保留关键字，这种传递方式不能通过 ```props``` 接收，但是只有通过关键字传递，React 才会把实例赋值给它；

但是你可以把 Refs 当做一个普通的 ```props``` 传递，只要不用 ref 这个关键字做名字就可以，只要最后一步——传给目标元素的时候采用 ```ref={Refs}``` 这种方式就行了。

在运行时，当目标元素挂载时，也就是在 ```componentDidMount``` 调用前会给 ```current``` 赋值，卸载时又把它设为 null；当其父组件 re-render 时，也就是在 ```componentDidUpdate``` 被调用前重新给它赋值，因为它指向的实例又重新生成了，原来的实例已经没用了。

元素又分 DOM 元素和组件元素两种，不管是哪一种，其目的都是为了能访问它们的实例，进而可以调用他们的方法。

### 传给 DOM 元素

把 Refs 传给 DOM 元素的 ```ref```：

    class CustomTextInput extends React.Component {
        constructor(props) {
            super(props);
            // create a ref to store the textInput DOM element
            this.textInput = React.createRef();
            this.focusTextInput = this.focusTextInput.bind(this);
        }

        focusTextInput() {
            // Explicitly focus the text input using the raw DOM API
            // Note: we're accessing "current" to get the DOM node
            this.textInput.current.focus();
        }

        render() {
            // tell React that we want to associate the <input> ref
            // with the `textInput` that we created in the constructor
            return (
                <div>
                    <input type="text" ref={this.textInput} />
                    <input type="button"
                        value="Focus the text input"
                        onClick={this.focusTextInput}
                    />
                </div>
            );
        }
    }

现在 ```textInput``` 就指向 ```<input/>``` 实例，通过 ```foucusTextInput()``` 你就可以让它获得焦点。

### 传给类组件

现在我们再定义一个新组件，这个组件一挂载就自动获取焦点。我们只用用一个 Refs 指向 ```CustomTextInput``` ，这样就可以通过这个 ```Refs.current``` 直接调它的 ```textInput```方法，从而调到 DOM 元素 ```<input/>``` 的方法 ```foucs()```。

    class AutoFocusTextInput extends React.Component {
        constructor(props) {
            super(props);
            this.textInput = React.createRef();
        }

        componentDidMount() {
            this.textInput.current.focusTextInput();
        }

        render() {
            return (
                <CustomTextInput ref={this.textInput} />
            );
        }
    }

对于自定义类组件或者 DOM 元素，其本身不需要做什么特殊处理，由 React 控制把传进来的 ref 绑定到它们的实例上。换句话来说，你也没办法控制传进来的 ref，你如果想把传进来的 ref 传递给深层的 child，就要把 Refs 当普通 props 传递，或者用 ```React.forwardRef()```。
 
### 函数组件传递 Refs 给它的 child

我们知道了怎么指向类组件元素、DOM 元素的实例，那么对与函数组件类型的元素呢？

函数组件是函数，不是类，它没有实例。所以你没办法让 Refs **指向**一个函数组件。

但是不影响你在函数组件中创建一个 Refs 对象，然后让它指向自己的子组件，**只要这个组件是类组件或者 DOM 元素就是可以的**。

    function CustomTextInput(props) {
        // textInput must be declared here so the ref can refer to it
        const textInput = useRef(null);
        
        function handleClick() {
            textInput.current.focus();
        }

        return (
            <div>
            <input
                type="text"
                ref={textInput} />
            <input
                type="button"
                value="Focus the text input"
                onClick={handleClick}
            />
            </div>
        );
    }

### useRef()

在函数组件中创建 Refs 就要用这个函数，因为函数组件不像类组件有构造函数。为了保证只创建一次只能用 ```useRef()```。

在用于 ref 的传递这方面，它几乎和类组件里面用的 ```createRef()``` 一样，唯一不同的是函数组件没有生命周期方法，不知道 ```current``` 什么时候被赋值，什么时候被设置为 null，如果想在这时候执行一些代码的话可以用 Callback Refs。

我们来看看它的细节：

    const refContainer = useRef(initialValue);

+ 它的返回值和 ```React.createRef()``` 一样是一个 Refs 对象，它可以接受一个参数做为 ```current``` 的初始值。
+ 它在组件的同一个生命周期里只执行一次，在组件的整个生命周期内这个 Refs 对象都存在——不会变，这也是和你手动创建一个 ```{ current:xxx }``` 唯一的区别。
+ ```current``` 的改变也不会触发 re-render。

**用它创建的 Refs 就相当于类组件里面的成员变量**，所以它的作用不仅仅是用在 ref 上，你可以用它在函数组件里面声明变量，就像类里面的成员变量一样。


# Callback Refs

Callback Refs 概念很简单，传给 ```ref``` 的不是对象 Refs ，而是一个回调函数 Refs，当指向和取消指向实例时都会回调这个函数，并把指向的实体作为参数传回。这样有更精准的控制(不知道为什么会有更精准的控制，后面再研究)。

下面是类组件使用 Callback Refs 的例子：

    class CustomTextInput extends React.Component {
        constructor(props) {
            super(props);

            this.textInput = null;

            this.setTextInputRef = element => {
                this.textInput = element;
            };

            this.focusTextInput = () => {
                // Focus the text input using the raw DOM API
                if (this.textInput) this.textInput.focus();
            };
        }

        componentDidMount() {
            // autofocus the input on mount
            this.focusTextInput();
        }

        render() {
            // Use the `ref` callback to store a reference to the text input DOM
            // element in an instance field (for example, this.textInput).
            return (
                <div>
                    <input
                        type="text"
                        ref={this.setTextInputRef}
                    />
                    <input
                        type="button"
                        value="Focus the text input"
                        onClick={this.focusTextInput}
                    />
                </div>
            );
        }
    }

这种方式比 ```createRef()```有什么优势呢？说实话，真没看出来。官方文档说它有更精准的控制，但是它的回调函数被调用的时机、或者说 ```ref``` 被赋值/置 null 的时机和 Refs 的方式没什么两样，都是在挂载的时候赋值，在卸载的时候置为 null，保证在 ```componentDidmount```、```componentDidUpdate``` 调用之前，所指向的实例已经更新到最新。

只能找到一点明显的不同：

就是你通过回调拿到实例后会把它存在一个普通变量上，而不是 ```.current``` 上面。

如果你想在 ```ref``` 被赋值/置 null 的时候做一些操作，那么这种方式就有优势，它可以把代码都集中在回调函数中，而不需要把代码分散在 ```componentDidmount```、```componentDidUpdate``` 中。并且对于函数组件而言要做这些事情只能用这种方式，因为函数组件没有生命周期方法（是不是可以在 ```useEffect``` 里面做这些事情呢？）


# Forwarding Refs

前面种种都是教我们如何在组件内部创建一个 Refs, 然后传给 child，从而拿到 child 的实例。那有没有办法直接把 child 实例暴露给组件的父组件呢？ 这样父组件就可以直接穿透组件，直接访问到孙子组件的实例。Forwarding Refs 就是做这个的。

    function FancyButton(props) {
        return (
            <button className="FancyButton">
                {props.children}
            </button>
        );
    }

如上例：通常调用 ```FancyButton``` 的地方(组件) 不需要知道 ```FancyButton``` 的实现细节，也不需要访问内部的 ```<button/>```。但是同样是上面说的那些原因，我们需要在外面直接访问 ```<button/>``` 的实例，这时候就用 ```React.forwardRef``` 把外面传进来了 Refs 转给自己的 child：

    const FancyButton = React.forwardRef((props, ref) => (
        <button ref={ref} className="FancyButton">
            {props.children}
        </button>
    ));

    // You can now get a ref directly to the DOM button:
    const ref = React.createRef();
    <FancyButton ref={ref}>Click me!</FancyButton>;

所以，看到给一个元素的 ref 传值的时候就要注意了，你要看看这个元素是否被 ```React.forwardRef``` 函数包裹过，如果不是，这个 Refs 指向的只是组件本身的实例，并不是指向它的某一个 child。

```React.forwardRef``` 既可以包裹函数组件，也可以包裹类组件，并没有限制。

### 在高阶组件中的运用

    function logProps(WrappedComponent) {
        class LogProps extends React.Component {
                componentDidUpdate(prevProps) {
                console.log('old props:', prevProps);
                console.log('new props:', this.props);
            }

            render() {
                return <WrappedComponent {...this.props} />;
            }
        }

        return LogProps;
    }

这个高阶组件的功能很简单，就是给被包裹的组件添加打印 ```props``` 的功能，例如，上面的 ```FrancyButton``` 就可以用：

    class FancyButton extends React.Component {
        focus() {
            // ...
        }

        // ...
    }

    // Rather than exporting FancyButton, we export LogProps.
    // It will render a FancyButton though.
    export default logProps(FancyButton);

从高阶组件 ```render``` 中我们看到，它转发了外面传进来的所有 ```props```，但是外面通过 ```ref = xxx``` 传进来的 Refs 它不会转发给被包裹的组件，因为 ref 不是props，它和 key 一样是 React 保留关键字。所以这样传的话 Refs 并不会指向被包裹组件的实例，而是指向包裹组件的实例。

要想把外面传进来的 Refs 转给被包裹的组件，就要用 ```React.forwardRef```，像上面这个例子就这样改：

    function logProps(Component) {
        class LogProps extends React.Component {
            componentDidUpdate(prevProps) {
                console.log('old props:', prevProps);
                console.log('new props:', this.props);
            }

            render() {
                //这里取出被转发的 Refs 
                const {forwardedRef, ...rest} = this.props;

                // 这里才真正的把转发的 Refs 绑定到 Component 组件上
                return <Component ref={forwardedRef} {...rest} />;
            }
        }

        // 和修改前相比，这里返回容器组件之前用 React.forwardRef 包裹了一下.
        // 然后接受到 ref 参数后把它作为容器组件的一个普通 props 传递
        // 这里是关键，因为作为普通 props 传递就不会绑定到容器组件上，就可以继续传递下去
        return React.forwardRef((props, ref) => {
            return <LogProps {...props} forwardedRef={ref} />;
        });
    }





