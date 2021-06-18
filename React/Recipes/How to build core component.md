## The Problem ⚡

When we think about design (slicing of the web page) we usually think in a page based context.  
In design file we get set of pages which consists of UI elements, and if we start extracting our components based on the single page view we often end up with a lot of un-reusable one-off components. Those component will bloat out codebase more and more as project grows. Bundle size will become bigger because we will generate a lot of JS code per each new page, which will eventually slow down our page.

To solve this component we need to shift our mindset from vertical (page based) "slicing" flow to horizontal flow.

![Vertical vs Horizontal Flow](/img/react-recipes/vertical_vs_horizontal.svg)

To do this we need to focus more on Design System and reusability of components.
If your design doesn't have any Design System you should as designer to make on.

### Card design

![Cards](/img/react-recipes/cards.png)

### Multiple similar cards

First step is to bootstrap the layout with inline props.

```tsx
// ./components/shared/user/UserCard/UserCard.tsx

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
// ./components/shared/user/JobCard/JobCard.tsx

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

Problem with inline styles is that they can easily clutter the code and decrease readability. 
When that happens we need to find a way to fix this problem.

## The Solution ✅

### Solution #1

Extending utility components into styled components and extracting them to `ComponentName.elements.tsx` file.

```tsx
// ./components/shared/user/JobCard/UserCardV2.elements.tsx

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
// ./components/shared/user/JobCard/UserCardV2.tsx

import * as React from "react";
import { FC } from "react";

import { IUser } from "../../../../interfaces/IUser";

import {
  Card,
  CardDescription,
  CardHeading,
  CardImage,
  CardImageAspectRatio
} from "./uderCardV2.elements";

export interface IUserCardProps {
  user: IUser;
}

export const UserCardV1: FC<IUserCardProps> = ({ user }) => (
  <Card>
    <CardImageAspectRatio ratio={1 / 1}>
      <CardImage src={user.picture} />
    </CardImageAspectRatio>
    <CardHeading size="sm">{`${user.name} ${user.surname}`}</CardHeading>
    <CardDescription noOfLines={3}>{user.bio}</CardDescription>
  </Card>
);
```

> The same thing should be done with the JobCard component.

<iframe 
  src="https://codesandbox.io/embed/recipa-how-to-build-core-components-v1-b2xv0?fontsize=14&hidenavigation=1&theme=dark"
  class="codesandbox"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components-v1"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

### Solution #2

Extracting styled props into `SystemStyleObject` objects and extracting them to `ComponentName.styles.tsx` file.

```tsx
// ./components/shared/user/JobCard/UserCardV3.tsx

import { SystemStyleObject } from "@chakra-ui/react";

export const cardStyles: SystemStyleObject = {
  borderRadius: "md",
  border: "1px",
  borderColor: "gray.200",
  boxShadow: "xl"
};

export const cardAspectRationStyles: SystemStyleObject = {
  my: 2,
  mb: 4,
  mx: { base: 4, md: 10 }
};

export const cardImage = {
  borderRadius: "full",
  w: "100%",
  h: "100%",
  bg: "gray.200"
};

export const cardHeading = {
  px: "5",
  mb: "4",
  textAlign: "center"
};

export const cardDescription = {
  fontSize: "xs",
  px: "5",
  mb: "5",
  textAlign: "center"
};
```

```tsx
// ./components/shared/user/JobCard/UserCardV3.tsx

import * as React from "react";
import { AspectRatio, Box, Heading, Image, Text } from "@chakra-ui/react";
import { FC } from "react";
import { IUser } from "../../../../interfaces/IUser";
import {
  cardAspectRationStyles,
  cardImage,
  cardStyles,
  cardHeading,
  cardDescription
} from "./UserCardV3.styles";

export interface IUserCardProps {
  user: IUser;
}

export const UserCardV2: FC<IUserCardProps> = ({ user }) => (
  <Box sx={cardStyles}>
    <AspectRatio ratio={1 / 1} sx={cardAspectRationStyles}>
      <Image src={user.picture} fit="cover" sx={cardImage} />
    </AspectRatio>
    <Heading as="h1" size="sm" sx={cardHeading}>
      {`${user.name} ${user.surname}`}
    </Heading>
    <Text noOfLines={3} sx={cardDescription}>
      {user.bio}
    </Text>
  </Box>
);
```

<iframe 
  src="https://codesandbox.io/embed/recipa-how-to-build-core-components-v2-pyokb?fontsize=14&hidenavigation=1&theme=dark"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components-v2"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>

### Solution #3

Creating `core` card component with theme driven and reusable compound components.

```tsx
```

<iframe 
  src="https://codesandbox.io/embed/recipa-how-to-build-core-components-ko2g9?fontsize=14&hidenavigation=1&theme=dark"
  class="codesandbox"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="recipa-how-to-build-core-components"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>
