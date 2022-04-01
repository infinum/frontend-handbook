## datx

> [datx.dev - A mobx data store](https://datx.dev/)
>
> DatX is an opinionated JS/TS data store. It features support for simple property definition, references to other models, and first-class TypeScript support.
>
> To find out more about DatX, check out the [official documentation](https://datx.dev/).

To set up datx store, first, we need to install DatX dependency. To do this, follow [the official documentation](https://datx.dev/docs/getting-started/installation).

After that, we can [create a collection](https://datx.dev/docs/getting-started/configuring-the-collection) and [models](https://datx.dev/docs/getting-started/defining-models).

```ts
import { Collection } from '@datx/core';

export class Client extends Collection {
  public static types = [];
}
```

`types` variable should be a populated array with the model that your app uses. For the sake of this demonstration, it will be empty, but in the last section of this page, some examples will be shown.

> To find out more about Datx, check out the [official documentation](https://datx.dev/).

### JSON:API Client

If you are working with a JSON:API, `datx` can handle that too - hello `datx-jsonapi`. This library provides multiple decorators that you can use to adapt collection and models to work with JSON:API. Read more about [JSON:API](https://jsonapi.org/) and [datx-jsonapi](https://datx.dev/docs/jsonapi/jsonapi-getting-started)

### Extra

If you want to follow along completely, here is a list of all needed dependencies:

- [datx](https://datx.dev/)
- [dequal](https://github.com/lukeed/dequal)
- [mobx](https://mobx.js.org/README.html)
- [swr](https://swr.vercel.app/)

After datx store is initialized, we will create a datx context that will enable us to use DatX in our web application. For the context, we will need a provider - `DatxProvider`and a hook - `useDatx`.

## DatxProvider

> For frontend network layer we will use [SWR](https://swr.vercel.app/) library created by Vercel.

```tsx
// DatxProvider.tsx

import { createContext, FC } from 'react';
import { SWRConfig } from 'swr';
import { dequal } from 'dequal/lite';

import { Client } from './Client';

export const DatxContext = createContext<Client>(null);

interface IDatxProviderProps {
  client: Client;
}

export const DatxProvider: FC<IDatxProviderProps> = ({ client, children }) => (
  <DatxContext.Provider value={client}>
    <SWRConfig value={{}}>{children}</SWRConfig>
  </DatxContext.Provider>
);
```

Let's break things up into sections:

- `DatxContext` is just a new context that will be used for our datx store

- `Client` is our datx store

- `DatxProvider` is a wrapper that will serve the context along with some default swr config

- `SWRConfig` is a provider for swr config defaults. Currently, this is empty but if there is a config that should be used for every swr hook, this is a good place to change that.

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

## Usage

In this section, an example will be shown. Before we start, we'll create a model, i.e. Todo.

```ts
import { Attribute, Model } from '@datx/core';

export class Todo extends Model {
  static type = 'todo';

  @Attribute({ isIdentifier: true })
  public id!: string | number;

  @Attribute()
  public title!: string;

  @Attribute()
  public completed!: boolean;

  @Attribute()
  public createdAt!: string;
}
```

Now we need to modify our collection and add this model to types array.

```ts
import { Todo } from 'models';

export class Client extends Collection {
  public static types = [Todo];
}
```

Once this is set up, and if we assume that all steps defined in this document are done, we can start fetching. For this example, we'll create a component that will fetch `todos` from the API and render them on the page.

Before we create a component, we will create a fetcher.

```tsx
export const getTodos = async (datx) => {
  const response = await fetch('/todos');
  const rawTodos = await response.json();
  const todos = rawTodos.map((rawTodo) => datx.add(rawTodo, 'todo'));

  return todos;
};
```

Now, we can use the fetcher to create a component.

```tsx
import React, { FC } from 'react';
import useSWR from 'swr';

import { useDatx } from 'store';

export const TodoListSection: FC = (props) => {
  const datx = useDatx();
  const { data: todos, error } = useSWR('/todos', () => getTodos(datx));

  if (error) {
    return (
      <div {...props}>
        <p>Something went wrong.</p>
      </div>
    );
  }

  if (!todos) {
    return <div {...props}>Loading todos...</div>;
  }

  return (
    <section {...props}>
      {todos.length === 0 && <p>No todos.</p>}
      {todos.map((todo) => (
        <div className='flex'>
          <input type='checkbox' defaultChecked={todo.completed} />
          <p>{todo.title}</p>
        </div>
      ))}
    </section>
  );
};
```

## JSON:API usage

Here at Infinum, we like to use [JSON:API specification](https://jsonapi.org/). To make the stack easier to maintain, DatX has an extension just for that, JSON:API, specification - [@datx/jsonapi](https://datx.dev/docs/jsonapi/jsonapi-getting-started). If you want to know more about that, please read more at [the official documentation site](https://datx.dev/docs/jsonapi/jsonapi-getting-started). For the purposes of this example, we'll create a model and show a short demonstration how to implement DatX jsonapi it in your application.

To create a JSON:API model you need to decorate existing DatX model using `jsonapi` decorator method from the package.

```ts
import { Attribute, Model } from '@datx/core';
import { jsonapi } from '@datx/jsonapi';

export class Flight extends jsonapi(Model) {
  static type = 'flight';

  @Attribute({ isIdentifier: true })
  public id!: string | number;

  @Attribute()
  public airplaneModel!: string;

  @Attribute()
  public departsAt!: string;

  @Attribute()
  public arrivesAt!: string;

  @Attribute()
  public basePrice!: string;

  @Attribute()
  public currentSeatPrice!: string;

  @Attribute()
  public name!: string;
}
```

Then, we can use this model to create out store:

```ts
import { Collection } from '@datx/core';

import { Flight } from 'models/flight';

export class AppCollection extends jsonapi(Collection) {
  public static types = [Flight];
}
```

Now, once this is setup we also need to provide out config to DatX. To do this, we need to use `config` object from `@datx/jsonapi`.

```ts
import { config, CachingStrategy, IRawResponse, ICollectionFetchOpts } from '@datx/jsonapi';

import { apify, deapify } from 'utils/api';

// since we are using swr, swr will handle cache
config.cache = CachingStrategy.NetworkOnly;

// base url of an api service
config.baseUrl = 'http://localhost:3000/api/v1/';

// these fetch options will be included on every request
config.defaultFetchOptions = {
  // JSON:API standard requires to provide following headers. The default headers required by the spec are added by default, but you're able to override this.
  headers: {
    Accept: 'application/vnd.api+json',
    'Content-Type': 'application/vnd.api+json',
  },
  credentials: 'include',
};

// in order to handle attributes in camelCase rather than snake_case, we'll transform all prop names to camelCase
config.transformResponse = (opts: IRawResponse) => {
  return { ...opts, data: deapify(opts.data) };
};

// same as previous step, but in reverse; all prop names from camelCase to snake_case
config.transformRequest = (opts: ICollectionFetchOpts) => {
  return { ...opts, data: apify(opts.data) };
};
```

Mentioned methods `apify` and `deapify` are using `lodash` methods under the hood. For that you'll need to install `lodash` (or only the specific lodash methods) as well:

```bash
npm install lodash
```

```ts
import camelCase from 'lodash/camelCase';
import snakeCase from 'lodash/snakeCase';

/**
 * Deep iteration trough an object and transformation
 *
 * @param obj - Object that needs to be Transformed
 * @param transformer - Transformer function
 * @return Transformed object
 */
export function iterator(
  obj: object | undefined,
  transformer: typeof snakeCase | typeof camelCase
): object | undefined {
  if (isArray(obj)) {
    return map(obj, (value) => iterator(value, transformer));
  }
  if (isObject(obj)) {
    const copy = mapValues(obj, (value) => iterator(value, transformer));

    return mapKeys(copy, (_, key) => transformer(key));
  }

  return obj;
}

export function apify(obj?: object) {
  return iterator(obj, snakeCase);
}

export function deapify(obj?: object) {
  return iterator(obj, camelCase);
}
```

Now, everything is ready for usage in the components:

```tsx
import React, { FC } from 'react';
import useSWR from 'swr';

import { useDatx } from 'store';
import { Flight } from 'models/flight';

export const FlightDetailsSection: FC = ({ flightId, ...rest }) => {
  const datx = useDatx();

  const { data: flightResponse, error } = useSWR(
    () => (flightId ? `${Flight.type}-${flightId}` : null), // swr key
    () => datx.getOne(Flight, flightId) // swr fetcher
  );

  if (error) {
    // implement better error handling, this is just for demo purposes
    return <div {...rest}>Flight fetch error occurred</div>;
  }

  if (!flightResponse.data) {
    // implement better loading state, this is just for demo purposes
    return <div {...rest}>Loading data...</div>;
  }

  return (
    <div {...rest}>
      {/* Handle flight data */}
      {JSON.stringify(flightResponse.data)}
    </div>
  );
};
```

As you can see this is not really complicated to set up. But all of this becomes hard to track once you have multiple places with request filters etc. Because swr uses the key attribute to cache data and JSON:API supports multiple different parameters, key handling becomes an difficult task. To bypass that, we created a piece of code with better abstraction that will connect three things - DatX, React and SWR. You can checkout the implementation in [our React example repo](https://github.com/infinum/JS-React-Example/tree/master/src/libs/%40datx/jsonapi-react).
