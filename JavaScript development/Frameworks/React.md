# React Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

// TODO: find a consistant example theme and rewrite all

### File Organization and Naming
As it is a good practice to divide React components as container and presentational components, you should place them in separate folders (example: **containers** and **components**):

```
...
├─ components
├─ containers 
...
```

- use **camelCase** for general folder naming and **PascalCase** for React components, their stylesheet files and their containing folders 
- each component type's root folder should have an **index.js** file which exports all components, so you can later import them from a single place

```
...
├─ components
⎮   ├─ LoginContent
⎮   ⎮   └─ LoginContent.js
⎮   ⎮   └─ LoginContent.scss
⎮   ├─ LoginHeader
⎮   ⎮   └─ LoginHeader.js
⎮   ⎮   └─ LoginHeader.scss
⎮   └─ index.js
├─ containers 
⎮   ├─ Login
⎮   ⎮   └─ Login.js
⎮   ⎮   └─ Login.scss 
⎮   └─ index.js
...
```

### Component Declaration

For declaring class components:
- use ES2015 **class syntax** for component declaration
- don't use *React.createClass*

``` jsx
// bad
const ComponentName = React.createClass({
  // ...
});

// good
export default class ComponentName extends Component {
  // ...
}
```

If components don't have and don't change state, use **pure functions**:

``` jsx
function PureComponent({message}) => {
  // ...
};
```

### Formatting

Use the self-closing tag, separated with an empty space, if the element doesn't have children.

``` jsx
// bad
<ChildComponent onClick={this.onClickHandler}><ChildComponent/>
}

// good
<ChildComponent onClick={this.onClickHandler} />
```

If props don't fit on one line separate them with line breaks and also, put opening and closing tags in separate lines.

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
}
```

- always wrap *render* method's return statement in parentheses (it's optional while returning a single line of JSX)

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
render() {
  // ...
  return (
    <div>
      <ChildComponent />
    </div>
  );
}
```

### Passing Props
Use camelCase for prop names and double quotes for JSX attribute values if you are passing them as a *string*.

``` jsx
class ComponentName extends Component {
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

Declare functions before passing them as props instead of passing inline anonymous (arrow) functions. They will be needlessy re-rendered each time, thus effecting performance.

``` jsx
// bad
<ChildComponent onClick={() => { /* ... */ } />
    
// good
class ComponentName extends Component {
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

### Receiving Props
*Checkout [`prop-types`](https://github.com/facebook/prop-types) library.*

In class component:
- always use prop validation
- use **static member** for declaring prop types and default props (if your transpiler doesn't support static members, set them after class declaration and then export the class; See: ["Exports" section](#Exports))
- use *defaultProps* if you need to set default values

``` jsx
export default class ComponentName extends Component {
  static propTypes = {
    id: PropTypes.string.isRequired,
    name: PropTypes.string,
  };

  static defaultProps = {
    name: 'John Doe',
  }
  // ...
  render() {
    const {name} = this.props;

    return (
      <div>{name}</div>
    );
  }
}
```

In stateless components:
- use PropTypes for type checking but use native JS for default values

``` jsx
function ComponentName({id, name = 'John Doe'}) {
  // ...
};

ComponentName.propTypes = {
  id: PropTypes.number.isRequired,
  name: PropTypes.string
}
```

### Using props

If you are using more than one property in a method, preffer accessing them with **object destructuring**.

``` jsx
class ComponentName extends Component {
  // ...
  render() {
    const {id, name, onClickHandler} = this.props;
    // ...
  }
}
```

### Imports
Import specific React modules, such as **Component**, to avoid accessing them with the React namespace every time.

``` jsx
// bad
import React from 'react';

class ComponentName extends React.Component {
  // ...
}

// good
import React, {Component} from 'react';

export default class ComponentName extends Component {
  // ...
}
```

### Exports
Each React component file should have a single exported component.
 
``` jsx
// class component
export default class ComponentName extends Component {
  // ...
}

// class component when static members are not supported
class ComponentName extends Component {
  // ...
}

ComponentName.propTypes = {
  // ..
};

export default ComponentName;

// stateless components
function PureComponent(/* props */) {
  // ...
}

PureComponent.propTypes = {
  // ...
};

export default PureComponent;

// stateless components with no prop validation
export default () => (/* ... */);

```

### Rendering Arrays of Data

For iterating arrays:
- use **map** method to render items
- you should have a unique key for each element
- **don't use array's index as a key**

``` jsx
render() {
  // ...
  return (
    <ul>
      {
        booksArray.map((book) => {
          <ListItem key={book.id} title={book.title} /> 
        }); 
      }
    </ul>
  );
}
```

### Binding
Don't bind functions inline, bind them in the class constructor.
``` jsx
// bad
class ComponentName extends Component {
  onClickHandler() {
    // ...
  }

  render() {
    return (
      <ChildComponent onClick={this.onClickHandler.bind(this)} />
    );
  }
}
    
// good
class ComponentName extends Component {
  constructor() {
    super();

    this.onClickHandler = this.onClickHandler.bind(this);
  }

  onClickHandler() {
    // ...
  }

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
⎮   └─ Login.js
...
```

Importing:

``` jsx
// ...
import * as styles from './ComponentName.scss';

class ComponentName extends Component {
  // ...
  render() {
    return (
      <Heading
        className={styles.specialHeading}
      />
    );
  }
}
```

If you need to pass more than one CSS class name to a component or want to use conditional classes, use the [`classnames`](https://www.npmjs.com/package/classnames) library.

``` jsx
import classNames from 'classnames';
// ...
import * as styles from './ComponentName.scss';

class ComponentName extends Component {
  // ...
  render() {
    return (
      <Heading
        className={classNames(styles.specialHeading, styles.largeHeading)}
      />
    );
  }
}
```

## React + MobX
*For general information about MobX, checkout* [`mobx.js.org`](https://mobx.js.org/).

## Decorators
Use decorators wherever you can because it makes code more readable and declarative.

### Component Declaration
Wrap components that handle app's state with *observer* from the `mobx-react` library.

``` jsx
import {observer} from 'mobx-react';
// ...
@observer
class ComponentName extends Component {
  // ...
}
```

### Local Component State
If MobX is used for state management:
- preffer using MobX *observable* instead of *this.state* for component's local state; don't use both
- wrap local state in a single object just as it would be in the built in state

``` jsx
import {observable} from 'mobx';
import {observer} from 'mobx-react';
// ...
@observer
class ComponentName extends Component {
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
class ComponentName extends Component {
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
Wrap methods and functions that modify state with *@action*.

``` jsx
  import {action, observable} from 'mobx';
  import {observer} from 'mobx-react';
// ...
class ComponentName extends Component {
  constructor() {
    super();

    this.increaseCounter = this.increaseCounter.bind(this);
  }

  @observable componentState = {
    counter: 0,
  };

  @action increaseCounter() {
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

For binding action methods, you can use *bound* modifier.
``` jsx
  import {action, observable} from 'mobx';
  import {observer} from 'mobx-react';
// ...
class ComponentName extends Component {
  // ...
  @observable componentState = {
    counter: 0,
  };

  @action.bound increaseCounter() {
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
In actions implemented as async functions, after each await, any code that modifies the state should be wrapped in a *runInAction* utility. This is because *async await* is just syntax sugar, and in reallity, a new function is run after each await.

``` jsx
  import {action, observable, runInAction} from 'mobx';
  import {observer} from 'mobx-react';
// ...
class ComponentName extends Component {
  // ...
  @observable componentState = {
    userData: null,
  };

  @action.bound async saveToLocalState() {
    const userData = await getUserData();
    // ...
    runInAction(() => {this.componentState.userData = userData});
  }

  render() { /* ... */ }
}
```

// TODO: add additional MobX stuff if needed


// TODO: remove all TODO comments