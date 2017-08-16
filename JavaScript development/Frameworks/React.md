
### File organization and naming
- as it is a good practice to divide React components as controllers and presentational components, you should place them in separate folders (example: **containers** and **components**)

```
...
├── components
├── containers 
...
```

- use **camelCase** for general folder naming and **PascalCase** for React components, their stylesheet files and their containing folders 
- each component type's root folder should have an **index.jsx** file which exports all components, so you can later import them from a single place

```
...
├── components
    ├── LoginContent
        └── LoginContent.jsx
        └── LoginContent.scss
    ├── LoginHeader
        └── LoginHeader.jsx
        └── LoginHeader.scss
    └── index.jsx
├── containers 
    ├── Login
        └── Login.jsx
        └── Login.scss 
    └── index.jsx
...
```

### Component Declaration
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

### Formatting
- use self-closing tags if the element doesn't have children
- put a **single space** before the self-closing tag

``` javascript
// bad
<ChildComponent onClick={this.onClickHandler}></>
}

// good
<ChildComponent onClick={this.onClickHandler} />
```

- if props don't fit on one line separate them with line breaks
- put opening and closing tags in separate lines

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

// ...
render() {
  // ...
  return (
    <div>
      <ChildComponent />
    </div>
  );
}
```


### Imports
- import specific React modules, such as **Component**, to avoid accessing them with the React namespace every time

``` javascript
// not that good
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

### Exports
- each React file should have a single exported component
 
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

### Passing Props

- use camelCase for props
- use double quotes for JSX attribute values if you are passing them a *string*

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

Omit boolean values if default value is **true**:

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

### Receiving Props
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
- add `?` at the end of parameter that are optional

``` javascript
class componentName extends Component<{
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}, {}> {
  // ...
}
```

- if you want cleaner component declaration, or if the component receives a great number of props, use interfaces
- use **"I"** prefix in interface names


``` javascript
interface IComponentProps {
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}

class ComponentName extends Component<{IComponentProps}, {}> {
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


### Conditional Rendering
- use ternary operators for conditional rendering
- return **null** if condition is not met

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

### Rendering Arrays of Data
- use array's **map** method to render items from an array
- you should have a unique key for each element
- don't use array's index as a key

``` javascript
render() {
    // ...
    return (
        <ul>
            {
                booksArray.map((book) => {
                    <ListItem key={book.id}>{book.title}></ListItem> 
                }); 
            }
        </ul>
    );
}
```

### Binding
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

// good

```

### CSS Modules And Components
- each component should have it's own stylesheet CSS is needed for it

```
├── Login
    └── Login.jsx
    └── Login.scss 
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

- if you need to pass more than one CSS class name to a component, use the [classnames](https://www.npmjs.com/package/classnames) library

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
