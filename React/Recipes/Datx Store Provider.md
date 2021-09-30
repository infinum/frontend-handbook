## datx

> [datx.dev - A mobx data store](https://datx.dev/)
>
> DatX is an opinionated JS/TS data store. It features support for simple property definition, references to other models, and first-class TypeScript support.
>
> To find out more about Datx, check out the [official documentation](https://datx.dev/).

To set up datx store, first, we need to install Datx dependency. To do this, follow [the official documentation](https://datx.dev/docs/getting-started/installation).

After that, we can [create a collection](https://datx.dev/docs/getting-started/configuring-the-collection) and [models](https://datx.dev/docs/getting-started/defining-models).

```ts
import { Collection } from '@datx/core';

export class Client extends Collection {
  public static types = [];
}
```

> To find out more about Datx, check out the [official documentation](https://datx.dev/).

### JSON:API Client

If you are working with a JSON:API, `datx` can handle that too - hello `datx-jsonapi`. This library provides multiple decorators that you can use to adapt collection and models to work with JSON:API. Read more about [JSON:API](https://jsonapi.org/) and [datx-jsonapi](https://datx.dev/docs/jsonapi/jsonapi-getting-started)

### Extra

If you want to follow along completely, here is a list of all needed dependencies:

- [datx](https://datx.dev/)
- [dequal](https://github.com/lukeed/dequal)
- [mobx](https://mobx.js.org/README.html)
- [swr](https://swr.vercel.app/)

After datx store is initialized, we will create a datx context that will enable us to use datx in our web application. For the context, we will need a provider - `DatxProvider`and a hook - `useHook`.

## DatxProvider

> For frontend network layer we will use [SWR](https://swr.vercel.app/) library created by Vercel.

```tsx
// DatxProvider.tsx

import { createContext, FC } from 'react';
import { SWRConfig } from 'swr';
import { dequal } from 'dequal/lite';

import { compare } from 'utils/compare';
import { Client } from './Client';

export const DatxContext = createContext<Client>(null);

interface IDatxProviderProps {
  client: Client;
}

export const DatxProvider: FC<IDatxProviderProps> = ({ client, children }) => (
  <DatxContext.Provider value={client}>
    <SWRConfig
      value={{
        compare: dequal,
      }}
    >
      {children}
    </SWRConfig>
  </DatxContext.Provider>
);
```

Let's break things up into sections:

- `DatxContext` is just a new context that will be used for our datx store

- `Client` is our datx store

- `DatxProvider` is a wrapper that will serve the context along with some default swr config

- `SWRConfig` is a provider for default swr config. We are providing a custom compare function, `dequal`, since datx works with class objects and by default, swr's default compare method does not work well with class objects.

Once the provider is created, wrap your whole application inside the created provider. In React application, you can do this in `index.ts` file and in Next.js application, you can do this in `_app.ts` file.

## useDatx

As mentioned, `useDatx` is a hook that will expose datx context in our component. To make it simpler, we will wrap `useContext` hook in our new `useDatx` hook.

```ts
// useDatx.ts

import { useContext } from 'react';

import { DatxContext } from './context';

export function useDatx() {
  const client = useContext(DatxContext);

  if (!client) {
    throw new Error('useDatx must be used inside DatxProvider');
  }

  return client;
}
```

This hook will throw an error if `useDatx` is called without `DatxProvider`. _If you followed the previous step, this will be already implemented in a `index` or `_app` file._
