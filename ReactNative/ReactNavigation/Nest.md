嵌套 navigation 就是在一个 screen component 里面嵌套一个 navigator，嵌套 navigation
和组件嵌套一样，因为它本身也是 React Component。

    function Home() {
        return (
            <Tab.Navigator>
                <Tab.Screen name="Feed" component={Feed} />
                <Tab.Screen name="Messages" component={Messages} />
            </Tab.Navigator>
        );
    }

    function App() {
        return (
            <NavigationContainer>
                <Stack.Navigator>
                    <Stack.Screen
                        name="Home"
                        component={Home}
                        options={{ headerShown: false }}
                    />
                    <Stack.Screen name="Profile" component={Profile} />
                    <Stack.Screen name="Settings" component={Settings} />
                </Stack.Navigator>
            </NavigationContainer>
        );
    }


# 注意事项

### 每个 navigator 的导航 history 都是独立的

当你在一个 sub stack navigator screen 中按返回按钮时，不管它有没有父 navigator，它
都会返回到它自己 stack 中的上一个 screen。

### 每一个 navigator 的 ```options``` 都是独立的

例如，你通过 sub stack navigator screen 的 title option 给了一个 title string，
它不会影响父 navigator 中的 title。

### 每个 navigator 的 ```params``` 都是独立的

例如，传给一个嵌套 navigator 中 screen 的 ```params```，只会存在于
该 screen 的 ```route``` prop 中，它的 sub navigator 或者 父 navigator
中的 screen 都访问不了这些 ```params```。

如果 sub screen 要访问父 screen 的 ```params```，可以通过 React Contex
把 ```params``` 暴露给 sub screen。

### 只有 sub navigator 处理不了的动作，才会抛给父 navigator

例如，如果已经在 sub navigator 的第一个 screen 了，再 call ```navigation.goBack()```, 就会导航到父 navigator 中。其它的导航动作也一样，只有 sub navigator 处理不了的动作，才会抛给父 navigator。

对于上面的例子，如果在 Feed screen call ```navigate('Message')```，tab
navigator 就处理了，如果 call ```navigate('Settings')```,就会交由 stack navigator 来处理。

### sub navigator 可以 call 父 navigator 的特有函数

例如，stack 嵌套在 drawer 里面，那么 stack screen 的 ```props.navigation``` 里面也包含了 drawer 的 ```openDrawer```,```closeDrawer```, ```toggleDrawer``` 这些函数可以调用，但是反过来就不行：父 navigator screen 就不能 call sub navigator 的特有函数。  
其它 navigator 相互嵌套也是一样。

如果你想从父 navigator dispatch actions 到 sub navigator，可以用
```navigation.dispatch```:

    navigation.dispatch(DrawerActions.toggleDrawer());