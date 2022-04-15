## The Problem ⚡

When we think about design (slicing of the web page) we usually think in a page based context.  
In design file we get set of pages which consists of UI elements, and if we start extracting our components based on the single page view we often end up with a lot of un-reusable one-off components. Those component will bloat out codebase more and more as project grows. Bundle size will become bigger because we will generate a lot of JS code per each new page, which will eventually slow down our page.

To solve this component we need to shift our mindset from vertical (page based) "slicing" flow to horizontal flow.

<figure>
  <img src="/handbook/img/frontend/react-recipes/vertical_vs_horizontal.svg">
  <figcaption style="text-align: center">
    <a href="/handbook/img/frontend/react-recipes/vertical_vs_horizontal.svg" >Vertical vs Horizontal Flow</a>
  </figcaption>
</figure>

To do this we need to focus more on Design System and reusability of our components.
If your design doesn't have any Design System you should ask designer to make on. Here is the example of [Chakra UI Design System](https://www.figma.com/community/file/971408767069651759) made in Figma.

### Card design

Here is the simple example of two different card designs, `user` card in the first row and the `job` card.

![Cards](/img/react-recipes/cards.png)

### Multiple similar cards

First step is to bootstrap the layout with inline props.

```tsx
// ./src/components/shared/user/UserCard/UserCard.tsx

import * as React from "react";
import { Box, Heading, Image, Text } from "@chakra-ui/react";
import { FC } from "react";
import { IUser } from "../../../../interfaces/IUser";

export interface IUserCardProps {
  user: IUser;
}

export const UserCard: FC<IUserCardProps> = ({ user }) => (
  <Box borderRadius="md" border="1px" borderColor="gray.200">
    <Image
      src={user.picture}
      borderRadius="full"
      w="150px"
      h="150px"
      fit="cover"
      mb="4"
      mt="4"
      mx="auto"
      bg="gray.200"
    />
    <Heading as="h1" size="sm" px="5" mb="4" textAlign="center">
      {`${user.name} ${user.surname}`}
    </Heading>
    <Text fontSize="xs" px="5" mb="5" noOfLines={3} textAlign="center">
      {user.bio}
    </Text>
  </Box>
);
```

```tsx
// ./src/components/shared/user/JobCard/JobCard.tsx

import * as React from "react";
import { Box, Heading, Image, Text } from "@chakra-ui/react";
import { FC } from "react";
import { IJob } from "../../../../interfaces/IJob";

export interface IJobCardProps {
  job: IJob;
}

export const JobCard: FC<IJobCardProps> = ({ job }) => (
  <Box borderRadius="md" border="1px" borderColor="gray.200">
    <Image
      src={job.thumbnail}
      borderTopRadius="md"
      w="100%"
      h="200px"
      fit="cover"
      mb="4"
    />
    <Heading as="h1" size="sm" px="5" mb="4">
      {job.title}
    </Heading>
    <Text fontSize="xs" px="5" pb="5" noOfLines={3}>
      {job.description}
    </Text>
  </Box>
);
```

### When this is OK?

When we are building a small one-off components which doesn't contain a lot of code and  are not shared across components.  

### Why this approach can become a problem?

This example is not so bad as it seems at firs, but as this component grows and things are added inline style will become a problem.
Problem with inline styles is that they can easily clutter the code and decrease readability.  
When that happens we need to find a way to fix this problem.

## The Solution ✅

### Solution #1

Extending utility components into styled components and extracting them to `ComponentName.elements.tsx` file.

```tsx
// ./src/components/shared/user/UserCard/UserCard.elements.tsx

import { AspectRatio, chakra, Heading, Text } from "@chakra-ui/react";

export const Card = chakra("div", {
  baseStyle: {
    borderRadius: "md",
    border: "1px",
    borderColor: "gray.200",
    boxShadow: "xl"
  }
});

export const CardImageAspectRatio = chakra(AspectRatio, {
  baseStyle: {
    my: 2,
    mb: 4,
    mx: { base: 4, md: 10 }
  }
});

export const CardImage = chakra("img", {
  baseStyle: {
    borderRadius: "full",
    w: "100%",
    h: "100%",
    objectFit: "cover",
    bg: "gray.200"
  }
});

export const CardHeading = chakra(Heading, {
  baseStyle: {
    px: 5,
    mb: 4,
    textAlign: "center"
  }
});

export const CardDescription = chakra(Text, {
  baseStyle: {
    fontSize: "xs",
    px: 5,
    mb: 5,
    textAlign: "center"
  }
});
```

```tsx
// ./src/components/shared/user/UserCard/UserCard.tsx

import * as React from "react";
import { FC } from "react";

import { IUser } from "../../../../interfaces/IUser";

import {
  Card,
  CardText,
  CardHeading,
  CardImage,
  CardImageAspectRatio
} from "./UserCard.elements";

export interface IUserCardProps {
  user: IUser;
}

export const UserCard: FC<IUserCardProps> = ({ user }) => (
  <Card>
    <CardImageAspectRatio ratio={1 / 1}>
      <CardImage src={user.picture} />
    </CardImageAspectRatio>
    <CardHeading size="sm">{`${user.name} ${user.surname}`}</CardHeading>
    <CardText noOfLines={3}>{user.bio}</CardText>
  </Card>
);
```

> The same thing should be done with the JobCard component.

<iframe 
  src="https://codesandbox.io/embed/recipe-how-to-build-core-components-v1-l3ytyy?fontsize=14&hidenavigation=1&theme=dark"
  class="codesandbox"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components-v1"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>


### When to use solution #1:

1. when your component becomes cluttered and hard to read
2. you don't have a clear design indications this component is designed in multipart manner

### Solution #2

Extracting styled props into `SystemStyleObject` objects and extracting them to `ComponentName.styles.tsx` file.

```tsx
// ./src/components/shared/user/UserCard/UserCard.tsx

import { SystemProps } from "@chakra-ui/react";

export const cardStyles: SystemProps = {
  borderRadius: "md",
  border: "1px",
  borderColor: "gray.200",
  boxShadow: "xl"
};

export const cardImageContainerStyles: SystemProps = {
  my: 2,
  mb: 4,
  mx: { base: 4, md: 10 }
};

export const cardImageStyles: SystemProps = {
  borderRadius: "full",
  w: "100%",
  h: "100%",
  bg: "gray.200",
  objectFit: "cover"
};

export const cardHeadingStyles: SystemProps = {
  px: "5",
  mb: "4",
  textAlign: "center"
};

export const cardTextStyles: SystemProps = {
  fontSize: "xs",
  px: "5",
  mb: "5",
  textAlign: "center"
};
```

```tsx
// ./components/shared/user/UserCard/UserCard.tsx

import * as React from "react";
import { AspectRatio, Box, Heading, Image, Text } from "@chakra-ui/react";
import { FC } from "react";
import { IUser } from "../../../../interfaces/IUser";
import {
  cardImageContainerStyles,
  cardImageStyles,
  cardStyles,
  cardHeadingStyles,
  cardTextStyles
} from "./UserCard.styles";

export interface IUserCardProps {
  user: IUser;
}

export const UserCard: FC<IUserCardProps> = ({ user }) => (
  <Box {...cardStyles}>
    <AspectRatio ratio={1 / 1} {...cardImageContainerStyles}>
      <Image src={user.picture} {...cardImageStyles} />
    </AspectRatio>
    <Heading as="h1" size="sm" {...cardHeadingStyles}>
      {`${user.name} ${user.surname}`}
    </Heading>
    <Text noOfLines={3} {...cardTextStyles}>
      {user.bio}
    </Text>
  </Box>
);

```

<iframe 
  src="https://codesandbox.io/embed/recipe-how-to-build-core-components-v2-pyokb?fontsize=14&hidenavigation=1&theme=dark"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components-v2"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

### When to use solution #2:

There is no significant difference between solution #1 except this way you can compose multiple styles together.

For example:

```tsx
<chakra.div sx={{ ...cardImageStyles, ...cardHeadingStyles }} />
```

### Solution #3

Creating a `core` card component by using a theme and make it reusable [compound component](https://kentcdodds.com/blog/compound-components-with-react-hooks).

> Compound Component is also called Multipart or Composite Component

#### Create a theme

To make highly reusable compound component we need to think about component anatomy.
Component anatomy means that we have to break component into a multiple parts.

In the case of our Card component we can break it in these four parts:
1. card
2. image
3. title
4. body

#### Defining the anatomy

```tsx
import { anatomy, Anatomy } from "@chakra-ui/theme-tools";

const parts = anatomy("card").parts("card", "image", "title", "body");

type CardAnatomy = Anatomy<typeof parts.__type>;
```

#### baseStyles

The base styles for each part

```tsx
const baseStyle: PartsStyleObject<CardAnatomy> = {
  card: {
    borderRadius: "md",
    border: "1px",
    borderColor: "gray.200",
    boxShadow: "xl"
  },
  body: {
    fontSize: "xs",
    px: "5",
    mb: "5"
  }
};
```

#### variants

The variant styles for each part.

```tsx
const variantSolid: PartsStyleFunction<CardAnatomy> = (props) => {
  return {
    image: {
      borderTopRadius: "md",
      w: "100%",
      h: "200px",
      objectFit: "cover",
      mb: "4"
    },
    title: {
      px: "5",
      mb: "4"
    }
  };
};

const variantRounded: PartsStyleFunction<CardAnatomy> = (props) => {
  return {
    image: {
      borderRadius: "full",
      w: "100%",
      h: "100%",
      objectFit: "cover",
      bg: "gray.200"
    },
    title: {
      px: "5",
      mb: "4",
      textAlign: "center"
    },
    body: {
      textAlign: "center"
    }
  };
};

const variants: MultiStyleConfig<CardAnatomy>["variants"] = {
  solid: variantSolid,
  rounded: variantRounded
};
```

#### Complete implementation of the card theme

```tsx
// ./src/styles/components/card.ts

import {
  MultiStyleConfig,
  PartsStyleObject,
  PartsStyleFunction,
  anatomy,
  Anatomy
} from "@chakra-ui/theme-tools";

const parts = anatomy("card").parts("card", "image", "title", "body");
type CardAnatomy = Anatomy<typeof parts.__type>;

const baseStyle: PartsStyleObject<CardAnatomy> = {
  card: {
    borderRadius: "md",
    border: "1px",
    borderColor: "gray.200",
    boxShadow: "xl"
  },
  body: {
    fontSize: "xs",
    px: "5",
    mb: "5"
  }
};

const variantSolid: PartsStyleFunction<CardAnatomy> = (props) => {
  return {
    image: {
      borderTopRadius: "md",
      w: "100%",
      h: "200px",
      objectFit: "cover",
      mb: "4"
    },
    title: {
      px: "5",
      mb: "4"
    }
  };
};

const variantRounded: PartsStyleFunction<CardAnatomy> = (props) => {
  return {
    image: {
      borderRadius: "full",
      w: "100%",
      h: "100%",
      objectFit: "cover",
      bg: "gray.200"
    },
    title: {
      px: "5",
      mb: "4",
      textAlign: "center"
    },
    body: {
      textAlign: "center"
    }
  };
};

const variants: MultiStyleConfig<CardAnatomy>["variants"] = {
  solid: variantSolid,
  rounded: variantRounded
};

const defaultProps: MultiStyleConfig<CardAnatomy>["defaultProps"] = {
  variant: "solid"
};

export default {
  parts: parts.keys,
  baseStyle,
  defaultProps,
  variants
} as MultiStyleConfig<CardAnatomy>;
```

After a card theme is done we'll update the theme to include the new Card component style.

```tsx
import { extendTheme } from "@chakra-ui/react";

import Card from "./components/card";

export const theme = extendTheme({
  components: {
    Card
  }
});
```

#### Card component

Now we can implement the actual Card component.

#### Consuming style config

Since the new Card component is not part of Chakra UI we need to create a new React component and consume the style we just created. We can do that using `useMultiStyleConfig` hook.

#### useMultiStyleConfig API

```tsx
const styles = useMultiStyleConfig("Card", props);
```

> [API Docs](https://chakra-ui.com/docs/styled-system/theming/component-style#usemultistyleconfig-api)

#### Using StylesProvider

In the rood component we can access the whole style config and provide it to all the parts via `StylesProvider` context provider. This way we can for example set the variant only on the root element and all the parts will inherit that variant styles.

```tsx
import {
  chakra,
  forwardRef,
  StylesProvider,
  useStyles,
  ThemingProps,
  useMultiStyleConfig,
} from "@chakra-ui/react";

export const Card = forwardRef<CardProps, "div">((props, ref) => {
  const { children } = props;

  const styles = useMultiStyleConfig("Card", props);

  return (
    <StylesProvider value={styles}>
      <chakra.div ref={ref} __css={styles.card} {...props}>
        {children}
      </chakra.div>
    </StylesProvider>
  );
});

export const CardImage = forwardRef<ImageProps, "img">((props, ref) => {
  const { image } = useStyles();

  return <Image ref={ref} {...image} {...props} />;
});

```

#### Complete example

```tsx
// ./src/components/core/Card/Card.tsx

import {
  chakra,
  forwardRef,
  Heading,
  HeadingProps,
  HTMLChakraProps,
  Image,
  ImageProps,
  StylesProvider,
  Text,
  TextProps,
  ThemingProps,
  useMultiStyleConfig,
  useStyles
} from "@chakra-ui/react";
import { __DEV__ } from "@chakra-ui/utils";

export interface CardOptions {}

export interface CardProps
  extends HTMLChakraProps<"div">,
    CardOptions,
    ThemingProps {}

export const Card = forwardRef<CardProps, "div">((props, ref) => {
  const { children } = props;

  const styles = useMultiStyleConfig("Card", props);

  return (
    <StylesProvider value={styles}>
      <chakra.div ref={ref} __css={styles.card} {...props}>
        {children}
      </chakra.div>
    </StylesProvider>
  );
});

if (__DEV__) {
  Card.displayName = "Card";
}

export const CardImage = forwardRef<ImageProps, "img">((props, ref) => {
  const { image } = useStyles();

  return <Image ref={ref} __css={image} {...props} />;
});

if (__DEV__) {
  CardImage.displayName = "CardImage";
}

export const CardTitle = forwardRef<HeadingProps, "h2">((props, ref) => {
  const { title } = useStyles();

  return <Heading ref={ref} {...title} {...props} />;
});

if (__DEV__) {
  CardTitle.displayName = "CardTitle";
}

export const CardBody = forwardRef<TextProps, "h2">((props, ref) => {
  const { body } = useStyles();

  return <Text ref={ref} {...body} {...props} />;
});

if (__DEV__) {
  CardBody.displayName = "CardBody";
}
```

<iframe 
  src="https://codesandbox.io/embed/recipe-how-to-build-core-components-v3-ko2g9?fontsize=14&hidenavigation=1&theme=dark"
  class="codesandbox"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

### When to use solution #3:

1. When you have clear design indications of multiple similar components with similar features but slightly different styles
2. Designer prepared component variants and anatomy
