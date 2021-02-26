## Organizing components

### Utility components

Utility components are basically components that don't have any impact on the UI itself.
For example, Meta component for populating `<Head>`.

Example:

```tsx
import React, { FC } from 'react';
import Head from 'next/head';

export const Meta: FC = () => {
  return (
    <Head>
      <link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png" />
      <link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png" />
      <link rel="shortcut icon" href="/favicon.ico" />
      <title>Infinum</title>
    </Head>
  );
};
```

```
src
.
└── components
    ├── utilities
    │   └── Meta
    │       └── Meta.tsx
    ...
```
### UI Components

When adding UI components, you should be able to distinguish them by:

1. Page scope based components
2. Components that are shared all across the app
3. Large and complex components which contain a lot of stylings, different layouts and child components.
   
Lowercase folders are indicators of a page scoped components, shared components and meta components, while PascalCase folders and filenames should be used for components naming.

```
components
.
├── shared
│   └── WelcomeBox
│       └── WelcomeBox.tsx
├── admin
│   └── AdminBanner
│       └── AdminBanner.tsx
└── Gallery
    ├── Gallery.tsx
    └── components
        ├── Arrows.tsx
        ├── Breadcrumbs.tsx
        └── ...
```

#### `components` folder

When adding `components` folder, you basically extracting smaller chunks of your main component that are not going to be used anywhere else, only in that component.

Note: There _should_ be only one level of `components` folder inside the component folder.

## Elements

If you have a lot of style declarations inside you component file (enough to make the file difficult to read), you should create a separated file `elements.ts` where you will store you chakra factories.

Example:

```
...
.
└── WelcomeCard
    ├── WelcomeCard.tsx
    └── WelcomeCard.elements.ts
```

## Different component layouts

Sometimes, you might want to create a component specific for mobile and desktop. You might use a tool like [fresnel](https://github.com/artsy/fresnel) for detecting media queries.

In this case, inside a specific component, we could add a subfolder `layouts` (not to be confused with actual layout described below) to define how our component would look like on specific media query.

```
.
└── admin
    └── UserCard
        ├── layouts
        │   ├── UserCard.mobile.tsx
        │   └── UserCard.desktop.tsx
        └── UserCard.tsx
```

For this case, inside `index.tsx` we would have something like this:

```tsx
  export const UserCard = () => {
    return (
      <Media at="sm">
        <UserCardMobile />
      </Media>
      <Media at="md">
        <UserCardDesktop />
      </Media>
    )
  }
```

This is the case **only** for layouts that would add too much complexity when using standard css queries.


### Extracting utility functions and hooks

Sometimes, you will have complex functions or effects inside your component that will affect the readability of your component.
In that case, you should extract them into separated files `utils.ts` and `hooks.ts`.

Main goal of these files is to store functions and hooks that are **specific for that component**, so we could keep our root `hooks` and `utils` folders clean and for global usage purposes only.

Example:

```
.
├── ...
└── AlbumsCarousel
    ├── AlbumsCarousel.tsx
    └── AlbumsCarousel.utils.ts
    └── AlbumsCarousel.hooks.ts
```

**Cluttered component**

```tsx
... imports ...

export const AlbumsCarousel = (props) => {
  const [isPlaying, setIsPlaying] = useState(true);
  const [highlighted, setHighlighted] = useState(null);
  const [zoom, setZoom] = useState(1);

  const modals = useModals();
  const carouselRef = useRef();

  const showLoginModal = () => {
    modals.open(Modals.Login);
  };

  const highlightAlbum = (id: number) => {
    setIsPlaying(false);
    setHighlighted(id);
    carouselRef.current.collapseItems();
  };

  const formatReleaseDate = (startDate: string, endDate: string) => {
    const start = new Date(startDate);
    const end = new Date(endDate);
    const timezonedStart = new Date(start.valueOf() + start.getTimezoneOffset() * 60 * 1000);
    const timezonedEnd = new Date(end.valueOf() + end.getTimezoneOffset() * 60 * 1000);

    if (isSameDay(timezonedStart, timezonedEnd)) {
      return `${format(timezonedStart, 'd MMMM yyyy')}`;
    }

    if (!isSameYear(timezonedStart, timezonedEnd)) {
      return `${format(timezonedStart, 'd MMMM yyyy')} ${String.fromCharCode(8212)} ${format(
        timezonedEnd,
        'd MMMM yyyy',
      )}`;
    }

    if (!isSameMonth(timezonedStart, timezonedEnd)) {
      return `${format(timezonedStart, 'd MMMM')} ${String.fromCharCode(8212)} ${format(
        timezonedEnd,
        'd MMMM yyyy',
      )}`;
    }

    return `${format(timezonedStart, 'd')} ${String.fromCharCode(8212)} ${format(
      timezonedEnd,
      'd MMMM yyyy',
    )}`;
  }

  return (
    //...
  )
}
```

**Cleaned**

```tsx
... other imports ...
import { formatReleaseDate } from './utils';

export const AlbumsCarousel = (props) => {
  const [isPlaying, setIsPlaying] = useState(true);
  const [highlighted, setHighlighted] = useState(null);
  const [zoom, setZoom] = useState(1);

  const modals = useModals();
  const carouselRef = useRef();

  const showLoginModal = () => {
    modals.open(Modals.Login);
  };

  const highlightAlbum = (id: number) => {
    setIsPlaying(false);
    setHighlighted(id);
    carouselRef.current.collapseItems();
  };

  return (
    //...
    // formatReleaseDate used somewhere here
  )
}
```

### Layouts

With Layouts, we generally define overall page structure that will be presented to the user based on a specific auth state or a specific page.

For example, your app could have an Admin dashboard on `/admin` route which has a fixed header and sidebars on both right and left, and scrollable content in the middle (you're most likely familiar with this kind of layout).

You will define your different layouts inside `layouts` folder:

```
.
.
└── components
    └── layouts
        ├── AdminLayout
        │   └── AdminLayout.tsx
        ├── MainLayout
        │   └── MainLayout.tsx
        └── BlogLayout
            └── BlogLayout.tsx
```

Then, when creating your routes (pages), you will wrap your page in the layout that represents the current page:

```tsx
// pages/index.tsx

export default function Home() {
  return (
    <MainLayout>
      ...
    </MainLayout>
  )
}

// pages/admin.tsx
export default function Admin() {
  return (
    <AdminLayout>
      ...
    </AdminLayout>
  )
}
```

## Setting up store

Your datx store and models will be placed in the root of the `src` folder as follows:

```
src
├── models
│   ├── User.ts
│   └── Session.ts
└── store
    ├── utlis
    │   ├── config.ts
    │   └── initializeStore.ts
    └── index.ts
```

## Fetchers

In `src/fetchers` you will organize your (swr) fetchers, usually by the model on which you will make API calls.

```ts
// src/fetchers/user.ts

import { AppCollection } from '../store';
import { User } from '../models/User';

export async function fetchUser(store: AppCollection, id: string): Promise<User> {
  try {
    const response = await store.fetch(User, id, {
      include: ['albums']
    });

    return response.data as User;
  } catch(response) {
    // handle response
  }
}

```

You will use your fetcher functions with `useSwr` hook inside of your components.


## Setting up theming and styles

When creating styles for your core components, you will create `components` folder inside of styles folder. Styles folder will contain all core stylings and theme setup.

Check out [styling guide(needs update)](styling-guide-section-link) for other styling related stuff.


```
src
├── styles
    └── theme
        ├── index.ts # main theme endpoint
        ├── styles.ts # global styles
        ├── foundations # colors, typography, sizes...
        │   ├── fontSizes.ts
        │   └── colors.ts
        └── components # components styles
            └── button.ts
```

## Tests

When organizing test files, here are couple of quick rules:

- Components, utils, fetchers... should have a test file in the same folder where they are placed
- When testing pages, create `__tests__/pages` folder because how next treats pages folder.
- All mocks should be placed in `__mocks__` folder

For other in depth guides for testing take a look at the [testing guide(needs update)](link-to-testing-section).

Folder structure would look something like this:

```
src
.
├── __mocks__
│   └── react-i18next.tsx
├── __tests__
│   ├── pages
│   │   └── user.test.ts
│   └── test-utils.tsx
├── pages
│   └── user.ts
├── fetchers
│   └── users
│       ├── users.ts
│       └── users.test.ts
└── components
    └── shared
        └── Button
            ├── Button.test.tsx
            └── Button.tsx
```
