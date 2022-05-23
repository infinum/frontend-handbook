Tools we use for testing are [React-Testing-Library](https://testing-library.com/docs/) and [Jest](https://jestjs.io/docs/en/getting-started)

## Video guide

If you want to see more practical usage, [here](https://www.youtube.com/watch?v=KfaFyB0uedk) is the recording form JS Standup about next.js testing presented by 🦌 . The project tested in the video is LearnReact project.

## Setup

First of all, install Jest:

```bash
npm install --save-dev jest @types/jest
```

Add a test script to `package.json`:

```js
  "scripts": {
    "test": "jest",

    // optional?
    "test:ci": "jest --ci --coverage",
    "test:update": "npm run test -- --u",
    "test:watch": "npm run test -- --watch",
    // ...other scripts
  }
```

Create the `jest.config.js` file in the project root and follow the instructions for the [Rust compiler setup](https://nextjs.org/docs/testing#setting-up-jest-with-the-rust-compiler).


Next, install `react-test-library`:

```bash
npm i --save-dev @testing-library/react
```

### User events

Use [testing-library/user-event](https://github.com/testing-library/user-event) for mocking events:

```bash
npm install --save-dev @testing-library/user-event
```

### Testing hooks

Use [react-hooks-testing-library](https://github.com/testing-library/react-hooks-testing-library) for testing hooks:

```bash
npm install --save-dev @testing-library/react-hooks
```

## Utils

### Mock providers and store

While testing, we need to mock various providers like [`<ChakraProvider>`](https://chakra-ui.com/docs/getting-started#setup-provider/). To mock all providers, we can create a `./__tests__/test-utils.tsx` file where we will export the [custom render](https://testing-library.com/docs/react-testing-library/setup/#custom-render) method with all providers:

```tsx
const AllProviders = ({ children }) => (
  <ChakraProvider theme={theme}>{children}</ChakraProvider>
);
```

Beside providers, we need to mock the [datx](https://datx.dev/) store:

```tsx
export const StoreMockContext = React.createContext<AppCollection | null>(null);

const withMockStore = (PageComponent: NextPage) => {
  const WithMockStore: FC = (props) => {
    const store: AppCollection = new AppCollection();

    return (
      <StoreMockContext.Provider value={store}>
        <PageComponent {...props} />
      </StoreMockContext.Provider>
    );
  };

  return WithMockStore;
};
```

While testing we can wrap the component in `<StoreMockContext.Consumer>` to get access to the store. The store is needed for mocking the datx model with relationships.

After mocking all providers and the store, we export everything:

```tsx
const customRender = (ui: React.ReactElement, options?: any) =>
  render(ui, { wrapper: withMockStore(AllProviders), ...options });

// re-export everything
export * from "@testing-library/react";

// override render method
export { customRender as render };
```

Now, when testing components we'll import `render` from `__tests__/test-utils.tsx` instead of `@testing-library/react`, like this:

```tsx
import { render } from "__tests__/test-utils";
```

Or if we add `path` to `tsconfig`:

```json
{
  "compilerOptions": {
    ...
    "paths": {
      ...
      "@test-utils": ["__tests__/test-utils.tsx"]
    }
  },
}
```

Then we can import render like this:

```tsx
import { render } from "@test-utils";
```

### Folder structure

All tests should be defined in the same directory where the file being tested is. There is one exception, and that is the `pages` folder because Next.js doesn't allow tests in the `pages` folder. That's why we have the `__tests__` folder in the root of our application.

Example:

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
    └── shared
        └── Button
            ├── Button.test.tsx
            └── Button.tsx
```

### `__mocks__`

Manual mocks are used to stub out functionality with mock data. For example, instead of accessing a resource from `node_modules` you might want to create a mock module that allows you to use fake data. Manual mocks are defined by writing a module in the `__mocks__/` subdirectory immediately adjacent to the module. ([docs](https://jestjs.io/docs/en/manual-mocks))

For example, if we want to mock [react-i18next](https://react.i18next.com/), we will create `./__mocks__/react-i18next.tsx`

```tsx
import {
  I18nextProvider,
  initReactI18next,
  setDefaults,
  getDefaults,
  setI18n,
  getI18n,
} from "react-i18next";

const useMock = [(k) => k, {}];
useMock.t = (k) => k;
useMock.i18n = {
  language: "en-GB",
};

module.exports = {
  // this mock makes sure any components using the translate HoC receive the t function as a prop
  withTranslation: () => (Component) => (props) => (
    <Component t={(k) => k} {...props} />
  ),
  useTranslation: jest.fn(() => useMock),

  // mock if needed
  I18nextProvider,
  initReactI18next,
  setDefaults,
  getDefaults,
  setI18n,
  getI18n,
};
```

## Introduction

Based on [the Guiding Principles](https://testing-library.com/docs/guiding-principles/), your tests should resemble how users interact with your code (component, page, etc.) as much as possible. In this context, the user is not the end application user, but some parent component that would use the component that is being tested.

**Query priorities** ([more info](https://testing-library.com/docs/guide-which-query/)):

1. Queries Accessible to Everyone queries that reflect the experience of visual/mouse users as well as those that use assistive technology
  - `getByRole` - this can be used to query every element that is exposed in the accessibility tree. With the `name` option you can filter the returned elements by their accessible name. This should be your top preference for just about everything. There's not much you can't get with this (if you can't, it's possible your UI is inaccessible). Most often, this will be used with the name option like so: `getByRole('button', {name: /submit/i})`. Check the [list of roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques#roles).
  - `getByLabelText` - only really good for form fields, but this is the number one method a user finds those elements, so it should be your top preference.
  - `getByPlaceholderText` - a placeholder is not a substitute for a label. But if that's all you have, then it's better than alternatives.
  - `getByText` - not useful for forms, but this is the number 1 method a user finds most non-interactive elements (like divs and spans).
  - `getByDisplayValue` - the current value of a form element can be useful when navigating a page with filled-in values.

2. Semantic Queries HTML5 and ARIA compliant selectors. Note that the user experience of interacting with these attributes varies greatly across browsers and assistive technology.

  - `getByAltText` - if your element is one which supports `alt` text (`img`, `area`, and `input`), then you can use this to find that element
  - `getByTitle` - the title attribute is not consistently read by screenreaders, and is not visible by default for sighted users

3. Test IDs

  - `getByTestId` - The user cannot see (or hear) these, so this is only recommended for cases where you can't match by role or text or it doesn't make sense (e.g. the text is dynamic).

**Avoid unnecessary "is rendering" test**

Since we have no value in testing whether our component will correctly render, avoid writing these type of tests and focus on more valuable tests instead.

**Avoid destructuring `render` result for querying/finding elements**

It is recommended that you avoid destructuring the `render(...)` result and use `screen` object instead.
To see more info about [why you should use screen](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#not-using-screen), and other common mistakes in RTL usage, please refer to the [Kents](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library) blog post.

### Naming Convention

Tests should have meaningful names and should be nested properly by following the next pattern:

1. Root `describe` must be the same as the component/page/hook/util we're testing
2. Nested `describe`s must have the `when` prefix (to indicate specific scenarios)
3. Description of `it` must be a use case sentence

An example:

```tsx
  describe('useAuth', () => {
    it('should throw context error', () => {
      // ...
    });
    it('should toggle loading state', () => {
      // ..
    });

    describe('when user exists', () => {
      it('should return the user object', () => {
        // ...
      });
      it('should log out user', () => {
        // ...
      });
    });

    describe('when user does not exist', () => {
      it('should return guest user', () => {
        // ...
      });
      it('should destroy session on window close', () => {
        // ...
      });
    });
  });
```

### Basic test example

Base Component:

```tsx
const Button: FC<ButtonProps> = (props) => (
  <button {...props} />
);
```

Test:

```tsx
describe("Button", () => {
  // or something more meaningful
  it("should handle click", () => {
    const buttonText = "click here";
    const testOnClick = jest.fn();

    render(
      <Button onClick={testOnClick}>{buttonText}</Button>
    );

    user.click(screen.getByText(buttonText));

    expect(testOnClick).toBeCalledTimes(1);
  });
});
```

**Components that use a base component test example**

Component:

```tsx
import { Button } from "components/Button";

const UserCard: FC<UserCardProps> = ({ title }) => (
  <Card>
    <h3>{title}</h3>
    <Button>click</Button>
  </Card>
);
```

Test:

Here, the `Button` component is mocked since we only care about the specifics of the `UserCard` component. This is especially useful when you don't want to generate a large tree inside your tests for components that don't have any impact on the actual test.

```jsx
import { screen } from '@testing-library/react';

jest.mock("components/button");
(Button as jest.Mock).mockReturnValue(<button />);

describe("UserCard", () => {
  it("should display the correct title", () => {
    const username = "Test User";

    render(<UserCard title={username} />);

    expect(screen.getByText(username)).toBeDefined();
  });
});
```

## Repeated component rendering
Describe your rendering inside `beforeEach` so you could use `screen.{getBySomething}` in your tests later, to reduce number of unnecessary renders.

An example:

```tsx
  describe('AlertButton', () => {
    let title: string;
    let confirmButtonText: string;
    let onConfirm: () => void;

    beforeEach(() => {
      title = 'Bonjour';
      confirmButtonText = 'Like, share, subscribe';
      onConfirm = jest.fn();

      render(
        <AlertButton
          title={title}
          confirmButtonText={confirmButtonText}
          onConfirm={onConfirm}
        />
      );
    });

    describe('when clicked', () => {
      beforeEach(async () => {
        await waitFor(() => {
          user.click(screen.getByText(buttonText));
        });
      });

      it('should open alert dialog', () => {
        expect(screen.queryByRole('alertdialog')).toBeNull();
      });
      it('should display correct title', () => {
        expect(screen.queryByText(title)).not.toBeNull();
      });
    });

    describe('when confirm is clicked', () => {
      beforeEach(async () => {
        await waitFor(() => {
          user.click(screen.getByText(buttonText));
        });
      });

      it('should close the dialog', () => {
        expect(screen.queryByRole('alertdialog')).toBeNull();
      });

      // ... more test cases
    });
  });
```


> **Note:** This is a shortened example of this concept, you can refer to the standup video section for more info:
[Next.js testing - Testing shared components](https://youtu.be/KfaFyB0uedk?t=1005)


## Testing user events

If any part of your test is performing the state update, that action needs to be wrapped into `waitFor`.

You would be also warned for the bad testing practices by the react-testing-library linter plugin.

For simulating an event, we use the `@testing-library/user-event` library.

An example:

```tsx
  import { waitFor, render } from '@test-utils';
  import userEvent from '@testing-library/user-event';

  // ...mocks

  describe('ResetPasswordForm', () => {
    it('should submit without validation errors', async () => {
      const mockSuccess = jest.fn();

      render(
        <ResetPasswordForm onSuccess={mockSuccess} />
      );

      userEvent.type(screen.getByRole('input'), '12345678');

      // this event will trigger react-hook-form to rerender our component
      // so we have to properly handle this state change before making further assertions
      await waitFor(() => {
        userEvent.click(screen.getByRole('button'));
      });

      expect(mockSuccess).toHaveBeenCalled();
    });
  })
```

Check out the list of other [utils](https://testing-library.com/docs/user-event/utility) for using `userEvent`.

> Important note: you should use the `userEvent` object over the `fireEvent` to properly simulate user interactions across the application. Check out [Kent's blog post](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#not-using-testing-libraryuser-event) for more info on this topic.


## Page component testing

Pages tests should be located in `src/__tests__/pages` folder because Next.js is not allowing tests in `/pages` folder. Test name should be the same as the page file with `test.tsx` extension.

Page example:

```tsx
const UserPage: NextPage = () => {
  const { data, error } = useSWR<Array<User>, IResponseError>(
    USER_KEY,
    fetchUsers
  );

  if (error) {
    return <ErrorPage />;
  }

  if (!data) {
    return <LoadingPage />;
  }

  return <UserTemplate userList={data} />;
};

export default UserPage;
```

In this page example we should test all three states of the page component. To do that we should mock `<ErrorPage>`, `<LoadingPage>` and `<UserTemplate>` and check if it's rendered based on the fetcher response.

Test example:

```tsx
jest.mock("components/error-page");
jest.mock("components/loading-page");
jest.mock("components/user-template");
jest.mock("fetchers/users");

(ErrorPage as jest.Mock).mockReturnValue(
  <div data-testid="error-page-testid" />
);
(LoadingPage as jest.Mock).mockReturnValue(
  <div data-testid="loading-page-testid" />
);
(UserTemplate as jest.Mock).mockReturnValue(
  <div data-testid="user-template-page-testid" />
);

describe("User Page", () => {
  it("should render error page", async () => {
    (fetchUsers as jest.Mock).mockResolvedValue(new Error("Error occurred!"));

    render(<UserPage />);

    expect(await screen.findByTestId("error-page-testid")).toBeDefined();
  });
  it("should render loading page", async () => {
    (fetchUsers as jest.Mock).mockResolvedValue(null);

    render(<UserPage />);

    expect(await screen.findByTestId("loading-page-testid")).toBeDefined();
  });
  it("should render user page", async () => {
    (fetchUsers as jest.Mock).mockResolvedValue([{ username: "test user" }]);

    render(<UserPage />);

    expect(await screen.findByTestId("user-template-page-testid")).toBeDefined();
  });
});
```

Because of testing `useSWR` behavior (which will re-render DOM after fetcher promise is resolved), we need to use `findByTestId` method that returns a promise that will resolve when the element is added to DOM.

### SWR testing

To use SWR in your tests, add `SWRConfig` to your `AllProviders` mock with the following setup (note: your cases could require more customization):


```tsx
const AllProviders = ({ children }) => (
  <SWRConfig value={{ dedupingInterval: 0, provider: () => new Map() }}>
    <ChakraProvider theme={theme}>{children}</ChakraProvider>
  </SWRConfig>
);
```

With this setup, we are sure that each component we're testing will have its own cache, so we don't have to manually clear the cache before each test.

Check out [this issue](https://github.com/vercel/swr/issues/781) and [this answer](https://github.com/vercel/swr/issues/781#issuecomment-952738214) that explains fixing the known problem.

### Server side rendering

For testing pages that are rendered on server we use [next-page-tester](https://github.com/toomuchdesign/next-page-tester).
It is used for testing pages that fetch data in `getServerSideProps` or `getStaticProps`.

User page example:

```tsx
const UserPage: NextPage<any> = ({ data }) => {
  return <UserTemplate users={data} />;
};

export async function getServerSideProps() {
  const store = new AppCollection();

  const data = await fetchUsers(store);

  return { props: { data } };
}

export default UserPage;
```

Test example:

```tsx
describe("User Page", () => {
  it("displays user data", async () => {
    (fetchUsers as jest.Mock).mockResolvedValue([
      new User({
        id: "1",
        name: "Test user",
        role: new Role({ name: RoleTypes.User }),
      }),
    ]);

    const { render } = await getPage({
      route: "/user",
    });

    render();

    expect(screen.queryByText('Test user')).not.toBeNull();
  });
});
```

## Fetchers

Fetcher tests should be located in `/fetchers/{{ fetcher name }}` folder next to the fetcher that is tested.

```
...
└── fetchers
    └── users
        ├── user.ts
        └── user.test.ts
```

### Using datx:

When testing fetchers that use Datx, we create mocked store Datx store and instead of making API calls with `request` we need to mock `request` and test the functionality of the fetcher.

In the example below our fetcher fetches one User model and if the user doesn't have a relationship to Role model we add a default one.

```tsx
export async function fetchUser(
  store: AppCollection,
  id: string
): Promise<User> {
  try {
    const response = await store.request(`users/${id}`, "GET", undefined, {
      include: ["role"],
    });
    const user = response.data as User;

    if (user.role === null) {
      const newRole = store.add({ name: RoleTypes.User }, Role);
      user.role = newRole;
    }

    return user;
  } catch (resError) {
    throw resError.error;
  }
}
```

Test example:

```tsx
describe("fetchUser", () => {
  it("should return success with only user", async () => {
    const mockStore = new AppCollection();
    const mockUser = mockStore.add({}, User);

    mockStore.request = jest.fn().mockResolvedValue({ data: mockUser });

    const fetchResponse = await fetchUsers(mockStore, "1");

    expect(fetchResponse).toBeInstanceOf(User);
    expect(fetchResponse.role).toBeDefined();
    expect(fetchResponse.role.name).toBe(RoleTypes.User);
  });

  it("should return success with user and role", async () => {
    const mockStore = new AppCollection();
    const mockRole = mockStore.add({ name: RoleTypes.Superadmin }, Role);
    const mockUser = mockStore.add({ role: mockRole }, User);

    mockStore.request = jest.fn().mockResolvedValue({ data: mockUser });

    const fetchResponse = await fetchUsers(mockStore, "1");

    expect(fetchResponse).toBeInstanceOf(User);
    expect(fetchResponse.role).toBeDefined();
    expect(fetchResponse.role.name).toBe(RoleTypes.Superadmin);
  });

  it("should return an error", async () => {
    const mockError = { description: "Error occurred!" };
    const mockStore = new AppCollection();
    mockStore.request = jest.fn().mockRejectedValue({ error: [mockError] });

    try {
      await fetchUsers(mockStore, "1");
    } catch (fetchError) {
      expect(fetchError.length).toBe(1);
      expect(fetchError[0]).toBe(mockError);
    }
  });
});
```

For more info about testing asynchronous code read [docs](https://jestjs.io/docs/en/asynchronous).

## Hooks

`react-hooks-testing-library` creates a simple test harness for React hooks that handles running them within the body of a function component, as well as providing various useful utility functions for updating the inputs and retrieving the outputs of your custom hook. This library aims to provide a testing experience as close as possible to how your hook is used in a real component.

### Example hook:

```tsx
function useModal(initialOpen = false) {
  const [isOpen, setOpen] = useState(initialOpen);

  const toggle = useCallback(() => {
    setIsOpen(!isOpen);
  }, [isOpen]);

  const close = useCallback(() => {
    setIsOpen(false);
  }, []);

  return { isOpen, close, toggle };
}
```

### Example test:

```tsx
describe("useModal", () => {
  it("should toggle isOpen on toggle call", async () => {
    const { result } = renderHook(() => useModal());

    await waitFor(() => {
      result.current.toggle();
    });

    expect(result.current.isOpen).toBe(true);
  });

  it("should change isOpen to false on close call", async () => {
    const { result } = renderHook(() => useModal(true));

    await waitFor(() => {
      result.current.close();
    });

    expect(result.current.isOpen).toBe(false);
  });
});
```
