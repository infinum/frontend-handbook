# Angular Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

## Libraries and Tools

### [Angular CLI](https://github.com/angular/angular-cli)

If you are starting a new project, the best way to create a new project is by using Angular CLI. It will hide a lot of configuration behind the scenes (you will not have a webpack config exposed for editing), it does some optimizations when compiling your code and it offers some handy scaffolding tools.

Install Angular CLI globally with `npm install -g @angular/cli` and check out what you can do with `ng help`.

#### Creating a new project

Make sure to use SCSS for styling. You can change this later if you forget but it can be tedious. Default is plain CSS
`ng new my-app --style=scss`

#### Creating new components

Create component with `ng g c MyComponent` or `ng g c my-component`. In both cases resulting component class name will be `MyComponent` and dir name will be `my-component`.

#### Creating new services

Use `ng g s MyService` to generate new service. NOTE: unlike during component generation, service generation will not create a new dir for the service (although that might change in the future).

#### Ejecting

If you **ABSOLUTELY HAVE TO**, you can eject the application with `ng eject`. Please avoid this. Doing this will expose all config files (e.g. webpack config). This also means that you will no longer be able to run your app with `ng serve`, instead you will have to use `npm start serve`. You will still be able to use scaffolding functionality of the Angular CLI, but that is pretty much it.

Most importantly, this means that you will have to maintain webpack config yourself, whereas non-ejected projects might see behind-the-scenes webpack config updates when updating app's angular-cli dependency version. Great example would be updating from webpack 3 to 4, which happens automatically if the project is not ejected.

One of the reasons why you might want to eject is if you want to use OS environment variables and/or inject things like git commit hash into your code. Currently Angular CLI has no support for those things. *There are* ways to do it without ejecting, but it requires running custom scripts before compiling the code. If you need to do something like that, make sure to discuss it with other developers on the project and decide what the best course of action is.

#### Other commands

This handbook will not cover all `ng` commands, please check out the [wiki](https://github.com/angular/angular-cli/wiki).

### [RxJS](https://github.com/ReactiveX/rxjs)

Angular depends heavily on RxJS so getting to know RxJS is pretty important. It is used for all event handling and can even be used for application state management of some description.

Please check out some RxJS tutorials if you are not familiar with Rx.

#### Observers and Subscribers

When you want to observe some value, use some of the classes derived from `Observable`. Most use cases are coverd with one of these:

- Subject
- BehaviorSubject
- ReplaySubject

All of classed derived from `Observable` can be subscribed to in order to receive value changes. You can also use `operators` to manipulate how and when the value changes are sent to the subscribers.

One convention is that variables which are `Observable` are postfixed with a dollar sign, like so:

```
// bad
const observable: Observable;

// good
const observable$: Observable;
```

#### Operators usage and creation

If you are using RxJS 5.5 or newer, make sure to use *[pipeable operators](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)* instead of *patch* operators. Some common operators are:

- map
- filter
- tap
- delay
- debounceTime
- distinctUntilChanged
- catchError

You can also write your own operators. If you do so, it is strongly recommended to write them as [pure pipeable operator functions](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md#operator-as-a-pure-function).

### Recommended Editor

Recommended editor for Angular development is [VSCode](https://code.visualstudio.com/) as it has great integration with TypeScript and has some handy Angular plugins (e.g. [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template)).

## File/Module Organization and Naming

If you created the app using Angular CLI, there is already some clear structure in place. Building on that structure, you will probably want to add some more dirs to help you organize your code.

Try to split up your application into components which you can reuse. If, for example, you have to handle lazy loading of all image in the app, create a `Image` module and component in `app/components`. Export the declared component and import/provide dependencies which your component has. In general if a component is used in multiple places, put it in `app/components`. One little exception here: if there is some very specific component tied to some routing submodule which is used only in that submodule, it is OK to leave it in that submodule (but as soon as it is used in another routing submodule, move it to `app/components`.

Store all your animations in `app/animations/` and import them for usage in component decorators.

Classes which might be used throughout the application (e.g. generic `Dictionary` class) should be in `app/classes`.

Entity models (`User`, `Post`, etc.) should be in `app/models`.

If you are using some JS libs which do not have typing, add your own custom typing in `app/custom-typings`.

Whenever you have some attribute whose value can be some value from a set of values, make an enum and store that enum in `app/enums`. Good example is an enum for HTTP status codes. If you need some metadata for the enum, create an enum data file which exportst what is basically a dictionary with enum values used as keys and use whatever you need for values. One such example would be: `export const httpStatusCodeData = { [HttpStatus.OK_200]: { translation: 'Success' } }`

Put all route guards in `app/guards`.

If you need some helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `app/helpers` in its own dir, make sure to write tests for it and simply export it in `app/helpers/helpers.ts`.

Save all your interface files to `app/interfaces`.

If you have some custom pipes for use in templates, create their own dirs in `app/pipes`, similar to helpers. Declare, export and provide the pipes in `app/pipes/pipes.module.ts`.

Your src folder might look something like this:
*(please note the postfixes in filenames like .guard, .interface, .model, etc.)*

- app/
  - animations/
    - fade.animations.ts
  - classes/
    - dictionary.ts
  - components/
    - header/
      - header.component.html
      - header.component.scss
      - header.component.spec.ts
      - header.component.ts
      - header.module.ts
    - modals/
  - custom-typings/
    - some-js-lib.d.ts
  - enums/
    - http-response-code.enum.ts
    - http-response-code-data.enum.ts
    - user-status.enum.ts
  - guards/
    - authentication/
      - authentication.guard.ts
      - authentication.guard.spec.ts
  - helpers/
    - round/
      - round.helper.ts
      - round.helper.spec.ts
    - helpers.ts
  - interfaces/
    - user-data.interface.ts
  - models/
    - user.model.ts
  - pages/
    - home-container/
    - login-container/
    - users-container/
  - pipes/
    - order-by/
      - order-by.pipe.ts
      - order-by.pipe.spec.ts
    - pipes.module.ts
  - services/
    - user/
      - user.service.ts
      - user.service.spec.ts
  - shared-modules/
    - material/
      - material.module.ts
  - types/
    - sorting-function.type.ts
- assets/
  - fonts/
    - CustomFont.eot
    - CustomFont.ttf
    - CustomFont.woff
    - CustomFont.woff2
  - images/
    - icons/
      - app-icon.svg
  - styles/
    - components/
      - _title.scss
    - overrides/
      - _buttons.scss
      - _card.scss
      - _media-blender.scss
    - themes/
      - _main-theme.scss
    - utils/
      - _colors.scss
      - _fonts.scss
      - _mixins.scss
      - _shared-variables.scss
      - _z-indices.scss
    - main.scss
    - _core.scss
- environments/
  - environment.base.ts
  - environment.prod.ts
  - environment.staging.ts
  - environment.ts
- testing/
  - stub-services/
    - user.service.stub.ts
  - stub-models/
    - user.model.stub.ts
  - stub-data/
    - some-json-api-response.json

### Presentational (dumb) and Container (smart) Components



## Environments

## Formatting

## Routing and Lazy Loading

## Importing

## Server-side rendering

## Inner Workings and Common Pitfalls