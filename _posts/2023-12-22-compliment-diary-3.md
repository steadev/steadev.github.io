---
title: "[칭찬일기 개발기 #3] Routing"
author: steadev
date: 2023-12-22 00:30:00 +0900
categories: [React Native]
tags: [React Native, Expo]
---

최대한 단순한 UI로 MVP 제품을 만들어 출시하려한다.

피그마로 대충 그려본 UI...

<img src="https://steadev.github.io/assets/images/compliment-diary/2023-12-22-1.png" />

iPhone8(375 \* 667)을 기준으로 해보니 엄청 짧게 느껴지지만 작은 기기로 먼저 개발하고 큰 기기 대응하는게 개인적으로는 좀 더 수월했던 것 같다.

원래 iPhone8에는 safeArea가 없지만 그냥 넣어봤다.. 느낌 보려고

Routing은 하단 메인 탭 / 세팅화면 크게는 이렇게 2개로만 구성이 된다.

메인 탭에서 첫째 탭은 칭찬 input과 출첵용 달력

둘째 탭은 칭찬한 것을 list로 보여주는 화면

설정화면은 `로그인` / `일기 export` 정도로만 생각하고 있다. 일단 배포 후에 하나씩 develop할 생각.

---

Navigation Stack별로 Navigation 파일을 따로 만들어 관리하려한다.

처음에는 구분없이 한 파일에 다 때려박아서 했었는데, 앱이 커질 수록 감당이 안되서 하나씩 분리해나갔었다..

그렇게 해보니 나중에 해당 navigation쪽에만 따로 로직을 넣는다던가 UX를 다르게 구성한다던가 할 때 매우 유용했다.

이하는 간단하게 구성한 코드

```
// App.tsx
export default function App() {
	return (
		<SafeAreaProvider>
			<RootNavigator />
		</SafeAreaProvider>
	);
}
```

##### navigations

RootNavigator(index.tsx)

```
...

const Stack = createNativeStackNavigator();

const RootNavigator = () => {
	return (
		<NavigationContainer fallback={<ActivityIndicator size="large" />}>
			<Stack.Navigator
				initialRouteName={"Main"}
				screenOptions={{
				headerShown: false,
				contentStyle: { backgroundColor: "#fff" },
			}}>
				<Stack.Screen name="Main" component={MainNavigator} />
				<Stack.Screen name="Setting" component={SettingNavigator} />
			</Stack.Navigator>
		 </NavigationContainer>
	);
};

export default RootNavigator;
```

MainNavigator(하단 탭바)
기본으로 하면 보기싫은 label이 보여서.. icon만으로 구성했다.

```
...

const Tab = createBottomTabNavigator();
export type TabName = "Home" | "History";

const MainNavigator = () => {
	return (
		<Tab.Navigator initialRouteName={"Home"}>
			<Tab.Screen
				name="Home"
				component={HomeScreen}
				options={{
					tabBarShowLabel: false,
					tabBarIcon: ({ focused }) => (
						<FontAwesome
							name="home"
							size={24}
							color={focused ? "black" : "lightgrey"}
						/>
					),
				}}
			/>
			<Tab.Screen
				name="History"
				component={HistoryScreen}
				options={{
					tabBarShowLabel: false,
					tabBarIcon: ({ focused }) => (
						<FontAwesome
							name="list-ul"
							size={24}
							color={focused ? "black" : "lightgrey"}
						/>
					),
				}}
			/>
		</Tab.Navigator>
	);
};

export default MainNavigator;
```

SettingNavigator(세팅 Navigator)

```
...

const Stack = createNativeStackNavigator();

const SettingNavigator = () => {
	return (
		<Stack.Navigator initialRouteName="Home">
			<Stack.Screen name="Home" component={SettingScreen} />
		</Stack.Navigator>
	);
};

export default SettingNavigator;
```

이하는 위 세팅의 기본 view

<img src="https://steadev.github.io/assets/images/compliment-diary/2023-12-22-2.png" />
