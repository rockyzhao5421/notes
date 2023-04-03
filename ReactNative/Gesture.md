# Gesture Responder System

React Native 中，响应手势的基本单位是 Responder，具体来说，就是最常见的 View 组件。任何 View 组件，都是潜在的 Responder，如果某个 View 组件没有响应手势操作，那是因为它还没有被“开发”。

将一个普通的 View 组件开发成为一个能响应手势操作的 Responder，非常简单，只需要按照 React Native 的 gesture responder system 的规范，在 props 上设置几个生命周期函数即可：

+ View.props.onStartShouldSetResponder
+ View.props.onMoveShouldSetResponder
+ View.props.onResponderGrant
+ View.props.onResponderReject
+ View.props.onResponderMove
+ View.props.onResponderRelease
+ View.props.onResponderTerminationRequest
+ View.props.onResponderTerminate

一个触摸事件处理周期，是从用户手指按下屏幕，到用户抬起手指抬起结束，这是用户的一次完整触摸操作，也就是一个 Responser 的生命周期，在这个生命周期的不同阶段，会调用上面相应的生命周期函数。

在看具体的调用流程之前我们要记住一点：**一个应用中同时只能有一个 Responder (被激活)**。

### 申请成为 Responder

一个组建要成为一个 Responder 是通过下面两个生命周期回调函数：

    View.props.onStartShouldSetResponder: (evt) => [true | false]

在用户开始触摸的时候（手指刚刚接触屏幕的瞬间），是否愿意成为 Responder。

**注意：此回调方法只在按下的时候调用一次！**

    View.props.onMoveShouldSetResponder: (evt) => [true | false]

如果第一个函数返回 false， 那么**每次** move 都会调一次这个函数，询问是否愿意成为 Responder。

如果这两个函数没有一个愿意成为 Responder, 本次周期到此结束。

### 申请成功了吗？

不是说你让 onStartShouldSetResponder 或者 onMoveShouldSetResponder 返回 true 就一定能申请成功，简单来说那只是表明你愿意成为 Responder。

    View.props.onResponderGrant: (evt) => {}

申请成功就会回调这个函数，表示发出申请的 View 已经可以开始响应触摸事件了。一般来说，这个时候需要去改变组建的底色或者透明度，来表示组件已经被激活。

    View.props.onResponderReject: (evt) => {}

申请失败就会回调这个函数，申请失败的原因就是我们前面所说的：**同时只能有一个处理事情的 Responder**。如果你在申请成为 Responder 的时候，此时别的组件正在响应用户操作，也就是说别的组件现在是 Responder，而且不放弃当前的响应权利，那么就会回调这个方法，通知你申请失败。本次周期也就到此结束。

### 开始正式响应

如果 View 已经开始响应触摸事件了，那么下列这些处理函数会被一一调用：

    View.props.onResponderMove: (evt) => {}

用户在移动手指时会不断调用这个函数。

    View.props.onResponderRelease: (evt) => {}

手指松开，此时组件一般要恢复到触摸之前的状态，例如底色和透明度恢复之前的样式，完成一次手势操作。

    View.props.onResponderTerminationRequest: (evt) => true | false

有其他组件请求成为 Responder 时回调当前组建的该函数，当前的 View 是否“放权”，如果返回 ture，则当前 Responder 组件释放响应者权利，发出请求的组件请求成功（回调 ```onResponderGrant```），相反的返回 false，则表示当前组件不放权，则发出请求的组件请求失败（回调 ```onResponderReject```）。

    View.props.onResponderTerminate: (evt) => {} 

当前组建的 Responder 权限已经被取走时调用，可能是被其他组件通过 ```onResponderTerminationRequest``` 申请取走，也可能是被系统强制取走，比如如然来电话。

### 回调函数参数 evt

结构如下：


+ nativeEvent
  + changedTouches - evt 数组，从上次回调上报的触摸事件，到这次上报之间的所有事件数组。因为用户触摸过程中，会产生大量事件，有时候可能没有及时上报，系统用这种方式批量上报；
  + identifier - 触摸的 ID，一般对应手指，在多点触控的时候，用来区分是哪个手指的触摸事件；
  + locationX / locationY - 触摸点相对组件的位置
  + pageX / pageY - 触摸点相对于屏幕的位置
  + target - The node id of the element receiving the touch event
  + timestamp - A time identifier for the touch, useful for velocity calculation
  + touches - evt 数组，多点触摸的时候，包含当前所有触摸点的事件

  这些数据中，最常用的是 locationX 和 locationY 数据，需要注意的是，因为这里是 Native 的数据，所以他们的单位是实际像素。如果要转换为 RN 中的逻辑单位，可以示使用如下方法：

        var pX = evt.nativeEvent.locationX / PixelRatio.get(); 

### 劫持 ShouldSet 事件处理

```onStartShouldSetResponder``` 与 ```onMoveShouldSetResponder``` 是以冒泡的形式调用的，即最深的节点最先调用。这意味着当多个 View 同时在 *ShouldSetResponder 中返回 true 时，最底层的 View 将优先“夺权”。

但是有些时候，某个父 View 会希望能先成为响应者。RN 提供了一个劫持机制，也就是在触摸事件往下传递的时候，先询问父组件是否需要劫持，不给子组件传递事件，也就是如下两个回调：

    View.props.onStartShouldSetResponderCapture: (evt) => true,

在触摸事件开始（touchDown）的时候，会先回调此函数，询问组件是否要劫持事件响应者设置，自己接收事件处理，如果返回 true，表示需要劫持；

    View.props.onMoveShouldSetResponderCapture: (evt) => true,

此函数类似，不过是在触摸移动事件（touchMove）询问容器组件是否劫持。

上面两个函数任意一个返回 true 就表示劫持。可以把这种劫持机制看成是一种下沉机制，与上面的冒泡机制对应。

例如：有三个组件 A，B，C。B 是 A 的子组件，C 是 B 的子组件。触摸事件开始，首先调用 A 组件的 ```on*ShouldSetResponderCapture```，若返回 false，则传递到 B 组件，然后调用 B 组件 ```on*ShouldSetResponderCapture```，若返回 true，则事件不再传递给 C 组件，直接调用本组件的 ```onResponder*```(我觉得应该先调用 ```on*ShouldSetResponder```，后面再验证吧)，则 B 组件就成为事件响应者，后续事件直接传递给它。

### Touchable*

一般情况下直接使用 Gesture Responder System 太麻烦了，所以 RN 提了几个实现好的 touch 组件:

    TouchableHighlight
    TouchableOpacity
    TouchableNativeFeedback
    TouchableWithoutFeedback


前三个组件都是在第四个TouchableWithoutFeedback的基础上增强而来，所以TouchableWithoutFeedback的所有基本属性，前边3个组件都有。

因为这几个组件的功能和使用方法基本类似，只是 Touch 的反馈效果不一样，所以一般我们用 Touchable** 代替。

    TouchableHighlight

这个组件的作用，除了给内部元素增加绑定事件之外，还负责给内部元素增加“点击状态”。所谓的“点击状态”，就是在用户在点击的时候，会产生一个短暂出现覆盖层，用来告诉用户这个区块被点击到了。

    TouchableOpacity


这个组件用来给为内部元素在点击时添加透明度。

    TouchableNativeFeedback

这个组件只能用在安卓上，它可以针对点击，在点击区域中显示不同的效果，例如最新安卓系统中的Material Design的点击波纹效果。

    TouchableWithoutFeedback

这个组件只响应touch手势，不增加点击态，不推荐使用。

Touchable** 有如下几个回调方法：

    onPressIn：点击开始；
    onPressOut：点击结束或者离开；
    onPress：单击事件回调；
    onLongPress：长按事件回调。

+ 快速点击，只会触发 press 事件
+ 只要在点击时有一个“按”的操作，就是比快速点击要按的久一点，就会触发 pressin 事件
+ 如果同时绑定了 pressIn, pressOut 和 press 事件，那么当 pressIn 事件触发之后，如果用户的手指在绑定的组件中释放，那么接着会连续触发 pressOut 和 press 事件，此时的顺序是 pressIn -> pressOut -> press。而如果用户的手指滑到了绑定组件之外才释放，那么此时将会不触发 press 事件，只会触发 pressOut 事件，此时的顺序是 pressIn -> pressOut。后一种情况就是中途取消，如果我们将回调函数绑定给press事件，那么后一种情况中回调函数并不会被触发，相当于"被取消"。
+ 如果绑定了 longPress 事件，那么在pressIn事件被触发之后，press 事件不会被触发，通过打点计时，可以发现 longPress 事件的触发时间大概是在 pressIn 事件发生383ms之后，当longPress 事件触发之后，无论用户的手指在哪里释放，都会接着触发pressOut事件，此时的触发顺序是 pressIn -> longPress -> pressOut

# PanResponder

前面只是介绍了简单的触摸事件处理机制及其使用方法，其实连续的触摸事件，可以组成一些更高级手势，例如我们最常见的滑动屏幕内容，双指缩放（Pinch）或者旋转图片都是通过手势识别完成的。

因为有些手势是很常用的，RN 也提供了内置的手势识别库 PanResponder，它封装了上面的事件回调函数，对触摸事件数据进行加工，完成滑动手势识别，向我们提供更加高级有意义的接口，如下：

+ onMoveShouldSetPanResponder: (e, gestureState) => {...}
+ onMoveShouldSetPanResponderCapture: (e, gestureState) => {...}
+ onStartShouldSetPanResponder: (e, gestureState) => {...}
+ onStartShouldSetPanResponderCapture: (e, gestureState) => {...}
+ onPanResponderReject: (e, gestureState) => {...}
+ onPanResponderGrant: (e, gestureState) => {...}
+ onPanResponderStart: (e, gestureState) => {...}
+ onPanResponderEnd: (e, gestureState) => {...}
+ onPanResponderRelease: (e, gestureState) => {...}
+ onPanResponderMove: (e, gestureState) => {...}
+ onPanResponderTerminate: (e, gestureState) => {...}
+ onPanResponderTerminationRequest: (e, gestureState) => {...}
+ onShouldBlockNativeResponder: (e, gestureState) => {...}

可以看到，这些接口与前面接收的基础回调基本上是一一对应的，其功能也是类似。其实这些新的回调都是原来回调函数的增强版本，其实就是多了一个 ```gestureState``` 参数。他的结构如下：

+ stateID：滑动手势的 ID，在一次完整的交互中此 ID 保持不变；
+ moveX 和 moveY：最近一次移动时的屏幕坐标；
+ x0 和 y0：滑动手势识别开始的时候的在屏幕中的坐标，就是从　Grant　开始算起；
+ dx 和 dy：从手势开始时，到当前回调时移动的距离；
+ vx 和 vy：当前手势移动的速度；
+ numberActiveTouches：当期触摸手指数量。

In general, for events that have capture equivalents, we update the gestureState once in the capture phase and can use it in the bubble phase as well.

Be careful with onStartShould* callbacks. They only reflect updated gestureState for start/end events that bubble/capture to the Node. Once the node is the responder, you can rely on every start/end event being processed by the gesture and gestureState being updated accordingly. (numberActiveTouches) may not be totally accurate unless you are the responder.

官方拖动的例子：

    const App = () => {
    const pan = useRef(new Animated.ValueXY()).current;

    const panResponder = useRef(
        PanResponder.create({
            onMoveShouldSetPanResponder: () => true,
            onPanResponderGrant: () => {
                pan.setOffset({
                    x: pan.x._value,
                    y: pan.y._value
                });
            },
            onPanResponderMove: Animated.event(
                [
                    null,
                    { dx: pan.x, dy: pan.y }
                ]
            ),
            onPanResponderRelease: () => {
                pan.flattenOffset();
            }
        })
    ).current;

    return (
        <View style={styles.container}>
            <Text style={styles.titleText}>Drag this box!</Text>
            <Animated.View
                style={{
                    transform: [{ translateX: pan.x }, { translateY: pan.y }]
                }}
                {...panResponder.panHandlers}
            >
                <View style={styles.box} />
            </Animated.View>
        </View>
    );
    }

    const styles = StyleSheet.create({
    container: {
        flex: 1,
        alignItems: "center",
        justifyContent: "center"
    },
    titleText: {
        fontSize: 14,
        lineHeight: 24,
        fontWeight: "bold"
    },
    box: {
        height: 150,
        width: 150,
        backgroundColor: "blue",
        borderRadius: 5
    }
    });

    export default App;

可以看到用法很简单

+ 建一个 PanResponder 实例，并设置想好相关的属性。
+ 然后把这个对象设置给 View 的属性，如下:

        <View  
            {...this._panResponder.panHandlers}
        />