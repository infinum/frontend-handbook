Environment files should be looked at like "build configuration" files and should be used for some global configuration changes that can differ between development and production builds—things like toggling source-maps on and off. Keep in mind that these values are bundled in build at build-time and cannot be changed after the build (at least not in an acceptable way).

Contrary to their name, environment files should not be used for storing values that can change depending on the deployment environment. These file should really have been called `configuration` files instead of `environment` files. Even the [Angular CLI renamed them (partially)](https://angular.io/guide/workspace-config#alternate-build-configurations) to `configuration` files. However, people saw these files and naturally started using them for injecting environment-specific URLs and similar.

The reason why we do not recommend using the environment files for that purpose is because it gets hard to maintain if you have many environments and you do not want to commit potentially sensitive values. It can also be an issue if the application is deployed using Docker—in that case the build is run only and the same image is deployed multiple times to different environments, requiring different API URLs to be injected into the frontend code. Environment files are not a good fit for this and some other mechanism should be used.

Your application source code repository should not be defining environment-specific values for URLs, secrets and similar—it should instead provide a way for those values to be injected after the build step.

In case that SSR is used, the SSR server can read system environment variables at runtime and transfer them to the client. SSR server can also use some secret variables whose values should not be exposed to the client, but can be use on the server side. Please check out [this repository](https://github.com/fvoska/angular-universal-demo), as an example of how this could be done.

In case that SSR is not used, there should be a post-build script that can be run after the build and before deployment. This script should provide the client application with correct values for a specific environment. In short, such a script would inject a piece of code in the built index.html file, containing the values for the environment-specific variables. This can be done by attaching some object to the `Window` object. Working with the global object would generally be considered a bad practice, but in this particular case there is a need for this, and accessing the global object will be contained to a very specific service that reads those values and provides them to the rest of the application.

In any case, for things that you do use the environment files for, we recommend defining an interface for the environment files: `app/interfaces/environment.interface`:

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
