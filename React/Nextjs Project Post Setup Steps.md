## Next.js project post setup steps


#### Project example

[JS React Example project](https://github.com/infinum/JS-React-Example)


### General setup

Before jumping into sections below, read pages for folder structure, naming conventions and other to see how general structure and setup should be in the project.

Do not skip sections when doing setup below since some configurations depend on previous ones.

In each section there will be a noted version for core packages because at the time you are reading this some configurations might
change by package authors especially if new version exists. This doesn't mean setup will not work and if you encounter changes
please inform your team lead.

This post setup was done for Next.js **v9.5.3**. 


### Folder restructuring

1. Delete `styles` folder in the project root. In the project remove all code that was used from that folder.
1. Create `src` folder in the project root and move `pages` folder into `src`
1. Create `tests` folder in the project root. In there create `e2e`, `integration` and `unit` folders.
1. Create `bin` folder in the project root with `build.sh` file
1. Create `.github` folder in the project root with `CODEOWNERS` file
1. Update `README` file
1. Add `src/utils` folder
1. Add `components/ui` and `components/containers` folders in `src` folder

Note: Git will not track empty folders


### Linter, formatter and Git hooks

##### Depends on sections: none

| Package | Version |
|---|---:|
| eslint | 7.9.x |
| prettier | 2.1.x |
| @infinumjs/eslint-config-react | 2.0.x |
| husky | 4.3.x |
| lint-staged | 10.4.x |

To add linter, formatter and Git hooks support do the following:
1. Run command
    ```bash
    yarn add --dev eslint prettier @infinumjs/eslint-config-react husky lint-staged
    ```
    or
    ```bash
    npm install --save-dev eslint prettier @infinumjs/eslint-config-react husky lint-staged
    ```
   to install `eslint` linter, `prettier` formatter, Infinum `eslint` config, `husky` and `lint-staged` to run 
   pre-commit hooks on code.
1. In project root create `.editorconfig` file and add following content
    ```text
    root = true
    
    [*]
    charset = utf-8
    indent_style = tab
    insert_final_newline = true
    trim_trailing_whitespace = true
    ```
1. In project root create `.prettierrc.json` file and add following content
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
        "trailingComma": "es5",
        "parser": "babel-ts"
    }
    ```
1. In project root create `.eslintrc` file and add following content
    ```json
    {
       "extends": "@infinumjs/eslint-config-react"
    }
    ```
1. In `package.json` file add following entry in JSON root
    ```json
    {
         // ...
        "husky": {
            "hooks": {
                "pre-push": "npm test",
                "pre-commit": "prettier --write"
            }
            },
        "lint-staged": {
            "{src,tests}/**/*.{js,ts,tsx}": [
                "eslint",
                "prettier --write"
            ]
        }
    }
    ```
   Note for `pre-push` hook: Tests configuration is required in order to run `npm test` on `pre-push`. This will be done later.
1. Based on your IDE, some changes might be required in IDE configurations
    1. VS Code
        - Create file `settings.json` in `.vscode` folder with following content
        ```json
        {
            "editor.defaultFormatter": "esbenp.prettier-vscode",
            "editor.formatOnSave": true,
            "editor.insertSpaces": false,
            "[scss]": {
                "editor.formatOnSave": false
            }
        }
        ```
       - Commit and push changes
   1. Intellij IDEA / WebStorm
    - In `preferences` on `macOS` or `settings` on `Windows`, enter search for `Prettier` and make setup for project
        - `Prettier package` path should be set to `[project_path]/node_modules/prettier`
        - `On save` option should be checked
    - In `preferences` on `macOS` or `settings` on `Windows`, enter search for `ESLint` and make setup for project
        - `Manual ESLint configuration ` option should be checked and path should be set to `[project_path]/node_modules/eslint`
        - `Configuration File:` path should be set to `[project_path]/.eslintrc`
        - `Run eslint —fix on save` option should be checked
   - Commit and push these changes

##### References:
[Code quality](/books/frontend/javascript/code-quality)

[EditorConfig](https://editorconfig.org/)


### TypeScript

##### Depends on sections: none

| Package | Version |
|---|---:|
| typescript | 4.0.x |

To add `TypeScript` support do the following:

1. In project root create a new file with a name `tsconfig.json` (manually or via terminal).
    ```bash
    touch tsconfig.json
    ```
1.  Run command
    ```bash
    yarn add --dev typescript @types/react @types/node
    ```
    or
    ```bash
    npm install --save-dev typescript @types/react @types/node
    ```
    to install required packages.
1. Run command
    ```bash
    yarn dev
    ```
    or
    ```bash
    npm run dev
    ```
    and `Next.js` will populate `tsconfig.json` file with default values.
    
    Edit these values or add new based on the project requirements.
    ```bash
    // add these additional rules in `compilerOptions` property
    "experimentalDecorators": true,
    "baseUrl": "src"
    ```
1. In the app rename all `js/jsx` files to `ts/tsx` files.
1. Start adding types.
    ```tsx
   // _app.tsx
    import { AppProps } from 'next/app';
    
    function MyApp({ Component, pageProps }: AppProps): ReactElement { }
    ```
   
##### References:
[TypeScript](https://www.typescriptlang.org/)

[TypeScript in Next.js](https://nextjs.org/docs/basic-features/typescript)


### Localisation

##### Depends on sections: none

| Package | Version |
|---|---:|
| i18next | 19.7.x |
| react-i18next | 11.7.x |

To add localisation support (`i18n`) do the following:
1. Run command
    ```bash
    yarn add i18next react-i18next
    ```
    or
    ```bash
    npm install --save i18next react-i18next
    ```
   to install localisation packages.
1. Run command
    ```bash
    yarn global add polyglot-cli
    ```
    or
    ```bash
    npm install -g polyglot-cli
    ```
    to either install `polyglot-cli` package globally or to update it. If installing for the first time, running
     `polyglot login` is also required. 
1. Run command
    ```bash
    polyglot init
    ```
   and type project number (if project isn't listed contact a team lead).
   
   Select options based on project requirements but for "*Relative path to the translations folder...*" enter:
   ```bash
   src/localisation/locales
   ```
   
   After initialisation, there are 2 approaches for `localisation` and/or `locales` folder generation:
   1. Go to `https://infinum.polyglothq.com/projects/[project_id]/translation_keys`, add a key and run 
       ```bash
       polyglot pull
       ```
       This will create `localisation/locales` with `json` files automatically.
   1. Manually create `localisation` folder and `locales` folder  inside `localisation` and next time translations 
   are pulled, locale files will be available.
1. In `package.json` add following entry at the end of `scripts` property
    ```bash
    "polyglot:pull": "polyglot pull -c .polyglot.json"
    ```
1. In `localisation` folder create file `localisation.ts` and add following content
    ```bash
    import i18n from 'i18next';
    import { initReactI18next } from 'react-i18next';
    
    import en from './locales/en_US.json';
    
    i18n.use(initReactI18next).init({
        resources: {
            // add new locales here; key names depend on Polyglot setup
            'en_US': {
                translations: en,
            },
        },
        defaultNS: 'translations',
        lng: 'en_US',
        fallbackLng: 'en_US',
        interpolation: {
            escapeValue: false,
        },
    });

    export default i18n;
    ```
1. In `_app.tsx` add following import
    ```bash
    import 'localisation/localisation';
    ```

Check if integration is correct by adding translated text in the UI.
```tsx
// example
import React, { ReactElement } from 'react';
import { useTranslation } from 'react-i18next';

export const myComp = () => {
  const { t } = useTranslation();
  
  return (
    <div>
      {t('some.key.registeredInPolyglot')}
      ...
    </div>
  );
};
```

Pulling translations is required when adding new keys in [Infinum Polyglot](https://infinum.polyglothq.com/).

##### References:
[i18n](https://github.com/mashpie/i18n-node)

[i18n in Next.js](https://react.i18next.com/)

[Polyglot CLI](https://github.com/infinum/js-polyglot-cli)

[Infinum Polyglot](https://infinum.polyglothq.com/)


### Theming
##### Depends on sections: TypeScript

| Package | Version |
|---|---:|
| @emotion/core | 10.0.x |
| @emotion/styled | 10.0.x |
| emotion-theming | 10.0.x |
| emotion-normalize | 10.1.x |

To add theming support do the following:
1. Run command
    ```bash
    yarn add @emotion/core @emotion/styled emotion-theming emotion-normalize
    ```
   or
   ```bash
   npm install --save @emotion/core @emotion/styled emotion-theming emotion-normalize
   ```
   to install theming packages.
1. In `src` folder create new folder `theming` and add `Theme.tsx` file. `Theme.tsx` content depends on the project, but boilerplate
below can be copied as a start. Important things are:
    - Export theme and its interface
    - Use `emotionNormalize` in global styles
    - Export `GlobalStyles` component
    - Export default `styled` (this instance should be used further on since it contains theme interface)
    ```tsx
    // e.g.
    import React from 'react';
    import { css, Global } from '@emotion/core';
    import styled, { CreateStyled } from '@emotion/styled';
    import emotionNormalize from 'emotion-normalize';
    
    export interface ITheme {
        colors: {
            primary: string;
        };
        spacings: {
            l: string;
        };
    }
    
    export const theme: ITheme = {
        colors: {
            primary: 'lightblue',
        },
        spacings: {
            l: '1rem',
        },
    };
    
    export const GlobalStyles = () => (
        <Global
            styles={css`
                // e.g. font families registered here
    
                ${emotionNormalize}
    
                * {
                    box-sizing: border-box;
                }
    
                html,
                body {
                    padding: 0;
                    margin: 0;
                    font-family: -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen, Ubuntu, Cantarell, Fira Sans,
                        Droid Sans, Helvetica Neue, sans-serif;
                }
    
                a {
                    color: inherit;
                    text-decoration: none;
                }
    
                header a {
                    margin-right: 10px;
                }
            `}
        />
    );
    
    export default styled as CreateStyled<ITheme>;
    ```
1. In `_app.tsx` add following content
    ```tsx
    // imports
    import { ThemeProvider } from 'emotion-theming';
   
    import { GlobalStyles, theme } from 'theming/Theme';
   
    // in the return statement wrap all with ThemeProvider and add GlobalStyles component inside
    <ThemeProvider theme={theme}>
        <GlobalStyles />
        <Component {...pageProps} />
    </ThemeProvider>
    ```

There are multiple ways how to work with styles in `emotion` and here is an example:
```tsx
import { ReactElement } from 'react';
// 2 lines below are needed for Alternative 2
/** @jsx jsx */
import { jsx } from '@emotion/core';
import { useTheme } from 'emotion-theming';

import styled, { ITheme } from 'theming/Theme';

// Alternative 1
const Select = styled.select`
  padding: ${({ theme }) => theme.spacings.l};
`;

export const CountryChooser = (): ReactElement => {
    const theme = useTheme<ITheme>(); // notice ITheme interface
    
    const handleCountrySelect = (event) => {
      // do something with country
    };

    return (
        // Alternative 2
        <Select css={{ backgroundColor: theme.colors.primary }} onChange={handleCountrySelect}>
            <option value="cro">Croatia</option>
            <option value="slo">Slovenia</option>
        </Select>
    );
};
```

##### References:

[Emotion](https://emotion.sh/docs/introduction)

[Emotion Normalize](https://github.com/infinum/emotion-normalize)


### Tests

##### Depends on sections: TypeScript, Localisation, Theming

| Package | Version |
|---|---:|
| @testing-library/react | 11.0.x |
| @testing-library/jest-dom | 5.11.x |
| @testing-library/user-event | 12.1.x |
| jest | 26.4.x |
| ts-jest | 26.4.x |

To add test support do the following:
1. Run command
    ```bash
    yarn add --dev @types/jest @testing-library/react @testing-library/jest-dom @testing-library/user-event jest ts-jest @babel/plugin-proposal-decorators
    ```
   or
   ```bash
   npm install --save-dev @types/jest @testing-library/react @testing-library/jest-dom @testing-library/user-event jest ts-jest @babel/plugin-proposal-decorators
   ```
   to install Jest - a JS testing framework, React Testing Library - wrapper around React components to test them more easily, and User Event for better event testing in React Testing Utilities.
1. In project root create `.babelrc` file and add following content
    ```json
    {
      "presets": ["next/babel"],
      "plugins": [["@babel/plugin-proposal-decorators", { "legacy": true }]]
    }
    ```
1. In `package.json` add following entries in JSON root 
    ```json
   {
     "scripts": {
        // ...
        "test": "jest --coverage",
        "watch": "jest --watch --coverage"
     },
     "jest": {
        "coveragePathIgnorePatterns": [
            "/tests/",
            "/node_modules/"
        ],
        "moduleFileExtensions": [
            "ts",
            "js",
            "tsx"
        ],
        "modulePaths": [
   		    "<rootDir>/src/"
   		],
        "testRegex": "tests/(.*).test.tsx?$",
        "globals": {
            "ts-jest": {
                "diagnostics": {
                    "warnOnly": true
                }
            }
        },
        "preset": "ts-jest",
        "transform": {
             "^.+\\.tsx?$": "babel-jest"
         },
        "testMatch": null
     }
   }    
    ```
1. Additionally, global configuration and mocking of external modules might be required. Create
`test-utils.tsx` file in `tests` folder and add following content
    ```tsx
    /**
     * Use this file to mock theme, store, localisation, ...
     * Based on: https://testing-library.com/docs/react-testing-library/setup
     */
    import React, { ReactElement } from 'react';
    import { render } from '@testing-library/react';
    import userEvent from '@testing-library/user-event';
    import { ThemeProvider } from 'emotion-theming';

    import localisation from 'localisation/localisation';
    import { theme } from 'theming/Theme';

    /*
    Remove if no need for test keys in tests
    An usecase:
    it('should have ability to change locale', () => {
        const Component = () => (
            <>
                <span>{t('testYolo')}</span>
                <LanguagePicker />
            </>
        );
        
        const { rerender } = render(<Component />);
        
        const option1 = screen.getByTestId('option1') as HTMLOptionElement;
        const option2 = screen.getByTestId('option2') as HTMLOptionElement;
        const translatedText = screen.getByText(t('testYolo'));
        
        expect(translatedText.textContent).toEqual(testYoloEnUsValue);
        expect(option1.selected).toBe(true);
        expect(option2.selected).toBe(false);
        
        userEvent.selectOptions(screen.getByTestId('language-picker'), ['hr_HR']);
        
        rerender(<Component />);
        
        expect(translatedText.textContent).toEqual(testYoloHrHrValue);
        expect(option1.selected).toBe(false);
        expect(option2.selected).toBe(true);
    });
    */
    export const testYoloEnUsValue = 'You only live once';
    export const testYoloHrHrValue = 'Samo jednom se zivi';
    localisation.addResource('en_US', 'translations', 'testYolo', testYoloEnUsValue);
    localisation.addResource('hr_HR', 'translations', 'testYolo', testYoloHrHrValue);   
    
    const wrapper = ({ children }): ReactElement => <ThemeProvider theme={theme}>{children}</ThemeProvider>;
    
    const customRender = (ui: ReactElement, options?: any) => render(ui, { wrapper, ...options });
    
    const customT = (key) => localisation.t(key);
    
    export * from '@testing-library/react';
    
    export { customRender as render, userEvent, customT as t };
    ```
   This configuration is dependent on project configuration and used packages. Add additional configuration if required.

Check if integration is correct by adding test in the project. Read more about where to put test files in [Folder structure](/books/react/folder-structure) page.
```tsx
// example

/* Todos.tsx */

import React, { ReactElement } from 'react';
import { useTranslation } from 'react-i18next';

interface ITodoItem {
    id: string;
    label: string;
    completed: boolean;
}

interface ITodosProps {
  todos: ITodoItem[];
}

export const Todos = ({ todos }: ITodosProps): ReactElement => {
    const { t } = useTranslation();
    
    if (todos.length === 0) {
        return <div>{t('todoListIsEmpty')}</div>;
    }
    
    const hasUncompletedTodo = todos.some(({ completed }) => !completed);
    
    if (!hasUncompletedTodo) {
        return <div>{t('allTodosCompleted')}</div>;
    }
    
    return (
        <div>
            {todos.map(({ id, label, completed }) => (
                <div data-testid="todo" key={id}>
                    <span>{label}</span>
                    <span className={completed ? 'todo__icon--complete' : 'todo__icon--uncomplete'} />
                </div>
            ))}
        </div>
    );
};



/* Todos.test.tsx */

import 'jest';
import React from 'react';

import { render, screen, t } from 'path-to-test-utils';
import { Todos } from 'path-to-todos';

describe('Todos', () => {
    it('should show message for empty list if no todos', () => {
        const emptyTodos = [];
    
        render(<Todos todos={emptyTodos} />);
    
        expect(screen.queryByText(t('todoListIsEmpty'))).not.toBeNull();
    
        expect(screen.queryByText(t('allTodosCompleted'))).toBeNull();
        expect(screen.queryAllByTestId('todo')).toHaveLength(0);
    });
    
    it('should show message for all todos being completed', () => {
        const allTodosCompleted = [
            { id: 1, label: 'Fix window', completed: true },
            { id: 2, label: 'Buy keyboard', completed: true },
        ];
    
        render(<Todos todos={allTodosCompleted} />);
    
        expect(screen.queryByText(t('allTodosCompleted'))).not.toBeNull();
    
        expect(screen.queryByText(t('todoListIsEmpty'))).toBeNull();
        expect(screen.queryAllByTestId('todo')).toHaveLength(0);
    });

    it('should show all todos in correct order', () => {
        const todos = [
            { id: 1, label: 'Fix window', completed: true },
            { id: 2, label: 'Buy keyboard', completed: false },
        ];
    
        render(<Todos todos={todos} />);
    
        const allTodos = screen.queryAllByTestId('todo');
    
        expect(allTodos).toHaveLength(2);
        allTodos.forEach((todo, index) => {
            expect(todo.textContent).toEqual(todos[index].label);
        });
    
        expect(screen.queryByText(t('allTodosCompleted'))).toBeNull();
        expect(screen.queryByText(t('todoListIsEmpty'))).toBeNull();
    });
    
    // ...
});

```

##### References:
[Jest](https://jestjs.io/)

[React Testing Library](https://testing-library.com/)

[React Testing Library User Event](https://testing-library.com/docs/ecosystem-user-event)