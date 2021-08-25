## datx

> [datx.dev - A mobx data store](https://datx.dev/)
>
> DatX is an opinionated JS/TS data store. It features support for simple property definition, references to other models, and first-class TypeScript support.
>
> To find out more about Datx, check out the [official documentation](https://datx.dev/).

We'll create two elements to access the data and datx store in our React components: the `DatxProvider` component and the `useDatx` hook. To utilize this, we'll use React Context.

## DatxProvider

```tsx
// DatxProvider.tsx

import { createContext, FC } from 'react';
import { SWRConfig } from 'swr';

import { compare } from '@/utils/compare';

import { Client } from './Client';

export const DatxContext = createContext<Client>(null);

interface IDatxProviderProps {
  client: Client;
}

export const DatxProvider: FC<IDatxProviderProps> = ({ client, children }) => (
  <DatxContext.Provider value={client}>
    <SWRConfig
      value={{
        compare,
      }}
    >
      {children}
    </SWRConfig>
  </DatxContext.Provider>
);
```

Let's break up the `DatxProvider.tsx ` into sections: `Client`, `DatxContext` and `SWRConfig`.

### Client

`Client` is our class that extends the core, Datx collection. Learn how to [create your collection](https://datx.dev/docs/getting-started/configuring-the-collection).

#### JSON:API Client

If you are working with a JSON:API, `datx` can handle that too - hello `datx-jsonapi`. This library provides multiple decorators that you can use to adapt collection and models to work with JSON:API. Read more about [JSON:API](https://jsonapi.org/) and [datx-jsonapi](https://datx.dev/docs/jsonapi/jsonapi-getting-started)

### DatxContext/DatxProvider

`DatxContext` is a ReactContext that will provide us with the Datx store. Read more about [ReactContext](https://reactjs.org/docs/context.html).

### SWRConfig

`swr` is a library that handles caching and revalidating data (hence the name - `stale-while-revalidate`). Read more about [`swr`](https://swr.vercel.app/).

Since Datx uses classes and objects to store models, we need to tell `swr` how to compare them since `swr` can only compare primitive types, arrays, and/or objects out of the box. To do that, using `SWRConfig`, we need to create and set a `compare` function:

```ts
// compare.ts

import { dequal } from 'dequal/lite';

export const compare = (a: any, b: any): boolean => {
  if (a instanceof Response && b instanceof Response) {
    return dequal(a.snapshot.response.data, b.snapshot.response.data);
  }

  return dequal(a, b);
};
```

Once the provider is created, wrap your whole application inside the created provider. In React application, you can do this in `index.ts` file and in Next.js application, you can do this in `_app.ts` file.

## useDatx

To use the context value in our component, we'll create a simple custom hook.

```ts
// useDatx.ts

import { useContext } from 'react';

import { DatxContext } from '../context';

export function useDatx() {
  const client = useContext(DatxContext);

  if (!client) {
    throw new Error('useDatx must be used inside DatxProvider.');
  }

  return client;
}
```

This hook will throw an error if `useDatx` is called without `DatxProvider`. _If you followed the previous step, this will be already implemented in a `index` or `_app` file._
