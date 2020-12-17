Environment files should be used for some global configuration changes that can differ between development and production builds. Keep in mind that these values are bundled in build at build-time and cannot be changed after the build (at least not in an acceptable way). This can be an issue if the application is deployed using Docker, in which case you should use some SSR solution or script injection instead of defining variables via Angular's environment files (more on this in the SSR section).

Define the interface for the environment: `app/interfaces/environment.interface`:

``` typescript
export interface IEnvironment {
  production: boolean;
}
```

Set base values `environments/environment.base.ts`:

``` typescript
import { IEnvironment } from 'app/interfaces/environment.interface';

export const baseEnvironment: IEnvironment = {
  production: false,
}
```

This is the default `dev` environment: `environments/environment.ts`:

``` typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  // Nothing is overriden, all will be as defined in base environment
};
```

This is the production environment: `environments/environment.prod.ts`:

``` typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  production: true,
};
```

Make sure to update `angular.json` if you add/remove environment files.

After everything is defined, run/build the app for different environments by specifying the environment in an `ng` command: `ng build --env=prod` for production build; `ng build --env=staging` for staging build; `ng build` will use the default `dev` environment.
