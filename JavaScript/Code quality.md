## Code quality

There are many tools for ensuring a constant level of code quality is maintained on a project. Different tools do different things in different ways. This handbook covers some of those tools.

If you have already read this section of the Handbook and are here just for the config files, [jump ahead](#putting-it-all-together) (+ [editor files](#editor-files)).

### Git hooks

Git hooks allow us to run scripts during various `git` commands. This verification step can be used to run various tools which ensure that the code satisfies some conditions before it is added to the repository. If the verification fails, the git command will abort.

No matter which code quality tools we use, git hooks are a great way to run those tools. There are multiple ways to add git hooks. For JavaScript projects, we recommend using [husky](https://github.com/typicode/husky). Husky scripts are configured in `package.json`. Here is a basic example which runs `test` before any code is pushed to the repository:

```js
// package.json
{
  "husky": {
    "hooks": {
      "pre-push": "npm test"
    }
  }
}
```

In this example, if you try pushing and the tests fail, code will not get pushed to the remote. We do not necessarily recommend running tests on push, it is just an example (there are better ways to run automated tests using a proper CI/CD set-up).

Most of our code quality tools are run on either `pre-commit` or `pre-push` hooks, so using git hooks is kind of a prerequisite for the rest of the Code quality handbook section.

### Lint-staged

[Lint-staged](https://github.com/okonet/lint-staged) works hand-in-hand with commit hooks - pre-commit hook in particular. It allows us to run scripts only on those files which were staged for committing. This makes hooks run faster since they only need to run on a subset of project files instead of all of them. The assumption is that code quality tools have to be run only on modified code while the code that was untouched should already have been checked.

Similarly to `husky`, `lint-staged` is also configured in `package.json`. It uses `glob` patterns which allow you to run different scripts on different file types/patterns.

Here is an example which runs `eslint` and `prettier` on all staged `.js` and `.ts` files, and `stylelint` on all staged `.scss` files via a pre-commit hook:

```js
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "**/*.{js,ts}": [
      "eslint",
      "prettier --write"
    ]
    "**/*.scss": [
      "stylelint --syntax=scss"
    ]
  }
}
```

What `lint-staged` does is it matches files to `glob` patterns and passes the list of files as an argument to scripts. Tooling developers should ensure that their scripts can receive the list of files in the correct format. Most common tools like eslint, tslint, stylelint and others are compatible with the way `lint-staged` passes the list of files.

It is important to note that `lint-staged` matches files to `glob` patterns in parallel and runs tasks on those matched groups in parallel as well. Within one matched group, the commands are executed in-order. There is an option to force `lint-staged` to process groups sequentially if you need to. In the above example `.scss` files will be processed in parallel with `.js` and `.ts` files. For `.js` and `.ts` files `eslint` will be executed first and only after it is done will `prettier` be executed.

Check the following chapters for specifics about Prettier, ESLint and Stylelint.

### Automatic code formatting with Prettier and IDE settings

![Bikeshedding](/img/work.png)

_Source: [XKCD](https://xkcd.com/1741/)_

Many people are very passionate about the way they format their code. While we appreciate everyone's opinion, we believe it is best to leave this bikeshedding to automated tooling. It might not format the code in a way that is satisfying to everyone, but it will be consistent across projects and, more importantly, it will format the code written by different people in the same way. This also eliminates discussions around formatting.

[Prettier](https://prettier.io/) is one of the most popular tools for this job. It is very opinionated and not very configurable. It might not be perfect, but it is a good way to ensure consistency when it comes to formatting and it format multiple different file types.

Here is our recommended Prettier configuration:

```js
// .prettierrc.json
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
  "trailingComma": "es5",
}
```

If you are using Angular, make sure to add an override for templates parser:

```js
// addendum to the above .prettierrc.json
{
  ...
  "overrides": [{
    "files": "*.component.html",
    "options": {
      "parser": "angular"
    }
  }]
}
```

If you just added Prettier to an existing codebase, you should probably run it once and let it format the whole codebase. This will probably create a lot of modifications which you can skim through and if it checks out you can commit the changes. It is possible that some things might break after Prettier is run. In particular, we noticed some issues with the way Prettier formats SCSS, so you might want to exclude SCSS from Prettier:

```bash
# .prettierignore

# It sometimes breaks SCSS (https://github.com/prettier/prettier/issues/6092)
*.scss
```

Developers should set up their code editors to run Prettier whenever they save a file. This is not a bullet-proof solution because some editors might not have support for this (either natively or via plug-ins). Going one step further, we recommend running Prettier via the pre-commit hook. This ensures that the committed code is formatted even if the developer who wrote it did not have his editor set up to format on file save.

```js
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "prettier --write"
    }
  }
}
```

<a id="editor-files"></a>
To ensure editor settings are in-line with prettier settings, create a workspace `.vscode` settings and `.editorconfig` files:

```js
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.insertSpaces": false,
  "[scss]": {
    "editor.formatOnSave": false
  }
}
```

```bash
# .editorconfig
root = true

[*]
charset = utf-8
indent_style = tab
insert_final_newline = true
trim_trailing_whitespace = true
```

### ESLint and TSLint

Even though [TSLint](https://palantir.github.io/tslint/) is deprecated, it is still used on some projects for legacy reasons. The main reason being that some custom rules packages have not yet migrated to ESLint. If your project does not use custom TSLint rules or you do not use TS, use [ESLint](https://eslint.org/).

Infinum created various rulesets for ESLint and TSLint which you can check out [here](https://github.com/infinum/js-linters). We encourage you to use some of these presets, as per your needs:

  - [Base ESLint config](https://github.com/infinum/js-linters/blob/master/eslint-config/README.md)
  - [React ESLint config](https://github.com/infinum/js-linters/blob/master/eslint-config-react/README.md)
  - [Angular TSLint config](https://github.com/infinum/js-linters/tree/master/tslint-config-angular)

### Stylelint

Just like with ESLint and TSLint, Infinum provides a ruleset for Stylelint as well. You can find it [here](https://github.com/infinum/stylelint-config).

### Putting it all together

Here is the complete example which runs TypeScript compilation check on all files, prettier, tslint and stylelint on an Angular (v10) project:

```js
// package.json
{
  "scripts": {
    "lint:ng": "ng lint",
    "tsc": "concurrently \"npm run tsc:app\" \"npm run tsc:spec\"",
    "tsc:app": "tsc --noEmit -p ./src/tsconfig.app.json",
    "tsc:spec": "tsc --noEmit -p ./src/tsconfig.spec.json"
  },
  "husky": {
    "hooks": {
      "pre-commit": "npm run tsc && lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{json,md,component.html}": [
      "prettier --write"
    ],
    "src/**/*.ts": [
      "prettier --write",
      "ng-lint-staged lint:ng --tsConfig=./tsconfig.base.json --"
    ],
    "src/*.scss": [
      "stylelint --syntax=scss"
    ]
  }
}
```

```js
// tslint.json
{
  "extends": ["@infinumjs/tslint-config-angular", "tslint-config-prettier"],
  "rules": {
    // remember to replace `app` with your selector
    "directive-selector": [true, "attribute", "app", "camelCase"],
    "component-selector": [true, "element", "app", "kebab-case"]
  }
}
```

```js
// stylelint.config.js
module.exports = {
  "extends": "@infinumjs/stylelint-config"
}

```

For this example, you will have to install the following `devDependencies`:

```bash
npm i -D concurrently husky lint-staged ng-lint-staged stylelint prettier tslint-config-prettier @infinumjs/tslint-config-angular @infinumjs/stylelint-config
```

Some notes:

  - it is important to run `tsc` on all files because changes in staged files can affect compilation of unmodified files
  - `tsc` is run on both the application `tsconfig` files and tests `tsconfig` files
      - `concurrently` speeds up things by running these two `tsc` checks in parallel
  - `prettier --write` is run separately for `.ts` and other files in order to prevent any possible race conditions before running TSLint (via `lint:ng`) and Prettier
  - `ng-lint-staged` is required because `ng lint` does not accept the list of files as provided by `lint-staged`, so some transformations are necessary
