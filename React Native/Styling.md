React Native implements `Stylesheet API` which uses javascript to create styles. It is an abstraction similar to CSS StyleSheets. Apart from `Stylesheet`, styles in RN can be applied using regular javascript object.

Applying styles in RN is done through `styles` prop. All core components in RN implement `styles` prop which accepts styles as javascript objects or array of objects with style parameters written in `camelCase` format.

```javascript
<View  style={{ flex: 1, backgroundColor: '#0fd0fd', borderRadius:  4 }} />
```

## StyleSheet API
Using `Stylesheet API` for creating styles is highly recommended since it consists of several optimisations. Each style object created with the API is registered with unique ID. This way styles can be reused by referring to their ID. Also, this allows javascript to send a single style configuration over bridge only once and then be reused on native side, instead of sending multiple configurations for same styles. In the end this allows styles to be more efficient, performant and apps / components more scalable and easier to maintain.

To create styles, the API implements a **`create`** method.

```javascript
const styles = StyleSheet.create({
	container:  { borderRadius:  4, borderWidth:  0.5, borderColor:  '#d6d7da'  },
	title:  { fontSize:  19, fontWeight:  'bold'  },
	activeTitle:  { color:  'red'  }
});
```

```javascript
<View style={styles.container}>
	<Text style={styles.title} />
</View>
```

**Note:** Styles should always be defined outside of the `render` function.

```javascript
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';

export default App = () => (
  <View style={container}>
    <Text style={text}>React Native</Text>
  </View>
);

const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#fff',
  },
  text: {
    fontSize: 30,
    color: '#000'
  },
});
```

More on the `Stylesheet API` can be found on:

[https://reactnative.dev/docs/stylesheet](https://reactnative.dev/docs/stylesheet)

Important thing to recognize here is that all measurement values are written as numbers, not pixels like we are used to in web development. That is because all dimensions in React Native are unitless, and represent density-independent pixels. Setting dimensions this way is common for components that should always render at exactly the same size, regardless of screen dimensions.

### Flexbox
RN uses Flexbox algorithm for defining the component layout. Flexbox properties on RN are practicly identical to web with one difference and that is  `flexDirection`. Default value for `flexDirection` property in RN is `column`, where on web it is `row`.

More on Flexbox layout can be found on:
[https://reactnative.dev/docs/flexbox](https://reactnative.dev/docs/flexbox)

## STYLED-COMPONENTS

Styling in RN can also be done with `styled components` which is more familiar approach for React developers.

When starting a new project **`styled-components`** should be your default styling option as it consumes styling rules which are identical to React, and provides theming.

Currently there are 2 main libraries used:
1. [Styled components](https://styled-components.com/docs/basics#react-native)
2. [Emotion](https://emotion.sh/docs/@emotion/native)

There are almost no differences between them, but we decided that `styled-components` should be the option when starting a new project. Main reason for that is theming typings which can easily be defined when using `styled-components` while on the other hand `emotion` library doesn't have that clean solution yet.

**Note:** `emotion`  had announced that theming will get better typings in version 11.

Next paragraphs inside the section are related to configuration & usage of `styled-components` library.

#### INSTALLATION
`npm install --save styled-components` or `yarn add styled-components`

#### USAGE
To use `styled-components` inside RN app make sure to import `styled-components/native` package as it provides all styling rules needed for RN.

```javascript
import React from 'react'
import styled from 'styled-components/native'

const StyledView = styled.View`
  background-color: papayawhip;
`

const StyledText = styled.Text`
  color: palevioletred;
`

const MyReactNativeComponent: React.FunctionComponent = () => {
  render() {
    return (
      <CircleView>
        <StyledText>Hello World!</StyledText>
      </CircleView>
    )
  }
}
```

If you want to style your own (custom) components using the `styled-components` there are 2 main steps:
1. Top element of your component needs implement / use `style` props and accept it's value from component prop.

```javascript
import React from 'react'
import { ViewStyle } from 'react-native';
import styled from 'styled-components/native'

interface ICircleProps {
	style?: ViewStyle | Array<ViewStyle>;
	radius: number;
}

const CircleWrapper = styled.View`
  background-color: papayawhip;
  height: ${({ radius }) => radius * 2};
  width: ${({ radius }) => radius * 2};
  border-radius: ${({ radius }) => radius};
`

export const Circle: React.FunctionComponent = ({ children, style, radius }) => {
  render() {
    return (
      <CircleWrapper style={style} radius={radius}>
        {children}
      </CircleWrapper>
    )
  }
}
```

2. When you use component (`Circle`) inside your code you can override it's style with:

```javascript
import React from 'react'
import { ViewStyle } from 'react-native';
import { Circle } from '../components/Circle'; // path to Circle component
import styled from 'styled-components/native'

const AppCircle = styled(Circle)`
  background-color: tomato;
`

const App: React.FunctionComponent = () => {
  render() {
    return (
      <AppCircle radius={50}>
        {children}
      </AppCircle>
    )
  }
}

```
This means that when you're defining your styles inside `AppCircle`,  you're actually creating a normal `Circle` component, that has your styles attached to it. Important thing to mention here is that `AppCircle` inherits all the props from `Circle` component.

When defining styled component in RN you can provide style properties directly:

***styled-components:***
```javascript
import React from 'react'
import styled from 'styled-components/native'

const Container = styled.View`
  background-color: papayawhip;
`

export const Circle: React.FunctionComponent = () => {
  render() {
    return (
      <Container height={100} width={100} paddingLeft={10} paddingRight={10} />
    )
  }
}
```

This kind of styling should be applied only in case you have some dynamic styles which is calculated during the application pre or post layouting phase. There is no sense in creating an empty styled component and applying the styled properties directly. Also, be aware that each styled component can accept any kind of prop (like a regular React component), after which you can extract the prop and use it, inside the component style declaration. This leads to a question, should you define your own style prop or use this feature. Well, this depends on a case and scenario but you should stick to one of those and try not to mix them, since it may hurt readability of the code and will eventually lead to overcomplicated style definitions.

**Note:** *style properties need to follow camelCase naming rule and have **number** or **percentage** value.* Using These value types are specific rule to `styled-components` library. If you are using `emotion` you can defined values in `px` aswell (have in mind that `px` values will cause error in `styled-components`):

**Note:** *flex property works like CSS shorthand, and not the legacy flex property in React Native. Setting flex: 1 sets flexShrink to 1 in addition to setting flexGrow to 1 and flexBasis to 0.*

***Emotion:***
```javascript
import React from 'react'
import styled from 'styled-components/native'

const Container = styled.View`
  background-color: papayawhip;
`

export const Circle: React.FunctionComponent = () => {
  render() {
    return (
      <Container height="100px" width="100px" paddingLeft="10px" paddingRight="10px" />
    )
  }
}
```

#### THEMING

Theming in RN with `styled-components` is identical to React.
More on theming can be found on:
[link to React guide] // TODO
[https://styled-components.com/docs/advanced#theming](https://styled-components.com/docs/advanced#theming)
