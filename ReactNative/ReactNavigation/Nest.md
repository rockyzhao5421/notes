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

### sub navigator 收不到父 navigator 的 Event

例如，stack 嵌套在 tab 里面，stack 中的 screen component 通过 ```navigation.addListener()```监听不到 tab
的 Event（比如 ```tabPress```）。  

如果要实现这一点，只能通过 ```navigation.getParent``` 来做：

    const unsubscribe = navigation
            .getParent('MyTabs')
            .addListener('tabPress', (e) => {
            // Do something
        });

```getParent``` 返回父 navigator 的 ```props.navigation```，如果有多个（层） parent，还可以给 parent navigator 一个 ```id``` 来标识：


    <Drawer.Navigator id="LeftDrawer">
        {/* .. */}
    </Drawer.Navigator>

### parent navigator 的 UI 会盖在 sub navigator UI 的上面

例如，如果你把 stack 嵌套在 drawer 里面，你就会发现测边抽屉栏会出现在 stack header的上面，反过来，如果你把 drawer
嵌套在 stack 的里面，你就会发现侧边抽屉栏会出现在 stack header下面。这一点在考虑如何嵌套 navigator 的时候很重要。

以下是常规用法：

+ 把 tab navigator 嵌套在 stack navigator 的 inital screen 中，这样跳转到其它 stack screen 的时候就会盖住
  tab bar。
+ 把 drawer navigator 嵌套在 stack navigator 的 inital screen 中，并且隐藏 stack header，这样 drawer 就只能
  在第一个 stack screen 中打开了。
+ 给 drawer navigator 每个 screen 都套上一个 stack navigator，这样 drawer 就会出现在 stack header 上面。
+ 给 tab navigator 每个 screen 都套上一个 stack navigator，这样 tab bar 就会一直显示，并且再次按下 tab ，
  stack 也会跑到顶部（这一句不太理解）。

# 跳转到 sub navigator

看下面的例子：

    function Root() {
        return (
            <Drawer.Navigator>
                <Drawer.Screen name="Home" component={Home} />
                <Drawer.Screen name="Profile" component={Profile} />
                <Drawer.Screen name="Settings" component={Settings} />
            </Drawer.Navigator>
        );
    }

    function App() {
        return (
            <NavigationContainer>
                <Stack.Navigator>
                    <Stack.Screen
                    name="Root"
                    component={Root}
                    options={{ headerShown: false }}
                    />
                    <Stack.Screen name="Feed" component={Feed} />
                </Stack.Navigator>
            </NavigationContainer>
        );
    }

如果想导航到 Root:

    navigation.navigate('Root');

这样会跳到 Root 的 initial screen： ```Home```，如果想直接跳到 ```Profile```：

    navigation.navigate('Root', { screen: 'Profile' });

接下来这段直接翻译，不太懂：

这看起来和之前嵌套 navigation 的导航方式有很大不同。之前的版本，所有的 navigator configuration 都是静态的，所以，React Navigation 可以静态的递归遍历所有的 navigator 以及 它们的 screen 列表。但是对于动态 configuration，React Navigation
直到 navigator screen 被渲染才能直到哪些 screen 是 available 的，以及这些 screen 在哪儿。通常， 直到跳转到一个 screen
的时候，这个 screen 才会渲染。这导致我们跳转的时候需要明确指出要跳到哪个 screen。这就要求我们尽量少嵌套一些层数，让代码简单清楚一点。

### 还可以传 params 进去

    navigation.navigate('Root', {
        screen: 'Profile',
        params: { user: 'jane' },
    });

要跳转到一个深度嵌套 screen ，可以用下面的方法，注意 ```navigatte``` 的第二个参数只是一个 ```params```:

    navigation.navigate('Root', {
        screen: 'Settings',
        params: {
            screen: 'Sound',
            params: {
                screen: 'Media',
            },
        },
    });

上面的例子是要跳转到 Media，Media 嵌套在 Sound 里面，Sound 又嵌套在 Settings 里面， Settings 又嵌套在 Root里面。  
我试了一下，其实不传层级关系，直接就可以跳 Media，可能是名字不重复的话应该可以直接跳。

### Rendering initial route defined in the navigator

默认情况下，当你跳转到 sub navigator 里面一个指定的 screen 时，那么定义 navigator 时，那个指定的 initial screen 
就不起作用了，你指定的那个 screen 当做了 initial screen。

如果你不想要这个行为，就是要渲染定义 navigator 时指定的那个 initial screen，你可以通过 ```initail: false``` 来关掉
它：

    navigation.navigate('Root', {
        screen: 'Settings',
        initial: false,
    });

如果设为 false，这次跳转， 按返回按钮时就会返回 initial screen，并不会直接返回到执行跳转动作的那个 screen。

# 多层嵌套

当嵌套多个 stack、drawer 或者 bottom tab navigator 时，sub navigator 和 parent navigator 的 header 都会显示出来。但是，通常来说，我们更想要显示 sub navigator screen 的
header，隐藏 parent navigator 的 header，当然这个 option 也可以放在 navigator 中。

要隐藏哪个 screen 的 header，就把它的 ```headerShow``` option 设置为 false 就可以了：

    function Home() {
        return (
            <Tab.Navigator>
                <Tab.Screen name="Profile" component={Profile} />
                <Tab.Screen name="Settings" component={Settings} />
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
                    <Stack.Screen name="EditPost" component={EditPost} />
                </Stack.Navigator>
            </NavigationContainer>
        );
    }

# 最佳实践

把 嵌套 navigator 作为实现 UI 的手段，不要用它来组织代码。如果你只是想把 screen 分组管理，
不要通过创建独立的 navigator 来分，用 [Group](https://reactnavigation.org/docs/group) component 来分：

    <Stack.Navigator>
        {isLoggedIn ? (
            // Screens for logged in users
            <Stack.Group>
            <Stack.Screen name="Home" component={Home} />
            <Stack.Screen name="Profile" component={Profile} />
            </Stack.Group>
        ) : (
            // Auth screens
            <Stack.Group screenOptions={{ headerShown: false }}>
            <Stack.Screen name="SignIn" component={SignIn} />
            <Stack.Screen name="SignUp" component={SignUp} />
            </Stack.Group>
        )}
        {/* Common modal screens */}
        <Stack.Group screenOptions={{ presentation: 'modal' }}>
            <Stack.Screen name="Help" component={Help} />
            <Stack.Screen name="Invite" component={Invite} />
        </Stack.Group>
    </Stack.Navigator>