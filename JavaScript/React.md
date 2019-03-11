# React guidelines and best practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

### File organization and naming
As it is good practice to divide React components into container and presentational components, you should place them in separate folders (this example uses **containers** and **components** for folder naming, but there are other naming conventions, depending on the use case and preference of developers).

```
...
├─ components
├─ containers 
...
```

Use **camelCase** for general folder naming, and **PascalCase** for React components, their style sheet files, and their containing folders.

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

### Importing files

JS Imports should be grouped in three categories:
1. 3rd-party dependencies
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

### Component declaration

For declaring class components:
- use ES2015 **class syntax**
- if you **need** `React.createClass`, use [`create-react-class`](https://www.npmjs.com/package/create-react-class) (`createClass` was removed in React 16)

``` jsx
// bad
export const ComponentName = React.createClass({
  // ...
});

// good
export class ComponentName extends React.Component {
  // ...
}
```

If components are stateless (they don't change state), use **pure components**:

``` jsx
export class FunctionalComponent extends React.PureComponent {
  // ...
}
```

### Formatting

Use self-closing tags separated with an empty space for elements that don't have children.

``` jsx
// bad
<ChildComponent onClick={this.onClickHandler}></ChildComponent>

// good
<ChildComponent onClick={this.onClickHandler} />
```

If there is more than one prop, separate them with line breaks. Also, put opening and closing tags in separate lines.

``` jsx
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

- always wrap the *render* method's return statement in parentheses (it's optional if it's returning a single line of JSX)

``` jsx
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
```

### Passing props
Use camelCase for prop names, and double quotes for JSX attribute values if you are passing them as a *string*.

``` jsx
export class ComponentName extends React.Component {
  // ...
  render () {
    return (
      <ChildComponent
        defaultTitle="Some Title"
        title={this.props.title}
      />
    );
  }
}
```

Omit values for boolean props if the default value is **true**.

``` jsx
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

Declare functions before passing them as props instead of passing inline anonymous (arrow) functions. They will be needlessly instanced every time, thus affecting performance.

``` jsx
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

Avoid writing any logic in line. Extract it to the outside of the JSX block.

``` jsx
export class ComponentName extends React.Component {
  // ...

  render () {
    const title = specialTitle || defaultTitle;
    
    return (
      <ChildComponent title={title} />
    );
  }
}
```

### Receiving props
*Check out [`Typechecking With PropTypes`](https://reactjs.org/docs/typechecking-with-proptypes.html) in React's official documentation for general info.*

*Note: when using TypeScript with React, use TypeScript's built-in types for type checking. For more info, check out the [`JSX Handbook`](https://www.typescriptlang.org/docs/handbook/jsx.html#type-checking) in TypeScript's official documentation.*

In class components:
- always use prop validation
- use a **static member** for declaring prop types and default props (if your transpiler doesn't support static members, set them after the class declaration and then export the class; see: ["Exporting Components" section](#exporting-components))
- use *defaultProps* if you need to set default values
- prop types should be as detailed as possible 

``` jsx
export default class ComponentName extends React.Component {
  static propTypes = {
    heading: PropTypes.string,
    users: PropTypes.arrayOf(
      PropTypes.shape({
        id: PropTypes.string.isRequired,
        firstName: PropTypes.string.isRequired,
        lastName: PropTypes.string.isRequired,
        age: PropTypes.number
      })
    ).isRequired,
  };

  static defaultProps = {
    heading: 'Default Heading',
  };

  // ...

  render() {
    const { name } = this.props;

    return (
      <div>{name}</div>
    );
  }
}
```

### Using props

If you are using more than one property in a method, you should access them with **object destructuring**.

``` jsx
export class ComponentName extends React.Component {
  // ...
  render() {
    const { id, name, onClickHandler } = this.props;
    // ...
  }
}
```

### Importing React
``` jsx
import React from 'react';

export class ComponentName extends React.Component {
  // ...
}
```

### Exporting components
- each React component file should have a single exported component
- don't use default exports, always use named exports instead
 
Exporting class components:

``` jsx
// bad
export default class ComponentName extends React.Component {
  // ...
}

// good
export class ComponentName extends React.Component {
  // ...
}
```

### Rendering arrays of data

For iterating arrays:
- use the **map** method to render items
- you should have a unique key for each element
- **don't use an array index as a key**, unless you have no choice (no unique property in each element of the array); in that case, make sure that you do not modify the order of that array

``` jsx
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

### Conditional rendering
When you want to render either something or nothing, you can use the && operator.
Make sure the condition is boolean. If it is not, cast it to boolean.

```javascript
// bad
{
  condition ? 
  <div>
    // ...some stuff
  </div> : 
  null
}

// good
{condition && 
  <div>
    // ...some stuff
  </div>
}
```

### Binding
- don't bind functions in line or in a class constructor if you have decorators
- use the [`developit/decko`](https://github.com/developit/decko) lib's *@bind* decorator for binding
- don't use arrow functions as class properties

``` jsx
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

### CSS modules and components
Each component should have its own style sheet if CSS is necessary for it.

```
...
├─ Login
⎮   └─ Login.js
⎮   └─ Login.scss
...
```

Importing:

``` jsx
// ...
import styles from './ComponentName.scss';

export class ComponentName extends React.Component {
  // ...
  render() {
    return (
      <Heading className={styles.specialHeading} />
    );
  }
}
```

If you need to pass more than one CSS class name to a component or want to use conditional classes, use the [`classnames`](https://www.npmjs.com/package/classnames) library.

``` jsx
import classNames from 'classnames';
// ...
import styles from './ComponentName.scss';

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
Make sure that your components stay readable and extract complex parts into separate and smaller components. In this way, you'll avoid huge files. A good rule of thumb for *presentational* components is around 150 lines, and around 300 loc for *containers* and *pages*.

## React + MobX
*For general information about MobX, check out the official documentation:* [`mobx.js.org`](https://mobx.js.org/).

### Decorators
Use decorators wherever you can if your transpiler supports it because it makes code more readable and declarative.

### Component declaration
Wrap components that handle the app's state with an *observer* from the `mobx-react` library.

``` jsx
import { observer } from 'mobx-react';
// ...
@observer
export class ComponentName extends React.Component {
  // ...
}
```

### Local component state
If MobX is used for state management:
- prefer using a MobX *observable* instead of *this.state* for a component's local state; don't use both
- wrap the local state in a single observable object named **componentState**

``` jsx
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

### Computed values
If you need values derived from observable data, use *@computed*.

``` jsx
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
    return (
      <div>{this.fullName}</div>
    );
  }
}
```

### Actions
Wrap methods and functions that modify state with *@action*. Use the aforementioned *@bind* decorator from the [`developit/decko`](https://github.com/developit/decko) library for binding action methods.

``` jsx
  import { action, observable } from 'mobx';
  import { observer } from 'mobx-react';
// ...
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
```

### Async await
In actions implemented as *async* functions, after each *await*, any code that modifies the state should be wrapped in a *runInAction* utility.

``` jsx
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

  render() { /* ... */ }
}
```

### Fragments
When using React fragments, always use explicit syntax with either `<Fragment></Fragment>` or `<React.Fragment></React.Fragment>`.
