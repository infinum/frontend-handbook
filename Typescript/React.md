# TypeScript + React

*For general info on using JSX with TypeScript checkout the guidelines in TypeScript's official documentation [JSX · TypeScript](https://www.typescriptlang.org/docs/handbook/jsx.html)*.

### Component Declaration

When declaring a React class component in TypeScript, set two generic types that describe props and state.

``` typescript
export class ComponentName extends React.Component<{/*prop types*/}, {/*state types*/}> {
  // ...
}
```

### Props

For validating props in TypeScript classes:
- use TypeScript's built in types instead of `prop-types`
- if no local component state is set, skip it (when MobX is used for state management this is usually always the case)

``` typescript
export class ComponentName extends React.Component<{
  className?: string;
  isConditionMet: boolean;
  onClick?(event: any): void;
}> {
  // ...
}
```

Use interfaces when you have a set of common props used in more than a few components (in which case the interfaces should be imported from a separate file).

``` typescript
interface CommonProps {
  // prop types
}
``` 

``` typescript
import { CommonProps } from 'interfaces/CommonProps';

export class ComponentName extends React.Component<CommonProps & { active?: boolean }> {
  // ...
}
```

### Class Members and Methods
React's lifecycle hooks and render method have to be declared public. Local data and methods that are not passed to child components are generally declared private.

``` typescript
export class ComponentName extends React.Component {
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
