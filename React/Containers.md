## ./src/components/containers/Login.tsx

- fetch should not be used here, only thru appData
- this is **not a partial** do not use containers as such (e.g. `PaginationContainer`)
- no styled components here (margins should happen on the components thru the props)

```tsx
import React from 'react';

import LoginForm from 'components/ui/LoginForm';
import appData from 'collections/app-data';

const LoginContainer = () => {
  const [apiErrors, setApiErrors] = React.useState({});

  const onSubmit = ({ username, password }) => {
    appData
      .login(username, password)
      .then(() => {})
      .catch((e) => {});
  };

  return (
    <>
      <LoginForm margin={30} onSubmit={onSubmit} apiErrors={apiErrors} />
    </>
  );
};

export default LoginContainer;
```
