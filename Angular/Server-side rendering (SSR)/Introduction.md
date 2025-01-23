A normal Angular application executes in the browser, rendering pages in the DOM in response to user actions. Angular
Universal executes on the server, generating static application pages that later get bootstrapped on the client. This
means that the application generally renders more quickly, giving users a chance to view the application layout before
it becomes fully interactive.

## What is Server Side Rendering (SSR)?

Server-side rendering (SSR) is an application’s ability to render HTML content of a single-page-app on the server and
serve it to the client. This is in contrast to how regular SPAs without SSR work, where all the HTML is rendered on the
client after the JS application is started. With SSR, the web browser submits a request to the server, which responds by
sending back a fully rendered page to the client.

## Why we use Server Side Rendering (SSR)?

There are 3 main reasons to add SSR to an Angular app:

1. Search engine optimization (SEO)
2. Social media link sharing
3. Improving initial page load performance on all types of devices and networks

## Setup

For a detailed guide on setting up SSR in Angular, refer to the [official Angular Server-side rendering documentation](https://angular.dev/guide/ssr). We also recommend exploring some of the [gotchas](https://github.com/angular/universal/blob/main/docs/gotchas.md).

