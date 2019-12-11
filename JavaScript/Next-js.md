## Introduction

Next.js supports some very cool features which can be used in different kinds of projects. For example, some projects with public pages can't be built with create-react-app (CRA) because it doesn't support server-side rendering, which makes SEO much harder. So, in projects that require SSR, a different stack must be used that would probably include something like Express on top of node, and custom Webpack and Babel configurations.

Also, any customization of Webpack or Babel inside CRA requires ejecting, which is a one-way operation. From that point onward, you are responsible for maintaining configuration files.

Multiple custom configuration files is something that we want to avoid.
Next.js solves above-stated problems and allows us to use a single stack for many different projects. Plus Next.js introduces some handy features, which come built-in:

- Code splitting
- SSR
- Page prefetching
- Dynamic import of modules
- Built-in Zero-Config TypeScript Support
- Integrated client-side routing
- Custom server with custom express routes
- static HTML export

## POC

We made POC which uses some of the above features. We built an app which can be exported to static HTML files (wouldn't need any server processing) and that contains modals which we can access with a link.

Since many of our projects use modals, it proved to be very useful to be able to access them directly by adding query parameters to the URL.

### App structure

```
...
├─ components
⎮   ├─ hoc
⎮   ⎮   └─ withAuth.tsx
⎮   ⎮   └─ withModal.tsx
⎮   ├─ layouts
⎮   ⎮   └─ Layout.tsx
⎮   ├─ modals
|   |   └─ index.tsx
|   |   └─ Newsletter.tsx
|   └─ LoginRegisterForm.tsx
├─ pages
|   ├─ index.tsx
⎮   ├─ login.tsx
⎮   ├─ register.tsx
|   └─ with-hoc-modal.tsx
├─ services
|   └─ auth.ts
...
```

`login.tsx` and `register.tsx` are public pages whose functionality is self-explanatory. Once login is successful, the user is redirected to the index page which acts as the home page.

### Layouts

`Layout.tsx` is an example of a layout page for all pages. Here is an example of how we can extend `Head` section for adding global fonts.

```jsx
// /components/layouts/layout.tsx
import React, { Fragment } from 'react';
import Head from 'next/head';

import withAuth from '../hoc/withAuth';

const Layout = (props) => {
  return (
    <Fragment>
      <Head>
        <link
          href="https://fonts.googleapis.com/css?family=Roboto"
          rel="stylesheet"
          key="google-font-cabin"
        />
      </Head>

      <main>
        <div className="container">{props.children}</div>
      </main>

      <style global jsx>{`
        body {
          font-family: 'Roboto', sans-serif;
        }
      `}</style>
    </Fragment>
  );
};

export default Layout;
```

Here is an example of how we can use `Layout`:

```jsx
// /pages/index.tsx
import React from 'react';
// ...
import Layout from '../components/layouts/Layout';
import withAuth from '../components/hoc/withAuth';

const Index = () => {
  // ...
  return <Layout>Home Page content</Layout>;
};
export default withAuth(Index);
```

Index page in this example is used as a private page, so we are using withAuth hoc.

### Higher order components

From the React documentation, Higher-Order Components (HOC) are **functions which take components and returns a new component**. They are used when we need to re-use some component logic.

### WithAuth

`withAuth` is simple HOC which checks if the user is authenticated, and if not, the user is redirected to the login page.

```jsx
// /components/higherOrderComponents/withAuth.tsx
import * as React from 'react';
import { useEffectOnce } from 'react-use';
import { useRouter } from 'next/router';

function isLoggedIn() {
  const user = localStorage.getItem('user');
  const userObject = JSON.parse(user);

  return userObject ? !!userObject.token : false;
}

const withAuth = (Component) => (props) => {
  const router = useRouter();

  useEffectOnce(() => {
    if (!isLoggedIn()) {
      router.push('/login');
    }
  });

  return <Component {...props} />;
};
export default withAuth;
```

### WithModal

Another HOC is `WithModal`, which pushes modalName inside props that can be used inside components.

```jsx
// /components/higherOrderComponents/withModal.tsx
import React from 'react';
import { useRouter } from 'next/router';

import useModalName from '../../customEffects/useModalName';

const withModal = (Component) => (props) => {
  const modalName = useModalName();

  return <Component modalName={modalName} {...props} />;
};
export default withModal;
```

### Page with Modal

Example of how can we use modal on a page:

```jsx
// /pages/with-hoc-modal.tsx
import React, { useState } from 'react';
import Link from 'next/link';
import Router from 'next/router';
import modals from '../components/modals';

import WithModal from '../components/hoc/withModal';
import Layout from '../components/layouts/Layout';
import withAuth from '../components/hoc/withAuth';

const pageUrlSlug = 'with-hoc-modal';

export const withHocModalPage = (props: { modalName: string, onModalClose: () => void }) => {
  const [name, setName] = useState(0);
  React.useEffect(() => {
    setName(name || Date.now());
  }, []);

  const onModalClose = () => {
    Router.push(`/${pageUrlSlug}`);
  };

  const Modal = modals[props.modalName];

  return (
    <Layout>
      <h1>Page with hoc modal</h1>
      {name}
      <Link href={`/${pageUrlSlug}?email=testtt&modal=newsletter`} shallow>
        <a>here</a>
      </Link>
      {Modal && <Modal onClose={onModalClose} />}
    </Layout>
  );
};

export default withAuth(WithModal(withHocModalPage));
```

We are storing the `Date.now()` value in a local state, and it's easy to show that the state is preserved between toggling the modal.
To `Layout` we are passing the `onClose` callback, so we would know where to route after closing the modal. In other words, we want to show the same page without query params. If there were some params we needed to preserve, we would use something like the following:

```jsx
Router.push({
  pathname: `/${pageUrlSlug}`,
  query: { name: 'some name' },
});
```

### Dynamic Modal rendering

The only thing that is left to do is to show how we are rendering the modal. In the next code snippet, we can see that modal is loaded dynamically. In that way, no component will load a modal component until URL params contain the modal name.

```jsx
// /components/modals/index.tsx
import dynamic from 'next/dynamic';

const NewsletterModal = dynamic(() => import('./Newsletter'), {
  ssr: false,
  loading: () => <p>Loading</p>,
});

export default {
  newsletter: NewsletterModal,
};
```

## Testing

For testing we used [Jest](https://jestjs.io/) (Javascript testing framework) and [Testing library](https://testing-library.com/) for easier querying and interacting with DOM nodes.
Setup is simple, we need few npm packages `jest`, `@testing-library/jest-dom`, `@testing-library/react`. and we need to add `jest.config.js` file with few options set.

```js
// /jest.config.js
module.exports = {
  setupFilesAfterEnv: ['@testing-library/jest-dom/extend-expect'],
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
};
```

To run tests we just need to add script to package.json

```js
  ...
  "scripts": {
    "test": "jest"
    ...
  }
  ...
```

## Next.js starter

We published to npm our [**Infinum Next.js starter**](https://www.npmjs.com/package/@infinumjs/js-next-starter).

Generated project features previously explained modals example, along with styling and a theming using [emotion](https://www.npmjs.com/package/emotion) for CSS in JS and [emotion theming](https://www.npmjs.com/package/emotion-theming).

The project contains a store that uses [datx](https://www.npmjs.com/package/datx). Also, the store example shows how to add a model to store and how to use model data inside react render function.
The store is initialized and later on, injected using [Dependable react](https://www.npmjs.com/package/dependable-react)

The testing environment is also set, and two example tests are added. `/src/components/modals` folder contains a testing file with an example of snapshot testing and testing modal component UI. We can do that using DOM querying (getByTestId) and firing events (fireEvent), which are few of the many features provided by `testing-library`.
