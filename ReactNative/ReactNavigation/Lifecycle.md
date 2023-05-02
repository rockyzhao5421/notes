# 生命周期

假设有一个 stack navigator，里面有两个 route: ```Home``` 和 ```Detail```,
 当从 ```Home``` 跳到 ```Detail``` 的时候， ```Home``` 会发生什么事情？ 跳回来的时候又
 会发生什么？ ```Home``` 怎么知道用户什么时候离开？什么时候回来？

 React 是通过生命周期函数来实现上面提到的功能，那么 React Navigation 库呢？  
 虽然这些生命周期函数依然可用，但是在 React Navigation 里面，它们的用法却不同。

 以上面的 stack navigator 为例，当你导航到 ```Home``` 时，它的 ```componentDidMount``` 函数会被调用；  
 从 ```Home```导航到 ```Detail```, ```Detail``` 的 ```componentDidMount``` 也会被调用，但是这时候 ```Home``` 仍然挂载在 stack 上。因此，它的 ```componentWillUnmount``` 也不会调用。

 当从 ```Detail``` 回到 ```Home``` 的时候，```Detail``` 的 ```componentWillUnmount``` 会被调用，但是 ```Home``` 的 ```componentDidMount```
 就不会调了，因为 ```Home``` 一直挂载着呢。

其它类型的 navigator 也是一样：假设一个 tab navigator 包含两个 tab，每个 tab 都是一个
stack navigator：

    function App() {
        return (
            <NavigationContainer>
                <Tab.Navigator>
                    <Tab.Screen name="First">
                        {() => (
                            <SettingsStack.Navigator>
                                <SettingsStack.Screen
                                    name="Settings"
                                    component={SettingsScreen}
                                />
                                <SettingsStack.Screen name="Profile" component={ProfileScreen} />
                            </SettingsStack.Navigator>
                        )}
                    </Tab.Screen>
                    <Tab.Screen name="Second">
                        {() => (
                            <HomeStack.Navigator>
                                <HomeStack.Screen name="Home" component={HomeScreen} />
                                <HomeStack.Screen name="Details" component={DetailsScreen} />
                            </HomeStack.Navigator>
                        )}
                    </Tab.Screen>
                </Tab.Navigator>
            </NavigationContainer>
        );
    }

假设我们开始是在 ```HomeScreen```，然后跳到 ```DetailScreen```。然后通过 tab 切换到 ```SettingScreen```，最后跳到 ```ProfileScreen```。这一通操作之后，4个 screen 都挂载了，这时候你再通过 tab 切回到 Home stack，你看到将是 ```DetailScreen```，因为之前 Home stack 的状态已经保存了——最后显示的是 ```DetailScreen```。


# 生命周期事件

现在我们大概知道了在 React Nivagation中，React 的生命周期函数是如何工作的了。现在我们可以回答开始提出的问题了:
怎么知道用户离开或者返回到了 route？

React Navigation 是通过发送 Event 给 screen component，然后 screen component 通过 ```props.addListener()```
监听 Event 来知道什么时候获取焦点，什么时候失去焦点的。

例子：

    function Profile({ navigation }) {
        React.useEffect(() => {
                const unsubscribe = navigation.addListener('focus', () => {
                // Screen was focused
                // Do something
            });

            return unsubscribe;
        }, [navigation]);

        return <ProfileContent />;
    }

具体 React Navigation 都提供了哪些 Event API ，请参考 [Navigation events](https://reactnavigation.org/docs/navigation-events)。

上面的例子是用 React 的钩子函数来手动添加 listener，React Navigation 还提供了专门的钩子函数，可以自动添加 listener：

import { useFocusEffect } from '@react-navigation/native';

    function Profile() {
        useFocusEffect(
            React.useCallback(() => {
                // Do something when the screen is focused

                return () => {
                    // Do something when the screen is unfocused
                    // Useful for cleanup functions
                };
            }, [])
        );

        return <ProfileContent />;
    }

如果你想在获取焦点和失去焦点的时候，渲染不同的 UI ，可以用 useIsFocused 函数。