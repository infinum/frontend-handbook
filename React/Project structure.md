## Organizing components

### UI Components

When adding UI components, you should be able to group them in three root domains:

**1. Core Domain**
Core components are the smallest building blocks, highly reusable and composable. Examples include Card and Section.

**2. Shared Domain**
Shared components are built out of core components and shared between feature components. Examples include InputField and MainLayout.

**3. Features Domain**
Feature components are specific to a particular feature or section of the app. Note that the structure of the features folder may vary from project to project. Developers should evaluate if this pattern is suitable for their specific project requirements.

Folder naming rules:

1. `kebab-case` folder name indicates domain name
2. `PascalCase` folders and filenames should be used for components naming

#### Pages Router

```
src
├── components
│   ├── core
│   │   ├── Section
│   │   │   └── Section.tsx
│   │   └── Card
│   │       └── Card.tsx
│   ├── features
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
│       ├── fields
│       │   └── TextField
│       │       └── TextField.tsx
│       ├── todo
│       │   ├── TodoCard
│       │   │   └── TodoCard.tsx
│       │   ├── TodoList
│       │   │   └── TodoList.tsx
│       │   └── TodoCreateForm
│       │       └── TodoCreateForm.tsx
│       └── utilities
│             ├── BugsnagErrorBoundary
│             │   └── BugsnagErrorBoundary.tsx
│             └── Meta
│                 └── Meta.tsx
└── pages
    ├── index.tsx
    └── todo
        └── [id]
            └── index.tsx
```

### *Core* domain

We can refer to them as ***atoms***, smallest building blocks, highly reusable and composable.
You can check the [Open UI](https://open-ui.org/components/card.research) standard proposal for inspiration how to split components into small segments. Components could be designed as [Compound Components](https://kentcdodds.com/blog/compound-components-with-react-hooks) or Black-box Components with good ["inversion of control" interface](https://kentcdodds.com/blog/inversion-of-control) like [ReactSelect](https://react-select.com/components).

Here are some examples of core components:

<table>
  <tr>
    <th>Components</th>
    <th>Parts</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Card</td>
    <td>
      <code>Card</code>, <code>CardImage</code>, <code>CardImageOverlay</code>, <code>CardTitle</code>, <code>CardDescription</code>, ...
    </td>
    <td>
      From these parts, you'll be able to compose multiple more specific <b>molecules</b> like <code>ProductCard</code> or <code>UserCard</code>.
    </td>
  </tr>
  <tr>
    <td>Section</td>
    <td>
      <code>Section</code>, <code>SectionHeader</code>, <code>SectionBody</code>, ..
    </td>
    <td>
      This might have multiple background schemes like <code>dimmed</code>, <code>inverted</code>, <code>light</code>.
    </td>
  </tr>
  <tr>
    <td>Search</td>
    <td>
      <code>Search</code>, <code>SearchInput</code>, <code>SearchEmpty</code>, <code>SearchResults</code>, ...
    </td>
    <td>
      <code>Search</code> uses context to provide shared state to other parts.
      <code>SearchInput</code> renders input and it could be placed anywhere in the DOM structure (for example, in the page <code>Header</code>).
      <code>SearchEmpty</code> and <code>SearchResults</code> handles switching between states and showing the result.
    </td>
  </tr>
  <tr>
    <td>
      ReactSelect
    </td>
    <td>
      <code>ReactSelect</code>,<br>
      <code>./components/ClearIndicator</code>,<br>
      <code>./components/Control</code>, ...
    </td>
    <td>
     The list of custom components can be found <a href="https://react-select.com/components">here</a>
    </td>
  </tr>
</table>

### *Shared* domain

We can refer to them as ***molecules***. They are more specific components built out of ***atoms*** (core components).
They could be shared between feature components and encapsulates some specific logic of an feature.

We can split them into three domains:

1. `UI` - higher order user interface components
2. `Entity` - UI representation of a data models
3. `Utility` - headless utility components

#### Shared *UI* domain

Component name is always composed out of two parts `Context` + `Domain`, for example `InputField` where `Input` is context and `Field` is domain.

Here are some examples of feature domain names:

<table>
  <tr>
    <th>Domains</th>
    <th>Components</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>fields</code></td>
    <td><code>InputField</code>, <code>TextareaField</code></td>
    <td>
      Specific form fields prepared to be used with <a href="https://react-hook-form.com/">React Hook Form</a> library.
      Built out of multiple parts, for example <code>InputGroup</code>, <code>InputLeftElement</code>, <code>Input</code> form <a href="https://chakra-ui.com/docs/form/input#add-elements-inside-input">Chakra UI</a>
    </td>
  </tr>
  <tr>
    <td><code>overlays</code></td>
    <td><code>UnsupportedBrowserOverlay</code>, <code>BugsnagErrorOverlay</code></td>
    <td>Components that covers the whole page and prevents user to interact with the page in some degree.</td>
  </tr>
  <tr>
    <td><code>layouts</code></td>
    <td><code>MainLayout</code>, <code>AdminLayout</code></td>
    <td>Components that are shared across the pages and renders the application shell (navigation and footer)</td>
  </tr>
  <tr>
    <td><code>messages</code></td>
    <td><code>NoResultsMessage</code>, <code>EmptyListMessage</code>, <code>LoadingMessage</code>, <code>ErrorMessage</code></td>
    <td>Reusable messages components that could be shared across the pages for handling empty list results, loading states or ErrorBoundaries fallback</td>
  </tr>
  <tr>
    <td><code>navigations</code></td>
    <td><code>MainNavigation</code>, <code>AdminNavigation</code></td>
    <td>Different navigations used in layouts to support different app shell styles. They could handle user logged-in/logged-out states and mobile/desktop layouts</td>
  </tr>
  <tr>
    <td><code>footers</code></td>
    <td><code>MainFooter</code>, <code>AdminFooter</code></td>
    <td>Different footers used in layouts to support different app shell styles. Serves the same purpose as <code>navigations</code></td>
  </tr>
  <tr>
    <td><code>panels</code></td>
    <td><code>ArticlesPanel</code>, <code>EventPanel</code>, <code>EventSidebarPanel</code>, <code>GroupPanel</code></td>
    <td>
      Specific panels that holds filtering dropdowns for narrowing down the list results. Usually consists of core <code>Panel</code> compound component for sharing the styles and sorting dropdowns.
    </td>
  </tr>
  <tr>
    <td><code>markdowns</code></td>
    <td><code>ArticleMarkdown</code>, <code>AnnouncementMarkdown</code></td>
    <td>Components that handles parsing of the markdown and styling of the generated HTML</td>
  </tr>
  <tr>
    <td><code>icons</code></td>
    <td><code>PlusIcon</code>, <code>TrashIcon</code></td>
    <td>SVG icons used throughout the application. The icons should be named by what they are, not where they are used, e.g. <code>TrashIcon</code> instad of <code>DeleteIcon</code> or <code>ExclamationCircleIcon</code> instead of <code>ErrorIcon</code></td>
  </tr>
</table>

#### Shared *Entity* domain

We can refer to them as ***molecules*** also, but they are tied to some entity, for example Datx model, algolia resource, google map entity.

Component name is always composed out of two parts `Entity` + `Context`, for example `TodoList` where `Todo` is entity and `List` is context.

<table>
  <tr>
    <th>Domains</th>
    <th>Components</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>todo</code></td>
    <td><code>TodoList</code>, <code>TodoCreateForm</code>, <code>TodoCard</code>, ...</td>
    <td rowspan="3" style="max-width: 400px;">
      Primarily they should accept entity prop like this <code>&lt;UserCard user={user} /&gt;</code> where <code>user</code> is resource form the API, or in the rare occasions they could accept primitive props like <code>resourceId</code> and do the resource fetching via <code>SWR</code>.
    </td>
  </tr>
  <tr>
    <td><code>user</code></td>
    <td><code>UserList</code>, <code>UserCreateForm</code>, <code>UserCard</code>, ...</td>

  </tr>
  <tr>
    <td><code>ticket</code></td>
    <td><code>TicketList</code>, <code>TicketCreateForm</code>, <code>TicketCard</code>, ...</td>
  </tr>
</table>

#### Shared *utility* domain

Utility components usually does not have any visual representation on the screen, but they are still reusable declarative components.

<table>
  <tr>
    <th>Domains</th>
    <th>Components</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>utilities</code></td>
    <td><code>Meta</code>,  <code>BugsnagErrorBoundary</code></td>
    <td>
     <code>Meta</code> inserts <code>meta</code> tags into document <code>head</code>. <code>BugsnagErrorBoundary</code> catches the error, triggers the Bugsnag report and render fallback component
    </td>
  </tr>
</table>

#### `components` folder

When adding a components folder, you are essentially breaking down larger main components into smaller, reusable chunks that are only relevant within that specific component.

Guidelines:

1. **Single Level Nesting**: Only one level of component nesting should be used inside the components folder to keep the structure simple and maintainable.
2. **Component Purpose**: These smaller components should not be reused outside their parent component.
3. **Naming Consideration**: To avoid confusion with the root components folder, consider renaming this folder to elements. This renaming helps differentiate between global components and component-specific elements.

Example:
For a MainTable component with a unique TableHeader, the structure should look like this:

```jsx
// src/components/features/MainTable/components/TableHeader.tsx
export const TableHeader: FC<FlexProps> = (props) => {
  const { t } = useTranslation();
  return (
    <Flex align="center" p={20} {...props}>
      <Heading size="md" colorScheme="secondary" as="h3">
        {t("table.title")}
      </Heading>
      <Button leftIcon={<ArrowForwardIcon />} colorScheme="teal" variant="solid">
        {t("table.viewAll")}
      </Button>
    </Flex>
  );
};
```

Folder structure for this feature:

```
src
└── components
    └── features
        └── MainTable
            ├── components
            │   └── TableHeader.tsx
            └── MainTable.tsx
```

## Elements

If you have many style declarations inside your component file, making it difficult to read, create a separate file named `ComponentName.elements.ts` to store your custom styled components.

Example:

```
└── WelcomeCard
    ├── WelcomeCard.tsx
    └── WelcomeCard.elements.ts
```

**WelcomeCard.tsx:**

```js
import { chakra } from '@chakra-ui/react';

export const WelcomeCardLayout = chakra('div', {
  baseStyle: {
    padding: '16px',
    borderRadius: '8px',
    boxShadow: '0 4px 6px rgba(0, 0, 0, 0.1)',
  },
});

export const WelcomeCardTitle = chakra('h1', {
  baseStyle: {
    fontSize: '24px',
    color: 'teal',
  },
});
```

**WelcomeCard.elements.ts:**

```js
import { chakra } from '@chakra-ui/react';

export const WelcomeCardLayout = chakra('div', {
  baseStyle: {
    padding: '16px',
    borderRadius: '8px',
    boxShadow: '0 4px 6px rgba(0, 0, 0, 0.1)',
  },
});

export const WelcomeCardTitle = chakra('h1', {
  baseStyle: {
    fontSize: '24px',
    color: 'teal',
  },
});
```

Moving things to .elements.tsx should be the last step in the development process and it should only be used for organizational purposes, i.e., when the main component becomes cluttered and unreadable.

**Rules for Creating Elements:**

* Use `.elements.tsx` for organizational purposes only.
* Custom components in `.elements.tsx` should not be reused outside their root component.
* Combining chakra factory and functional components is allowed.
* Avoid using hooks inside elements.

> For more information check [Chakra UI - Style Props section](https://infinum.com/handbook/frontend/react/chakra-ui#style-props).

## Different component layouts for different screen sizes

In most of the cases we should strive to create responsive components that consist of one DOM structure that is going to be rendered on all screen sizes by utilizing CSS only solutions. Work closely with your designer to achieve this.

But, there are cases when we need to create different DOM structures for different screen sizes.

## Collocating different layouts in one component

With the help of Chakra UI [Display](https://chakra-ui.com/docs/styled-system/style-props#display) helper props:

```tsx
export const UserCard = (props) => {
  return (
    <Box {...props}>
      <Box hideFrom='md'>
        // Mobile layout
      </Box>
      <Box hideBelow='md'>
        // Desktop layout
      </Box>
    </Box>
  )
}
```

## Different component layouts for different screen sizes in separate `layout` components

In this case, inside a specific component, we could add a subfolder `layouts` (not to be confused with actual layout described below) to define how our component would look like on a specific media query. We recommend using this approach only for layouts that would add too much complexity when using Chakra UI [Display](https://chakra-ui.com/docs/styled-system/style-props#display) helper props. We realize that this approach usually results in a lot of code duplication, so use this approach only when necessary.

```
.
└── admin
    └── UserCard
        ├── layouts
        │   ├── UserCard.mobile.tsx
        │   └── UserCard.desktop.tsx
        └── UserCard.tsx
```

For this case, inside `UserCard.tsx` we would have something like this:

With help of Chakra UI [Display](https://chakra-ui.com/docs/styled-system/style-props#display) helper props:

```tsx
export const UserCardMobile = () => (
  <Box hideFrom='md'>
    // Mobile layout
  </Box>
)

export const UserCardDesktop = () => (
  <Box hideBelow='md'>
    // Desktop layout
  </Box>
)

export const UserCard = () => {
  return (
    <Fragment>
      <UserCardMobile />
      <UserCardDesktop />
    </Fragment>
  )
}
```

With help of Chakra UI [Show/Hide](https://chakra-ui.com/docs/components/show-hide):

```tsx
import { Show, Hide } from '@chakra-ui/react';

export const UserCard = () => {
  return (
    <Fragment>
      <Hide above="md">
        <UserCardMobile />
      </Hide>
      <Show above="md">
        <UserCardDesktop />
      </Show>
    </Fragment>
  )
}
```

> Use of [Show/Hide](https://chakra-ui.com/docs/components/show-hide) helpers is advisable to be used only for client side rendering. For server side rendering use [Display](https://chakra-ui.com/docs/styled-system/style-props#display) helper props.

### Extracting utility functions and hooks

Sometimes, you will have complex functions or effects inside your component that will affect the readability of your component.
In that case, you should extract them into separated files `*.utils.ts` and `*.hooks.ts`.

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
    <div>{formatReleaseDate(props.startDate, props.endDate)</div>
    //...

  )
}
```

**Cleaned**

```tsx
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
    <div>{formatReleaseDate(props.startDate, props.endDate)</div>
    //...
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
    └── shared
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

### Utility components

Utility components are headless, which means that they don't have any impact on the UI itself.
For example, Meta component for injecting meta tags inside document `<head>`.

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
    ├── shared
    │   ├── utilities
    │   │   └── Meta
    │   │       └── Meta.tsx
    ...
```

## Setting up the store

Your datx store and models will be placed in the root of the `src` folder as follows:

```
src
├── models
│   ├── User.ts
│   └── Session.ts
└── datx
    └── create-client.ts
```

## Fetchers

> This section is considered deprecated. For the newer approach check the [@datx/swr](https://datx.dev/docs/jsonapi-swr/overview) documentation.

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

When creating styles for your core components, you will create the `components` folder inside of the styles folder. The styles folder will contain all core stylings and the theme setup.

```
src
└── styles
    └── theme
        ├── index.ts # main theme endpoint
        ├── styles.ts # global styles
        ├── foundations # colors, typography, sizes...
        │   ├── font-sizes.ts
        │   └── colors.ts
        └── components # components styles
            └── button.ts
```

## Tests

When organizing test files, here are couple of quick rules:

* Components, utils, fetchers... should have a test file in the same folder where they are placed
* When testing pages, create the `__tests__/pages` folder because of how Next.js treats pages folder.
* All mocks should be placed in `__mocks__` folder

For other in depth guides for testing take a look at the [testing guide](https://infinum.com/handbook/frontend/react/testing-best-practices).

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
    └── core
        └── Button
            ├── Button.test.tsx
            └── Button.tsx
```

## The complete structure

```
src
├── __mocks__
│   └── react-i18next.tsx
├── __tests__
│   ├── pages
│   │   └── user.test.ts
│   └── test-utils.tsx
├── components
│   ├── core
│   │   ├── Button
│   │   │   ├── Button.test.tsx
│   │   │   └── Button.tsx
│   │   ├── Section
│   │   │   ├── Section.test.tsx
│   │   │   └── Section.tsx
│   │   └── Card
│   │       ├── Card.test.tsx
│   │       └── Card.tsx
│   ├── features
│   │   ├── home
│   │   │   ├── HomeHeaderSection
│   │   │   │   ├── HomeHeaderSection.test.tsx
│   │   │   │   └── HomeHeaderSection.tsx
│   │   │   └── HomeTodoListSection
│   │   │       ├── HomeTodoListSection.test.tsx
│   │   │       └── HomeTodoListSection.tsx
│   │   ├── album
│   │   │   └── AlbumsCarousel
│   │   │       ├── AlbumsCarousel.tsx
│   │   │       ├── AlbumsCarousel.test.ts
│   │   │       ├── AlbumsCarousel.utils.ts
│   │   │       └── AlbumsCarousel.hooks.ts
│   │   └── todo
│   │       ├── TodoHeaderSection
│   │       │   ├── TodoHeaderSection.test.tsx
│   │       │   └── TodoHeaderSection.tsx
│   │       └── TodoCreateFormSection
│   │           ├── TodoCreateFormSection.test.tsx
│   │           └── TodoCreateFormSection.tsx
│   └── shared
│       ├── layouts
│       │   ├── AdminLayout
│       │   │   └── AdminLayout.tsx
│       │   ├── MainLayout
│       │   │   └── MainLayout.tsx
│       │   └── BlogLayout
│       │       └── BlogLayout.tsx
│       ├── fields
│       │   └── TextField
│       │       ├── TextField.test.tsx
│       │       └── TextField.tsx
│       ├── cards
│       │   ├── UserCard
│       │   │   ├── layouts
│       │   │   │   ├── UserCard.mobile.tsx
│       │   │   │   └── UserCard.desktop.tsx
│       │   │   ├── UserCard.test.tsx
│       │   │   └── UserCard.tsx
│       │   └── WelcomeCard
│       │       ├── WelcomeCard.test.tsx
│       │       ├── WelcomeCard.elements.ts
│       │       └── WelcomeCard.tsx
│       ├── todo
│       │   ├── TodoCard
│       │   │   ├── TodoCard.test.tsx
│       │   │   └── TodoCard.tsx
│       │   ├── TodoList
│       │   │   ├── TodoList.test.tsx
│       │   │   └── TodoList.tsx
│       │   └── TodoCreateForm
│       │       ├── TodoCreateForm.test.tsx
│       │       └── TodoCreateForm.tsx
│       └── utilities
│             ├── BugsnagErrorBoundary
│             │   └── BugsnagErrorBoundary.tsx
│             └── Meta
│                 └── Meta.tsx
├── models
│   ├── User.ts
│   └── Session.ts
├── datx
│   └── create-client.ts
├── styles
│   └── theme
│       ├── index.ts
│       ├── styles.ts
│       ├── foundations
│       │   ├── font-sizes.ts
│       │   └── colors.ts
│       └── components
│           └── button.ts
└── pages
    ├── index.tsx
    └── todo
        └── [id]
            └── index.tsx
```

## App Router

By using the App Router, you gain more granular control over your layouts, data fetching, and code structure - yielding a cleaner, more scalable, and more performant application in the long run.

### Key Differences from the Pages Router

With the release of Next.js 13, the App Router introduces several new concepts that differ significantly from the Pages Router. Here are the key differences and benefits:

1. `app/` Directory
   * Replaces `pages/` as the main place for routing.
   * Allows colocation of files like `layout.tsx`, `page.tsx`, `loading.tsx`, `error.tsx`, and more within each route segment.
   * Promotes splitting the application by `route/feature` instead of having a flat `pages/` structure.
2. Route Groups
   * Introduced to conceptually group routes without affecting the final URL (`(groupName)` folders).
   * Ideal for organizing large projects (e.g., `(authorization)`, `(admin)`, `(marketing)`).
3. Nested Layouts
   * Each folder in app/ can have a `layout.tsx` file that wraps all of its nested routes.
   * Eliminates the need for layout components in every page.
   * Example:
     * `app/(marketing)/layout.tsx` wraps all `(marketing)` pages like `dashboard` and `about`.
     * `app/(admin)/layout.tsx` applies only to admin pages.
4. Improved File Organization
   * You can colocate `_components`, `_hooks`, `_utils`, `_types`, etc. within each route directory.
   * Encourages a feature-driven, modular code structure instead of a strictly pages-driven one.
5. Server vs Client Components
   * By default, files in `app/` are Server Components, offering better performance and reduced bundle sizes.
   * You can opt into Client Components with `"use client"` directive when you need interactivity.
6. No more `getStaticProps` or `getServerSideProps`
   * Data fetching is now more flexible. You can fetch data directly in a Server Component (using standard `async/await`), or use RSC primitives like `fetch()`.
   * `layout.tsx` can also fetch data and pass it to child components as props.
7. API Routes in `app/api/`
   * Replaces `pages/api/` for route handlers.
   * Uses **Request** & **Response** objects instead of API handlers.
8. `loading.tsx` and `error.tsx` for Better UX
   * `loading.tsx` shows a skeleton loader while fetching data.
   * `error.tsx` catches errors for that route.
   * Works per-route or globally in `app/`.

**Why it's better**

1. More Flexible & Scalable
   * Nested layouts and route groups let you organize complex UIs in a way that was harder or more verbose in the Pages Router.
   * The file-system routing remains intuitive while providing new capabilities like colocation of components, hooks, and utilities.
2. Performance & Bundle Size
   * Server Components help keep initial payload sizes down by only shipping necessary code to the client.
   * Automatic code splitting ensures only essential bits are loaded per route.
3. Better Developer Experience
   * Colocation of related files (tests, types, hooks) simplifies discovering and maintaining logic.
   * The new data-fetching model (removing `getServerSideProps` and `getStaticProps`) feels more natural to React developers.
4. Incremental Adoption
   * You can run App Router and Pages Router in the same project if you are migrating gradually.
   * This means no big-bang rewrite, so you can adopt new features at your own pace.

### App Router structure

Below is an example of a complete App Router folder structure, similar to what you have in the Pages Router section, but adapted to Next.js 13 best practices. This structure follows the **Colocation** principle, which encourages grouping related files together within their respective feature or route directories. This approach improves maintainability, reduces unnecessary imports, and keeps the project modular. You can learn more about Colocation in the [Next.js documentation](https://nextjs.org/docs/app/getting-started/project-structure#colocation). Note that this example includes route groups like `(authorization)`, `(admin)`, and `(marketing)`, a `[locale]` directory for **i18n**, plus an `api` folder for route handlers:

```
src
├── app
│   ├── api                              // Server API endpoints (App Router)
│   │   ├── health                       // Example server endpoint: /api/health
│   │   │   └── route.ts
│   │   └── auth
│   │       └── [..nextauth]
│   │           └── route.ts            // NextAuth route
│   └── [locale]                         // Top-level for i18n (e.g. /en, /fr)
│       ├── layout.tsx                   // Root layout for everything under /[locale]
│       ├── _components                  // Shared "global" components
│       │   └── Button
│       │       ├── Button.tsx
│       │       └── Button.test.tsx
│       ├── _hooks                       // Shared "global" hooks
│       │   └── useResponsive
│       │       ├── useResponsive.ts
│       │       └── useResponsive.test.ts
│       ├── _types                       // Shared "global" types/interfaces
│       │   └── globalTypes.ts
│       ├── _utils                       // Shared "global" utils/helpers
│       |   └── fetchHelpers
│       |       ├── fetchHelpers.ts
│       |       └── fetchHelpers.test.ts
│       ├── (root)
|       |   ├── _components              // Components used only by the root page
|       |   └── page.tsx                 // Root page -> /[locale]
│       ├── (authorization)              // Route group for auth pages
│       │   ├── layout.tsx               // Optional layout for (authorization) routes
│       │   ├── login                    // /[locale]/login
│       │   │   ├── page.tsx
│       │   │   ├── _components
│       │   │   │   └── LoginForm
│       │   │   │       ├── LoginForm.tsx
│       │   │   │       ├── LoginForm.test.tsx
│       │   │   │       └── LoginForm.types.ts     // Types used by LoginForm (props interfaces, etc.)
│       │   │   ├── _hooks
│       │   │   │   └── useLogin
│       │   │   │       ├── useLogin.ts
│       │   │   │       ├── useLogin.test.ts
│       │   │   │       └── useLogin.types.ts      // Types used only by useLogin hook
│       │   │   ├── _types
│       │   │   │   └── loginTypes.ts              // Shared types across login page
│       │   │   └── _utils
│       │   │       └── loginHelpers
│       │   │           ├── loginHelpers.ts
│       │   │           └── loginHelpers.test.ts
│       │   └── signup                   // /[locale]/signup
│       │       ├── page.tsx
│       │       └── _components
│       │           └── SignupForm
│       │               ├── SignupForm.tsx
│       │               └── SignupForm.test.tsx
│       ├── (admin)                      // Route group for admin pages
│       │   ├── layout.tsx               // Optional layout for (admin) routes
│       │   └── users                    // /[locale]/users
│       │       ├── page.tsx
│       │       ├── _components
│       │       │   └── UsersList
│       │       │       ├── UsersList.tsx
│       │       │       └── UsersList.test.tsx
│       │       ├── _hooks
│       │       │   └── useUsersList
│       │       │       ├── useUsersList.ts
│       │       │       └── useUsersList.test.ts
│       │       ├── _types
│       │       │   └── usersTypes.ts
│       │       └── _utils
│       │           └── usersHelpers
│       │               ├── usersHelpers.ts
│       │               └── usersHelpers.test.ts
│       └── (marketing)                  // Route group for marketing pages
│           ├── layout.tsx               // Optional layout for (marketing) routes
│           ├── dashboard                // /[locale]/dashboard
│           │   ├── page.tsx
│           │   ├── _components
│           │   │   ├── DashboardHeader
│           │   │   │   ├── DashboardHeader.tsx
│           │   │   │   └── DashboardHeader.test.tsx
│           │   │   └── DashboardStats
│           │   │       ├── DashboardStats.tsx
│           │   │       └── DashboardStats.test.tsx
│           │   ├── _hooks
│           │   │   └── useDashboard
│           │   │       ├── useDashboardData.ts
│           │   │       └── useDashboardData.test.ts
│           │   ├── _types
│           │   │   └── dashboardTypes.ts
│           │   └── _utils
│           │       └── dashboardHelpers
│           │           ├── dashboardHelpers.ts
│           │           └── dashboardHelpers.test.ts
│           ├── about                    // /[locale]/about
│           │   └── page.tsx
│           └── contact                  // /[locale]/contact
│               └── page.tsx
├── assets                                // Folder for images, icons, or other static assets
│   └── images
│       └── example.png
└── lib                                   // External dependency config (CASL, Stripe, etc.)
    ├── casl.ts
    └── stripe.ts
```
