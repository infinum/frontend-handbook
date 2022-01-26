When you are working with refs you have control over the `update` phase but you don't have a way to trigger it imperatively. But, if you still need to trigger an update imperatively, you can use a custom `useUpdate` hook.

_Note: This example demonstrates how `react-hook-form` works internally._

```jsx
const updateReducer = (num: number): number => (num + 1) % 1_000_000;

const useUpdate = () => {
  const [, update] = useReducer(updateReducer, 0);

  return update;
}

// Large form
// This is how react-hook-form works, in a nutshell
const Form: FC = () => {
  const formRef = useRef();

  if (!formRef.current) {
    formRef.current = {
      name: "",
      surname: "",
      // ...many fields
    };
  }

  const update = useUpdate();

  const register = useCallback((input) => {
    input?.addEventListener(
      "input",
      () => (formRef.current[input.name] = input.value)
    );
  }, []);

  const submit = useCallback(
    (event) => {
      event.preventDefault();
      // triggers update and renders the form data
      update();
    },
    [update]
  );

  return (
    <form onSubmit={submit}>
      <div>Form Data: {JSON.stringify(formRef.current)}</div>
      <input name="name" ref={register} />
      <input name="surname" ref={register} />
      // ...many fields
      <input type="submit" />
    </form>
  )
}
```

<iframe src="https://codesandbox.io/embed/large-form-update-uskuh?fontsize=14&hidenavigation=1&theme=dark"
  style="width:100%; max-width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden; box-shadow:rgba(0, 0, 0, 0.1) 0px 10px 15px -3px, rgba(0, 0, 0, 0.05) 0px 4px 6px -2px;"
  title="large-form-update"
  allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
  sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
></iframe>