Because we are using SSR, the SSR server can read system environment variables at runtime and transfer them to the client with `TransferState`. For this we can use loader `EnvironmentVariablesSSRLoaderModule` from the library [ngx-nuts-and-bolts](https://github.com/infinum/ngx-nuts-and-bolts).
You can read more details in the [official documentation](https://infinum.github.io/ngx-nuts-and-bolts/docs/services/environment-variables).

## Configuration
In the `app.module.ts` add `EnvironmentVariablesModule` and `EnvironmentVariablesSSRLoaderModule`:

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { TransferHttpCacheModule } from '@nguniversal/common';
import { EnvironmentVariablesModule } from '@infinumjs/ngx-nuts-and-bolts';
import { EnvironmentVariablesSSRLoaderModule } from '@infinumjs/ngx-nuts-and-bolts-ssr';
import { exposedEnvironmentVariables } from './enums/environment-variable.enum';

@NgModule({
    imports: [
        BrowserModule.withServerTransition({ appId: 'serverApp' }),
        TransferHttpCacheModule,
        EnvironmentVariablesModule,
        EnvironmentVariablesSSRLoaderModule.withConfig({
            variablesToLoad: exposedEnvironmentVariables,
        }),
    ],
})
```

We can create `environment-variable.enum.ts` file and declare environment variables and variables we want to expose:

```typescript
export enum EnvironmentVariable {
	APP_URL = 'APP_URL',
	API_URL = 'API_URL',
}

export const exposedEnvironmentVariables: Array<string> = [
    EnvironmentVariable.APP_URL,
    EnvironmentVariable.API_URL,
];
```

After this configuration you can start using `EnvironmentVariablesService` to get environment variables.
