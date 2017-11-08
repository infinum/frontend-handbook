# TypeScript

## TypeScript+React
*For general info on using JSX with TypeScript checkout the guidelines in TypeScript's official documentation [JSX · TypeScript](https://www.typescriptlang.org/docs/handbook/jsx.html)*.

### Component Declaration

When declaring a React class component in TypeScript, set two generic types that describe props and state.

``` typescript
export class ComponentName extends Component<{/*prop types*/}, {/*state types*/}> {
    // ...
}
```

Declare functional components as arrow functions

``` typescript
export const FunctionalComponent = (/*props*/): JSX.Element => {
  // ...
}
```

### Props

For validating props in TypeScript classes:
- use TypeScript's built in types instead of `prop-types`
- if no local component state is set, use **{}** (when MobX is used for state management this is usually always the case)

``` typescript
export class ComponentName extends Component<{
  className?: string;
  isConditionMet: boolean;
  onClick?: (event: any) => void;
}, {}> {
  // ...
}
```

In stateless components, use regular object destructuring with TypeScript's type annotations.
``` typescript
export const FunctionalComponent = ({classname, title}: {className?: string, title: string}): JSX.Element => {
    // ...
}
```

Use interfaces when you have a set of common props used in more than a few components (in which case the interfaces should be imported from a separate file).

``` typescript
interface SpecificProps {
  // prop types
}

export class ComponentName extends Component<SpecificProps, {}> {
  // ...
}
```

### Class Members and Methods
React's lifecycle hooks and render method have to be declared public. Local data and methods that are not passed to child components are generally declared private.

``` typescript
export class ComponentName extends Component<{}, {}> {
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
export class ComponentName extends Component<{}, {}> {
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

  @bind
  @action
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