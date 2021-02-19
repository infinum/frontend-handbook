## POC

We made POC which uses some of the above features. We built an app which can be exported to static HTML files (wouldn't need any server processing) and that contains modals which we can access with a link.

Since many of our projects use modals, it proved to be very useful to be able to access them directly by adding query parameters to the URL.

### App structure

```
...
├─ components
⎮   ├─ layouts
⎮   ⎮   └─ Layout.tsx
⎮   ├─ modals
|   |   └─ Newsletter.tsx
|   └─ LoginRegisterForm.tsx
├─ hooks
|   ├─ useUser.tsx
├─ pages
|   ├─ index.tsx
⎮   ├─ login.tsx
⎮   ├─ register.tsx
|   └─ modal.tsx
├─ fetchers
|   └─ user.ts
...
```

`login.tsx` and `register.tsx` are public pages whose functionality is self-explanatory. Once login is successful, the user is redirected to the index page which acts as the home page.

### Layouts

`Layout.tsx` is an example of a layout page for all pages. Here is an example of how we can extend `Head` section for adding global fonts.

```jsx
// /components/layouts/Layout.tsx
import React, { Fragment } from 'react';
import Head from 'next/head';

export const Layout = (props) => {
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
```

Here is an example of how we can use `Layout`:

```jsx
// /pages/index.tsx
import React from 'react';
// ...
import Layout from '../components/layouts/Layout';

const Index = () => {
  // ...
  return <Layout>Home Page content</Layout>;
};
export default Index;
```

### Auth

`useUser` is simple hook which checks if the user is authenticated, and if not, the user is redirected to the login page.

```jsx
// /hooks/useUser.tsx
import React, { useEffect } from 'react';
import { useRouter } from 'next/router';

function getUser() {
  const user = localStorage.getItem('user');
  const userObject = JSON.parse(user);

  return userObject;
}

export function useUser({
  redirectTo = false,
  redirectIfFound = false,
} = {}) {
  const router = useRouter();
  // you should use swr to fetch user from the api
  const user = getUser();

  useEffect(() => {
    // if no redirect needed, just return (example: already on /dashboard)
    if (!redirectTo) return;

    if (
      // If redirectTo is set, redirect if the user was not found.
      (redirectTo && !redirectIfFound && !user) ||
      // If redirectIfFound is also set, redirect if the user was found
      (redirectIfFound && user)
    ) {
      router.push(redirectTo);
    }
  }, [router, redirectTo, user]);

  return user;
};
```

Here is an example of how we can use `useUser`:

```jsx
// /pages/admin/index.tsx
import React from 'react';
// ...
import { Layout } from '../components/layouts/Layout';
import { useUser } from '../hooks/useUser';

const Admin = () => {
  const user = useUser({
    redirectTo: '/login'
  });

  return (
    <Layout>
      {user ? (
        <div>Admin Page content</div>
      ) : (
        <Skeleton />
      )}
    </Layout>
  );
};
export default Index;
```

### Page with Modal

Example of how can we use modal on a page:

```jsx
// /pages/modal.tsx
import React, { useState, useEffect, useCallback } from 'react';
import Link from 'next/link';
import { useRouter } from 'next/router';

import Layout from '../components/layouts/Layout';
import modals from '../components/modals';
import { useUser } from '../hooks/useUser';

export const ModalPage = () => {
  const router = useRouter();

  const onModalClose = useCallback(() => {
    router.push({
      pathname: router.pathname
    });
  }, [router]);

  const Modal = modals[props.modalName];

  return (
    <Layout>
      <h1>Page modal</h1>
      {name}
      <Link
        href={{
          pathname: router.pathname,
          query: {
            modal: 'newsletter'
          },
        }}
        shallow
      >
        <a>here</a>
      </Link>
      {Modal && <Modal onClose={onModalClose} />}
    </Layout>
  );
};

export default ModalPage;
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

