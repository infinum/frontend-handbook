A normal Angular application executes in the browser, rendering pages in the DOM in response to user actions. Angular Universal executes on the server, generating static application pages that later get bootstrapped on the client. This means that the application generally renders more quickly, giving users a chance to view the application layout before it becomes fully interactive.

## What is Server Side Rendering (SSR)?
Server-side rendering (SSR) is an application’s ability to convert HTML files on the server into a fully rendered HTML page for the client. The web browser submits a request for information from the server, which instantly responds by sending a fully rendered page to the client.

## Why we use Server Side Rendering (SSR)?
There are 3 reasons for create angular application with server side rendering.

 1. Search engine optimization (SEO).
 2. Improve performance on web and mobile.
 3. Load the page quickly.

## Angular Universal
Angular Universal is tool which allows server to pre-render Angular application.
For detailed guide you can read it more [here](https://angular.io/guide/universal) in the Angular official documentation.

To add Angular Universal run the following CLI command:

```
ng add @nguniversal/express-engine
```

The command creates the following folder structure:

```text
src
|-- index.html                             // <-- app web page
|-- main.ts                                // <-- bootstrapper for client app
|-- main.server.ts                         // <-- * bootstrapper for server app
|-- style.css                              // <-- styles for the app
|-- app/  …                                // <-- application code
|   |-- app.server.module.ts               // <-- * server-side application module
|-- server.ts                              // <-- * express web server
|-- tsconfig.json                          // <-- TypeScript base configuration
|-- tsconfig.app.json                      // <-- TypeScript browser application configuration
|-- tsconfig.server.json                   // <-- * TypeScript server application configuration
|-- tsconfig.spec.json                     // <-- TypeScript tests configuration
```

The files marked with `*` are new added files after running the command.

### Package.json file configuration
In the package.json file update the following scripts:

```json
{
  "start": "ng run APP_NAME:serve-ssr",
  "build": "ng build && ng run APP_NAME:server",
  "serve": "node dist/APP_NAME/server/main.js"
}
```

_APP_NAME is application name. For example `js-infinum-website`._

This configuration uses SSR for development. This helps catch any SSR-specific errors during development. 



