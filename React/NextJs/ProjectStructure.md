# Project file structure

## Organizing components
(For PR Review):
I believe this section will be refactored once we get around with all chakra stuff and component organization methodology.
I kept going with only atomic design approach.

### UI Components

Before implementing [atomic design methodology](https://github.com/danilowoz/react-atomic-design), our common practice was separating components by the scope of the page + core components + shared components like this:

Lowercase folders were indicators of a page scope (except of core).

```
.
├── src
└── components
    ├── core
    │   └── Button
    │       ├── index.tsx
    │       └── stories.tsx
    ├── admin
    │   └── WelcomeBanner
    │       └── index.tsx
    ├── Carousel
    │   └── index.tsx
    └── Card
        └── index.tsx
```

With atomic design, we would completely separate all UI components and build them as a blocks of atoms, molecules and organisms (together with templates and pages for putting everything together at the end) 

```
src
.
└── components
    └── ui
        ├── atoms
        │   └── Button
        │       ├── Button.tsx
        │       └── stories.tsx
        ├── molecules
        │   ├── Card
        │   │   └── index.tsx
        │   └── WelcomeBanner
        │       └── index.tsx
        └── organisms
            └── Carousel
                └── index.tsx
```

## Different molecule layouts

Sometimes, you might want to create a molecule specific for mobile and desktop. You might use a tool like [fresnel](https://github.com/artsy/fresnel) for detecting mediaqueries.

In this case, inside a specific molecule, we could add a subfolder `layouts` (not to be confused with actual layout described below) to define how our molecule would look like on specific media query.

```
.
└── molecules
    └── Card
        ├── layouts
        │   ├── Desktop.tsx
        │   └── Mobile.tsx
        └── index.tsx
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
        ├── AdminLayout
        │   └── index.tsx
        ├── MainLayout
        │   └── index.tsx
        └── BlogLayout
            └── index.tsx
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

Your datx store will be placed in the root of the `src` folder as follows:

```
src
└── store
    ├── models
    │   ├── User.ts
    │   └── Session.ts
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
import { User } from '../store/models/User';

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

_Sync with @mamihi_official_
* Styles will probably be a part of components organization

## Tests

(For PR Review):
I personally did not do testing that much, so I found inspiration for writing in some projects and blogs.
I believe this section will be refactored.

### Components

When testing atoms or molecules, you ussualy want to place test file within the same folder:
```
└── atoms
    └── Button
        ├── __snapshots__
        ├── index.tsx
        ├── stories.tsx
        └── Button.test.tsx
```

### Utils and hooks

Testing files for hooks and files are gona be placed in a special subfolder so it could be catched when running tests.

```
  .
  ├── hooks
  │   ├── __tests__
  │   │   ├── useModal.test.ts
  │   │   └── useNotification.test.ts
  │   ├── useModal.ts
  │   └── useNotification.ts
  └── utils
      ├── __tests__
      │   └── generateFileTree.test.ts
      └── generateFileTree.ts

```