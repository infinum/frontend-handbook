## Always expose internal constants (magic numbers) as props

If we need some "magic number" inside our component we will usually do something like this:

The problem ⚡

```tsx
const HEIGHT = 72;

const Example: FC = ({ children }) => {

  const position = calculatePositionBasedOnHeight(HEIGHT);

  return (
    <div style={{ height: HEIGHT }}>
      <div style={{ position: 'absolute', left: position.x, top: position.y }}>
       {children}
      </div>
    </div>
  );
}
```

Maybe we don't see it right away and we might think that this component will always have the same height everywhere. But it usually turns out differently.

The solution ✅

Always expose "magic numbers" as props because there is a really big chance that we will need to change for some specific use-case.

```tsx
const HEIGHT = 72;

export interface IExampleProps {
  height?: number;
}

const Example: FC<IExampleProps> = ({ children, height = HEIGHT }) => {

  const position = calculatePositionBasedOnHeight(HEIGHT);

  return (
    <div style={{ height: HEIGHT }}>
      <div style={{ position: 'absolute', left: position.x, top: position.y }}>
       {children}
      </div>
    </div>
  );
}
```

## Shadowing props in higher abstractions is a bad habit

When we are creating a higher abstractions of our base components, we usually think that we want to disallow some props if we override them internally. But usually it's a bad thing because you are narrowing down reusability of that component.

The problem ⚡

```tsx
// Base component
interface IImageProps {
  src?: string;
  loader?: (src) => string;
}

const Image: FC<IImageProps> ({ loader, src }) => {
  const internalSrc = loader?.(src) || src;

  return <img src={src} {...props} />;
}

// Higher abstraction component
interface IMyImageProps extends Omit<IImageProps, 'loader'> {
  // ...
}

// Cache busting!
const myLoader = (src) = `${src}?timestamp=${Date.now()}`;

const MyImage: FC<IMyImageProps> (props) => {
  <Image {...props} loader={myLoader} />
};
```

This is a problem because we shadowed the `loader` prop and we can't opt-out from this behavior anymore.
What will we usually do in this situation is start adding multiple boolean flags for every edge-case we encounter.
And we ship that `if/else` code with every component include, whether we use it or not.

```tsx
// Higher abstraction component
interface IMyImageProps extends Omit<IImageProps, 'loader'> {
  // ...
  isSomeEdgeCase?: boolean;
}

// Cache busting!
const myLoader = (src) = `${src}?timestamp=${Date.now()}`;
const edgeCaseLoader = (src) = `${src}?timestamp=${Date.now()}&w=500`;

const MyImage: FC<IMyImageProps> (props) => {
  const loader = props.isSomeEdgeCase ? edgeCaseLoader : myLoader;

  // we start adding more and more if/else statements here code grows out of proportions

  <Image {...props} loader={loader} />
};
```

The solution ✅

There is no real reason to omit `loader` prop. We can just provide sane default value and leave the overriding ability for specific cases.

```tsx
interface IMyImageProps extends IImageProps {
  // ...
}

const myLoader = (src) = `${src}?timestamp=${Date.now()}`;

const MyImage: FC<IMyImageProps> (props) => {
  <Image loader={myLoader} {...props} />
};
```

And if for one edge-case we need to include `w=500` in the URL, we can do it like this:

```tsx
<MyImage src="..." loader={(...args) => `${myLoader(...args)}&w=500`} />
```
