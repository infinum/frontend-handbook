## ./src/pages/login.tsx

- no fetching should happen here
- no loading/modifying data should happen here

```tsx
import React from 'react';

import LoginContainer from 'components/containers/Login';

/**
 * Page accessible on /login; used to login in to the app.
 * Once user is logged in the session token is stored in the
 * local storage.
*/
const LoginPage = () => {
	return {
		<LoginContainer />
	};
};

export default LoginPage;
```