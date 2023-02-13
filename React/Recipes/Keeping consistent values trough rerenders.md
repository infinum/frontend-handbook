Do not use `useMemo` as a semantic guarantee that it will be a constant throughout component re-renders

If you need a value to stay the same throughout re-renders, you might think of `useMemo` as a nice way to "mimic" a constant. While it might seem that way, [it is not guaranteed](https://reactjs.org/docs/hooks-reference.html#usememo).

> **You may rely on useMemo as a performance optimization, not as a semantic guarantee.** In the future, React may choose to “forget” some previously memoized values and recalculate them on next render, e.g. to free memory for off-screen components. Write your code so that it still works without useMemo — and then add it to optimize performance when it's necessary.

```jsx
// BAD
export const MyComponent: FC = () => {
  const id = useMemo(() => generateId(), []); // Not guaranteed to be a constant

  // ...
}
```

The best approach is to use lazy initialization:

```jsx
// GOOD
export const MyComponent: FC = () => {
  const [id, _] = useState(() => generateId()); // Initial value will stay the same throughout re-renders

  // ...
}
```

You can also store the value in a ref:

```jsx
type Result<T> = { v: T };

function useConstant<T>(fn: () => T): T {
  const ref = useRef<Result<T>>();

  if (!ref.current) {
    ref.current = { value: fn() };
  }

  return ref.current.value;
}

export const MyComponent: FC = () => {
  const id = useConstant(() => generateId());

  // ...
}
```

_Note: updating ref values will not trigger a re-render._

Storing a "constant" value in a ref might not work with Concurrent Mode. You can see an explanation in this [twitter thread](https://twitter.com/sstur_/status/1384934568285900806).

Consider this case:

1. Render starts
2. You update the ref
3. React concurrent mode has to throw away the work and doesn't commit

You now have a ref that isn't the latest value, it's a value for a render that was never committed.