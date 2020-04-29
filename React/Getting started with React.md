# Getting started with React

## React

[React](https://reactjs.org/) is not a Framework, it is a JavaScript library for building user interfaces.
With React you will use declarative programming paradigm for building component based systems which could be reused anywhere.

In the next sections we will cover some topics related to React ecosystem used inside our company.

## Next.js Framework

[Next.js](https://nextjs.org/) is a production ready framework for building React applications.

- [Official documentation](https://nextjs.org/docs/getting-started)
- [Learn next](https://nextjs.org/learn/basics/create-nextjs-app)

### Typescript

Next.js supports TypeScript by default and has built-in types for pages and the API.
- [Get started with TypeScript in Next.js](https://nextjs.org/docs/basic-features/typescript).


If you are unfamiliar with Typescript you can go through [documentation](https://www.typescriptlang.org/docs/home.html) or play with it in a [playground](https://www.typescriptlang.org/play/index.html).

Next.js uses `'@babel/preset-typescript'` instead of `tsc` so some features are not supported

> Does not support namespaces or const enums because those require type information to transpile. Also does not support export = and import =, because those cannot be transpiled to ES.next.

### Prettier

- infinum prettier settings
- maybe add note for optional format on save settings in VSCode

### Eslint

We use [Eslint](https://eslint.org/) to find and fix problems in our JavaScript code.

- [Eslint config React](https://www.npmjs.com/package/@infinumjs/eslint-config-react)

### Testing

Automated testing is very important in software development. It gives us the assurance that code won't break when we add new features or change some existing implementation.

For testing React applications we use these libraries:
- [jest](https://jestjs.io/)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)
- [React Hooks Testing Library](https://github.com/testing-library/react-hooks-testing-library)

### Internationalization

- polyglot
- react-i18Next
