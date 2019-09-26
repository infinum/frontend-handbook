## Introduction

Next.js supports some very cool features which can be used on diffrent kind of projects. For example some projects with public pages couldn't use create-react-app beacause it doesn't support server side rendering which makes SEO much easier.
So in projects that requres SSR, different stack must be used that would probably include something like express on top of node, and custom webpack and babel configuration.
Also any customization of webpack or babel inside CRA requires ejecting which is one way operation and from that point you are responsible for maintaining cofiguration files.
Soon every project would probably introduce something new and we would end up with many different projects very hard to maintain.

Next.js solves above stated problems, and allowes us to use single stack for many different projects. Plus next.js introduces some very usefool features, which comes out of the box:

- Code sppliting
- SSR
- Page prefetching
- Dynamic import of modules
- Built-in Zero-Config TypeScript Support
- Integrated client side routing
- Custom server with custom express routes
- static HTML export

## POC

We made POC which used some of above features. We wanted to build an app which could be exported to static html files (wouldn't need any server processing) and would contain modals which we could be accessed via link.
We used this requirement since many our projects use modals, and it proved to be very usefull to access them directly by adding query params to url.

### App structure

```
...
├─ components
⎮   ├─ higherOrderComponents
⎮   ⎮   └─ withAuth.tsx
⎮   ⎮   └─ withModal.tsx
⎮   ├─ layouts
⎮   ⎮   └─ privatePage.tsx
⎮   ├─ modals
|   |   └─ index.tsx
|   |   └─ newsletter.tsx
|   └─ loginRegisterForm.tsx
├─ pages
|   ├─ index.tsx
⎮   ├─ login.tsx
⎮   ├─ register.tsx
|   └─ withHocModal.tsx
├─ services
|   └─ auth.ts
...
```

`login.tsx` and `register.tsx` are public pages whose functionality is self explanatory. Once login is successful user is redirected to index page which acts as home page.

### Layouts

`privatePage.tsx` is a example of layout page for all private pages. Here is example how we can extend `Head` section for adding global fonts, but more importantly we use layout page with two HOC-s composed with ramda.
`compose` is just a helper function which enables function composition.

```jsx
import React, { Fragment } from 'react';
import Head from 'next/head';
import compose from 'ramda/src/compose';

import withAuth from '../higherOrderComponents/withAuth';
import withModal from '../higherOrderComponents/withModal';

const privatePage = compose(
  withModal,
  withAuth,
);

const PrivatePage = (props) => {
  return (
    <Fragment>
      <Head>
        <link
          href="https://fonts.googleapis.com/css?family=Roboto"
          rel="stylesheet"
          key="google-font-cabin"
        />
      </Head>

      <div id="portal"></div>

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

export default privatePage(PrivatePage);
```

Here is example how we can use `privatePage` layout:

```jsx
import * as React from 'react';
// ...
import PrivatePage from '../components/layouts/privatePage';

const Index = () => {
  // ...
  return <PrivatePage>Home Page content</PrivatePage>;
};

export default Index;
```

### Higher order components

From react docs HOC-s are _function who takes components and returns a new component_. They are used when we need to re use some component logic.

### withAuth

Simple HOC which checks if user is authenticated. If not user is redirected to login

```jsx
import * as React from 'react';
import { useEffectOnce } from 'react-use';
import { useRouter } from 'next/router';

import { isLoggedIn } from '../../services/auth';

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

### withModal

For this POC much more interesting component is `withModal` HOC, which enables pages to show any of supported modals. In this example we have just newsletter modal.

```jsx
import React from 'react';
import Router, { useRouter } from 'next/router';

import useModalName from '../../utils/useModalName';
import Modals from '../modals';
import { ModalNames } from '../../utils/types';

const withModal = (Component) => (props) => {
  const router = useRouter();
  const modalName = useModalName();

  const email = (router.query.email as string) || '';

  const onModalSubmit = (email: string) => {
    console.log(email);
  };

  const onModalClose = () => {
    Router.push(`/${props.pageUrlSlug}`);
  };

  const renderModal = () => {
    switch (modalName) {
      case ModalNames.newsletter:
        return Modals.newsletter({
          email,
          onSubmit: onModalSubmit,
          onClose: onModalClose,
        });
    }
  };

  return (
    <React.Fragment>
      {modalName && renderModal()}
      <Component {...props} />
    </React.Fragment>
  );
};
export default withModal;
```

Here we are returning component, along with modal if query parametar is present and if it matches one of our modals (in this case newsletter modal). HOC is dynamically rendering modal and `/components/modal/index.tsx` is responsible for it.

```jsx
import dynamic from 'next/dynamic';

const NewsletterModal = dynamic(() => import('./newsletter'), {
  ssr: false,
  loading: () => <p>Loading</p>,
});

export default {
  newsletter: (props: {
    email?: string,
    isModalVisible?: boolean,
    onSubmit?: (email: string) => void,
    onClose?: () => void,
  }) => <NewsletterModal email={props.email} onSubmit={props.onSubmit} onClose={props.onClose} />,
};
```

This way we can simply dynamically load modal, which means that modal will be loaded asynchronously if on private pages query param containes matching modal name.

### Page with Modal

Example how can we use modal in a page:

`/pages/withHocModal.tsx`

```jsx
import React, { useState } from 'react';
import Link from 'next/link';

import PrivatePage from '../components/layouts/privatePage';

const pageUrlSlug = 'withHocModal';

export const ShowModal = () => {
  const [name, setName] = useState(0);
  React.useEffect(() => {
    setName(name || Date.now());
  }, []);

  return (
    <PrivatePage pageUrlSlug={pageUrlSlug}>
      <h1>Page with hoc modal</h1>
      {name}
      <Link href={`/${pageUrlSlug}?email=testtt&modal=newsletter`} shallow>
        <a>here</a>
      </Link>
    </PrivatePage>
  );
};

export default ShowModal;
```

In a local state we are storing `Date.now()` value and it's easy to show that the state is preserved between toggling modal.
To `PrivatePage` layout we are passing pageUrlSlug so `withModal` HOC would know where to route after closing modal. In other words we are showing the same page without query params.
