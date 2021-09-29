## Session model

Most of the application we build have some kind of a login and protected pages. To make this possible, we use sessions. Using Datx, let's create a session model class.

```ts
import { Model, prop } from 'datx';

export class SessionModel extends Model {
  public static type = 'session';
}
```

Session model is not really helpful on it's own and because of that it usually has a relationship to the user model. Now our session model looks like this

```ts
import { Model, prop } from 'datx';
import { UserModel } from 'models/UserModel';

export class SessionModel extends Model {
  public static type = 'session';

  @prop.toOne(UserModel)
  public user: UserModel;
}
```

## Network

Now, when we have defined session model we can create basic fetchers for the session. We'll create three fetchers - `createSession`, `readSession` and `deleteSession`. We'll create those fetchers using Datx.

_Following fetchers assume that application is using cookies for authentication. To use token or something else, the following methods can easily be extended._

```ts
import { SessionModel } from 'models/SessionModel';

/**
  Creates and returns a session from an endpoint.
  User model is included in the response.
*/
export async function createSession(datx, loginData) {
  return fetch(SESSION_API_ENDPOINT, {
    method: 'POST',
    body: loginData,
    // ...rest of the options
  })
    .then((res) => res.json())
    .then((data) => datx.add(data, SessionModel));
}

/**
  Based on a cookie, request returns a current session if any.
  User model is included in the response.
*/
export async function readSession(datx) {
  return fetch(SESSION_API_ENDPOINT, {
    method: 'GET',
    // ...rest of the options
  })
    .then((res) => res.json())
    .then((data) => datx.add(data, SessionModel));
}

/**
  Deletes a session and clears the cookie.
*/
export async function deleteSession(datx) {
  return (
    fetch(SESSION_API_ENDPOINT, {
      method: 'DELETE',
      // ...rest of the options
    })
      .then((res) => res.json())
      // since only one session can be active per browser, following is OK to do
      .then((data) => datx.removeAll(SessionModel))
  );
}
```

## `useSession` hook

To make a session model accessible in the React components, we'll create a `useSession` hook that will expose a session. We have many ways to implement this, but for the sake of this example we'll stick to [SWR](https://swr.vercel.app/). _SWR is a React Hooks library for data fetching._

```tsx
import { useCallback } from 'react';
import useSWR, { SWRConfiguration } from 'swr';
import { useDatx } from 'hooks/useDatx';
import { createSession, deleteSession, readSession } from 'fetchers/session';

interface IUseSessionOptions extends SWRConfiguration {
  onLoginError?(error): void;
  onLoginSuccess?(session): void;

  onLogoutError?(error): void;
  onLogoutSuccess?(): void;
}

export function useSession({
  onLoginError,
  onLoginSuccess,
  onLogoutError,
  onLogoutSuccess,
  ...config
}: IUseSessionOptions = {}) {
  const datx = useDatx();

  const state = useSWR('session', () => readSession(), {
    shouldRetryOnError: false,
    errorRetryCount: 0,
    ...config,
  });

  const login = useCallback(
    async (attributes) => {
      const session = createSession(datx, attributes).then(
        (session) => {
          onLoginSuccess?.(session);

          return session;
        },
        (error) => {
          onLoginError?.(error);

          return Promise.reject(error);
        }
      );

      return state.mutate(session, false);
    },
    [datx, onLoginError, onLoginSuccess, state]
  );

  const logout = useCallback(async () => {
    const session = deleteSession(datx).then(
      () => onLogoutSuccess?.(),
      (error) => {
        onLogoutError?.(error);

        return Promise.reject(error);
      }
    );

    return state.mutate(session, false);
  }, [datx, onLogoutError, onLogoutSuccess, state]);

  return { login, logout, state };
}
```

Since this there is a lot of code, let's explain section by section.

`useDatx` hook is explained in [DatxStoreProvider](./datx-store-provider)

```ts
const state = useSWR('session', () => readSession(), {
  shouldRetryOnError: false,
  errorRetryCount: 0,
  ...config,
});
```

This part will create a swr state by using `readSession` method. It won't retry on error because if we get an error that should mean that we are not logged in.

Next thing we have are two callbacks - `login` and `logout`. Those two are returned by the hook and can be used in the, i.e., login form to make a login request to the backend API. They both have success and error callbacks that will be called if they are defined. Those two callback will also mutate the swr state.

Now, once we have defined and explained the `useSession` hook, it can be used in any React component that can access to `DatxProvider`.

```tsx
const SomeComponent = () => {
  const { state } = useSession();

  const session = state.data;

  return <p>{session ? 'Session exists' : 'Session does not exists'}</p>;
};
```

## `AuthRedirect`

As mentioned in the introduction of this section, often there is a need for private (protected) pages. To achieve this we can create following logic:

```tsx
const SomePrivatePage = () => {
  const { state } = useSession();
  const router = useRouter();

  const session = state.data;
  const sessionError = state.error;

  useEffect(() => {
    if (!session && sessionError) {
      router.push('/');
    }
  }, [session, sessionError, router]);

  return (
    <Layout>
      <Header />

      <Content />

      <Footer />
    </Layout>
  );
};

const Content = () => {
  const { state } = useSession();

  const session = state.data;

  return {session ? <PrivateContent /> : <Loading />}
}
```

Although this might look like a simple page redirect, it's not really optimized - rerender will occur and might hurt performance.

Once the page is loaded and the component is mounted, `session` and `sessionError` will be `undefined` since the request did not happen yet. Once request to read a session is triggered, `session` or `sessionError` will be defined, and re-render will occur, and this, potentially, might be an expensive operation. To optimize this, we'll create a component that will handle the redirect based on authentication.

```tsx
interface IAuthRedirectProps {
  /**
   * URL used to redirect a user to
   * By default, if only this prop is set, the component will redirect if no session is found
   */
  to: Url | string;
  /**
   * If this property is set to `true`, user will be redirected if he is logged in.
   * Useful when you don't want to show login page to already logged in users.
   *
   * Will be ignored if `condition` is defined.
   */
  ifFound?: boolean;
  /**
   * Callback that will trigger a redirect if true is returned.
   * Useful when you need to redirect base on some attribute, e.g. if user is not admin
   */
  condition?(session: SessionModel): boolean;
}

const AuthRedirect: FC<IAuthRedirectProps> = ({ to, ifFound, condition }) => {
  const {
    state: { data, isValidating, error },
  } = useSession();
  const router = useRouter();

  useEffect(() => {
    // if state is validating, wait until request is done
    if (isValidating) {
      return;
    }

    // https://swr.vercel.app/advanced/performance#dependency-collection
    const hydration = data === undefined && error === undefined && isValidating === false;
    if (hydration) {
      return;
    }

    // `condition` has a priority over a `ifFound` property
    if (condition) {
      if (condition(state.data)) {
        router.push(to);
      }

      return;
    }

    const shouldRedirect = (ifFound && data) || (!ifFound && !data);

    if (shouldRedirect) {
      router.push(to);
    }
  }, [data, isValidating, error, to, condition, ifFound, router]);

  // this component renders nothing since it is only used to redirect if needed.
  return null;
};
```

If we now implement this in our example, this now looks like this.

```tsx
const SomePrivatePage = () => {
  return (
    <>
      <Layout>
        <Header />

        <AuthRedirect to="/">
        <Content />

        <Footer />
      </Layout>
    </>
  );
};
```

Now, the `useSession` is not called on a page level and therefore won't cause a rerender of a entire page. As a result, this will decrease time needed for a rerender.

### Props

In `AuthRedirect` props we have defined multiple properties:

#### `to`

`to` property is used as an argument when calling `router.push`. If redirect needs to occur, this is where user will be redirected.

#### `ifFound`

`ifFound` property is an optional property. It can be used to change the logic when the redirect needs to occur. If set to `true, a redirect will occur if the session exists - this is useful to hide the login/registration page when the session exists.

```tsx
// redirect if session exits
<AuthRedirect to="/" ifFound>
```

_NOTE: if `condition` prop is defined, `isFound` prop will be ignored._

#### `condition`

`condition` property is an optional property. This property is a function that takes the session as an argument and returns a boolean. It will determine if a redirect should occur or not. This can be useful if you need to create a redirect on some condition based on a session.

```tsx
// redirect if logged in user is not an admin
<AuthRedirect to="/" condition={(session) => session?.user.role !== 'admin'}>
```
