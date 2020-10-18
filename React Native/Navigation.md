Routing in mobile applications presents more then mere navigation between routes (screens). It consists of functionalities and styles which are divided across platforms. Generally, routing systems inside native applications has its own look and feel as well as well defined UX which differs from android and iOS platforms.

In order to achieve this “native feel”, RN community implemented a [React Navigation](https://reactnavigation.org/) library. It’s main focus presents “native-look-and-feel” while keeping the performance and extensibility.

### Official documentation
[Official documentation](https://reactnavigation.org/docs/getting-started) has a lot of information and contains more than a few code examples of all important parts.
Make sure to get familiar with it before continuing to read further.

### 1. Navigation file structure
In order to keep your navigation maintainable and easy to upgrade, here are some guidelines which rely on library main concepts: ***Stack***, ***Tab***, ***Drawer*** and ***Parameters***.

 - navigation/
	 - stacks/
	 - tabs/
	 - drawer/
	 - params/
	  -- Navigator.tsx


#### 1.1 `Navigator.tsx`
 `Navigator.tsx` is a React functional component which defines navigation entry point. It should contain navigation structure wrapped inside `NavigationContainer`:


```typescript
const  Navigator:  React.FC<INavigatorProps>  =  (props)  =>  {
	return  (
		<NavigationContainer>
			// NAVIGATION STRUCTURE
		</NavigationContainer>
	);
};
```

#### 1.2 STACKS
All navigation stacks should be wrapped as functional components inside `navigation/stacks`.
For example, most common application structure consits of authentication (*login*, *register*) and main (*home*, etc..) workflow.
In React Navigation terms this means that you should split your navigation stacks between those 2 flows: `AuthenticationStack` and `HomeStack`. Even though, we could use only one stack with all screens, this is a bad practise in mobile development and makes the app harder to maintain as your application grows.

```typescript

// AuthenticationStack.tsx

const AuthenticationStack = createStackNavigator(AuthenticationStackParamList);

const AuthenticationStack: React.FC<IAuthenticationStackProps> =  (props) => {
	return (
		<Stack.Navigator>
			<Stack.Screen component={Login} name="Login" />
			<Stack.Screen component={Register} name="Register" />
			// ... other screens in "Authentication" flow
		</Stack.Navigator>
	);
};

// HomeStack.tsx

const HomeStack = createStackNavigator(HomeStackParamList);

const HomeStack: React.FC<IHomeStackProps> = (props) => {
	return (
		<Stack.Navigator>
			<Stack.Screen component={Home} name="Home" />
			// ... other screens in "Home" flow
		</Stack.Navigator>
	);
};
```

**IMPORTANT**: `NavigationContainer` component accepts single child component. This is not an issue if your application uses `Tab` or `Drawer` navigation. If you use only `Stacks` you need to create `RootStack` component which will then target all other `Stacks` inside separate `Stack.Screen` components.
Using example above this `Navigator.tsx` would look like:

 ```typescript
 const RootStack = createStackNavigator();

const Navigator: React.FC<INavigatorProps> = (props) => {
	return (
		<NavigationContainer>
			<RootStack.Navigator>
				<RootStack.Screen component={AuthenticationStack} name="AuthenticationStack" />
				<RootStack.Screen component={HomeStack} name="HomeStack" />
			</RootStack.Navigator>
		</NavigationContainer>
	);
};
```

#### 1.3 TABS
Usually, applications have one navigation tab which should be in `navigation/tabs`. When using tab navigation, each `Tab.Screen` component should target a stack with related screens.

 ```typescript
 const Tab = createTabNavigator();

const TabNavigator: React.FC<ITabNavigatorProps> = (props) => {
	return (
		<Tab.Navigator>
			<Tab.Screen component={AuthenticationStack} name="AuthenticationStack" />
			<Tab.Screen component={HomeStack} name="HomeStack" />
		</Tab.Navigator>
	);
};
```

#### 1.4 Drawer
Usually, applications have one navigation tab which should be in `navigation/drawer`. When using tab navigation, each `Tab.Screen` component should target a stack with related screens.

 ```typescript
const Drawer = createDrawerNavigator();

const DrawerNavigator: React.FC<IDrawerNavigatorProps> = (props) => {
	return (
		<Drawer.Navigator>
			<Drawer.Screen component={AuthenticationStack} name="AuthenticationStack" />
			<Drawer.Screen component={HomeStack} name="HomeStack" />
		</Drawer.Navigator>
	);
};
```

#### 1.4 Parameters (Typescript)
To fully use the power of Typescript, `navigation/params` folder should contain types and interfaces for all stacks used inside the application. Since React Navigation handles `Tab` and `Drawer` navigations in the background, there is no need for any type definitions.
Each stack should have its parameters defined in a separate file. The parameters file consists of 3 main parts:

 1. List of screens used inside of stack with related `route` params defined as `type`.
 2. Screen `route` parameters defined as `interface`
 3. Stack screen props defined as `type`

Example:
 1.

 ```typescript
export type HomeStackParamList = {
	Home: IHomeRouteProps;
	// ... other HomeStack screens
};
```
2.
 ```typescript
export interface IHomeRouteProps {
	propName: propValue;
	...
};
```
3.
 ```typescript
export type HomeStackScreenProps<T extends keyof HomeStackParamList = {
	navigation: StackNavigationProp<HomeStackParamList, T>;
	route: RouteProp<HomeStackParamList, T>;
};
```

Each screen inside the application inherits `navigation` and `route` props from React Navigation. Type (3) should be used as a typing for screen props which basically gives code completion for all navigation and routing parameters inside of the screen.

Correct use:

1. Create stack with related types:
 ```typescript
const HomeStack = createStackNavigator<HomeStackParamsList>();
```

2.  Add screen types to each screen components props:

 ```typescript
const Home: React.FC<HomeStackScreenProps<'Home'>> = ({ navigation, route}) => (...);
```

**IMPORTANT**:
Above route types (2) won't work for nested stacks.
Correct route types:
 ```typescript
type NestedRouteParams<T> = {
[K in keyof T]: undefined extends T[K] ? { screen: K; params?: T[K] } : { screen: K; params: T[K] };
}[keyof T];
```

Example:
`HomeStack` contains `Home` screen and `UserStack`, which then contains `Profile` screen. In that case our params list type looks like:

 ```typescript
export type HomeStackParamList = {
	Home: IHomeRouteProps;
	ProfileStack: NestedRouteParams<ProfileStackParamList>;
};
```

This way we can easily navigate to each screen inside `ProfileStack` from anywhere inside `HomeStack`:

 ```typescript
navigation.navigate('ProfileStack', {
	screen: 'Profile',
	params: { /* params */ } // route params
});
```

### 	TIPS

 1. `Header` component height differs between android & iOS.
`android: 56, iOS: 44 + insetTop`. `insetTop` equals to size of the status bar + top notch.
To get the exact size of the notch (top, bottom) you should use `react-native-safe-area-context` library.
Library implements `useSafeAreaInsets` hook which returns `inset` object containing definitions of top and bottom insets:

`const { top, bottom } = useSafeAreInsets()`

Since newer android devices started to add notch as well, you should always define your `Header` height using `height + topInset` value.
