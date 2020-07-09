## Next.js starter

We published the [**Infinum Next.js starter**](https://www.npmjs.com/package/@infinumjs/js-next-starter) to npm.

Generated project features previously explained modals example, along with styling and a theming using [Emotion](https://www.npmjs.com/package/emotion) for CSS in JS and [Emotion Theming](https://www.npmjs.com/package/emotion-theming).

The project contains a store that uses [DatX](https://www.npmjs.com/package/datx). Also, the store example shows how to add a model to store and how to use model data inside react render function.
The store is initialized and later injected using [Dependable react](https://www.npmjs.com/package/dependable-react).

The testing environment is also set up, and two example tests are added. `/src/components/modals` folder contains a testing file with an example of snapshot testing and testing modal component UI. We can do that using DOM querying (getByTestId) and firing events (fireEvent), which are few of the many features provided by `testing-library`.
