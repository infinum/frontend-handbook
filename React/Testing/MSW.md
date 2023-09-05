## Introduction to MSW (Mock Service Worker)

*Mock Service Worker (MSW) provides an innovative approach to mocking network requests, and it distinguishes itself from traditional mocking methods in several key ways:*

- Operates at the service worker level, which means it intercepts requests outside of the application's context.
- Mimics real network behavior since it intercepts actual outgoing requests.
- Non-intrusive. Doesn't require changing application code or network logic.
- Can mock both REST and GraphQL easily.
- Provides a realistic environment, given that it doesn't mock the actual fetch or XMLHttpRequest but intercepts them.
- Versatile for both development (mocking out yet-to-be-developed APIs) and testing (ensuring consistent responses).
- and more...

## Setup

1. We should install `msw`

```jsx
npm install msw --save-dev
```

2. Import `rest` & `setupServer` into our `.test.tsx` file:

```jsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';
```

3. After that, we can setup our server:

```jsx
const server = setupServer(
    // apiUrl() converts relative URL to an absolute (always use an absolute URL)
	rest.post(apiUrl('/api/login'), (req, res, ctx) => {
		return res(ctx.status(200), ctx.json(true));
	})
);

beforeAll(() => {
	server.listen();
});

afterEach(() => {
	server.resetHandlers();
});

afterAll(() => {
	server.close();
});
```

4. Add the MSW endpoint to our test case:

```jsx
it('should be render List of Flights', async () => {
	server.use(
		rest.get(apiUrl('/api/flights'), (req, res, ctx) => {
			return res(ctx.status(200), ctx.json({ data }));
		})
	);

	const { container } = render(<Flights />);

	await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

	expect(screen.getAllByRole('row')).toHaveLength(10);
});
```
