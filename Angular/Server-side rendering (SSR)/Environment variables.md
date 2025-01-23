When using SSR, the node server can read system environment variables at runtime and transfer them to the client using
the [`TransferState` API](https://angular.dev/api/core/TransferState). This is just one of the use cases for
the `TransferState` API.

To simplify working with environment variables in an SSR application, refer to the `Environment variables` documentation
in our [ngx-nuts-and-bolts](https://infinum.github.io/ngx-nuts-and-bolts/docs/environment-variables#232-for-ssr--angular-universal-apps) library where the setup and the usage is explained in detail.
