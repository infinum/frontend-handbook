# Angular Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

## Core Libraries and Tools

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

- `Subject`
- `BehaviorSubject`
- `ReplaySubject`

All of classed derived from `Observable` can be subscribed to in order to receive value changes. You can also use `operators` to manipulate how and when the value changes are sent to the subscribers.

One convention is that variables which are `Observable` are postfixed with a dollar sign, like so:

```typescript
// bad
const observable: Observable;

// good
const observable$: Observable;
```

#### Operators usage and creation

If you are using RxJS 5.5 or newer, make sure to use *[pipeable operators](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)* instead of *patch* operators.

```typescript
// bad
subject.do(() => { ... });

// good
subject.pipe(tap() => { ... });
```

Some common operators are:

- `map`
- `filter`
- `tap` (replaces `.do`)
- `delay`
- `debounceTime`
- `distinctUntilChanged`
- `catchError` (replaces `.catch`)

You can also write your own operators. If you do so, it is strongly recommended to write them as [pure pipeable operator functions](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md#operator-as-a-pure-function).

### Recommended Editor

Recommended editor for Angular development is [VSCode](https://code.visualstudio.com/) as it has great integration with TypeScript and has some handy Angular plugins (e.g. [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template)). This is just a recommendation, please feel free to use whatever editor you are most comfortable with.

## File/Module Organization and Naming

If you created the app using Angular CLI, there is already some clear structure in place. Building on that structure, you will probably want to add some more dirs to help you organize your code.

Try to split up your application into components which you can reuse. If, for example, you have to handle lazy loading of all images in the app, create a `Image` module and component in `app/components` and use it in other components whenever you have to show an image. Export the declared component and import/provide dependencies which your component has.

Container components which are loaded directly via routing (by either `component` or `loadChildren`) should be placed in `app/pages` dir. If component is loaded because you have used it in a template, that component should be in `app/components`

One little exception here: if there is some very specific component tied to some routing submodule which is used only in that submodule, it is OK to leave it in that submodule (but as soon as it is used in another routing submodule, move it to `app/components`.

Store all your animations in `app/animations/` and import them for usage in component decorators.

Classes which might be used throughout the application (e.g. generic `Dictionary` class) should be in `app/classes`.

Entity models (`User`, `Post`, etc.) should be in `app/models`.

If you are using some JS libs which do not have typing, add your own custom typing in `app/custom-typings`.

Whenever you have some attribute whose value can be some value from a set of values, make an enum and store that enum in `app/enums`. Good example is an enum for HTTP status codes. If you need some metadata for the enum, create an enum data file which exportst what is basically a dictionary with enum values used as keys and use whatever you need for values. One such example would be: `export const httpStatusCodeData = { [HttpStatus.OK_200]: { translation: 'Success' } }`

If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s and validators in `app/forms`. If you have form components which are reused throughout the app, put them in `app/components/forms`.

Put all route guards in `app/guards`.

If you need some helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `app/helpers` in its own dir, make sure to write tests for it and simply export it in `app/helpers/helpers.ts`.

Save all your interface files to `app/interfaces`.

If you have some custom pipes for use in templates, create their own dirs in `app/pipes`, similar to helpers. Declare, export and provide the pipes in `app/pipes/pipes.module.ts`.

Services should be placed in `app/services`, each service in its own dir.

If you have any `NgModule`s which are not components (e.g. a module which imports all specific material modules that you need), put them in `app/shared-modules`.

TypeScript `type` declarations should be placed in `app/types`.

As defined in angular-cli, assets placed in `assets` will be served statically. `assets/styles` dir has very similar structure as that described in SASS Styleguide, so please check it out.

All environment files are placed in `environments`.

Save all your mock models, services and data in `testing`. Make sure that dir is excluded from code coverage calculations.

When it all comes together, your src folder might look something like this:
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
    - forms/
  - custom-typings/
    - some-js-lib.d.ts
  - enums/
    - http-response-code.enum.ts
    - http-response-code-data.enum.ts
    - user-status.enum.ts
  - forms/
    - user/
      - user.form-object.ts
      - user.form-store.ts
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

## Presentational and Container Components

Design your components to be as small as possible and reuse them as much as possible. Philosophically, components can be split up into two types:

1. Presentational (dumb)
    - in charge of displaying the data which is passed down to them
    - should not make use of services
    - pass events up instead of doing complex things with them
    - use `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` declaration

2. Container (smart)
    - use multiple (if necessary) other components
    - serve data to presentational components
    - handle events from child components
    - control UX flow
    - work with services to make things happen (fetch, store, change state)
    - can be loaded directly or lazy loaded when routing

More on this philosophy can be found in various articles [Google]. Some articles have framework-specific information, but the core concepts apply no matter which framework you are using.

## Environments

Currently there is no way to use OS environment variables if you are using `angular-cli` to build the app. If something is different depending on the environment, it should be defined in a variable in corresponding environment file. Every environment should have its own environment file which extends base environment file and overrides any values which are different. All environments have to be defined in `.angular-cli.json` as well.

Example:

Define interface `/src/app/interfaces/environment.interface`:
```typescript
export interface IEnvironment {
  production: boolean;
  apiBaseUrl: string;
}
```

Set base values `/src/environments/environment.base.ts`:
```typescript
import { IEnvironment } from 'app/interfaces/environment.interface';

export const baseEnvironment: IEnvironment = {
  production: false,
  apiBaseUrl: 'https://api.dev.myapp.com',
}
```

This is the default `dev` environment: `/src/environments/environment.ts`:
```typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  // Nothing is overriden, all will be as defined in base environment
};
```

This is the staging environment: `/src/environments/environment.ts`:
```typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  apiBaseUrl: 'https://api.staging.myapp.com',
};
```

This is the production environment: `/src/environments/environment.prod.ts`:
```typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  production: true,
  apiBaseUrl: 'https://api.myapp.com',
};
```

Define environments in `.angular-cli.json`:
```json
{
  ...
  "apps": [{
    ...
    "environments": {
      "dev": "environments/environment.ts",
      "prod": "environments/environment.prod.ts",
      "staging": "environments/environment.staging.ts"
    }
    ...
  }]
  ...
}
```

After all is defined, run/build the app for different environments by specifying the environment in `ng` command: `ng build --env=prod` for production build; `ng build --env=staging` for staging build; `ng build` will use default `dev` environment.

## Formatting and Naming

### In Templates


For the purposes of future-proofing and avoiding conflicts with other libs, prefix all component selectors with something unique/app-specific. Specify the prefix in `.angular-cli.json` like so: `"prefix": "my-app",`.

----

Try to fit all inputs, outputs and regular HTML attributes in one line. If there is simply too many of them, break it up like so:

```html
// bad
<my-app-component *ngFor="let item of items" class="foo" [bar]="something ? 'foo' : 'oof'" (click)="onClick($event)">
  <div>Transcluded Content</div>
</my-app-component>

// bad
<my-app-component
  *ngFor="let item of items"
  class="foo"
  [bar]="something ? 'foo' : 'oof'"
  (click)="onClick($event)">
  <div>Transcluded Content</div>
</my-app-component>

// maybe could be good?
<my-app-component
  *ngFor="let item of items"
  class="foo"
  [bar]="something ? 'foo' : 'oof'"
  (click)="onClick($event)">

  <div>Transcluded Content</div>
</my-app-component>

// good
<my-app-component
  *ngFor="let item of items"
  class="foo"
  [bar]="something ? 'foo' : 'oof'"
  (click)="onClick($event)"
>
  <div>Transcluded Content</div>
</my-app-component>
```

----

If you are passing data to a component, use property binding only if necessary:

```html
// bad
<my-app-component [name]="'Steve'"></my-app-component>

// good
<my-app-component name="Steve"></my-app-component>
```

---

Prefix output handlers with `on`:

```html
// bad
<my-app-component (somethingHappened)="onSomethingHappened($event)">

// bad
<my-app-component (onSomethingHappened)="onSomethingHappened($event)">

// bad
<my-app-component (somethingHappened)="somethingHappened($event)">

// good
<my-app-component (somethingHappened)="onSomethingHappened($event)">
```

----

Use `ng-container` when possible to reduce the amount of generated DOM elements:

```html
// bad
<div *ngIf="something"></div>

// good
<ng-container *ngIf="something"></ng-container>

// good
<div class="i-need-this-class" *ngIf="something"></div>
```

----

Use `*ngIf; else`:

```html
// bad
<ng-container *ngIf="something"></ng-container>
<ng-container *ngIf="!something"></ng-container>

// good
<ng-container *ngIf="something; else anotherThing"></ng-container>
<ng-template #anotherThing></ng-template>
```

----

Subscribe to observables with `async` pipe if possible:

```typescript
// component
@Component(...)
export class ArticlesList {
  public articles$: Observable<Array<Article>> = this.articleService.fetchArticles();
  public menu$: Observable<Menu> = this.articleService.fetchMenu();
  ...
}
```
```html
// template
<my-app-menu *ngIf="menu$ | async as articlesMenu" [menu]="articlesMenu">
</my-app-menu>

<my-app-article-details *ngFor="let article of articles$ | async" [article]="article">
</my-app-article-details>
```

----

Use attributes for transclusion selectors:

```html
// Wrapper component
<div class="wrapper">
  <p>Wrapper content:</p>
  <ng-content select="[some-stuff]"></ng-content>
</div>
```
```html
// Some other component
<my-app-wrapper>
  <div class="some-item" some-stuff>Some stuff</div>
</my-app-wrapper>
```

### In Code

## Routing and Lazy Loading

## Importing

## Server-side Rendering

## Working with JSON API

## Working with Forms

## Inner Workings and Common Pitfalls
