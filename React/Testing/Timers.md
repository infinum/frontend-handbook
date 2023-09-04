## Introduction to Testing Timers

> JavaScript provides several timing functions that can be used to execute code after a specified period of time or at regular intervals. The most commonly used timers are setTimeout and setInterval. - Tatiana Maslyak, [Mastering Asynchronous Timing](https://marketsplash.com/tutorials/react-js/settimeout-in-react-js/)

## Common use-cases for timeouts

Using timeouts is a common practice, often driven by the need to manage or introduce delays in component behaviors, handle asynchronous operations, or enhance user experience. Here are some typical use-cases for using timeouts in React:

- **Debouncing User Input**: When users type in a search box, instead of making an API call for every keystroke, you might want to wait until the user stops typing to make the call.

- **Displaying Temporary Feedback**: After users submit a form, you want to display a success message that disappears after a few seconds.

- **Handling Animations**: You have an animation or transition effect, and you want to perform some action once the animation completes.

- **Lazy Loading Components or Modules**: To improve initial page load times, you might want to delay the loading of certain non-essential components.

- **Implementing Automatic Session Logout**: For security reasons, you want to automatically log users out after a period of inactivity.

*Remember, these are just a few scenarios. While timeouts can be handy in managing these scenarios, it's essential to handle them correctly, ensuring they are cleared appropriately (using `clearTimeout`) to avoid unwanted side-effects or memory leaks, especially when components unmount.*


## The Challenges of Testing Components with Timeouts:

Testing components with timeouts introduces specific challenges that can make the process trickier than testing synchronous operations or even other types of asynchronous behaviors. Let's delve into the challenges posed by timeouts:


- **Non-deterministic Behavior**: Timeouts introduce non-deterministic behavior into components. Since they operate on a delay, tests can become unpredictable, especially if not managed correctly.

- **Difficulties in Reproducing Failures**: Failures in tests involving timeouts can be harder to reproduce consistently. The problem might appear under one condition or system load but not another.

- **Cleanup and Memory Leaks**: If timeouts aren't managed properly, they can continue to run even after a test is finished, leading to side effects that might affect other tests or cause memory leaks.

- **Overhead of Mocking**: To effectively test components with timeouts, developers often have to mock or fake timers. This introduces an overhead in terms of both understanding the mocking tools and ensuring they mimic real-world behavior closely.

- **Over-reliance on Real Timers**: While using real timers (like JavaScript's `setTimeout`) might seem like a straightforward way to test, it's often unreliable. Tests might sometimes fail just because the operation took a millisecond longer than expected.

*To navigate these challenges, developers are often advised to make use of testing tools and utilities specifically designed for asynchronous operations, like the `jest.useFakeTimers()` function in Jest, which lets you mock timers and control their progression. Properly mocking and controlling timeouts can lead to more consistent, faster, and reliable tests.*

## Utilizing Testing Libraries and Tools

### Jest's Timer Functions
- **`jest.useFakeTimers()`**: This function allows Jest to replace `setTimeout`, `setInterval`, and other timer functions with mock implementations. 
- **`jest.runAllTimers()`**: Fast-forward until all timers have been executed.
- **`jest.runOnlyPendingTimers()`**: This runs only the timers that are currently scheduled.
- **`jest.advanceTimersByTime(msToRun)`**: To advance the timers by a specific time, this is useful.
- **`jest.clearAllTimers()`**: Clears all timer callbacks.

### React Testing Library
- **`waitFor` and `waitForElementToBeRemoved`**: These functions allow you to wait for elements to appear or disappear from the DOM, which can be useful for asynchronous UI updates following timeouts.
- **`findBy` queries**: These return a promise that resolves when the element is found, useful for async operations.

## Examples

### Waiting for API Response
A simple example is waiting for our component to fetch an external resource from an API endpoint (*We are using **MSW** to mock the endpoint*). `waitForElementToBeRemove` function from React Testing Library, which `awaits` for the `queryByText` to return `true`.

```jsx
it('should show empty state', async () => {
    server.use(
        rest.get('/api/lists', (req, res, ctx) => {
            return res(ctx.json([]));
        })
    );

    render(<TodoLists />);

    await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

    expect(screen.findByText('Empty List!'));
});
```

### Form Submit inside Modal

Here we are testing a full flow of a modal which includes a form where users can create new To-do Lists.

We are mocking the API response and return only 1 list item by using `res.once`. Later, we have the same `GET` request, but we are returning 2 list items. This makes sense, as if the `POST` request is successful, the next `GET` request will also return the list item we just created.

```jsx
it('should add item', async () => {
	const user = userEvent.setup();

	server.use(
	    rest.post('/api/lists', (req, res, ctx) => {
			return res(
				ctx.json(true)
			);
		}),
		rest.get('/api/lists', (req, res, ctx) => {
			return res.once(
				ctx.json([
					{
						uuid: '6e09fb6d-dc49-402f-a237-2b6018291a4a',
						title: 'List 1',
						created: '2023-06-19T08:41:59.417Z',
					},
				])
			);
		}),
		rest.get('/api/lists', (req, res, ctx) => {
			return res.once(
				ctx.json([
					{
						uuid: '6e09fb6d-dc49-402f-a237-2b6018291a4a',
						title: 'List 1',
						created: '2023-06-19T08:41:59.417Z',
					},
					{
						uuid: '16919678-8f95-474c-9eb1-8eb0f6ec249c',
						title: 'List 2',
						created: '2023-06-19T08:41:11.529Z',
					},
				])
			);
		})
	);

	render(<TodoLists />);

    // If we would not have waited for the Loader to be removed,
    // the expect below would throw an error, as no row would be rendered.
	await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));

    // We are expecting the List to include 1 item
	expect(screen.getAllByRole('row')).toHaveLength(1);

    // Here we are opening the Modal
	await user.click(screen.getByRole('button', { name: /Add New List/i }));

	const createModal = screen.getByRole('dialog', { name: /Create Todo list/i });

	expect(createModal).toBeInTheDocument();

	await user.type(screen.getByPlaceholderText('Title'), 'List 2');
	await user.click(screen.getByRole('button', { name: /List item/i }));
	await user.type(screen.getByPlaceholderText('Task 1'), 'Task 1');

    // We are submitting the form and closing the modal
	await user.click(within(createModal).getByRole('button', { name: /Create/i }));
	await user.click(within(createModal).getByRole('button', { name: /Close/i }));
	
    // Now we expect to have 2 list items rendered
	expect(screen.getAllByRole('row')).toHaveLength(2);
});
```


## Best Practices

- **Mock Timers Where Possible**: Instead of waiting for real-time durations, mock timers for faster and more deterministic tests.

- **Clean Up After Each Test**: Always clear any timeouts you've set up to avoid them spilling over to other tests. This can be done using `afterEach` hooks.

- **Avoid Arbitrary Waits**: Using arbitrary `setTimeout` delays in tests to wait for something to happen is a common but bad practice. It makes tests slow and flaky. Instead, use utilities provided by your testing library to wait for specific conditions.

- **Be Aware of External Async Effects**: If a component's asynchronous behavior depends on external factors (like API calls), ensure you mock or stub these out.

- **Test State Transitions**: If timeouts lead to state changes in your components, ensure you test the initial state, the state during the timeout, and the final state after the timeout completes.

- **Cover Branching Logic**: If timeouts lead to branching behaviors (like loading indicators, error messages, etc.), ensure your tests cover all these branches.

*By making the most of the tools at your disposal and adhering to best practices, you can write reliable, fast, and comprehensive tests for asynchronous behavior in your React components.*

