# React Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

## File Organization and Naming
- as it is a good practice to divide React components as container and presentational components, you should place them in separate folders (example: **containers** and **components**)

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

## Component Declaration
- don't use React.createClass
- use ES2015 **class syntax** for component declaration

``` javascript
// bad
const ComponentName = React.createClass({
  // ...
  render () {
    return (
      <div>
        {this.componentState.message}
      </div>
    );
  }
});
    
// good
import React, {Component} from 'react';

class ComponentName extends Component {
  // ...
  render () {
    return (
      <div>
        {this.componentState.message}
      </div>
    );
  }
}
    
export default ComponentName;
```

- if components don't have state, use **pure functions**:

``` javascript
import React from 'react';

const ComponentName = ({message}) => {
  return (
    <div>
      {message}
    </div>
  );
};
```

## Formatting
- use self-closing tags if the element doesn't have children
- put **single spaces** before self-closing tags

``` javascript
// bad
<ChildComponent onClick={this.onClickHandler}><ChildComponent/>
}

// good
<ChildComponent onClick={this.onClickHandler} />
```

- if props don't fit on one line separate them with line breaks and also, put opening and closing tags in separate lines

``` javascript
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

- always wrap *render* method's return statement in parentheses (with the exception of returning a single line)

``` javascript
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
    <div>
      <ChildComponent />
    </div>
  );
}
```


## Imports
- import specific React modules, such as **Component**, to avoid accessing them with the React namespace every time

``` javascript
// bad
import React from 'react';

class ComponentName extends React.Component {
  // ...
}

// good
import React, {Component} from 'react';

class ComponentName extends Component {
  // ...
}
    
export default ComponentName;
```

## Exports
- each React component file should have a single exported component
 
``` javascript
export default class ComponentName extends Component {
  // ...
}
    
// or

class ComponentName extends Component {
  // ...
}
    
export default ComponentName;
```

## Passing Props
- use camelCase for prop names
- use double quotes for JSX attribute values if you are passing them as a *string*

``` javascript
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

- declare functions before passing them as props instead of using anonymous functions:

``` javascript
// bad
<ChildComponent onClick={(event) => {console.log(event)}} />
    
// good
class ComponentName extends Component {
    // ...
    onClickHandler(event) {
        console.log(event);
    }
    // ...
    render () {
        return (
          <ChildComponent onClick={this.onClickHandler} />
        );
    }
}
```

- omit values for boolean props if default value is **true**:

``` javascript
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

## Receiving Props
- always use type checking for props
- use `defaultProps` if you need to set default values in cases when the prop value isn't received

``` javascript
import React, {Component} from 'react';
import PropTypes from 'prop-types';

class ComponentName extends Component {
    // ...
    render() {
        return (
          <div>{this.props.name}</div>
        );
      }
}

ComponentName.propTypes = {
    id: PropTypes.string.isRequired,
    name: PropTypes.string,
};

ComponentName.defaultProps = {
    name: 'Default',
};
```

- you can also declare property types as a **static member** inside a class:

``` javascript
class ComponentName extends Component {
    static propTypes = {
        id: PropTypes.string.isRequired,
        name: PropTypes.string,
    }
    // ...
    render() {
        return (
          <div>{this.props.name}</div>
        );
      }
}
```

- inside the *render* method, preffer accessing the props using **object destructuring**:

``` javascript
import React, {Component} from 'react';
import PropTypes from 'prop-types';

class ComponentName extends Component {
    // ...
    render() {
        const {id, name} = this.props;
        // ...
        return (
          <div>{name}</div>
        );
    }
}
```

- when using TypeScript with React, use built in types instead of React's `prop-types` package
- add `?` at the end of parameters that are optional

``` javascript
class componentName extends Component<{
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}, {}> {
  // ...
}
```

- if you want a cleaner component declaration, or if a component receives a great number of props, use an interface
- use **"I"** prefixes in interface names


``` javascript
interface IComponentProps {
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}

class ComponentName extends Component<IComponentProps, {}> {
  // ...
}

// in pure function components
interface IComponentProps {
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}

function ComponentName(props: IComponentProps) {
  return (
    // ...
  );
}
```


## Conditional Rendering
- use ternary operators for conditional rendering
- return `null` if the condition is not met

``` javascript
render() {
    // ...
    return (
        <AlwaysShownComponent />
        
        {
            isConditionMet
            ? <ConditionallyShownComponent />
            : null;
        }
    );
}
```

## Rendering Arrays of Data
- use array's **map** method to render items from an array
- you should have a unique key for each element
- **don't use array's index as a key**

``` javascript
render() {
    // ...
    return (
        <ul>
            {
                booksArray.map((book) => {
                    <ListItem key={book.id}>{book.title}</ListItem> 
                }); 
            }
        </ul>
    );
}
```

## Binding
- bind component's methods inside it's constructor


``` javascript
// not that good
class ComponentName extends Component {
    // ...
    
    onClickHandler(e) {
        // ...
    }
    
    render() {
        return (
            <ChildComponent
                onClick={this.onCLickHandler.bind(this)}
            />
        );
    }
}

// good
class ComponentName extends Component {
    // ...
    constructor(props) {
        super(props);
        this.onClickHandler = this.onClickHandler.bind(this);
    } 
    
    onClickHandler(e) {
        // ...
    }
    
    render() {
        return (
            <ChildComponent
                onClick={this.onCLickHandler}
            />
        );
    }
}
```

## CSS Modules and Components
- each component should have it's own stylesheet if CSS is needed for it

```
...
├─ Login
⎮   └─ Login.js
⎮   └─ Login.js
...
```

- importing:

``` javascript
// ...
import * as styles from 'ComponentName.scss';

class ComponentName extends Component {
    // ...
    render () {
        return (
            <Heading
                className={styles.specialHeading}
            />
        );
    }
}
```

- if you need to pass more than one CSS class name to a component or want to use conditional classes, use the [`classnames`](https://www.npmjs.com/package/classnames) library

``` javascript
import classNames from 'classnames';
// ...
import * as styles from 'ComponentName.scss';

class ComponentName extends Component {
    // ...
    render () {
        return (
            <Heading
                className={classNames(styles.specialHeading, styles.largeHeading)}
            />
        );
    }
}
```
