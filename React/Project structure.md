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

When adding UI components, you should be able to group them in two root domains:

1. `pages` - page scope based components
2. `shared` - Components that are shared all across the app
   
Folder naming rules:

 1. `kebab-case` folder name indicates domain name
 2. `PascalCase` folders and filenames should be used for components naming

```
src
├── components
│   ├── pages
│   │   ├── home
│   │   │   ├── HomeHeaderSection
│   │   │   │   └── HomeHeaderSection.tsx
│   │   │   └── HomeTodoListSection
│   │   │       └── HomeTodoListSection.tsx
│   │   └── todo
│   │       ├── TodoHeaderSection
│   │       │   └── TodoHeaderSection.tsx
│   │       └── TodoCreateFormSection
│   │           └── TodoCreateFormSection.tsx
│   └── shared
│       ├── core
│       │   ├── Section
│       │   │   └── Section.tsx
│       │   └── Card
│       │       └── Card.tsx
│       ├── inputs
│       │   └── TextField
│       │       └── TextField.tsx
│       └── todo
│           ├── TodoCard
│           │   └── TodoCard.tsx
│           ├── TodoList
│           │   └── TodoList.tsx
│           └── TodoCreateForm
│               └── TodoCreateForm.tsx
└── pages
    ├── index.tsx
    └── todo
        └── [id]
            └── index.tsx
```

#### Shared *core* components

We can refer to them as **_atoms_**, smallest building blocks, highly reusable and composable.
You can check the [Open UI](https://open-ui.org/components/card.research) standard proposal for inspiration how to split components into small segments. Components could be designed as [Compound Components](https://kentcdodds.com/blog/compound-components-with-react-hooks) or Black-box Components with good ["inversion of control" interface](https://kentcdodds.com/blog/inversion-of-control) like [ReactSelect](https://react-select.com/components).

<table>
  <tr>
    <th>Components</th>
    <th>Parts</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Card</td>
    <td>
      `Card`,  `CardImage`,  `CardImageOverlay`,  `CardTitle`, `CardDescription`, ...
    </td>
    <td>
      From these parts, you'll be able to compose multiple more specific __molecules__ like `ProductCard` or `UserCard`.
    </td>
  </tr>
  <tr>
    <td>Section</td>
    <td>
      `Section`,  `SectionHeader`, `SectionBody`, ..
    </td>
    <td>
      This might have multiple background schemes like `dimmed`, `inverted`, `light`.
    </td>
  </tr>
  <tr>
    <td>Search</td>
    <td>
      `Search`, `SearchInput`, `SearchEmpty`, SearchResults`, ...
    </td>
    <td>
      `Search` uses context to provide shared state to other parts.
      `SearchInput` renders input and it could be placed anywhere in the DOM structure (for example, in the page `Header`).
      `SearchEmpty` and `SearchResults` handles switching between states and showing the result.
    </td>
  </tr>
  <tr>
    <td>
      ReactSelect
    </td>
    <td>
      `ReactSelect`, 
      `./components/ClearIndicator`, 
      `./components/Control`, ...
    </td>
    <td>
     The list of custom components can be find [here](https://react-select.com/components)
    </td>
  </tr>
</table>

#### Shared *Feature* based domains

We can refer to them as **_molecules_**. They are more specific components built out of **_atoms_** (core components).
They are shared components that encapsulate some specific feature of an app.

Here are some examples of feature domain names:
<table>
  <tr>
    <th>Domains</th>
    <th>Components</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>`inputs`</td>
    <td>`InputField`, `TextField`</td>
    <td>
      Specific form fields prepared to be used with [React Hook Form](https://react-hook-form.com/) library.
      Built out of multiple parts, for example `InputGroup`, `InputLeftElement`, `Input` form [Chakra UI](https://chakra-ui.com/docs/form/input#add-elements-inside-input)
    </td>
  </tr>
  <tr>
    <td>`overlays`</td>
    <td>`UnsupportedBrowser`, `BugsnagError`</td>
    <td>Components that covers the whole page and prevents user to interact with the page in some degree.</td>
  </tr>
  <tr>
    <td>`layouts`</td>
    <td>`MainLayout`, `AdminLayout`</td>
    <td>Components that are shared across the pages and renders the application shell (navigation and footer)</td>
  </tr>
  <tr>
    <td>`messages`</td>
    <td>`NoResultsMessage`, `EmptyListMessage`, `LoadingMessage`, `ErrorMessage`</td>
    <td>Reusable messages components that could be shared across the pages for handling empty list results, loading states or ErrorBoundaries fallback</td>
  </tr>
  <tr>
    <td>`navigations`</td>
    <td>`MainNavigation`, `AdminNavigation`</td>
    <td>Different navigations used in layouts to support different app shell styles. They could handle user logged-in/logged-out states and mobile/desktop layouts</td>
  </tr>
  <tr>
    <td>`footers`</td>
    <td>`MainFooter`, `AdminFooter`</td>
    <td>Different footers used in layouts to support different app shell styles. Serves the same purpose as `navigations`</td>
  </tr>
  <tr>
    <td>`utilities`</td>
    <td>`Meta`, `BugsnagErrorBoundary`</td>
    <td>
    Utility components usually does not have any visual representation on the screen, but they are still reusable declarative components. `Meta` inserts `meta` tags into document `head`. `BugsnagErrorBoundary` catches the error, triggers the Bugsnag report and render fallback component
    </td>
  </tr>
  <tr>
    <td>`panels`</td>
    <td>`ArticlesPanel`, `EventPanel`, `EventSidebarPanel`, `GroupPanel`</td>
    <td>
      Specific panels that holds filtering dropdowns for narrowing down the list results. Usually consists of core `Panel` compound component for sharing the styles and sorting dropdowns.
    </td>
  </tr>
  <tr>
    <td>`markdowns`</td>
    <td>`ArticleMarkdown`, `AnnouncementMarkdown`</td>
    <td>Components that handles parsing of the markdown and styling of the generated HTML</td>
  </tr>
</table>

#### Shared *Entity* based domain

We can refer to them as **_molecules_** also, but they are tied to some entity, for example Datx model, algolia resource, google map entity.

<table>
  <tr>
    <th>Domains</th>
    <th>Components</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>`todo`</td>
    <td>`TodoList`, `TodoCreateForm`, `TodoCard`, ...</td>
    <td colspan="3">
      They should accept primitive props like `resourceId` and hook to the resource fetching layer via `SWR`. 
    </td>
  </tr>
  <tr>
    <td>`user`</td>
    <td>`UserList`, `UserCreateForm`, `UserCard`, ...</td>

  </tr>
  <tr>
    <td>`ticket`</td>
    <td>`TicketList`, `TicketCreateForm`, `TicketCard`, ...</td>
  </tr>
</table>

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

> Moving things to`.elements.tsx` should be the last step in the development process and it should only be used for organizational purposes, i.e. when the main component becomes cluttered and unreadable.

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
