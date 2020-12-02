# Chakra UI

Chakra UI is a simple, modular and accessible component library that gives you the building blocks you need to build your React applications.

## Getting started

### Installation:

```
npm i @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

### Setup provider:

For Chakra UI to work correctly, you need to setup the `ChakraProvider` at the root of your application.

Example:

```jsx
import { ChakraProvider } from "@chakra-ui/react";

function App({ Component, pageProps }: AppProps): ReactElement {
  return (
    <ChakraProvider theme={theme}>
      <Component {...pageProps} />
    </ChakraProvider>
  );
}

export default App;
```

Note:

- For Next.js, you need to set this up in `pages/_app.tsx`
- For Create React App, you need to set this up in `index.tsx`

### Style props

Style props are a way to alter the style of a component by simply passing props to it. It helps to save time by providing helpful shorthand ways to style components.

Alternative to that is using sx prop that takes an object with style rules.

```tsx
// shorthand width prop
<Box w="400px">Lorem Ipsum is simply dummy text</Box>

// sx prop
<Box sx={{ width: '400px'}}>Lorem Ipsum is simply dummy text</Box>
```

For more about style props read [docs](https://chakra-ui.com/docs/features/style-props)

#### The `as` prop

The `as` prop is a feature that all Chakra UI components have and it allows you to pass an HTML tag or component to be rendered.

For example, say you are using a Button component, and you need to make it a link instead. You can compose a and Button like this:

```tsx
<Button as="a" variant="outline" href="https://chakra-ui.com">
  Lorem Ipsum
</Button>
```

This allows you to use all of the Button props and all of the a props without having to wrap the Button in an a component.

### Setup custom theme

If you need to customize the default theme to match your design requirements, you can use `extendTheme` from `@chakra-ui/react`.

Example:

```jsx
import { extendTheme } from "@chakra-ui/react";

const overrides = {
  colors: {
    primary: {
      100: "#ff0000",
      80: "#ff1a1a",
    },
  },
};

const theme = extendTheme(overrides);

export default theme;
```

For more info about setup read [docs](https://chakra-ui.com/docs/getting-started)

## Theming rules

The theme object is where you define your application's color palette, type scale, font stacks, breakpoints, border radius values, and more. Theme overrides are placed in `src/styles/themes` folder.

Example:

```ts
const overrides = {
  colors: {
    primary: {
      100: "#ff0000",
      80: "#ff1a1a",
    },
  },
};
```

Except objects, theme override can take a function:

```ts
const overrides = {
  colors: (props) => {
    const { colorMode } = props;

    return {
      primary: {
        100: colorMode === "dark" ? "#ff0000" : "#ffffff",
        80: "#ff1a1a",
      },
    };
  },
};
```

Then in `props` we have access button props, `theme` object and `colorMode`.

Note:

- `mode(lightMode, darkMode)(props)` function is the same as `colorMode === "dark" ? darkMode : lightMode`.
- `mode` function is part of `chakra-ui/theme-tools` package.

Theme folder structure:

```bash
src
├── styles
    └── theme
        ├── index.ts # main theme endpoint
        ├── styles.ts # global styles
        ├── foundations # colors, typography, sizes...
            ├── fontSizes.ts
            └── colors.ts
        ├── components # components styles
            └── button.ts
```

<!-- TODO maybe detail description of each folder or list of files per folder? -->

### Colors

Color naming should a single value or an object with keys in range from 50 to 900.

Example:

```ts
export const colors = {
  white: "#ffffff",
  primary: {
    50: "#f6faff",
    100: "#d7e9fd",
    200: "#c0dcfc",
    300: "#a7cefb",
    400: "#8abdfa",
    500: "#68aaf8",
    600: "#3d92f6",
    700: "#0070f3",
    800: "#0064d8",
    900: "#0055b9",
  },
};
```

### Global styles

Global styles are theme-aware styles you can apply to any html element globally.

They are defined in `src/styles/theme/styles.ts` file.

<!-- TODO maybe what should be defined in global -->

Example:

```ts
const overrides = {
  styles: {
    global: {
      body: {
        fontFamily: "body",
        bg: "white",
        color: "primary.200",
      },
      "*": {
        boxSizing: "border-box",
      },
    },
  },
};
```

### Responsive

Chakra UI supports responsive styles out of the box. Instead of manually adding @media queries and adding nested styles throughout your code, Chakra UI allows you to provide object and array values to add mobile-first responsive styles.

> Under the hood `@media(min-width)` media query is used to ensure mobile-first

Chakra UI default breakpoints:

```ts
export const breakpoints = {
  sm: "30em",
  md: "48em",
  lg: "62em",
  xl: "80em",
};
```

Here's how to interpret this syntax:

- `base`: From `0em` upwards
- `md`: From `48em` upwards
- `lg`: From `62em` upwards
- `xl`: From `80em` upwards

For responsive styles, array or object syntax can be used.

```tsx
// array syntax
<Text fontSize={["24px", "40px", "56px"]}>
  Lorem Ipsum is simply dummy text
</Text>
```

```tsx
// object syntax
<Text fontSize={{ base: "24px", md: "40px", lg: "56px" }}>
  Lorem Ipsum is simply dummy text
</Text>
```

In case we need to skip cretan breakpoint, `null` is passed at that position in the array to avoid generating unnecessary CSS.

Example:

```jsx
<Text fontSize={["24px", null, "56px"]}>Lorem Ipsum is simply dummy text</Text>
```

Array and Object syntax work for every style prop in the them specification, which means you can change most properties at a given breakpoint.

To create custom breakpoints read [docs](https://chakra-ui.com/docs/features/responsive-styles#customizing-breakpoints).

### Custom components style

Chakra UI has a specific API for styling components. Most components have default or base styles `baseStyle`, styles for different sizes `sizes`, and styles for different visual variants `variants`.

Components styles are defined in `theme/components/<component-name>.ts`
Each component style will export these objects:

- `baseStyles`
- `sizes`
- `variants`
- `defaultProps`

Base style are styles that all button types share.

```tsx
const baseStyle = {
  lineHeight: "1.2",
  borderRadius: "md",
  _focus: {
    boxShadow: "outline",
  },
  _disabled: {
    opacity: 0.4,
    cursor: "not-allowed",
    boxShadow: "none",
  },
  _hover: {
    cursor: "pointer",
  },
};
```

<!-- TODO change this example to use color schema -->

Variant represents visual style.

```tsx
const variants = {
  solid: {
    bg: 'primary.100',,
    color: 'white',
    _hover: {
      bg: 'primary.400',
    },
    _disabled: {
      bg: 'neutral.200',
    }
  },
	outline: {
    bg: 'transparent',,
    border: '1px solid',
    borderColor: 'primary.100',
    _hover: {
      bg: 'primary.100',
    }
  },
};
```

Sizes determent all the component sizes like `width`, `height`, `font-size` etc

```tsx
const sizes = {
  sm: {
    h: 8,
    minW: 8,
    fontSize: "sm",
    px: 3,
  },
  xs: {
    h: 6,
    minW: 6,
    fontSize: "xs",
    px: 2,
  },
};
```

Naming used for sizes:

- `lg`
- `md`
- `sm`
- `xs`

Default values are default values for `size` and `variant`

```tsx
const defaultProps = {
  variant: "solid",
  size: "md",
};
```

For more info about custom component styles read [docs](https://chakra-ui.com/docs/theming/customize-theme#customizing-component-styles)

### Color mode

When you use the ChakraProvider at the root of your app, you can automatically use color mode in your apps. By default, most of Chakra UI component are dark mode compatible. To handle color mode manually in your application, use the `useColorMode` or `useColorModeValue` hooks.

> Tip: Chakra stores the color mode in localStorage and uses CSS variables to ensure the color mode is persistent.

#### useColorMode

`useColorMode` is a React hook that gives you access to the current color mode, and a function to toggle the color mode.

```tsx
function Example() {
  const { colorMode, toggleColorMode } = useColorMode();
  return (
    <header>
      <Button onClick={toggleColorMode}>
        Toggle {colorMode === "light" ? "Dark" : "Light"}
      </Button>
    </header>
  );
}
```

#### useColorModeValue

`useColorModeValue` is React hook that takes 2 arguments, first is value for light mode and second is value for dark mode, and returnees the the value based on the active color mode

```tsx
const value = useColorModeValue(lightModeValue, darkModeValue);
```

### Style 3rd party components

In this example we will style [react-select](https://react-select.com) with Chakra UI

First we need to create custom style object in theme

```tsx
// style/theme/components/reactSelect.ts
const ReactSelect = {
  baseStyle: () => ({
    container: {
      bg: "black",
      p: 8,
    },
    menu: {
      color: "blue.100",
      bg: "blue.500",
      padding: 8,
    },
  }),
};
```

Keys in `baseStyle` can be one of:

- clearIndicator
- container
- control
- dropdownIndicator
- group
- groupHeading
- indicatorsContainer
- indicatorSeparator
- input
- loadingIndicator
- loadingMessage
- menu
- menuList
- menuPortal
- multiValue
- multiValueLabel
- multiValueRemove
- noOptionsMessage
- option
- placeholder
- singleValue
- valueContainer

Reade more about react-select style objects in [docs](https://react-select.com/styles#style-object).

After we have `ReactSelect` theme object, we need to add that to `theme.components`

```tsx
// style/theme/index.ts
const overrides = {
  colors,
  // ...
  components: {
    Button,
    ReactSelect,
    // ...
  },
};
```

Now we can create Select component in components folder:

```tsx
const selectStyle = {
  container: (base, { theme }) => {
    return {
      ...base,
      ...theme.container,
    };
  },
  menu: (base, { theme }) => ({
    ...base,
    ...theme.menu,
  }),
};

const Select = (props) => {
  const styles = useStyleConfig("ReactSelect", props);

  return (
    <ReactSelect
      theme={css(styles)(useTheme())}
      styles={selectStyle}
      {...props}
    />
  );
};
```

Each key added to the theme needs to be added to `selectStyle`.

Additionally custom components can be added to react-select like `SelectContainer`

Example:

```tsx
const SelectContainer: FC<ContainerProps<BoxProps>> = ({
  children,
  ...rest
}) => {
  return (
    <components.SelectContainer {...rest}>
      <chakra.div sx={rest.selectProps.sx}>{children}</chakra.div>
    </components.SelectContainer>
  );
};
```

This will allow us to use `sx` prop on select component.

```tsx
<Select sx={{ mt: 12 }} options={[]} />
```
