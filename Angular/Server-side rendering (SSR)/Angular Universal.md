Angular Universal is tool which allows server to pre-render Angular application.
For detailed guide you can read it more [here](https://angular.io/guide/universal) in the Angular official documentation.

### Angular Universal installation guide
To add Angular Universal run the following CLI command:
```
ng add @nguniversal/express-engine
```
The command creates the following folder structure:
```
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
  "serve": "node dist/server/main.js"
}
```
_APP_NAME is application name. For example `js-infinum-website`._

This configuration uses SSR for development. This helps catch any SSR-specific errors during development. 
