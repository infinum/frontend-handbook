Chakra UI is a simple, modular and accessible component library that gives you the building blocks you need to build your React applications.

## Getting started

### Installation:

```
npm i @chakra-ui/react @emotion/react @emotion/styled framer-motion
```

For more about setup read [docs](https://chakra-ui.com/docs/getting-started)

### Setup provider:

For Chakra UI to work correctly, you need to set up the `ChakraProvider` at the root of your application.
Under the hood Chakra UI is using [emotion ThemeProvider](https://github.com/chakra-ui/chakra-ui/blob/develop/packages/system/src/providers.tsx)

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

For including `<CSSReset />` just add `resetCSS` prop to `<ChakraProvider>`

[All provider props](https://chakra-ui.com/docs/getting-started#chakraprovider-props)

### Style props

Style props are a way to alter the style of a component by simply passing props to it. It helps to save time by providing helpful shorthand ways to style components.
Alternative to that is using chakra factory.

Style props should be used for one-off styles (up to max 3 props):

```tsx
<Button display="block" mt="md">
  Click me!
</Button>
```

Components that have more than 3 styles should be moved to separate component.
Every `Box` or `Flex` components should be moved to separate component, regarding of number of styles, with a descriptive name.

```tsx
const ListWrapper = chakra(Box, {
  baseStyle: {
    shadow: "lg",
    rounded: "lg",
    bg: "white",
  },
});
```

For more about style props read [docs](https://chakra-ui.com/docs/features/style-props)

### The `sx` prop

`sx` prop works similar to `style` prop. It takes an object with styles.

Example:

```tsx
<Button
  sx={{
    display: "block",
    mt: "md",
    color: "primary.100",
    mx: 12,
  }}
>
  Click me!
</Button>
```

### The `__css` prop

`__css` prop is similar to `sx` but it is designed __ONLY__ for internal use (hence the private prefix `__`).
The main difference is that `__css` prop will be merged before `sx` prop ([github](https://github.com/chakra-ui/chakra-ui/blob/085891be806b855af90b86367f5b26e8151c3ff5/packages/system/src/system.ts#L106)).  
We should __ONLY__ use it for building primitive `core` components.

```jsx
import { chakra, HTMLChakraProps } from '@chakra-ui/react';
import { __DEV__ } from "@chakra-ui/utils";

export interface CardOptions {};

export interface CardProps
  extends HTMLChakraProps<"div">,
    CardOptions,
    ThemingProps<"Card"> {};

export const Card = forwardRef<CardProps, "div">(({ variant, ...rest }, ref) => {
  const styles = useStyleConfig("Card", { variant });

  return <chakra.div ref={ref} __css={styles} {...rest} />;
});

if (__DEV__) {
  Card.displayName = "Card"
}
```

### Chakra Factory

Chakra factory serves as an object of chakra JSX elements, and also a function that can be used to enable custom component receive chakra's style props.

```tsx
import { chakra } from "@chakra-ui/react";

// preferred way!
chakra("button", {
  baseStyle: {
    shadow: "lg",
    rounded: "lg",
    bg: "white",
  },
});

// using sx prop
<chakra.button
  sx={{
    shadow: "lg",
    rounded: "lg",
    bg: "white",
  }}
>
  Click me
</chakra.button>;
```

This reduces the need to create custom component wrappers and name them. Syntax is available for common html elements. See the reference for the full [list of elements](https://github.com/chakra-ui/chakra-ui/blob/main/packages/system/src/system.utils.ts#L9) supported. Beside list of elements, we can pass a function as [see custom components](#custom-component-example)

Chakra factory will be defined with base style and can be overridden by `sx` prop because `baseStyle` is merged before `sx`. Components created using chakra factory can be overridden by also using chakra factory ([merge order](https://github.com/chakra-ui/chakra-ui/blob/085891be806b855af90b86367f5b26e8151c3ff5/packages/system/src/system.ts#L104))

Example:

```tsx
const Button = chakra("button", {
  baseStyle: {
    bg: "white",
  },
});
```

__Update (14.05.2021.)__

Previously we had this example:
```tsx
const FbButton = chakra(Button, {
  baseStyle: {
    bg: "primary.500",
  },
});
```

We decided this is not the way to go and instead we should make special `colorScheme` in the theme.

Theme:
```ts
// ./src/styles/theme/foundations/colors.ts
const colors = {
  facebook: {
    50: "#E8F4F9",
    100: "#D9DEE9",
    200: "#B7C2DA",
    300: "#6482C0",
    400: "#4267B2",
    500: "#385898",
    600: "#314E89",
    700: "#29487D",
    800: "#223B67",
    900: "#1E355B",
  },
}

// ./src/styles/theme/components/button.ts
function variantSolid(props: Dict) {
  const { colorScheme } = props;

  if (colorScheme === 'facebook') {
    return {
      bg: 'facebook.500',
    }
  }

  return {
    bg: `${colorScheme}.200`,
  }
}

const variants = {
  solid: variantSolid
};

export default {
  variants,
};
```

Usage:
```tsx
const ExampleForm = () => {
  return (
    <form>
      // ...some inputs
      <Button colorScheme="facebook" />
    </form>
  )
}
```

> NOTE: `facebook` color scheme was used just for demonstration purpose, you can do the similar thing with any other social button. Chakra UI already supports `facebook`, `messenger`, `whatsapp`, `twitter`, `telegram` color schemes OTB. You can find the example [here](https://chakra-ui.com/docs/form/button#social-buttons)

> 

#### Custom component example

For example `react-datepicker` can be wrapped in chakra factory function so we can pass `sx` or style props to `<DatePicker />`

Example:

```tsx
import DatePicker from "react-datepicker";
import { chakra } from "@chakra-ui/react";

const StyledaDatepicker = chakra(DatePicker);

export const Datepicker = (props) => (
  <StyledaDatepicker w="100%" bg="blue.100" {...props} />
);
```

Chakra factory function will pass `className` to `DatePicker input` component with all styles. In this case `input` will be rendered with custom `bg` and `width` styles.

For more about chakra factory read [docs](https://chakra-ui.com/docs/features/chakra-factory)

#### The `as` polymorphic prop

The `as` prop is a feature that all Chakra UI components have and it allows you to pass an HTML tag or component to be rendered.

`as` prop (`polymorphic prop`) is a feature of emotion borrowed from styled-components

- [emotion as prop](https://emotion.sh/docs/styled#as-prop)
- [styled-components polymorphic prop](https://styled-components.com/docs/api#as-polymorphic-prop)

It allows us to use all of the `Button` props and all of the `a` props without having to wrap the `Button` in an `a` component.

Example:

```tsx
<Button as="a" href="https://chakra-ui.com/docs/getting-started">
  Chakra UI docs
</Button>
```

In example above `a` element will be rendered with styled of `Button` component.
Except HTML elements `as` prop can be another React component:

```tsx
<Button as={Link} someLinkProp={value}>
  Chakra UI docs
</Button>
```

- `Button` will be rendered as `Link` component
- `Link` props will become available on `Button`
- `Link` and `Button` styles will be combined

For more info about `as` prop read [docs](https://chakra-ui.com/docs/features/style-props#the-as-prop)

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

Theming with Chakra UI is based on the [Styled System Theme Specification](https://system-ui.com/theme/)

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

#### `chakra-ui/theme-tools`

Chakra has a whole pallet of useful helpers:

- `mode(lightMode, darkMode)(props)` function is the same as `colorMode === "dark" ? darkMode : lightMode`.
- `orient` define `vertical` and `horizontal` style for component: `<Divider orientation="horizontal | vertical" />`
- `transparentize` helper to make a color transparent `transparentize('blue.100', 0.3)`
- `darken` darken a specified color `darken('blue.100', 0.5)` (there is also `lighten`, `blacken`, `whiten`)

For more about `theme-tools` check out [github](https://github.com/chakra-ui/chakra-ui/tree/develop/packages/theme-tools/src)

Theme folder structure:

```bash
src
└── styles
    └── theme
        ├── index.ts # main theme endpoint
        ├── styles.ts # global styles
        ├── foundations # colors, typography, sizes...
        │   ├── fontSizes.ts
        │   └── colors.ts
        └── components # components styles
            └── button.ts
```

### Colors

Color naming should a single value or an object with keys in range from `50` to `900`.

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

#### How to generate colors

We recommend adding a palette that ranges from `50` to `900`. Tools like [Themera](https://themera.vercel.app/), [Smart Swatch](https://smart-swatch.netlify.app/), [Coolors](https://coolors.co/232020-553739-955e42-9c914f-748e54) or [Palx](https://palx.jxnblk.com/) are available to generate these palettes.

Sometimes you can get different or incomplete color palette from designer.
For example, designer provided us with only one `primary` color value `#68aaf8`.  
In this case you can use this [POC tool](https://codesandbox.io/s/chakra-color-palette-2w7wl?file=/src/App.tsx:1178-1191) to generate the color palette while retaining the exact color value, but have in mind that this tool is still in beta and have some bugs.

### Pseudo props

Pseudo props in ChakraUI can be passed as props to component. Full list of props can be found in [docs](https://chakra-ui.com/docs/features/style-props#pseudo)

Example:

```tsx
<Button
  bg="blue.100"
  _hover={{
    background: "white",
  }}
>
  Submit
</Button>
```

Also pseudo elements can be used in theme or in chakra factory:

```tsx
const Card = chakra("div", {
  baseStyle: {
    bg: "blue.100",
    _hover: {
      bg: "white",
    },
  },
});

// or
const Card = chakra("div", {
  baseStyle: {
    bg: "blue.100",
    ":hover": {
      bg: "white",
    },
  },
});
```

### Global styles

Global styles are theme-aware styles you can apply to any html element globally.

They are defined in `src/styles/theme/styles.ts` file.

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

In case we need to skip a certain breakpoint, `null` is passed at that position in the array to avoid generating unnecessary CSS.

Example:

```jsx
<Text fontSize={["24px", null, "56px"]}>Lorem Ipsum is simply dummy text</Text>
```

Array and Object syntax work for every style prop in the theme specification, which means you can change most properties at a given breakpoint. Preferred way is to use array.

To create custom breakpoints read [docs](https://chakra-ui.com/docs/features/responsive-styles#customizing-breakpoints).

_NOTE: there is an issue with having `strict: false` in tsconfig when using `createBreakpoints`.
Error that breaks the build: `Type '(() => string) & (() => string)' is not assignable to type 'string'`. Current workaround is to add `strictNullChecks: true` to tsconfig. ([github issue](https://github.com/chakra-ui/chakra-ui/issues/3372))_

For more about responsive styles read [styled-system responsive styles docs](https://styled-system.com/responsive-styles/)

### Custom components style

Chakra UI has a specific API for styling components. Most components have default or base styles `baseStyle`, styles for different sizes `sizes`, and styles for different visual variants `variants`.

Components styles are defined in `theme/components/<component-name>.ts`
Each component style will export these objects:

- `baseStyles`
- `sizes`
- `variants`
- `defaultProps`

`baseStyle` are styles that all button types share.

```tsx
const baseStyle: StyleObjectOrFn = {
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

`variants` represents visual style.

```tsx
const variants: { [variant: string]: StyleObjectOrFn } = {
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
  outline: (props: Dict): SystemStyleObject => ({
    bg: 'transparent',,
    border: '1px solid',
    borderColor: 'primary.100',
    _hover: {
      bg: 'primary.100',
    }
  }),
};
```

`sizes` determine all the component sizes like `width`, `height`, `font-size` etc

```tsx
const sizes: { [size: string]: StyleObjectOrFn } = {
  sm: {
    h: 8,
    minW: 8,
    fontSize: "sm",
    px: 3,
  },
  xs: (props: Dict): SystemStyleObject => ({
    h: 6,
    minW: 6,
    fontSize: "xs",
    px: 2,
  }),
};
```

Naming used for sizes:

- `lg`
- `md`
- `sm`
- `xs`

`defaultProps` are default values for `size` and `variant`

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

Additionally, custom components can be added to react-select like `SelectContainer`

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

### React hook form example

This [example](https://chakra-ui.com/guides/integrations/with-hook-form) shows how to build a simple form with Chakra UI form components and the React Hook Form form library.
