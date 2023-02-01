When we use Angular Universal, the API that delivers the content is hit twice. First time is when the server is
rendering the page and the second time is when the application is bootstrapped in the browser. This causes latency
issues and a bad user experience as the screen usually flickers when this happens. Not to mention that this is not
optimal.

We can use `TransferState` service to send information from the server to the client, which avoids making duplicate API
calls.

## TransferHttpCacheModule

`TransferHttpCacheModule` installs an HTTP interceptor that avoids duplicate `HttpClient` GET requests on the client,
for requests that were already made when the application was rendered on the server side.

When the module is installed in the application `NgModule`, it will intercept `HttpClient` requests on the server and
store the response in the `TransferState` key-value store. This is transferred to the client, which then uses it to
respond to the same `HttpClient` requests on the client.

Any requests other than GET will prevent any further requests so be aware of this if you have some initial request on
the page which is not GET request you will need to handle that case manually. You may alternatively
use `BrowserTransferStateModule` to write your own behaviour for caching information.

### Usage

To use the `TransferHttpCacheModule`, first install it as part of the `app.module.ts`:

```typescript
import {TransferHttpCacheModule} from '@nguniversal/common';

@NgModule({
    imports: [
        BrowserModule.withServerTransition({appId: 'serverApp'}),
        TransferHttpCacheModule,
    ]
})
```

Then, import `ServerTransferStateModule` in your `app.server.module.ts`.

```typescript
import {NgModule} from '@angular/core';
import {ServerModule, ServerTransferStateModule} from '@angular/platform-server';
import {AppModule} from './app.module';
import {AppComponent} from './app.component';

@NgModule({
    imports: [AppModule, ServerModule, ServerTransferStateModule],
    bootstrap: [AppComponent],
})
export class AppServerModule {
}
```

### Verification

You can test that `TransferState` has been configured successfully if you go the Network section in your browser and see
that all initial GET request are no longer being made. You might also notice that a script element was added to the
initial document response - `<script id="serverApp-state" type="application/json">...</script>`, including all the
transferred data.
