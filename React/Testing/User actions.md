## Introduction to Testing User Actions

> The more your tests resemble the way your software is used, the more confidence they can give you. - Kent D. Dodds, creator of React testing Library.

User actions refer to the various ways a user interacts with a web application, often triggering changes in the state of the application or causing certain functions to execute. These actions can include but are not limited to clicking buttons, submitting forms, hovering over elements, dragging and dropping items, and pressing keys. In essence, user actions serve as the bridge between the user and the application, enabling the two-way communication that makes interactivity possible.

These interactions are an integral part of any interactive application for several reasons:

- **Usability**: User actions help in navigating the application, performing tasks, and accessing features. Without the ability to effectively capture and respond to these actions, an app would be static and not user-friendly.

- **Dynamic Content**: Most modern web applications are designed to be dynamic, updating content or changing layout based on user interactions. The handling of user actions is fundamental to this dynamic behavior.

- **Data Capture**: Actions like form submissions are essential for capturing user data, whether it's a simple search query or a complex set of options in a multi-step form.

- **User Experience**: Properly responding to user actions is crucial for providing a positive user experience. For instance, failing to accurately capture and process a click event could result in a frustrating experience for the user.

Given the critical role that user actions play in interactive applications, it's crucial to rigorously test how your application responds to these actions to ensure reliability, accuracy, and a positive user experience.

## Importance of Testing User Actions
Testing user actions is a critical component of ensuring that an application is both reliable and robust for several key reasons:

- **Bug Identification**: Without testing user actions, bugs can easily go unnoticed until the application is in the hands of the user. By then, not only has the user had a frustrating experience, but it's also more costly in terms of both time and resources to fix the issue.

- **Predictable Behavior**: Testing ensures that the application behaves as expected when users interact with it. This predictability is crucial for building user trust and satisfaction. A button should perform the action it's designed for every time it’s clicked, and a form should submit data accurately and reliably.

- **Accessibility**: Testing user actions also ensures that your application is accessible to as many people as possible, including those using assistive technologies. This is not only good practice but also a legal requirement in many jurisdictions.

- **User Experience**: Ultimately, if user actions are not tested properly, the overall user experience suffers. Elements might not be clickable, forms may not submit correctly, and features may not be accessible, all leading to user dissatisfaction.

- **Edge Cases**: Users often interact with applications in unpredictable ways. Thorough testing of user actions helps to uncover edge cases where the application might behave unexpectedly, allowing for preemptive resolution of these issues.

- **Security**: User actions like form submissions can be vulnerable points for attacks such as SQL injection or cross-site scripting. Rigorous testing can help identify and fix these vulnerabilities.

Given these factors, it's evident that testing user actions is not just a good-to-have but a must-have practice for any serious development project. It contributes directly to the application’s reliability and robustness, ensuring that it meets both functional and non-functional requirements.

## Methods for Testing User Actions
The library we will be using is `@testing-library/user-event`. Here's a brief discussion on what `userEvent` is and why it's beneficial for simulating user actions.

### What is `userEvent`?

[userEvent](https://testing-library.com/docs/user-event/intro) is a library that simulates user actions like clicking, typing, tabbing, etc., in a way that closely mimics real user behavior. It builds upon the `fireEvent` utility, also from React Testing Library, but provides a more user-centric experience for firing events.

### Why Use `userEvent`?

1. **Realistic Simulation:** Unlike simpler methods of event simulation that trigger only individual events, `userEvent` methods generate a sequence of events that more closely mimic user interactions. For example, when you use `userEvent.type`, it doesn't just change the value of an input; it also fires the appropriate `keydown`, `keyup`, and `change` events in the correct sequence.

2. **Ease of Use:** `userEvent` offers a more straightforward, readable API for simulating complex user interactions, making your tests easier to write and understand.

3. **Focus on User Experience:** By simulating the full flow of events triggered by user interactions, `userEvent` helps you catch issues that simpler simulation methods might miss. This leads to a more robust application and better user experience.

4. **Comprehensive Testing:** Using `userEvent`, you can cover various edge cases, like what happens if a user types too quickly, or if they click a button multiple times in rapid succession.

Suggested reading: [Use @testing-library/user-event over fireEvent where possible.](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#not-using-testing-libraryuser-event)

### Commonly Used `userEvent` Methods

- **`click(element)`**: Simulates a mouse click event on a given element.
- **`type(element, value)`**: Types the given value into the input element, simulating individual keypress events.
- **`hover(element)`**: Simulates a hover event over a given element.
- **`dblClick(element)`**: Simulates a double-click event on a given element.
- **`clear(element)`**: Clears the content of an input field.

## Examples

### Testing Button Click

Here's a simple example using Jest and React Testing Library to test a button click:

```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should update the text', () => {
  	render(<MyButtonComponent />);
  
  	const button = screen.getByRole('button', { name: /click me/i });
  	userEvent.click(button);
  
  	expect(screen.getByText('Button clicked')).toBeInTheDocument();
});
```

In this example, `userEvent.click()` simulates a real user clicking the button, making the test more reflective of actual user behavior.

### Testing Toggle Button

When we are trying to assert elements that are not yet visible on the DOM, we can use the `queryBy*` method variants. Read more about this here: [Using query* variants](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#using-query-variants-for-anything-except-checking-for-non-existence)

```jsx
it('should render toggle show more/less button', async () => {
	const showMoreButton = screen.getByRole('button', { name: /showmore/i });
	const showLessButton = screen.queryByRole('button', { name: /showless/i });

	expect(showMoreButton).toBeInTheDocument();
	expect(showLessButton).not.toBeInTheDocument();

	userEvent.click(showMoreButton as HTMLButtonElement);

	waitFor(() => expect(screen.getByRole('button', { name: /showless/i })).toBeInTheDocument());
	waitFor(() => expect(screen.queryByRole('button', { name: /showmore/i })).not.toBeInTheDocument());
});
```

Suggested reading: [Advice: wait for a specific assertion inside waitFor.](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#passing-an-empty-callback-to-waitfor)

### Testing User Navigation

Here is a trickier scenario where we are testing the navigation of the user:

First, we create a Router Mock. This can be done by creating a new file: `__mocks__/router.ts`, which looks something like this:
```jsx
import { NextRouter } from 'next/router';

const createMockRouter = (router: Partial<NextRouter>): NextRouter => {
	return {
		basePath: '',
		pathname: '/',
		route: '/',
		query: {},
		asPath: '/',
		forward: jest.fn(),
		back: jest.fn(),
		beforePopState: jest.fn(),
		prefetch: jest.fn(() => Promise.resolve()),
		push: jest.fn(),
		reload: jest.fn(),
		replace: jest.fn(),
		events: {
			on: jest.fn(),
			off: jest.fn(),
			emit: jest.fn(),
		},
		isFallback: false,
		isLocaleDomain: false,
		isReady: true,
		defaultLocale: 'en',
		domainLocales: [],
		isPreview: false,
		...router,
	};
};

export default createMockRouter;
```

This allows us to use `Next Router` when testing for User Navigation. Look at the example code below:

```jsx
import { RouterContext } from 'next/dist/shared/lib/router-context';
import createMockRouter from '__mocks__/router';

const router = createMockRouter({});

it('should navigate to register page', async () => {
	const user = userEvent.setup();

	render(
		<RouterContext.Provider value={router}>
			<LoginForm />
		</RouterContext.Provider>
	);

	user.click(screen.getByRole('link', { name: /Register/i }));

	waitFor(() => expect(router.push).toHaveBeenCalled());
});
```

### Testing Form Submit

Many times, we have complex forms that are very important to our business running smoothly, so we can't risk them not working. Here is how we can write a simple test for a form:

First, we set up **MSW** (Mock Service Worker) to mock API requests made by the form. This will ensure we have more control over the responses, so we can test for different scenarios.

```jsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
	rest.post(testApiUrl('/api/login'), (req, res, ctx) => {
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

After, we can test the Login form that hits the `/api/login` endpoint:

```jsx
import { FormProvider, useForm } from 'react-hook-form';

it('should submit form', async () => {
	const user = userEvent.setup();
    const methods = useForm();

	return (
		<FormProvider {...methods}>
			<LoginForm />
		</FormProvider>
	);

	const emailInput = screen.getByRole('textbox', { name: /Email Address/i });
	const passwordInput = screen.getByPlaceholderText('Enter Password');

	user.type(emailInput, 'test@infinum.com');
	user.type(passwordInput, 'password');

	user.click(screen.getByRole('button', { name: /Submit/i }));

	waitFor(() => expect(screen.getByText('Login Successful.')).toBeInTheDocument());
});
```


## Best Practices

Testing user actions is an essential part of ensuring that your web application is robust and user-friendly. However, the way you approach this testing can have a significant impact on its effectiveness. Below are some do's and don'ts to consider when testing user actions in your application.

### Do's:

1. **Always use userEvent**: Use `userEvent` to simulate user actions as closely as possible to how a real user would interact with the application.

2. **Do Use Realistic Test Data**: Use data that closely mimics what actual users would input. This makes your tests more reliable and likely to catch edge cases.

3. **Do Test Multiple User Flows**: Don't just test the "happy path." Consider different user behaviors, such as what might happen if a user double-clicks a button, inputs invalid data, or tries to submit a form without filling it out.

4. **Do Check for Accessibility**: Use tools and techniques to ensure that your application is accessible. Test keyboard navigation, screen reader compatibility, and other accessibility features.

5. **Do Test Asynchrony**: If the user action triggers asynchronous operations like API calls, make sure to account for that in your tests.

6. **Do Validate UI Changes**: After a user action, check that the UI updates as expected. This could involve new elements becoming visible, text changing, or items being added to a list.

7. **Do Use Descriptive Test Names**: Make sure that the names of your test cases clearly indicate what is being tested.

### Don'ts:

1. **Don't Only Test the JavaScript**: While it’s crucial to test how your JavaScript handles user actions, don’t forget to test the end result from a user’s perspective, such as whether the correct page elements are displayed.

2. **Don't Ignore Edge Cases**: Users rarely follow the exact path you expect them to. Always test edge cases to make sure your application can handle them gracefully.

3. **Don't Assume Users Will Follow Instructions**: Users might not use your application the way you intend. Test for unintended or incorrect user actions as well.

4. **Don't Rely Solely on Unit Tests**: While unit tests are helpful, they can't capture the full range of user interactions with your application. Make sure to include integration and end-to-end tests.

5. **Don't Hardcode Test Data**: Hardcoding values can make your tests brittle and less reusable. Whenever possible, generate test data programmatically or fetch it from a controlled source.

6. **Don't Skip Cleanup**: Make sure to reset any changes your tests make to the application state so that one test doesn't affect the outcome of another.

By keeping these do's and don'ts in mind, you'll be better equipped to write effective tests for user actions, leading to a more robust and reliable application.

## Suggested Reading
- [Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Avoid Nesting when you're Testing](https://kentcdodds.com/blog/avoid-nesting-when-youre-testing)
- [Common Testing Mistakes](https://kentcdodds.com/blog/common-testing-mistakes)
- [Static vs Unit vs Integration vs E2E Testing for Frontend Apps](https://kentcdodds.com/blog/static-vs-unit-vs-integration-vs-e2e-tests)
- [Stop mocking fetch](https://kentcdodds.com/blog/stop-mocking-fetch)
- [Write tests. Not too many. Mostly integration.](https://kentcdodds.com/blog/write-tests)
- [Write fewer, longer tests](https://kentcdodds.com/blog/write-fewer-longer-tests)
- [Test Isolation with React](https://kentcdodds.com/blog/test-isolation-with-react)
- [How to know what to test](https://kentcdodds.com/blog/how-to-know-what-to-test)
