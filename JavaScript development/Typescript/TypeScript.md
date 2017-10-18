# TypeScript

## TypeScript+React
*For general info on using JSX with TypeScript checkout the guidelines in TypeScript's official documentation [JSX · TypeScript](https://www.typescriptlang.org/docs/handbook/jsx.html)*.

### Component Declaration

When declaring a React class component in TypeScript, set two generic types that describe props and state.

``` typescript
export default class ComponentName extends Component<{/*prop types*/}, {/*state types*/}> {
    // ...
}
```

Declare stateless components as arrow functions

``` typescript
export default (/*props*/): JSX.Element => {
  // ...
}
```

### Props

For validating props in TypeScript classes:
- use TypeScript's built in types instead of `prop-types`
- if no state (or prop) is set, use **{}** (when MobX is used for state management this is usually always the case)

``` typescript
class ComponentName extends Component<{
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}, {}> {
  // ...
}
```

In stateless components, use regular object destructuring with TypeScript's type annotations.
``` typescript
export default ({className?: string, title: string}): JSX.Element => {
    // ...
}
```

Use interfaces for props when you have a great amount of props/state to validate or when you have a set of common props used in more than a few components (in which case the interfaces should be imported from a separate file).

``` typescript
interface IComponentNameProps {
  // prop types
}

interface IComponentNameState {
  // state types
}

class ComponentName extends Component<{}, {}> {
  // ...
}
```

### Class Members and Methods
React's lifecycle hooks and render method have to be declared public. Local data and methods that are not passed to child components are generally declared private.

``` typescript
class ComponentName extends Component<IComponentNameProps, IComponentNameState> {
  constructor(args) {
    super(args);
    // ...
  }

  public componentDidMount() {
    // ...
  }

  private handleLocalData() {
    // ...
  }

  public render() {
    return {
      // ...
    };
  }
}
```

Example with MobX:

``` typescript
@observer
class ComponentName extends Component<{}, {}> {
  constructor(args) {
    super(args);
    // ...
  }

  @observable private componentState = {
    // ...
  };

  public componentDidMount() {
    // ...
  }

  @action.bound private handleLocalData() {
    // ...
  }

  public render() {
    return {
      // ...
    };
  }
}
```