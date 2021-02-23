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
└─ fetchers
    └─ fetcher.ts
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
import useSWR from 'swr'

import { fetcher } from '../fetchers/fetcher';

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
  const { data: user, mutate: mutateUser, error, isValidating } = useSWR('/api/user', fetcher);

  useEffect(() => {
    // https://swr.vercel.app/advanced/performance#dependency-collection
    const hydration = user === undefined && error === undefined && isValidating === false;

    // if no redirect needed, just return (example: already on admin /dashboard)
    // if user data not yet there (fetch in progress, logged in or not) then don't do anything yet
    if (!redirectTo || hydration || isValidating) return;

    if (
      // If redirectTo is set, redirect if the user was not found.
      (redirectTo && !redirectIfFound && !user) ||
      // If redirectIfFound is also set, redirect if the user was found
      (redirectIfFound && user)
    ) {
      router.push(redirectTo);
    }
  }, [router, redirectTo, user, error, isValidating]);

  return { user, mutateUser };
};
```

Here is the implementation of simple `fetcher`

```jsx
// ./fetchers/fetcher.js
export async function fetcher(...args) {
  try {
    const response = await fetch(...args)

    // if the server replies, there's always some data in json
    // if there's a network error, it will throw at the previous line
    const data = await response.json()

    if (response.ok) {
      return data
    }

    const error = new Error(response.statusText)
    error.response = response
    error.data = data
    throw error
  } catch (error) {
    if (!error.data) {
      error.data = { message: error.message }
    }
    throw error
  }
}
```


Here is an example of how we can use `useUser`:

```jsx
// ./pages/admin/index.tsx
import React from 'react';
// ...
import { Layout } from '../components/layouts/Layout';
import { useUser } from '../hooks/useUser';

const Admin = () => {
  const { user } = useUser({
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
export default Admin;
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

  const Modal = modals[router.query.modal];

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

