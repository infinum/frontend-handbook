When using SSR, the node server can read system environment variables at runtime and transfer them to the client using
the [`TransferState` API](https://angular.io/api/platform-browser/TransferState). This is just one of the use cases for
the `TransferState` API.

To make working with environment variables in an SSR application easier, we have created `EnvironmentVariablesSSRLoader`
in our [ngx-nuts-and-bolts](https://github.com/infinum/ngx-nuts-and-bolts) library.
The following section is just a quick-start demonstrating how the lib can be used. You can read into more details in
the [official documentation](https://infinum.github.io/ngx-nuts-and-bolts/docs/services/environment-variables).

## Configuration

First, add `EnvironmentVariablesModule` and `EnvironmentVariablesSSRLoaderModule` to `app.module.ts`:

```typescript
import {NgModule} from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
import {TransferHttpCacheModule} from '@nguniversal/common';
import {EnvironmentVariablesModule} from '@infinumjs/ngx-nuts-and-bolts';
import {EnvironmentVariablesSSRLoaderModule} from '@infinumjs/ngx-nuts-and-bolts-ssr';
import {exposedEnvironmentVariables} from './enums/environment-variable.enum';

@NgModule({
    imports: [
        BrowserModule.withServerTransition({appId: 'serverApp'}),
        TransferHttpCacheModule,
        EnvironmentVariablesModule,
        EnvironmentVariablesSSRLoaderModule.withConfig({
            variablesToLoad: exposedEnvironmentVariables,
        }),
    ],
})
```

Next, create `environment-variable.enum.ts` file and define environment variables to be exposed:

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

If there are some environment variables that are sensitive and should not be transferred to the client, you can still
add them to the enum and read them on the server-side, but you mustn't include them in the `exposedEnvironmentVariables`
array.
