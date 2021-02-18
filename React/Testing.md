## Testing

For testing we used [Jest](https://jestjs.io/) (Javascript testing framework) and [Testing library](https://testing-library.com/) for easier querying and interacting with DOM nodes.
Setup is simple, we need few npm packages `jest`, `@testing-library/jest-dom`, `@testing-library/react`, and we need to add `jest.config.js` file with few options set.

```js
// /jest.config.js
module.exports = {
  setupFilesAfterEnv: ['@testing-library/jest-dom/extend-expect'],
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
};
```

To run tests we just need to add script to package.json

```js
  ...
  "scripts": {
    "test": "jest"
    ...
  }
  ...
```
