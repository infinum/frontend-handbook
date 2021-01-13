# Project file structure

## Organizing components
### UI Components

Before implementing [atomic design methodology](https://github.com/danilowoz/react-atomic-design), our common practice was separating components by the scope of the page + core components + shared components like this:

Lowercase folders were indicators of a page scope (except of core).

```
.
├── src
└── components
    ├── core
    │   └── button
    │       ├── Button.tsx
    │       └── stories.tsx
    ├── admin
    │   └── welcome-banner
    │       └── WelcomeBanner.tsx
    ├── carousel
    │   └── Carousel.tsx
    └── card
        └── Card.tsx
```

With atomic design, we would completely separate all UI components and build them as a blocks of atoms, molecules and organisms (together with templates and pages for putting everything together at the end) 

```
src
.
└── components
    └── ui
        ├── atoms
        │   └── button
        │       ├── Button.tsx
        │       └── stories.tsx
        ├── molecules
        │   ├── card
        │   │   └── Card.tsx
        │   └── welcome-banner
        │       └── WelcomeBanner.tsx
        └── organisms
            └── carousel
                └── Carousel.tsx
```

## Different molecule layouts

Sometimes, you might want to create a molecule specific for mobile and desktop. You might use a tool like [fresnel](https://github.com/artsy/fresnel) for detecting mediaqueries.

In this case, inside a specific molecule, we could add a subfolder `layouts` (not to be confused with actual layout described below) to define how our molecule would look like on specific media query.

```
.
└── molecules
    └── card
        ├── layouts
        │   ├── Desktop.tsx
        │   └── Mobile.tsx
        └── Card.tsx
```

For this case, inside `index.tsx` we would have something like this:
```tsx
  export const Card = () => {
    return (
      <Media at="sm">
        <Mobile />
      </Media>
      <Media at="md">
        <Desktop />
      </Media>
    )
  }
```

This is the case **only** for layouts that would add too much complexity when using standard css queries.


### Layouts

With Layouts, we generally define overall page structure that will be presented to the user based on a specific auth state or a specific page.

For example, your app could have an Admin dashboard on `/admin` route which has a fixed header and sidebars on both right and left, and scrollable content in the middle (you're most likely familiar with this kind of layout).

You will define your different layouts inside `layouts` folder:

```
.
.
└── components
    └── layouts
        ├── admin-layout
        │   └── AdminLayout.tsx
        ├── main-layout
        │   └── MainLayout.tsx
        └── blog-layout
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

You will use your fetcher functions with `useSwr` hook inside of your components (organisms, molecules, pages).


## Setting up theming and styles

When creating styles for your core components, you will create `components` folder inside of styles folder. Styles folder will contain all core stylings and theme setup.

Check out [styling guide](styling-guide-section-link) for other styling related stuff.


```
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

## Tests

When organizing test files, here are couple of quick rules:
- Components, utils, fetchers... should have a test file in the same folder where they are placed
- When testing pages, create `__tests__/pages` folder because how next treats pages folder.
- All mocks should be placed in `__mocks__` folder

For other in depth guides for testing take a look at the [testing guide](link-to-testing-section).

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
│      ├── users.ts
│      └── users.test.ts
└── components
    └── ui
        ├── atoms
        │   └── button
        │       ├── Button.test.tsx
        │       └── Button.tsx
        └── molecules
            └── user-card
                ├── UserCard.test.tsx
                └── UserCard.tsx
```