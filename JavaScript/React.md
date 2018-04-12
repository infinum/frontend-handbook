# React Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

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

### Component Declaration

For declaring class components:
- use ES2015 **class syntax**
- if you **need** `React.createClass` use [`create-react-class`](https://www.npmjs.com/package/create-react-class) (`createClass` was removed in React 16)

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

Use self-closing tags, separated with an empty space, for elements that don't have children.

``` jsx
// bad
<ChildComponent onClick={this.onClickHandler}></ChildComponent>

// good
<ChildComponent onClick={this.onClickHandler} />
```

If there is more than one prop, separate them with line breaks and also, put opening and closing tags in separate lines.

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

- always wrap *render* method's return statement in parentheses (it's optional if it's returning a single line of JSX)

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

### Passing Props
Use camelCase for prop names and double quotes for JSX attribute values if you are passing them as a *string*.

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

Omit values for boolean props if default value is **true**.

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

Declare functions before passing them as props instead of passing inline anonymous (arrow) functions. They will be needlessy instanced each time, thus effecting performance.

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

Avoid writing any logic inline. Extract it to the outside of the JSX block.

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

### Receiving Props
*Checkout [`Typechecking With PropTypes`](https://reactjs.org/docs/typechecking-with-proptypes.html) in React's official documentation for general info.*

*Note: when using TypeScript with React, use TypeScript's built in types for type checking. For more info, checkout the [`JSX Handbook`](https://www.typescriptlang.org/docs/handbook/jsx.html#type-checking) in TypeScript's official documentation.*

In class components:
- always use prop validation
- use **static member** for declaring prop types and default props (if your transpiler doesn't support static members, set them after class declaration and then export the class; See: ["Exporting Components" section](#exporting-components))
- use *defaultProps* if you need to set default values
- prop types shoud be as detailed as possible 

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
    const {name} = this.props;

    return (
      <div>{name}</div>
    );
  }
}
```

### Using props

If you are using more than one property in a method, preffer accessing them with **object destructuring**.

``` jsx
export class ComponentName extends React.Component {
  // ...
  render() {
    const {id, name, onClickHandler} = this.props;
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

### Exporting Components
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

### Rendering Arrays of Data

For iterating arrays:
- use **map** method to render items
- you should have a unique key for each element
- **don't use array's index as a key**, unless you have no choice (no unique property in each element of the array) then make sure that you do not modify the order of that array

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

### Conditional Rendering
When you want to render either something or nothing, you can use the && operator.
```
// bad
{condition ? (<span />) : null}

// good
{condition && (<span />)}
```

### Binding
- don't bind functions inline or in a class constructor if you have decorators
- use [`developit/decko`](https://github.com/developit/decko) lib's *@bind* decorator for binding
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

### CSS Modules and Components
Each component should have it's own stylesheet if CSS is needed for it.

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
Make sure that your components stay readable and extract complex parts into separate and smaller components. This way you'll avoid huge files. A good rule of thumb for *presentional* components is around 150 lines, and for *containers* and *pages* around 300 loc.

## React + MobX
*For general information about MobX, checkout the official documentation:* [`mobx.js.org`](https://mobx.js.org/).

### Decorators
Use decorators wherever you can if your transpiler supports it, because it makes code more readable and declarative.

### Component Declaration
Wrap components that handle app's state with *observer* from the `mobx-react` library.

``` jsx
import {observer} from 'mobx-react';
// ...
@observer
export class ComponentName extends React.Component {
  // ...
}
```

### Local Component State
If MobX is used for state management:
- preffer using MobX *observable* instead of *this.state* for component's local state; don't use both
- wrap local state in a single observable object named **componentState**

``` jsx
import {observable} from 'mobx';
import {observer} from 'mobx-react';
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

### Computed Values
If you need values derived from observable data, use *@computed*.

``` jsx
import {observable, computed} from 'mobx';
import {observer} from 'mobx-react';
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
Wrap methods and functions that modify state with *@action*. For binding action methods, use the aforementioned *@bind* decorator from the [`developit/decko`](https://github.com/developit/decko) library.

``` jsx
  import {action, observable} from 'mobx';
  import {observer} from 'mobx-react';
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

### Async Await
In actions implemented as *async* functions, after each *await*, any code that modifies the state should be wrapped in a *runInAction* utility.

``` jsx
  import {action, observable, runInAction} from 'mobx';
  import {observer} from 'mobx-react';
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
