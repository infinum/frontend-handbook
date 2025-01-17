In the next sections we will cover some topics related to React ecosystem used inside our company.

## Next.js Framework

[Next.js](https://nextjs.org/) is a production ready framework for building React applications.

- [Official documentation](https://nextjs.org/docs/getting-started)
- [Learn next](https://nextjs.org/learn/basics/create-nextjs-app)
- [Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Handbook Introduction](/books/frontend/react/getting-started/nextjs)

### Typescript

Next.js supports TypeScript by default and has built-in types for pages and the API.

- [Get started with TypeScript in Next.js](https://nextjs.org/docs/app/building-your-application/configuring/typescript).

If you are unfamiliar with Typescript you can go through [documentation](https://www.typescriptlang.org/docs/home.html) or play with it in a [playground](https://www.typescriptlang.org/play/index.html).

> Next.js uses `SWC`, and it does not support features that require type information since it focuses on syntax transformation without checking types. Because of that `namespaces` or `const enums` will not work because those require type information to transpile, and `export =` or `import =` cannot be cleanly transpiled to ES module syntax (ESM).

### Prettier

Prettier is an opinionated code formatter that supports many languages
and integrates with most editors. By using Prettier we can achieve consistent code across all the project without spending too many time and energy on PR reviews.
We are obligated to use it on every project.

You can start using it by adding this VSCode plugin https://github.com/prettier/prettier-vscode and creating `.prettierrc` in the root of your project.

Here is an example of `.prettierrc`:

```json
{
  "$schema": "http://json.schemastore.org/prettierrc",
  "printWidth": 120,
  "endOfLine": "lf",
  "useTabs": true,
  "arrowParens": "always",
  "quoteProps": "as-needed",
  "bracketSpacing": true,
  "singleQuote": true,
  "semi": true,
  "trailingComma": "es5"
}
```

#### OPTIONAL:

You can enable `formatOnSave` in VSCode by creating `.vscode/settings.json` and adding these settings:

```
{
  "editor.formatOnSave": true,
}
```

This will format your code every time save file.

### Eslint

We use [Eslint](https://eslint.org/) to find and fix problems in our JavaScript code.

- [Eslint config React JS](https://www.npmjs.com/package/@infinumjs/eslint-config-react-js)
- [Eslint config React TS](https://www.npmjs.com/package/@infinumjs/eslint-config-react-ts)

### Testing

Automated testing is very important in software development. It gives us the assurance that code won't break when we add new features or change some existing implementation. You can read more about how to test in a [separate testing chapter](https://infinum.com/handbook/books/frontend/react/testing-best-practices).

### Internationalization

We use [polyglot-cli](https://www.npmjs.com/package/polyglot-cli) for managing translations and [react-i18next](https://react.i18next.com/) with [next-i18next](https://www.npmjs.com/package/next-i18next) for implementing internationalization in React applications.

Now that you have everything set and ready, you can start with the tutorials and official documentation explained in the [next chapter](/books/frontend/react/official-documentation).
