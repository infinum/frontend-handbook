Do not use `useMemo` as a semantic guarantee that it will be a constant throughout component re-renders

If you need a value to stay the same throughout re-renders, you might think of `useMemo` as a nice way to "mimic" a constant. While it might seem that way, [it is not guaranteed](https://react.dev/reference/react/useMemo#preventing-an-effect-from-firing-too-often).

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

*Note: updating ref values will not trigger a re-render, and they should never be used inside hook like useEffect, useCallback or useMemo dependency arrays.*

If you'll ever need to update the ref value, it's crucial to avoid modifying the .current property during the render phase. For example in Concurrent Mode, React may invoke the render function multiple times without committing the result, leading to potential inconsistencies if refs are modified during rendering.

For example, consider the following ["Latest ref pattern"](https://www.epicreact.dev/the-latest-ref-pattern-in-react) pattern:

```jsx
const callbackRef = React.useRef(callback);

React.useLayoutEffect(() => {
  callbackRef.current = callback;
});
```

In this pattern, callbackRef is initialized with the callback function. The useLayoutEffect hook updates callbackRef.current after each render, ensuring it always holds the latest callback without causing re-renders. This approach is safe in Concurrent Mode because the ref is updated outside the render phase.

In summary, while useRef is reliable for storing persistent values, ensure that any updates to the ref's .current property occur outside the render phase to maintain consistency and prevent potential issues.
