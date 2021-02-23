Tools we use for testing are [React-Testing-Library](https://testing-library.com/docs/) and [Jest](https://jestjs.io/docs/en/getting-started)

## Setup

First of all, install Jest:

```bash
npm install --save-dev jest @types/jest
```

Add a test script to `package.json`:

```js
  "scripts": {
    "test": "jest",
    ...
  }
```

Create the `jest.config.js` file in project root:

```js
module.exports = {
  testPathIgnorePatterns: ["<rootDir>/.next/", "<rootDir>/node_modules/"],
  moduleDirectories: ["node_modules", "src"],
};
```

Add the `preset` property to `.babelrc`:

```js
{
  "presets": [
    [
      "next/babel"
    ]
  ]
}
```

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

### Add svg support

```bash
npm install --save-dev jest-svg-transformer
```

Add the `transform` property to `jest.config.js`:

```js
module.exports = {
  ...
  transform: {
    '^.+\\.tsx?$': 'babel-jest',
    '^.+\\.svg$': 'jest-svg-transformer',
  },
}
```

## Utils

### Mock providers and store

While testing, we need to mock various providers like [`<ChakraProvider>`](https://chakra-ui.com/docs/getting-started#setup-provider/). To mock all providers, we can create `./__tests__/test-utils.tsx` file where we will export [custom render](https://testing-library.com/docs/react-testing-library/setup/#custom-render) method with all providers:

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

Now, while testing we can wrap the component in `<StoreMockContext.Consumer>` to get access to the store. The store is needed for mocking the datx model with relationships.

After mocking all providers and the store, we export everything:

```tsx
const customRender = (ui: React.ReactElement, options?: any) =>
  render(ui, { wrapper: withMockStore(AllProviders), ...options });

// re-export everything
export * from "@testing-library/react";

// override render method
export { customRender as render };
```

Now, when testing components we won't import `render` from `__tests__/test-utils.tsx` instead of `@testing-library/react`, like this:

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

## Basic component test

Based on [the Guiding Principles](https://testing-library.com/docs/guiding-principles/), your tests should resemble how users interact with your code (component, page, etc.) as much as possible. In this context, the user is not the end application user, but some parent component that would use the component that is being tested.

Query priorities ([more info](https://testing-library.com/docs/guide-which-query/)):

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

### Base components test example

Component:

```tsx
const Button: FC<ButtonProps> = ({ children, ...rest }) => (
  <button {...rest}>{children}</button>
);
```

Test:

```tsx
describe("Button", () => {
  it("Is rendering", () => {
    const buttonText = "click here";

    const { getByText } = render(<Button>{buttonText}</Button>);

    expect(getByText(buttonText)).toBeDefined();
  });

  it("Is onClick called", () => {
    const buttonText = "click here";
    const testOnClick = jest.fn();

    const { getByText } = render(
      <Button onClick={testOnClick}>{buttonText}</Button>
    );

    userEvent.click(getByText(buttonText));

    expect(testOnClick).toBeCalledTimes(1);
  });
});
```

### Components that uses a base component test example

Component:

```tsx
import { Button } from "components/Button";

const UserCard: FC<UserCardProps> = ({ label }) => (
  <Card>
    <h3>{label}</h3>
    <Button>click</Button>
  </Card>
);
```

Test:

```jsx
jest.mock("components/button");
(Button as jest.Mock).mockReturnValue(<button />);

describe("UserCard", () => {
  it("Is rendering", () => {
    const username = "Test User";

    const { getByText } = render(<UserCard label={username} />);

    expect(getByText(username)).toBeDefined();
  });
});
```

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

It this page example we should test all three states of the page component. To do that we should mock `<ErrorPage>`, `<LoadingPage>` and `<UserTemplate>` and check if it's rendered based on fetcher response.

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
  it("Is error page rendered", () => {
    (fetchUsers as jest.Mock).mockResolvedValue(new Error("Error occurred!"));

    const { findByTestId } = render(<UserPage />);

    expect(findByTestId("error-page-testid")).toBeDefined();
  });

  it("Is loading page rendered", () => {
    (fetchUsers as jest.Mock).mockResolvedValue(null);

    const { findByTestId } = render(<UserPage />);

    expect(findByTestId("loading-page-testid")).toBeDefined();
  });

  it("Is loading page rendered", () => {
    (fetchUsers as jest.Mock).mockResolvedValue([{ username: "test user" }]);

    const { findByTestId } = render(<UserPage />);

    expect(findByTestId("user-template-page-testid")).toBeDefined();
  });
});
```

Because of testing `useSWR` behavior (which will re-render DOM after fetcher promise is resolved), we need to use `findByTestId` method that returns a promise that will resolve when the element is added to DOM.

### SWR testing

Testing swr has a few known issues. One is cache persistency across tests. To fix that we need to add this:

```tsx
afterEach(async () => {
  await waitFor(() => cache.clear());
});
```

Another issue is deduping. We can fix that by adding `SWRConfig` provider to our custom render.

```tsx
const AllProviders = ({ children }) => (
  <SWRConfig value={{ dedupingInterval: 0 }}>
    <ChakraProvider theme={theme}>{children}</ChakraProvider>
  </SWRConfig>
);
```

More can be found in [this issue](https://github.com/vercel/swr/issues/781).
[Additional info](https://medium.com/frontend-digest/using-testing-libary-with-useswr-f595919de2fd).

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
  it("User page is rendered", async () => {
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

    expect(screen.getByText("Buzz")).toBeDefined();
  });
});
```

## Fetchers

Fetcher tests should be located in `/fetchers/{{ fetcher name }}` folder next to the fetcher that is tested.

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
describe("user fetchers", () => {
  it("api returns success with only user", async () => {
    const mockStore = new AppCollection();
    const mockUser = mockStore.add({}, User);

    mockStore.request = jest.fn().mockResolvedValue({ data: mockUser });

    const fetchResponse = await fetchUsers(mockStore, "1");

    expect(fetchResponse).toBeInstanceOf(User);
    expect(fetchResponse.role).toBeDefined();
    expect(fetchResponse.role.name).toBe(RoleTypes.User);
  });

  it("api returns success with user and role", async () => {
    const mockStore = new AppCollection();
    const mockRole = mockStore.add({ name: RoleTypes.Superadmin }, Role);
    const mockUser = mockStore.add({ role: mockRole }, User);

    mockStore.request = jest.fn().mockResolvedValue({ data: mockUser });

    const fetchResponse = await fetchUsers(mockStore, "1");

    expect(fetchResponse).toBeInstanceOf(User);
    expect(fetchResponse.role).toBeDefined();
    expect(fetchResponse.role.name).toBe(RoleTypes.Superadmin);
  });

  it("api returns error", async () => {
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
  it("isOpen should change to true when toggle", () => {
    const { result } = renderHook(() => useModal());

    act(() => {
      result.current.toggle();
    });

    expect(result.current.isOpen).toBe(true);
  });

  it("isOpen should change to false when close", () => {
    const { result } = renderHook(() => useModal(true));

    act(() => {
      result.current.close();
    });

    expect(result.current.isOpen).toBe(false);
  });
});
```

If any part of your test is preforming an update, that action needs to be wrapped into `act()`. More info about [act()](https://reactjs.org/docs/test-utils.html#act).
