## Motivation

In almost every application we create, there is some session. _What is saved in this session is not essential now._ The session needs to be accessible in our components and based on that data, we can restrict/allow users access to a page.

## `useSession` hook

To make a session accessible in the React component, we'll create a `useSession` hook that will expose a session.

### SWR example

```tsx
import { useCallback } from 'react';
import useSWR, { SWRConfiguration } from 'swr';

interface IUseAuthOptions extends SWRConfiguration {
  /**
   * on logout success callback
   */
  onLogout?(): void;
  /**
   * on login success callback
   */
  onLogin?(session: SessionModel): void;
}

export const useAuth = ({ onLogin, onLogout, ...config }: IUseAuthOptions = {}) => {
  const datx = useDatx();

  const state = useSWR('current_session', () => readSession(), {
    shouldRetryOnError: false,
    errorRetryCount: 0,
    ...config,
  });

  const login = useCallback(
    async (attributes: ICreateSessionAttributes) => {
      const session = await state.mutate(createSession(attributes), false);

      if (onLogin) {
        onLogin(session);
      }

      return session;
    },
    [state, datx, onLogin]
  );

  const logout = useCallback(async () => {
    const session = await state.mutate(deleteSession(), false);

    if (onLogout) {
      onLogout();
    }

    return session;
  }, [state, datx, onLogout]);

  return { state, login, logout };
};
```

This hook will expose session data and basic session handlers: `login` and `logout. Now, this hook can use this in any component to get session data.

```tsx
const SomeComponent = () => {
  const { state } = useSession();

  const session = state.data;

  return <p>{session ? 'Session exists' : 'Session does not exists'}</p>;
};
```

## `AuthRedirect`

As mentioned in motivation, often there is a need for private/protected pages. To make this possible we can do following:

```tsx
const SomePrivatePage = () => {
  const { state } = useSession();
  const router = useRouter();

  useEffect(() => {
    if (!state.data && state.error) {
      router.push('/');
    }
  }, [state, router]);

  return (
    <>
      <Layout>
        <Header />

        <Content />

        <Footer />
      </Layout>
    </>
  );
};

const Content = () => {
  const { state } = useAuth();

  const session = state.data;

  return {session ? <PrivateContent /> : <Loading />}
}
```

Although this might look like a simple page redirect, render will occur and might hurt app performance. Once the page is loaded and the component is mounted, `state.data` will be `undefined`. Once request to read a session, `state.data` or `state.error` will be defined, and re-render will occur. This means that all child components will be re-rendered, too, which can potentially be an expensive operation. To optimize this, we'll create a component that will handle the redirect.

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
   */
  ifFound?: boolean;
  /**
   * Callback that will trigger a redirect if true is returned.
   * Useful when you need to redirect base on some attribute, e.g. if user is not admin
   */
  condition?(session: SessionModel): boolean;
}

const AuthRedirect: FC<IAuthRedirectProps> = ({ to, ifFound, condition }) => {
  const { state } = useAuth();

  useEffect(() => {
    // if state is validating, wait until request is done
    if (state.isValidating) {
      return;
    }

    // https://swr.vercel.app/advanced/performance#dependency-collection
    const hydration =
      state.data === undefined && state.error === undefined && state.isValidating === false;

    if (hydration) {
      return;
    }

    // condition has a priority over a ifFound property
    if (condition) {
      if (condition(state.data)) {
        router.push(to);
      }

      return;
    }

    const shouldRedirect = (ifFound && state.data) || (!ifFound && !state.data);

    if (shouldRedirect) {
      router.push(to);
    }
  }, [state.data, state.isValidating, state.error, to, condition, ifFound, router]);

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

const Content = () => {
  const { state } = useAuth();

  const session = state.data;

  return {session ? <PrivateContent /> : <Loading />}
}
```

Now, any state update won't trigger a re-render when the redirect logic is separated into a separate component. As a result, this will speed up a re-render.

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

#### `condition`

`condition` property is an optional property. This property is a function that takes the session as an argument and returns a boolean. It will determine if a redirect should occur or not. This can be useful if you need to create a redirect on some condition based on a session.

```tsx
// redirect if logged in user is not an admin
<AuthRedirect to="/" condition={(session) => session?.user.role !== 'admin'}>
```

This property has a higher priority then a default behavior.
