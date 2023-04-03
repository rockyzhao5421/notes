# 概述

RN 提供了 Animated API 来实现动画;

我们知道所谓动画，无非就是通过平移、缩放、旋转，透明度变换，以及这些基本变换的组合; Animated API 也是通过对组件的 opacity、transform(translateX,translateY,rotate,scale)进行修改来实现平移，缩放，旋转，透明度变换；

RN 实现一个动画分为三步：

1. **声明一个 ```Value```**。组件需要变化的属性必须使用： ```Animated.Value``` 或者 ```Animated.ValueXY``` 作为值。

    Animated.Value提供了以下方法：

    + constructor：构造函数。new Animated.Value(0)
    + setValue：设置新的值，注意此方法会停止动画
    + setOffset：设置一个修正值，不论接下来的值是由setValue、一个动画，还是Animated.event产生的，都会加上这个值。常用来在拖动操作一开始的时候用来记录一个修正值（譬如当前手指位置和View位置）。
    + flattenOffset：该用来把相对值合并到值里，然后相对值设置为0，最终输出的值不会发生变化。常常在拖动操作结束以后调用。
    + addListener：异步监听动画值变化
    + removeListener：删除指定监听器
    + removeAllListeners：删除所有监听器
    + stopAnimation：停止动画，返回当前动画值。
    + interpolate：差值，可以将值映射成新的值，后面会具体介绍。

    Animated.ValueXY其实是就是2个Animated.Value，方法和Animated.Value一样，不过比Animated.Value 多了2个方法：

        getLayout();

    Converts {x, y} into {left, top} for use in style:

        style={this.state.anim.getLayout()}

    另外一个：

        getTranslateTransform();

    Converts {x, y} into a useable translation transform：

        style={{
            transform: this.state.anim.getTranslateTransform()
        }}

1. **把 ```Value``` 和动画组件的属性绑定**。需要进行动画的组件一般是 ```Animated```组件: 
    + ```Animated.View```
    + ```Animated.Image```
    + ```Animated.Text```
    + ```Animated.ScrollView```
    + ```Animated.FlatList``` 
    + ```Anmated.SectionList```
  
    当然，也可以用 ```Animated.createAnimatedComponent()``` 自定义动画组件。

2. **最后创建动画**，执行该动画就能按照某中方式不断的改变 ```Value```，这样就有了动画效果。```Animated``` 提供了三个函数来创建三种不同的动画：
   + ```Animated.timing()```：最常用的动画类型,使得值按照过渡曲线随时间变化
   + ```Animated.spring()```：弹簧变化效果
   + ```Animated.decay()```：衰变效果，以一个初始的速度和一个衰减系数逐渐减慢为 0
  
    这里只说一下 ```Animated.timing()```，另外两个用到再学吧:

        static timing(value, config)

   + value：就是 ```Animated.Value``` 或者 ```Animated.ValueXY```
   + config: 这是一个对象，里面的属性都是可选的：
      - duration: 不解释，默认 500 毫秒.
      - easing: Easing 函数，就是定义如何变化 value 的，是加速还是减速…… 默认是 Easing.inOut(Easing.ease).
      - delay: 调用 start() 后延迟多少毫秒开始执行. 默认是 0.
      - isInteraction: Whether or not this animation creates an "interaction handle" on the InteractionManager. Default true.
      - useNativeDriver: Uses the native driver when true. Default false.

### 示例

透明度：

    import React, { useRef, useEffect } from 'react';
    import { Animated, Text, View } from 'react-native';

    const FadeInView = (props) => {
    // 1. 声明 Value；
    const fadeAnim = useRef(new Animated.Value(0)).current  
    useEffect(() => {
        //3. 最后创建并执行动画，从而按照某中方式不断的改变这个值。
        Animated.timing(
            fadeAnim,
            {
                toValue: 1,
                duration: 10000,
            }
        ).start();
    }, [fadeAnim])

    return (
        // 2. 把这个 Animated.Value 和动画组件的属性绑定
        <Animated.View                
            style={{
                ...props.style,
                // Bind opacity to animated value
                opacity: fadeAnim,         
            }}
            >
            {props.children}
        </Animated.View>
    );
    }

    // You can then use your `FadeInView` in place of a `View` in your components:
    export default () => {
        return (
            <View style={{flex: 1, alignItems: 'center', justifyContent: 'center'}}>
                <FadeInView style={{width: 250, height: 50, backgroundColor: 'powderblue'}}>
                    <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>Fading in</Text>
                </FadeInView>
            </View>
        )
    }

平移：


    const TransView = (props) => {
        const transX = useRef(new Animated.Value(0)).current;
        useEffect(() => {
            Animated.timing(transX,
                {toValue:150,duration:3000}
            ).start();
        },[transX]);
       
        render() {
            return (
                <View>
                    <Animated.View style={{transform:[{translateX:transX}]}}/>
                </View>
        }
    }

旋转：

    const DegreeView = (props) => {
        const degree=useRef(new Animated.Value(0)).current;
        useEffect(() => {
            Animated.timing(degree,
                    {toValue:1,duration:3000,}
                ).start();
            }, [degree]);
        render() {
            const realDeg=degree.interpolate({
            inputRange:[0,1],outputRange:['0deg','360deg']});
            return 
                <View >
                    <Animated.Image style={{transform:[{rotate:realDeg}]}} />
                </View>
        }
    }

缩放：

    const ScaleView = (props) => {
        scale1=new Animated.Value(0)
        useEffect(() => {
            Animated.timing(scale,
                    {
                        toValue:1,
                        duration:3000,
                    }
                ).start();
            },[scale]);

        render() {
            return (
                <View >
                    <Animated.View style={{transform:[{scale:scale}]}}/>
                </View>
        }
    }

# 组合动画

复杂的动画就是上面基本动画的组合，Animated API中提供了以下方法来实现组合动画，这些函数都是接收一个动画数组为参数，返回值也是一个动画，也就是说数组的 item 也可以是数组：

       static sequence(animations)

按顺序依次执行数组中的动画，前一个结束后一个才开始执行。如果当前动画被终止-主动调用了 ```stop()``` 后面的就不会被执行。

       static parallel(animations, config?)

数组里面的动画同时开始，默认情况，其中有一个执行结束了或者被终止，其它的也会停止。可以通过 ```config``` 参数来关掉这个默认行为：
    {stopTogether： false}

       static stagger(time, animations)

参数是一个延迟时间和一个动画数组，前一个动画开始一段时间后，后一个动画开始；因为仅仅是前一个动画开始后指定时间，前一个动画并不一定结束了，所以会出现同时执行（重叠）的情况;

# 合成数值

还可以用两个 ```Value``` 进行加、减、乘、除，以及取模运算，得到一个新的 ```Value```。 Animated 也提供了对应的 API：

+ ```Animated.add()```
+ ```Animated.subtract()```
+ ```Animated.divide()```
+ ```Animated.modulo()```
+ ```Animated.multiply()```

# 插值函数

    interpolate(config);

这个函数是 ```Animated.Value``` 的函数，```AnimatedXY``` 中的 X，Y 也有这个函数，其中 ```config``` 参数有一下属性：

+ inputRange:数字数组，必须和 outputRange 个数相同 
+ outputRange:数字数组，或者字符串数组，必须和 inputRange 个数相同 
+ easing (optional):可选属性，提供一个 Easing 函数 
+ extrapolate (optional): a string such as 'extend', 'identity', or 'clamp'
+ extrapolateLeft (optional): a string such as 'extend', 'identity', or 'clamp'
+ extrapolateRight (optional): a string such as 'extend', 'identity', or 'clamp'

这个函数的作用就是将输入值范围转换为输出值范围，默认是线性转换，也可以传入 Easing 函数。**必须保持输入输出数组的元素个数是一样的**；并且 ```inputRange``` **里面的数字必须是递增的**。

### 简单的例子

最简单的例子，把 0～1 转换为 0～100：

    value.interpolate({
        inputRange: [0, 1],
        outputRange: [0, 100]
    }); 

### 从数字到字符串的映射

也支持从数字到字符串的映射，从而可以实现颜色以及带有单位的值的动画变换：

    value.interpolate({
        inputRange: [0, 360],
        outputRange: ['0deg', '360deg']
    });

### 设置边界

先看一个让透明度和拖动关联起来的例子：

    this._animatedValue = new Animated.ValueXY();

    this._opacityAnimation = this._animatedValue.x.interpolate({
        inputRange: [0, 150],
        outputRange: [1, .2]
    });

用户在拖动的过程中在X轴上位置的变化对应到透明度的值上。X轴的位置0到150，对应到透明度的1到0.2。所以，当x轴的位置为0的时候，透明度就是1，当x的值为75的时候透明度为0.4。

户拖动的距离在x轴上大于150呢？有些人会觉得透明度会保持在0.2上不再改变。这并不是默认的情况。

必须指定插值器如何处理这种情况。如果你想让透明度停留在0.2上，不管X轴的距离超出了多少，那么就指定```extrapolate:'clamp'```，默认值是 ```extend``` ，具体是这样的：

    this._opacityAnimation = this._animatedValue.x.interpolate({
        inputRange: [0, 150],
        outputRange: [1, .2],
        extrapolate: 'clamp'
    });




interpolate 一般用于多个动画共用一个 Animated.Value，只需要在每个属性里面映射好对应的值，就可以用一个变量控制多个动画：

    style={{
        opacity: this.state.fadeAnim, // Binds directly
        transform: [{
            translateY: this.state.fadeAnim.interpolate({
                inputRange: [0, 1],
                outputRange: [150, 0]  // 0 : 150, 0.5 : 75, 1 : 0
            }),
        }],
    }}

也**支持多个范围片段**，来实现一个非线性的映射：

    value.interpolate({
        inputRange: [-300, -100, 0, 100, 101],
        outputRange: [300, 0, 1, 0, 0]
    });

最终的映射结果大概是这样：

| 输入 | 输出 |
| ---- | ---- |
| -400 | 450  |
| -300 | 300  |
| -200 | 150  |
| -100 | 0    |
| -50  | 0.5  |
| 0    | 1    |
| 50   | 0.5  |
| 100  | 0    |
| 101  | 0    |
| 200  | 0    |


设想在 RN 如何实现一段抽奖转盘的动画：20%前慢，20-80%快，80-90%减慢，90-100%摇摆一下回正。 难道用顺序动画去写？感觉插值更方便，也许你没注意到它写动画中的作用。

    inputRange: [0, 0.2, 0.8, 0.9, 1], 
    outputRange: ['0deg', '100deg', '800deg', '830deg', '810deg'],

# Tracking dynamic values

```toValue``` 也可以设置成 ```Animated.Value```，比如我们可以用弹跳动画来实现聊天头像的闪动，又比如通过timing设置duration:0来实现快速的跟随。

    Animated.spring(follower, { toValue: leader }).start();
    Animated.timing(opacity, {
        toValue: pan.x.interpolate({
            inputRange: [0, 300],
            outputRange: [1, 0]
        })
    }).start();