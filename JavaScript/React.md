# React Guidelines and Best Practices

_Guides are not rules and should not be followed blindly. Use your head and think._

If you're using Prettier, there are some styling rules here that will be overriden by the formatter. In that case, ignore the styleguides mentioned here.

### File Organization and Naming

As it is a good practice to divide React components as container and presentational components, you should place them in separate folders (this example uses **containers** and **components** for folder naming, but there are other naming conventions, depending on the use case and preference of developers).

```
...
├─ components
├─ containers
...
```

Use **camelCase** for general folder naming and **PascalCase** for React components, their stylesheet files and their containing folders.

```
...
├─ components
⎮   ├─ LoginContent
⎮   ⎮   └─ LoginContent.js
⎮   ⎮   └─ LoginContent.scss
⎮   └─ LoginHeader
⎮       └─ LoginHeader.js
⎮       └─ LoginHeader.scss
├─ containers
⎮   └─ Login
⎮       └─ Login.js
⎮       └─ Login.scss
...
```

### File importing

JS Imports should be grouped in three categories:

1. 3rd party dependencies
2. Project modules
3. Assets

Example:

```javascript
import { css } from 'emotion';
import React from 'react';

import { Home } from '../containers/Home';
import { User } from '../state/models/User';

import logo from '../assets/logo.svg';
```

### Component Declaration

For declaring class components:

- use functional components for presentational components
- use ES2015 class syntax for containers

```jsx
// bad
export const ComponentName = React.createClass({
  // ...
});

// bad
export class ComponentName extends React.Component {
  // ...
}

// good
export const ComponentName = (props) => {
  // ...
}

// good
export class ContainerName extends React.Component {
  // ...
}
```

### Formatting

Use self-closing tags, separated with an empty space, for elements that don't have children.

```jsx
// bad
<ChildComponent onClick={this.onClickHandler}></ChildComponent>

// good
<ChildComponent onClick={this.onClickHandler} />
```

If there is more than one prop, separate them with line breaks and also, put opening and closing tags in separate lines.

```jsx
// bad
<ChildComponent className={styles.specialStyling}
  title={this.title}
  onClick={this.onClickHandler}
/>

// good
<ChildComponent
  className={styles.specialStyling}
  title={this.title}
  onClick={this.onClickHandler}
/>

// good
<OuterChildComponent
  className={styles.outerChild}
  title={this.title}
  onClick={this.onClickHandler}
>
  <InnerChildComponent className={styles.innerChild} />
</OuterChildComponent>
```

- always wrap _render_ method's return statement in parentheses (it's optional if it's returning a single line of JSX)

```jsx
// bad
render() {
  // ...
  return <div>
      <ChildComponent />
    </div>;
}

// bad
render() {
  // ...
  return
    <div>
      <ChildComponent />
    </div>;
}

// good
render() {
  // ...
  return (
    <div>
      <ChildComponent />
    </div>
  );
}

// good
render() {
  // ...
  return <ChildComponent />;
}

// good
render() {
  // ...
  return (
    <ChildComponent />
  );
}

// good
const Component = () => (
  <ChildComponent />
);
```

### Passing Props

Use camelCase for prop names and double quotes for JSX attribute values if you are passing them as a _string_.

```jsx
export class ComponentName extends React.Component {
  // ...
  render() {
    return <ChildComponent defaultTitle="Some Title" title={this.props.title} />;
  }
}
```

Omit values for boolean props if default value is **true**.

```jsx
// bad
<ChildComponent
  onClick={this.onClickHandler}
  isPropActive={true}
/>

// good
<ChildComponent
  onClick={this.onClickHandler}
  isPropActive
/>
```

When using class components, declare functions before passing them as props instead of passing inline anonymous (arrow) functions. They will be needlessy instanced each time, thus effecting performance.

```jsx
// bad
<ChildComponent onClick={() => { /* ... */ } />

// good
class ComponentName extends React.Component {
  onClickHandler() {
    // ...
  }

  render () {
    return (
      <ChildComponent onClick={this.onClickHandler} />
    );
  }
}
```

Avoid writing any logic inline. Extract it to the outside of the JSX block.

```jsx
export class ComponentName extends React.Component {
  // ...

  render() {
    const title = specialTitle || defaultTitle;

    return <ChildComponent title={title} />;
  }
}
```

### Receiving Props

_Checkout [`Typechecking With PropTypes`](https://reactjs.org/docs/typechecking-with-proptypes.html) in React's official documentation for general info._

_Note: when using TypeScript with React, use TypeScript's built in types for type checking. For more info, checkout the [`JSX Handbook`](https://www.typescriptlang.org/docs/handbook/jsx.html#type-checking) in TypeScript's official documentation._

- always use prop validation
- use **static member** for declaring prop types and default props (if your transpiler doesn't support static members, set them after class declaration and then export the class; See: ["Exporting Components" section](#exporting-components))
- use _defaultProps_ if you need to set default values
- prop types shoud be as detailed as possible

```jsx
// Class component
export default class ComponentName extends React.Component {
  static propTypes = {
    heading: PropTypes.string,
    users: PropTypes.arrayOf(
      PropTypes.shape({
        id: PropTypes.string.isRequired,
        firstName: PropTypes.string.isRequired,
        lastName: PropTypes.string.isRequired,
        age: PropTypes.number,
      }),
    ).isRequired,
  };

  static defaultProps = {
    heading: 'Default Heading',
  };

  // ...

  render() {
    const { name } = this.props;

    return <div>{name}</div>;
  }
}

// Functional component
export const ComponentName = ({ name }) => {
  return <div>{name}</div>;
};

ComponentName.propTypes = {
  heading: PropTypes.string,
  users: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.string.isRequired,
      firstName: PropTypes.string.isRequired,
      lastName: PropTypes.string.isRequired,
      age: PropTypes.number,
    }),
  ).isRequired,
};

ComponentName.defaultProps = {
  heading: 'Default Heading',
};
```

### Using props

If you are using more than one property in a method, preffer accessing them with **object destructuring**.

```jsx
export class ComponentName extends React.Component {
  // ...
  render() {
    const { id, name, onClickHandler } = this.props;
    // ...
  }
}
```

### Importing React

```jsx
import React from 'react';

export class ComponentName extends React.Component {
  // ...
}
```

### Exporting Components

- each React component file should have a single exported component
- don't use default exports, always use named exports instead

Exporting class components:

```jsx
// bad
export default class ComponentName extends React.Component {
  // ...
}

// good
export class ComponentName extends React.Component {
  // ...
}
```

### Rendering Arrays of Data

For iterating arrays:

- use **map** method to render items
- you should have a unique key for each element
- **don't use array's index as a key**, unless you have no choice (no unique property in each element of the array) then make sure that you do not modify the order of that array

```jsx
render() {
  // ...
  return (
    <ul>
      {
        booksArray.map((book) => (
          <li key={book.id}>
            <a href={book.href}>{book.title}</a>
          </li>
        ));
      }
    </ul>
  );
}
```

### Conditional Rendering

When you want to render either something or nothing, you can use the && operator.
Make sure condition is boolean, if not cast it to boolean.

```javascript
// bad
{
  condition ? <div>// ...some stuff</div> : null;
}

// bad
const condition = 1;
{
  condition && <div>// ...some stuff</div>;
}

// good
const condition = 1;
{
  Boolean(condition) && <div>// ...some stuff</div>;
}

// good
{
  condition && <div>// ...some stuff</div>;
}
```

### Binding

- don't bind functions inline or in a class constructor if you have decorators
- use [`developit/decko`](https://github.com/developit/decko) lib's _@bind_ decorator for binding
- don't use arrow functions as class properties

```jsx
// bad
export class ComponentName extends React.Component {
  onClickHandler(e) {/**/}

  render() {
    return (
      <ChildComponent onClick={this.onClickHandler.bind(this)} />
    );
  }
}

// bad if you have decorators
export class ComponentName extends React.Component {
  constructor(args) {
    super(args);
    this.onClickHandler = this.onClickHandler.bind(this);
  }

  onClickHandler(e) {/**/}

  render() {
    return (
      <ChildComponent onClick={this.onClickHandler} />
    );
  }
}

// bad
export class ComponentName extends React.Component {
  onClickHandler = () => {}

  render() {
    return (
      <ChildComponent onClick={this.onClickHandler} />
    );
  }
}

// good
export class ComponentName extends React.Component {

  @bind
  onClickHandler(e) {/**/}

  render() {
    return (
      <ChildComponent onClick={this.onClickHandler} />
    );
  }
}
```

### CSS Modules and Components

Each component should have it's own stylesheet if CSS is needed for it. The module filename should be suffixed with either `.module.scss` or `.module.css`.

```
...
├─ Login
⎮   └─ Login.js
⎮   └─ Login.module.scss
...
```

Importing:

```jsx
// ...
import styles from './ComponentName.module.scss';

export class ComponentName extends React.Component {
  // ...
  render() {
    return <Heading className={styles.specialHeading} />;
  }
}
```

If you need to pass more than one CSS class name to a component or want to use conditional classes, use the [`classnames`](https://www.npmjs.com/package/classnames) library.

```jsx
import classNames from 'classnames';
// ...
import styles from './ComponentName.module.scss';

export class ComponentName extends React.Component {
  // ...
  render() {
    return (
      <Heading
        className={classNames(styles.largeHeading, {
          [styles.importantHeading]: isImportant,
        })}
      />
    );
  }
}
```

## Maximum file line number

Make sure that your components stay readable and extract complex parts into separate and smaller components. This way you'll avoid huge files. A good rule of thumb for _presentional_ components is around 150 lines, and for _containers_ and _pages_ around 300 LOC.

## React + MobX

_For general information about MobX, checkout the official documentation:_ [`mobx.js.org`](https://mobx.js.org/).

### Decorators

Use decorators wherever you can if your transpiler supports it, because it makes code more readable and declarative.

### Component Declaration

Wrap components that handle app's state with _observer_ from the `mobx-react` library.

```jsx
import { observer } from 'mobx-react';
// ...
@observer
export class ComponentName extends React.Component {
  // ...
}

export const ComponentName = observer((props) => {
  // ...
});
```

### Local Component State

If MobX is used for state management:

- prefer using MobX _observable_ instead of _this.state_ for component's local state; don't use both

#### Class components

- wrap local state in a single observable object named **componentState**

```jsx
import { observable } from 'mobx';
import { observer } from 'mobx-react';
// ...
@observer
export class ComponentName extends React.Component {
  // ...
  @observable componentState = {
    selectedTitle: 'React+MobX',
  };

  render() {
    // ...
    return (
      <div>
        // ...
        <div>{this.componentState.selectedTitle}</div>
      </div>
    );
  }
}
```

#### Functional components

In functional components it's fine to use either `useState` or `useLocalStore`, but don't mix them together - choose one for the project and then stick with it.
`useLocalStore` might be simpler in the short run, but it might not work with new React features like concurrent rendering.

```jsx
import { observer, useLocalStore } from 'mobx-react';
// ...
export const ComponentName = observer(() => {
  const state = useLocalStore(() => ({
    selectedTitle: 'React+MobX',
  }));

  return (
    <div>
      // ...
      <div>{state.selectedTitle}</div>
    </div>
  );
});
```

### Computed Values

If you need values derived from observable data, use _@computed_ with class components or observable getters for functional components:

```jsx
import { observable, computed } from 'mobx';
import { observer } from 'mobx-react';
// ...

@observer
export class ComponentName extends React.Component {
  // ...
  @observable componentState = {
    firstName: 'John',
    lastName: 'Doe',
  };

  @computed get fullName() {
    return `${this.componentState.firstName} ${this.componentState.lastName}`;
  }

  render() {
    return <div>{this.fullName}</div>;
  }
}

export const ComponentName = observer(() => {
  const state = useLocalStore(() => ({
    firstName: 'John',
    lastName: 'Doe',
    get fullName() {
      return `${state.firstName} ${state.lastName}`;
    }
  }));

  return (
    <div>
      // ...
      <div>{state.fullName}</div>
    </div>
  );
});
```

### Actions

Wrap methods and functions that modify state with _@action_. For binding action methods, use the aforementioned _@bind_ decorator from the [`developit/decko`](https://github.com/developit/decko) library (for class components).

```jsx
  import { action, observable } from 'mobx';
  import { observer, useLocalState } from 'mobx-react';
// ...
@observer
export class ComponentName extends React.Component {
  // ...
  @observable componentState = {
    counter: 0,
  };

  public componentWillMount() {
    // ...
  }

  // Note: lifecycle methods also have to be wrapped as "actions" if they modify state
  @action public componentDidMount() {
    // ...
  }

  @bind
  @action
  increaseCounter() {
    this.componentState.counter++;
  }

  render() {
    return (
      <div>
        <div>{this.componentState.counter}</div>
        <button onClick={this.increaseCounter}>Increase by One</button>
      </div>
    );
  }
}

export const  ComponentName = observer(() => {
  const state = useLocalState(() => ({
    counter: 0,
  }));

  const increaseCounter = React.useCallback(action(() => {
    state.counter++;
  }));

  return (
    <div>
      <div>{state.counter}</div>
      <button onClick={increaseCounter}>Increase by One</button>
    </div>
  );
});
```

### Async Await

In actions implemented as _async_ functions, after each _await_, any code that modifies the state should be wrapped in a _runInAction_ utility.

```jsx
import { action, observable, runInAction } from 'mobx';
import { observer } from 'mobx-react';
// ...
export class ComponentName extends React.Component {
  // ...
  @observable componentState = {
    userData: null,
  };

  @bind
  async saveToLocalState() {
    const userData = await getUserData();
    // ...
    runInAction(() => (this.componentState.userData = userData));
  }

  render() {
    /* ... */
  }
}
```

For async operations in functional components, the simplest way is to use the [`useAsync`](https://github.com/streamich/react-use/blob/master/docs/useAsync.md) hook from the [`react-use`](https://github.com/streamich/react-use) package.

```jsx
import { useAsync } from 'react-use';

async function someAsyncOperation() {
  // ...
}

export const ComponentName = () => {
  const { loading, error, value } = useAsync(someAsyncOperation, []);

  if (loading) {
    return <div>Loading...</div>;
  } else if (error) {
    return <div>Error</div>;
  } else {
    return <div>Here is some data: {value}</div>;
  }
};
```

### Fragments

When using React fragments, always use the explicit syntax with either `<Fragment></Fragment>` or `<React.Fragment></React.Fragment>`.
