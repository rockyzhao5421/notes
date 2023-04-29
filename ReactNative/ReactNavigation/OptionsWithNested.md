```options``` 一定要写在正确的位置才能成功设置对应的 navigator，如果位置不对，最好的结果是啥效果也没有，坏的结果是可能发生异常。

# 只能通过直属 screen component 修改 navigation option

    const Tab = createTabNavigator();
    const HomeStack = createNativeStackNavigator();
    const SettingsStack = createNativeStackNavigator();

    function HomeStackScreen() {
        return (
            <HomeStack.Navigator>
                <HomeStack.Screen
                    name="A"
                    component={A}
                    options={{ tabBarLabel: 'Home!' }}
                />
            </HomeStack.Navigator>
        );
    }

    function SettingsStackScreen() {
        return (
            <SettingsStack.Navigator>
                <SettingsStack.Screen
                    name="B"
                    component={B}
                    options={{ tabBarLabel: 'Settings!' }}
                />
            </SettingsStack.Navigator>
        );
    }

    export default function App() {
        return (
            <NavigationContainer>
                <Tab.Navigator>
                    <Tab.Screen name="Home" component={HomeStackScreen} />
                    <Tab.Screen name="Settings" component={SettingsStackScreen} />
                </Tab.Navigator>
            </NavigationContainer>
        );
    }

Tab navigator 的两个 screen component 里面分别嵌套了一个 stack navigator。  

想修改 Tab navigator 两个 Tab 的显示名字，却写在里 Stack 的 screen component 里面，这肯定没效果的。
想修改 Tab navigator 就要写在 Tab navigator 的 screen component 里面：

    export default function App() {
        return (
            <NavigationContainer>
                <Tab.Navigator>
                    <Tab.Screen
                        name="Home"
                        component={HomeStackScreen}
                        options={{ tabBarLabel: 'Home!' }}
                    />
                    <Tab.Screen
                        name="Settings"
                        component={SettingsStackScreen}
                        options={{ tabBarLabel: 'Settings!' }}
                    />
                </Tab.Navigator>
            </NavigationContainer>
        );
    }

# 根据 sub navigator 的状态来设置 ```options```

    const Tab = createBottomTabNavigator();

    function HomeTabs() {
        return (
            <Tab.Navigator>
                <Tab.Screen name="Feed" component={FeedScreen} />
                <Tab.Screen name="Profile" component={ProfileScreen} />
                <Tab.Screen name="Account" component={AccountScreen} />
            </Tab.Navigator>
        );
    }

    const Stack = createNativeStackNavigator();

    export default function App() {
        return (
            <NavigationContainer>
                <Stack.Navigator>
                    <Stack.Screen name="Home" component={HomeTabs} />
                    <Stack.Screen name="Settings" component={SettingsScreen} />
                </Stack.Navigator>
            </NavigationContainer>
        );
    }

如果想显示 FeedScreen 的时候，通过 ```headerTitle``` 定制 Title 组件，好像行不通。因为
只有 Stack navigator 才有 ```headerTitle``` option，而这里的 stack 只会看它的直属
screen component ： HomeTabs 和 SettingsScreen，不会看 FeedScreen。

但是我们可以根据 Tab navigator 的状态来取 tilte 字符串：

    import { getFocusedRouteNameFromRoute } from '@react-navigation/native';

    function getHeaderTitle(route) {
        const routeName = getFocusedRouteNameFromRoute(route) ?? 'Feed';
        switch (routeName) {
            case 'Feed':
                return 'News feed';
            case 'Profile':
                return 'My profile';
            case 'Account':
                return 'My account';
        }
    }