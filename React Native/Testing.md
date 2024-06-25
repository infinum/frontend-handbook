Testing React Native is similar to default React testing, so for a quick intro in React testing - you can check
my video tutorial https://www.youtube.com/watch?v=AhH5xFPwNHg.

### Setup

By default - RN ships with Jest testing library that doesn’t require additional setup and works out of the box. To run tests - use:

`pnpm test`

But default test which can be found in **tests**/ folder is too simple and for more complicated and useful tests
we will require another library - https://github.com/callstack/react-native-testing-library (which replaces Enzyme if you’re familiar with that)

`pnpm install -D -E @testing-library/react-native`

To have more matchers we can also install https://github.com/testing-library/jest-native

`pnpm install -D -E @testing-library/jest-native`

### Testing Styled components

***!! NOT WORKING in ("jest-styled-components": "7.0.3") !! https://github.com/styled-components/jest-styled-components/issues/294***

As styled-components is our RN way of styling - we need to know how to test it. But not now :\\

### Mocking native modules

Native Modules requires a running React Native application to work properly. In order to use it in tests,
you have to provide its separate implementation.

1. Async storage - https://react-native-community.github.io/async-storage/docs/advanced/jest/#with-jest-setup-file

2. React Native Gesture Handler - https://docs.swmansion.com/react-native-gesture-handler/docs/#testing

### General recommendations about testing

1. Test actual result, not state(for example in case of opening/closing modal instead of checking modal state
   you should check if modal component appeared on the page.)
2. In case of testing API request you need to mock the result of the request(you can just copy response
   from a real request and store it as an object.) and pass it to the component you're testing as a prop.
3. For searching elements use `testID` prop on RN component. This is 100% way of finding correct
   element for your test.

### Examples

Test project with examples could be found here https://github.com/Hardronox/RN-testing-examples

### Documentation

1. https://jestjs.io/docs/en/tutorial-react-native
2. https://reactnative.dev/docs/testing-overview
3. https://github.com/styled-components/jest-styled-components
