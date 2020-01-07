# Angular guidelines and best practices

<center><img style="width: 100%; max-width: 1000px" src="/img/angular.svg"></center>

*Guides are not rules and should not be followed blindly. Use your head and think.*

## Core libraries, configuration and tools

### [Angular CLI](https://github.com/angular/angular-cli)

The best way to create a new project is by using Angular CLI. It will hide a lot of configuration behind the scenes (you will not have a webpack config exposed for editing), do some optimizations when compiling your code, and offer some handy scaffolding tools.

Install Angular CLI globally with `npm install -g @angular/cli` and check out what you can do with `ng help`.

**Creating a new project**

Use Angular CLI for the initial project setup:

```bash
ng new my-app
```

As of version 7, there will be a quick wizard-like setup advising you about routing and styling. Choose depending on your needs. All of our Angular projects here at Infinum use routing and SCSS styling.

**Scaffolding**

You can use Angular CLI to generate new modules, components, services, directives, etc.

The command is `ng generate [schematic-name]`, for example, `ng generate component`. There is also a shorthand for many generation commands, for example, `ng g c` === `ng generate component`. Run `ng generate --help` to see a list of all available options and schematics.

**Creating new modules**

Use `ng g m MyModule` to create a new module. If the module should have routes, you can generate a module with a routing skeleton with `ng g m MyModule --routing`.

**Creating new components**

Create components with `ng g c MyComponent` or `ng g c my-component`. In both cases, the resulting component class name will be `MyComponent`, and dir name will be `my-component`.

Usually, you want to create a component module when creating components. This makes the component more modular and ensures that everything that the component needs is provided by the component module. The component module should declare and export the component—you can think of it as a public API. The component module could also declare some other "internal" components, but it does not have to necessarily export those internal components. It is our recommendation that component modules export only one declared component.

Also remember that a component can be declared only once (in one module), so it makes sense for reusable components to have their own module.

Example:

```bash
# create a module
ng g m Calendar

# create the component - this will also automatically declare the component in the above created module
ng g c Calendar

# make sure to export CalendarComponent in CalendarModule

cd calendar

# add some "internal" components which are not exported
ng g c WeekView
ng g c DayView

# WeekView and DayView do not have to be exported, as they are used only internally
```

``` typescript
@NgModule({
  imports: [
    FormsModule,
    MatButtonModule,
  ],
  declarations: [
    CalendarComponent,
    WeekViewComponent,
    DayViewComponent,
  ],
  exports: [
    CalendarComponent,
  ]
})
export class CalendarModule { }
```

**Creating new services**

Use `ng g s MyService` to generate a new service. NOTE: unlike during component generation, service generation will not create a new dir for the service (although that might change in the future). The solution is to simply prefix the directory name, and the command will generate the directory and the service: `ng g s my-service/my-service`.

**Ejecting**

Do not do it, but if you *absolutely have to* edit the webpack config manually, you can eject the CLI's webpack config with `ng eject`.

**Extending Angular CLI's webpack config**

Since Angular CLI version 6, there has been a way to extend the internal webpack config which is used by the CLI by using custom builders. To check out the details on how to do this, please have a look at [@angular-builders/custom-webpack](https://github.com/meltedspark/angular-builders/tree/master/packages/custom-webpack). This should be enough in most cases, so you will most likely never have to eject the internal webpack config.

One good example of extending the CLI's webpack config can be seen in [Guess.js's Angular implementation](https://guess-js.github.io/docs/angular) for predictive prefetching of lazy-loaded modules.

**DevServer proxy**

If you have issues with CORS, it is OK to temporarily use the Webpack DevServer proxy for development purposes. You can do this without ejecting the webpack config, [as instructed](https://angular.io/guide/build#proxying-to-a-backend-server).

**Other commands**

This handbook will not cover all `ng` commands; please check out [Angular CLI Wiki](https://github.com/angular/angular-cli/wiki) for more info.

### [Typings](https://angular.io/guide/typescript-configuration)

If you are using a JS library that is not written in TypeScript, you can usually install typings separately. Install typings as a dev-dependency, for example:

```bash
npm install --save d3
npm install --save-dev @types/d3
```

If there are no typings available, you can create your own `typings.d.ts` file and write some custom typings for your needs. They do not even have to be 100% complete; they can cover only the functionality which you use.

### [Recommended Editor](https://xkcd.com/378/)

<center><img style="width: 100%; max-width: 740px" src="/img/real_programmers.png"></center>

At Infinum, we recommend using [VSCode](https://code.visualstudio.com/) for Angular development, as it has a great integration with TypeScript and has some handy Angular plugins:

- [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template)—IntelliSense in templates
- [Angular 7 Snippets](https://marketplace.visualstudio.com/items?itemName=Mikael.Angular-BeastCode)
- [TSLint](https://marketplace.visualstudio.com/items?itemName=eg2.tslint)
- [stylelint](https://marketplace.visualstudio.com/items?itemName=shinnn.stylelint)
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)

### [RxJS](https://github.com/ReactiveX/rxjs)

Angular depends heavily on RxJS, so getting familiar with RxJS is pretty important. It is used for all event handling and can even be used for state management.

RxJS is the implementation of the [Reactive Extensions Library](http://reactivex.io/) for JavaScript. Angular isn't the only place where you can come across RxJS—it can also be used with Vanilla JS or some other framework. Even some Android and iOS developers use Rx for their respective platforms. While the implementations can differ a bit, learning just one of the implementations (for example RxJS) will introduce you to a lot of concepts which are common for all Rx implementations. You can then get into a heated mergeMap vs switchMap discussion with Android and/or iOS developers. Isn't that magnificent?

Please check out some RxJS tutorials if you are not familiar with Rx. Please note that there have been some breaking changes in RxJS version 6. Some of the APIs have changed (mostly the way we use operators), and many tutorials that were created earlier are now using older versions of RxJS. That being said, not much has changed conceptually, just the way you write some RxJS functionalities.

Here are some good introductory tutorials to get you started:

- Academind—[Understanding RxJS](https://www.youtube.com/watch?v=T9wOu11uU6U&list=PL55RiY5tL51pHpagYcrN9ubNLVXF8rGVi) playlist
- Interactive diagrams—[RxJS Marbles](https://rxmarbles.com/)
  - this will help you a lot when trying to understand what specific operators do
- Angular documentation—[The RxJS Library](https://angular.io/guide/rx-library)

**Observers and subscribers**

When you want to observe a value, use some of the classes derived from `Observable`. Most use cases are covered with one of these:

- `Subject`
- `BehaviorSubject`
- `ReplaySubject`

You can subscribe to all classes derived from `Observable` in order to receive value changes. You can also use `operators` to manipulate how and when the value changes are sent to the subscribers.

There is a naming convention for `Observable` variables—postfixing with a dollar sign:

``` typescript
// bad
const observable: Observable;

// good
const observable$: Observable;

// good
const mySubject$: Subject;
```

**Using and creating operators**

If you are using RxJS 5.5 or newer, make sure to use *[pipeable operators](https://github.com/ReactiveX/rxjs/blob/master/doc/pipeable-operators.md)* instead of *patch* operators.

``` typescript
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
- `switchMap`

Avoid doing a lot of logic in the `subscribe` callback. Subscribe as late as possible, and do all the necessary error catching, side-effects, and transformations via operators.

You can also write your own operators. If you do so, it is strongly recommended to write them as [pure pipeable operator functions](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md#operator-as-a-pure-function).

**async/await**

You can use async/await to await the completion of observables. Keep in mind that this will not work if the observable emits more than one value. For example, it is OK for things like HTTP requests.

To convert an observable to a promise, just call `obs$.toPromise()` and then you can await it.

One important thing to note is that you are waiting for the **completion**, not **emission** of values. Knowing that, what do you expect the result to be in the following example?

``` typescript
const source$: Subject<string> = new Subject();
const sourcePromise: Promise<string> = source$.toPromise();

source$.next('foo');
source$.next('bar');
source$.complete();

const result = await sourcePromise;
```

Spoiler alert! The result will be `bar`, and if we do not complete the source observable, awaiting will hang indefinitely.

## File/module organization and naming

If you have created the app using Angular CLI, there is already some clear structure in place. Building on that structure, you will probably want to add some more dirs to help you organize your code. Here are some guidelines:

- Try to split up your application into modules which you can reuse. If, for example, you have to handle lazy loading of all images in the app, create an `Image` module and component in `app/components` and use it in other modules/components whenever you have to show an image. Export the declared component and import/provide dependencies which your component uses.

- Container components that are loaded directly via routing (by either `component` or `loadChildren`) should be placed in the `app/pages` dir. If a component is loaded because you have used it in a template, that component should be in `app/components`.
  - One little exception here: if there is a very specific component tied to some routing submodule which is used only in that submodule, it is OK to leave it in that submodule (but move it to `app/components` as soon as it is used in another routing submodule).

- Store all your animations in `app/animations/` and import them for use in component decorators. If you want to have some animation properties, such as a duration configurable, you can export the "animation factory" function which returns the actual `AnimationTriggerMetadata`.

- Classes that might be used throughout the application (e.g., generic `Dictionary` class) should be in `app/classes`.

- Entity models (`User`, `Post`, etc.) should be in `app/models`.

- If you are using some JS libs which do not have typings, add your own custom typing in `app/custom-typings`.

- Whenever you have an attribute whose value can be some value from a set of values, make an enum and store that enum in `app/enums`. A good example is an enum for HTTP status codes. If you need some metadata for the enum, create an enum data file which exports what is basically a dictionary with enum values used as keys, and use whatever you need for values. One such example would be: `export const httpStatusCodeData = { [HttpStatus.OK_200]: { translationKey: 'http.success' } }`. If the enum data is a bit more complex, defining an interface for it can be useful.

- If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s, and validators in `app/forms`. If you have form components that are reused throughout the app, put them in `app/components/forms`.

- Put all route guards in `app/guards`.

- If you need some helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `app/helpers` in its own dir. Make sure to write tests for it and simply export it in `app/helpers/index.ts`.

- Save all your common interface files to `app/interfaces`.

- If you have some custom pipes for use in templates, create their own dirs in `app/pipes`, similar to helpers. Declare, export, and provide the pipes in `app/pipes/pipes.module.ts`.

- Services should be placed in `app/services`, each service in its own dir.

- If you have any `NgModule`s which are not components (e.g., a module which imports all specific material modules that you need), put them in `app/shared-modules`.

- TypeScript `type` declarations should be placed in `app/types`.

- As defined in angular-cli, assets placed in `assets` will be served statically.

- Global styles should be placed in the `app/styles` dir. The styles dir has a very similar structure as that described in the [SASS Styleguide](https://handbook.infinum.co/books/frontend/SASS%20Styleguide/File%20organization), so please check it out.

- All environment files are placed in `environments`.

When it all comes together, your src folder might look something like this:
*(please note the postfixes in filenames, such as .guard, .interface, .model, etc.)*

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
      - index.ts
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
  - styles/
      - components/
          - _title.scss
      - overrides/
          - _button.scss
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
      - _core.scss
      - main.scss
- assets/
    - fonts/
        - CustomFont.eot
        - CustomFont.ttf
        - CustomFont.woff
        - CustomFont.woff2
    - images/
        - icons/
            - app-icon.svg
- environments/
    - environment.base.ts
    - environment.prod.ts
    - environment.staging.ts
    - environment.ts

## Presentational and smart/container components

Design your components to be as small as possible and reuse them as much as possible. Philosophically, components can be divided into two types:

- Presentational (dumb)
  - in charge of displaying the data which is passed down to them
  - should not make use of services
  - pass events up instead of doing complex things with them
  - use `changeDetection: ChangeDetectionStrategy.OnPush` in the `@Component` declaration

- Container (smart)
  - use multiple (if necessary) other components
  - serve data to presentational components
  - handle events from child components
  - control UX flow
  - work with services to make things happen (fetch, store, change state)
  - can be loaded directly or lazy loaded when routing
  - use `changeDetection: ChangeDetectionStrategy.OnPush` and pass data down using the `async` pipe

More on this philosophy can be found in various online articles. Some articles have framework-specific information, but the core concepts apply no matter which framework you use. A good article from Angular University on this topic: [Angular Architecture—Smart Components vs Presentational Components](https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/).

## Environments

Environment files should be used for some global configuration changes that can differ between development and production builds. Keep in mind that these values are bundled in build at build-time and cannot be changed after the build (at least not in an acceptable way). This can be an issue if the application is deployed using Docker, in which case you should use some SSR solution or script injection instead of defining variables via Angular's environment files (more on this in the SSR section).

Define the interface for the environment: `app/interfaces/environment.interface`:

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

Make sure to update `angular.json` if you add/remove environment files.

After everything is defined, run/build the app for different environments by specifying the environment in an `ng` command: `ng build --env=prod` for production build; `ng build --env=staging` for staging build; `ng build` will use the default `dev` environment.

## Formatting, naming and best practices

### In templates

For the purposes of future-proofing and avoiding conflicts with other libs, prefix all component selectors with something unique/app-specific. Specify the prefix in `angular.json` like this: `"prefix": "my-app"` (needless to say, pick a better prefix than `my-app`).

----

**Try to fit all inputs, outputs, and regular HTML attributes in one line. If there is simply too many of them, break it up like this:**

``` html
<!-- bad -->
<my-app-component *ngFor="let item of items" class="foo" [bar]="something ? 'foo' : 'oof'" (click)="onClick($event)">
  <div>Transcluded Content</div>
</my-app-component>

<!-- bad -->
<my-app-component
  *ngFor="let item of items"
  class="foo"
  [bar]="something ? 'foo' : 'oof'"
  (click)="onClick($event)">
  <div>Transcluded Content</div>
</my-app-component>

<!-- bad -->
<my-app-component
  *ngFor="let item of items"
  class="foo"
  [bar]="something ? 'foo' : 'oof'"
  (click)="onClick($event)">

  <div>Transcluded Content</div>
</my-app-component>

<!-- good -->
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

**Closing elements and angle brackets placement:**

Case without content:

``` html
<!-- bad -->
<my-component
  class="foo"
  [someInput]="someValue"
  (click)="onSomeClick()"
>
</my-component>

<!-- good -->
<my-component
  class="foo"
  [someInput]="someValue"
  (click)="onSomeClick()"
></my-component>
```

Case with content:

``` html
<!-- bad -->
<my-component
  class="foo"
  [someInput]="someValue"
  (click)="onSomeClick()"
>Some content</my-component>

<!-- bad -->
<my-component
  class="foo"
  [someInput]="someValue"
  (click)="onSomeClick()">

  Some content
</my-component>

<!-- good -->
<my-component
  class="foo"
  [someInput]="someValue"
  (click)="onSomeClick()"
>
  Some content
</my-component>
```

----

**If you are passing data to a component, use property binding only if necessary:**

``` html
<!-- bad -->
<my-app-component [name]="'Steve'"></my-app-component>

<!-- good -->
<my-app-component name="Steve"></my-app-component>
```

----

**Prefix output handlers with `on`:**

``` html
<!-- bad -->
<my-app-component (somethingHappened)="somethingHappened($event)">

<!-- bad -->
<my-app-component (onSomethingHappened)="somethingHappened($event)">

<!-- bad -->
<my-app-component (onSomethingHappened)="onSomethingHappened($event)">

<!-- good -->
<my-app-component (somethingHappened)="onSomethingHappened($event)">
```

----

**Name click handlers in a descriptive manner:**

``` html
<!-- bad -->
<button (click)="onClick()">Log in</button>

<!-- bad - use infinitive instead of past form -->
<button (click)="onLogInClicked()">Log in</button>

<!-- bad - too much -->
<button (click)="onLogInButtonClick()">Log in</button>

<!-- good -->
<button (click)="onLogInClick()">Log in</button>
```

----

**Use a safe navigation operator:**

``` html
<!-- bad -->
<div *ngIf="user && user.loggedIn"></div>

<!-- good -->
<div *ngIf="user?.loggedIn"></div>
```

----

**Add space around curly brackets used for interpolation:**

``` html
<!-- bad -->
<div>{{userName}}</div>

<!-- good -->
<div>{{ userName }}</div>
```

----

**Use `ng-container` when possible to reduce the amount of generated DOM elements:**

``` html
<!-- bad -->
<div *ngIf="something"></div>

<!-- good -->
<ng-container *ngIf="something"></ng-container>

<!-- good -->
<div *ngIf="something" class="i-need-this-class"></div>
```

----

**Use `*ngIf; else`:**

``` html
<!-- bad -->
<ng-container *ngIf="something"></ng-container>
<ng-container *ngIf="!something"></ng-container>

<!-- good -->
<ng-container *ngIf="something; else anotherThing"></ng-container>
<ng-template #anotherThing></ng-template>
```

----

**Subscribe to observables with the `async` pipe if possible:**

Component:

``` typescript
@Component(...)
export class ArticlesList {
  public articles$: Observable<Array<Article>> = this.articleService.fetchArticles();
  public menu$: Observable<Menu> = this.articleService.fetchMenu();
  ...
}
```

Template:

``` html
<my-app-menu
  *ngIf="menu$ | async as articlesMenu"
  [menu]="articlesMenu"
></my-app-menu>

<my-app-article-details
  *ngFor="let article of articles$ | async"
  [article]="article"
></my-app-article-details>
```

The `async` pipe is especially useful if `changeDetection` is `OnPush` since it subscribes, calls change detection on changes, and unsubscribes when the component is destroyed.

----

**Use attributes for transclusion selectors:**

The `my-app-wrapper` component template:

``` html
<div class="wrapper">
  <p>Wrapper content:</p>
  <ng-content select="[some-stuff]"></ng-content>
</div>
```

Some other component's template:

``` html
<my-app-wrapper>
  <div class="some-item" some-stuff>Some stuff</div>
</my-app-wrapper>
```

----

**Order attributes and bindings:**

1. Structural directives (`*ngFor`, `*ngIf`, etc.)
2. Animation triggers (`@fade`, `[@fade]`)
3. Element reference (`#myComponent`)
4. Ordinary attributes (`class`, etc.)
5. Non-interpolated string inputs (`foo="bar"`)
6. Interpolated inputs (`[foo]="theBar"`)
7. Two-way bindings (`[(ngModel)]="email"`)
8. Outputs (`(click)="onClick()"`)

``` html
<!-- bad -->
<my-app-component
  (click)="onClick($event)"
  [(ngModel)]="fooBar"
  class="foo"
  #myComponent
  [foo]="theBar"
  @fade
  foo="bar"
  *ngIf="shouldShow"
  (someEvent)="onSomeEvent($event)"
></my-app-component>

<!-- good -->
<my-app-component
  *ngIf="shouldShow"
  @fade
  #myComponent
  class="foo"
  foo="bar"
  [foo]="theBar"
  [(ngModel)]="fooBar"
  (click)="onClick($event)"
  (someEvent)="onSomeEvent($event)"
></my-app-component>
```

----

**Prefer the `[class]` over `[ngClass]` syntax:**

``` html
<!-- bad -->
<div [ngClass]="{ 'active': isActive }"></div>

<!-- good -->
<div [class.active]="isActive"></div>
```

### In code

**Prefix interfaces**

Interfaces should be prefixed with `I`. This might be a polarizing decision, but you should do it because you might have cases where some class implements an interface, and you also have a stub class which implements the same interface.

``` typescript
// bad
interface UserService { }

// good
interface IUserService { }

// if we prefix we can do this:
class UserService implements IUserService { }
class UserServiceStub implements IUserService { }
```

----

**Prefer interface over type**

When defining data structures, prefer using interfaces over types. Use types only for those things for which interfaces cannot be used—unions of different types.

``` typescript
// bad
export type User = {
  name: string;
};

// good
export interface IUser {
  name: string;
}

// good
export type CSSPropertyValue = number | string;
```

You might be tempted to use a type for a union of models. For example, in some CMS solutions you might have `AdminModel` and `AuthorModel`. The user can log in using either admin or author credentials.

``` typescript
export class AdminModel {
  public id: string;
  public permissions: Array<Permission>;
}

export class AuthorModel {
  public id: string;
  public posts: Array<Post>;
}

export type User = AdminModel | AuthorModel;

// in some component
@Input() public user: User;

// in template
User id: {{ user.id }}
```

This might seem fine at first, but it is actually an anti-pattern. What you should actually do in a case like this is create an abstraction above `AdminModel` and `AuthorModel`:

``` typescript
export abstract class User {
  public id: string;
}

export class AdminModel extends User {
  public permissions: Array<Permission>;
}

export class AuthorModel extends User {
  public posts: Array<Post>;
}
```

**TL;DR:** Prefer interfaces and abstractions over types. Use types only when you need union types.

----

**Renaming RxJS imports**

Some RxJS imports have very generic names, so you might want to rename them during import:

``` typescript
import {
  of as observableOf
  EMPTY as emptyObservable
} from 'rxjs';
```

----

**Subscribe late and pipe**

If you are new to RxJS, you will probably be overwhelmed by the amount of operators and the "Rx-way" of doing things. To do things in the "Rx-way", you will have to embrace the usage of operators and think carefully when and how you subscribe.

A rule of thumb is to subscribe as little and as late as possible, and do minimal amount of work in subscription callbacks.

We will demonstrate this with an example. Imagine you have a stream of data to which you would like to subscribe and transform the data in some way.

``` typescript
interface IUser {
  id: number;
  user_name: string;
  name: string;
  surname: string;
}
```

```
/account/me returns JSON:
{
  "id": 42,
  "user_name": "TheDude",
  "name": "Jeff",
  "surname": "Lebowski"
}
```

``` typescript
// bad
const user$: Observable<IUser> = http.get('/account/me');

user$.subscribe((response: IUser) => {
  console.log(response.user_name); // TheDude
});
```

``` typescript
// good
const userName$: Observable<string> = http.get('/account/me').pipe(
  map((response: IUser) => response.user_name)
);

userName$.subscribe(console.log); // TheDude
```

The reason why is because there might be multiple subscribers who want the same thing, and they would all have to do the same transformation in their respective success callbacks.

It is, of course, possible that the subscribers want different things, and for those cases you might want to expose some observable with fewer piped operations.

The point here is that, as soon as you notice that you are repeating yourself in subscription callbacks, you should move that repeated logic into some operator that is piped to the source observable.

----

**No subscriptions in guards**

Asynchronous guards are common, but they should not subscribe to anything; they should return an observable.

To demonstrate this on an example, imagine we have `AuthService` and the `getUserData` method which returns an observable. The observable will return the `User` model if the user is authenticated or emit an error if not.

We would like to implement a guard which allows only authenticated users to navigate to the route.

``` typescript
// bad
class UserAuthorizedGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  canActivate(): Observable<boolean> {
    const authStatus$ = new Subject();

    this.authService.getUserData().subscribe((user: User) => {
      authStatus$.next(true);
      authStatus$.complete();
    }, (error) => {
      console.error('User not authorized!', error);

      authStatus$.next(false);
      authStatus$.complete();
    })

    return authStatus$;
  }
}

// good
class UserAuthorizedGuard implements CanActivate {
  constructor(private authService: AuthService) {}

  canActivate(): Observable<boolean> {
    return this.authService.getUserData().pipe(
      catchError((error) => {
        console.error('User not authorized!', error);

        return observableOf(false);
      }),
      map(Boolean), // we could technically omit this mapping to Boolean since User model will be a truthy value anyway
    )
  }
}
```

----

**Be mindful of how and when data is fetched**

There are two basic approaches to data loading:

1. Via route resolve guards
2. Via container components

**Resolve guards**

A [resolve guard](https://angular.io/guide/router#resolve-pre-fetching-component-data) can be used to pre-fetch the data necessary for component rendering. The advantages of using resolve guards are:

- No need to fetch data on component initialization
  - Data is fetched during the routing event and ready when the component is initialized
- No need to implement logic for not showing the component until the data is fetched
  - When the component starts rendering, you know you will have the data
- A loader showing/hiding logic can be implemented in one place instead of in each component that loads data
  - If data is loaded via a resolve guard for all routes, you can have a global loader logic which hooks into router events, thus implementing the loaded showing/hiding logic only once

**Container components**

Even though guards are really easy to use and have some advantages, there might be situations where the data has to be loaded through multiple requests or the loading might be slow. In those cases, it might be worth to consider using a container component for data loading instead of guards. The container component would, of course, use some service to make the requests and then show the data once the requests have been completed. In this way, it is possible to show partial data as it comes in, and implement empty state design for each chunk of data that is shown.

A good example where partial data loading via a container component can be preferred over all-at-once loading via guards is a dashboard-like page with multiple graphs where each graph shows data from one request. In such cases, it is probably better to let the container component handle data loading, and presentational components (graphs) should implement an empty state with some nice loaders.

Bottom line:

- Use resolve guards if route data can be loaded in one big chunk and is also shown all-at-once.
- Use container components if data is loaded in chunks and results should be shown as they come in.

----

**Ordering class members (including getters and lifecycle hooks)**

Follow this guideline when ordering class members:

1. `@Input`s
2. `@Output`s
3. public, protected, and private properties
4. constructor
5. lifecycle hooks
6. public, protected, and private getters and setters
7. public, protected, and private methods

Example:

``` typescript
class MyComponent implements OnInit, OnChanges {
  @Input() public i1: string;
  @Input() public i2: string;
  @Output() public o2: EventEmitter<string> = new EventEmitter();
  public attr1: number;
  protected attr2: string;
  private attr3: Date;

  constructor(...) { ... }

  public ngOnInit(): void { ... }

  public ngOnChanges(): void { ... }

  public get computedProp(): string { ... }

  private get secretComputedProp(): number { ... }

  public onSomeButtonClick(event: MouseEvent): void { ... }

  private someInternalAction(): void { ... }
}
```

## Dependency injection

DI is utilized heavily in Angular, primarily for providing instances of either primitive values or class instances.

DI is a very complex topic and requires a separate discussion which would be out of the scope of this handbook. Make sure to check out the [official DI guide](https://angular.io/guide/dependency-injection) which explains many details. You can also have a look at [this repo](https://github.com/fvoska/angular-di-demo) for some examples.

As of Angular 6, make sure to use `{ providedIn: 'root' }` [for singletons](https://angular.io/guide/singleton-services#providing-a-singleton-service) whenever possible.

DI is very useful for testing purposes, as shown in a later chapter.

## Angular Universal (server-side rendering)

When setting up server-side rendering, follow the [official guide](https://angular.io/guide/universal).

There are many tricks with SSR, even after you finish the setup. You can check out [this repo](https://github.com/fvoska/angular-universal-demo) to find out how to solve many issues. This includes reading environment variables at runtime, so it is worth checking out.

## Two-way binding

Just like AngularJS, Angular has support for two-way binding, even though it works quite differently and is a lot less magical than in AngularJS.

Angular's [binding syntax](https://angular.io/guide/template-syntax#binding-syntax-an-overview) offers three directions:

- One-way from data source to view target (`[someInput]="expression"`)
- One-way from view target to data source (`(someEvent)="onSomeEvent()"`)
- Two-way (`[(ngModel)]="value"`)

If you have worked with template-driven forms, you've probably come across `ngModel` and used two-way data binding. Just like you can use two-way binding with `ngModel`, you can also use the two-way binding of any input of your custom component.

This handbook will not cover how to implement a custom two-way binding for your components. We suggest checking out this article: [Two-way Data Binding in Angular](https://blog.thoughtram.io/angular/2016/10/13/two-way-data-binding-in-angular-2.html), which explains how two-way binding works, and how you can implement your own.

One key takeaway which we will repeat in this handbook is how two-way binding is de-sugared.

``` html
<my-counter [(value)]="counterValue"></my-counter>
<my-counter [value]="counterValue" (valueChange)="counterValue=$event"></my-counter>
```

These two lines do exactly the same thing—that might give you an idea how to go about implementing custom two-way binding for your components. :) For more details, we suggest reading the official docs and articles that we mentioned a bit earlier.

### Should you use two-way binding?

If you've checked out the docs and the article, you've probably noticed that implementing two-way binding for your components isn't rocket science. You might be thinking "OH MY GOD, I WILL ENABLE TWO-WAY BINDING FOR ALL MY COMPONENTS". Hold on, you might not actually need it.

We recommend implementing two-way binding for components which hold some internal state, and you want to be able to pass values down and also update outer component value if something changes internally. A good example would be some kind of a counter component.

Another type of component for which you might consider implementing two-way binding are the components used in a similar manner as inputs, checkboxes, and similar elements that are used within forms. For such cases, it is usually better to implement a [ControlValueAccessor](https://angular.io/api/forms/ControlValueAccessor) instead. More on that in the section about working with forms.

## Working with forms

As you might already know, there are two approaches when working with forms in Angular—template-driven and reactive forms. There is a lot of great documentation on both:

- [Reactive Forms](https://angular.io/guide/reactive-forms)
- [Template-driven Forms](https://angular.io/guide/forms)

The real question is which approach should you use? General reasoning is that template-driven forms should suffice if the form is simple enough. If the form is complex and has a lot of validations, the reactive approach is easier to manage.

Going by that general reasoning, you might end up having a mix of reactive and template-driven forms in your application. For that reason, here at Infinum, we usually lean towards reactive forms. While they do have some initial overhead, forms often get more complex over time. If you start off with using reactive forms, there is no need to switch from template-driven forms when they get harder to manage.

### Our own library for working with forms!

Since we mostly use reactive forms, we've created a library, which makes things a bit easier. The library in question is [ngx-form-object](https://github.com/infinum/ngx-form-object). Check out the project's README on GitHub to find out how to use it.

Some of the benefits of using it:

- Definition of form attributes on the model level using decorators
- Support for single-values and relationships (Attribute, BelongsTo, HasMany)
- Introduces `FormObject` and `FormStore` for decoupling data and validation
- Form generation based on model definition

We definitely recommend trying out `ngx-form-object` (*shameful self-promotion*) if you decide to use reactive forms. We've used it on many complex projects with huge forms and it has helped us a lot!

### Making your components work with forms transparently

Another topic that should be covered when talking about forms is using your own components with both reactive and template-driven forms. To use your component with forms just like you would use the `input` element, you will have to implement [ControlValueAccessor](https://angular.io/api/forms/ControlValueAccessor) in your component. It isn't too straight-forward and might seem complex at first glance (what is `forwardRef` anyway, right?), but luckily, there are some great articles which we recommend reading:

- [Angular Custom Form Controls Made Easy](https://netbasal.com/angular-custom-form-controls-made-easy-4f963341c8e2)—quick introduction
- [Never Again be Confused when Implementing ControlValueAccessor in Angular Forms](https://blog.angularindepth.com/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms-93b9eee9ee83)—more in-depth, great article, recommend reading; but uses jQuery in the example—so démodé

You can implement both two-way binding and `ControlValueAccessor` if you want your component to be usable both inside a form and outside the form, as a stand-alone. However, you will usually not use the same component both inside and outside a form, so implementing just `ControlValueAccessor` should be enough.

## Testing

Writing good quality tests for critical functionalities is an important step in quality assurance. We do not recommend not writing any tests, nor do we recommend mindlessly trying to achieve 100% coverage. Applications with 100% coverage can still have bugs either because the tests are not good enough, or if the edge cases were not covered by the implementation in the first place.

There are many tips and tricks that go into testing Angular applications. This section will cover some of them.

### Unit vs. integration vs. end-to-end testing

Tests can usually be placed into one of the three categories: unit, integration, or end-to-end tests.

**Unit testing**

Unit tests, as the name suggests, test units. A unit is some individual piece of software which can also have some external dependencies. In the context of Angular, units are components, services, guards, pipes, helper functions, interceptors, models and other custom classes, etc.

Unit testing in Angular comes out-of-the-box with Jasmine, as both the testing framework and assertion library. It is also possible to generate good coverage reports as HTML files which can be presented to management or some other interested parties.

**Integration testing**

Integration testing includes multiple units which are tested together to check how they interact. In Angular, there is no special way of doing integration testing. There is a thin line between unit and integration tests, and that line is more conceptual than technical. For integration testing, you still use Jasmine, which comes with every project generated by Angular CLI.

What makes a test an integration test instead of a unit test? Well, it depends on how much you mock during testing. If component A which you are testing renders components B, C, and D, and depends on services S1 and S2, you have a choice:

1. You can test component A using mocks for components B, C, and D, and mocks for services S1 and S2. We could call that a unit test of component A since it is testing only component A's functionality, and it assumes that other components and services work as expected.
2. You can test component A using real components B, C, and D and real services S1 and S2. This will make the `TestBed` module configuration a bit heavier, and tests might run a bit slower. However, we could call this an integration test of the interaction between components A, B, C, and D and services S1 and S2.

So, it is really up to you to decide whether you want to test some component with all its dependencies mocked, or if you want to test interaction with other real components as well.

What you call those Jasmine tests—unit or integration—doesn't really matter, ♪ *anyone can see* ♪.

**E2E testing**

End-to-end testing is quite different when compared to unit/integration testing with Jasmine. Angular CLI projects come with a separate E2E testing project which uses Protractor. [Protractor](https://www.protractortest.org/) is basically a wrapper around [Selenium](https://www.seleniumhq.org/), which allows us to write tests in JavaScript, and it also has some Angular-specific helper functions.

When you start E2E tests, your application will be built, and a programmatically controlled instance of a web browser will be opened. The tests will then click on various elements of the webpage without any human input (via [WebDriver](https://www.seleniumhq.org/projects/webdriver/)). This allows us to create a quick smoke-testing type of tests which goes through the main flows of our application in an automated way.

Covering E2E testing in detail is out of the scope of this handbook. Please check out the [official documentation](https://angular.io/cli/e2e) if you would like to know more. However, we do have some quick tips and tricks:

- the `ng e2e` command builds your app and starts a local DevServer just like `ng serve`, and then it runs Protractor on that local instance of your app. If you want to run e2e tests on some specific environment instead of a local environment, you can run Protractor directly instead of via Angular CLI, simply by executing `protractor` (make sure to install it globally, use binary from node_modules, or create a script in package.json). In `protractor.conf.js`, you can configure `baseUrl` to point to the URL of your app on the desired environment.
- You can use environment variables in Protractor tests. For example, if you have a login functionality test, you can do something like this: `$('input[data-e2e-test="login-email"]').sendKeys(process.env.E2E_LOGIN_EMAIL);`.
- You can set up some pre-conditions using the `onPrepare` hook in `protractor.conf.js`.
- If you are using async/await in your e2e tests, make sure to [check this out](https://github.com/angular/protractor/blob/master/docs/async-await.md)

We should mention that Protractor isn't the only solution for e2e testing, but it does come out-of-the box with Angular CLI generated projects. Honestly, writing Protractor tests is often a painful process (especially if you are writing tests using [Cucumber](https://cucumber.io/)). There are some newer e2e testing frameworks which might be worth checking out:

- [Cypress.io](https://www.cypress.io/)
- [WebdriverIO](https://webdriver.io/)
- [Puppeteer](https://developers.google.com/web/tools/puppeteer/), although e2e testing is not its primary or only intended use

### Utilizing TestBed

Dependency injection is very powerful during testing, as it allows you to provide mock services for testing purposes. You will sometimes want to spy on some private service's methods. This might tempt you to make the service public or to bypass TS by doing something like `component['myService']`. To avoid these *dirty* solutions, you can utilize `TestBed`.

Consider this example with a header component and user service which is used for logging in:

``` html
...
<button (click)="onLogInClick">Log in</button>
...
```

``` typescript
@Component({ ... })
export class HeaderComponent {
  constructor(private userService: UserService)

  onLogInClick() {
    this.userService.logIn();
  }
}
```

In our test, we will use TestBed to configure a testing module with all the necessary dependencies. It is usually a good idea to provide mock dependencies instead of real dependencies. This makes it easier to set up different conditions for testing, and it makes actual unit tests and not integration tests.

You can use `TestBed.get` to get an instance of a dependency which the component that is being tested injects. In the following example, we will get an instance of `UserTestingService`, using the `UserService` class as a token. This has to match the token with which the instance is injected in the header component constructor.

Here is an example of how we could test our header component:

``` typescript
let component: HeaderComponent;
let fixture: ComponentFixture<HeaderComponent>;
let userService: UserTestingService;

beforeEach(async(() => {
  TestBed.configureTestingModule({
    declarations: [HeaderComponent],
    providers: [
      { provide: UserService, useClass: UserTestingService },
    ],
  })
  .compileComponents();

  userService = TestBed.get(UserService);
}));

beforeEach(() => {
  fixture = TestBed.createComponent(HeaderComponent);
  component = fixture.componentInstance;
  fixture.detectChanges();
});

it('should log the user in when Log in button is clicked', () => {
  spyOn(userService, 'logIn');
  expect(userService.logIn).not.toHaveBeenCalled();

  fixture.debugElement.query(By.css('.login-button')).nativeElement.click();

  expect(userService.logIn).toHaveBeenCalledTimes(1);
});
```

### Testing a service example

We will test `SomeService` which injects `UserService`:

``` typescript
@Injectable({ providedIn: 'root' })
export class SomeService {
  constructor(private userService: UserService) { }

  // ...rest of functionality
}
```

To make this test a "unit" test, we will mock all `SomeService`'s dependencies:

``` typescript
let service: SomeService;

beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [
      { provide: UserService, useClass: UserTestingService }, // Provide mock dependency
    ],
  });

  service = TestBed.get(SomeService);
});

it('should create a service instance', () => {
  expect(service).toBeTruthy();
});
```

A mock service for a UserService might look something like this:

``` typescript
// service/user/user.service.interface.ts
export interface IUserService {
  user$: Observable<UserModel>;
}

// service/user/user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService implements IUserService {
  public user$: Observable<UserModel>;
  //...
}

// service/user/user.testing.service.ts
// no need for @Injectable decorator
export class UserTestingService implements IUserService {
  public user$: Observable<UserModel> = of(new UserModelStub('steve'));
}
```

### Testing a component example

When testing complex components that use multiple services and render other components, make sure to provide them with stub services and components.

In this example, we will test the `ComponentToBeTested` component which renders `SomeOtherComponent` and depends on `UserService`:

``` typescript
@Component({
  selector: 'my-app-component-to-be-tested',
  template: `
    <div *ngIf="userService.user$ | async as user">
      User: {{ user.name }}
    </div>

    <my-app-some-other-component></my-app-some-other-component>
  `
})
export class ComponentToBeTested {
  constructor(public userService: UserService) { }
}
```

The SomeOtherComponent folder structure should look like this:

- components/
  - some-other/
      - some-other.module.ts
          - `export class SomeOtherModule`
          - should declare and export `SomeOtherComponent`
      - some-other.component.ts
          - `export class SomeOtherComponent implements ISomeOtherComponent`
      - some-other.component.spec.ts
      - some-other.component.interface.ts
          - `export interface ISomeOtherComponent`
          - interface should include all `@Input()`s and `@Output()`s
      - some-other.testing.component.ts
          - `export class SomeOtherTestingComponent implements ISomeOtherComponent`
          - **important** - should have the same selector as `SomeOtherComponent`
      - some-other.testing.module.ts
          - `export class SomeOtherTestingModule`
          - should declare and export `SomeOtherTestingComponent`

Make sure to exclude the `*.testing.*` files from code coverage reports in Jasmine (`angular.json` `codeCoverageExclude`) and any other reporting tools you might be using (such as SonarQube, for example).

The test should look like this:

``` typescript
let component: ComponentToBeTested;
let fixture: ComponentFixture<ComponentToBeTested>;

beforeEach(async(() => {
  TestBed.configureTestingModule({
    imports: [SomeOtherTestingModule], // use testing child component
    declarations: [ComponentToBeTested],
    providers: [
      { provide: UserService, useClass: UserTestingService }, // use testing service
    ],
  })
  .compileComponents();
}));

beforeEach(() => {
  fixture = TestBed.createComponent(ComponentToBeTested);
  component = fixture.componentInstance;
  fixture.detectChanges();
});

it('should create', () => {
  expect(component).toBeTruthy();
});
```

### Testing component inputs

If you want to test a component under different conditions, you will probably have to change some of its `@Input()` values and check if it emits `@Output()`s when expected.

To change the component input, you can simply:

``` typescript
@Component(...)
class SomeComponent {
  @Input() someInput: string;
}
```

``` typescript
component.someInput = 'foo';
fixture.detectChanges(); // To update the view
```

If you are doing stuff in `ngOnChanges`, you will have to call it manually since `ngOnChanges` is not called automatically in tests during programmatic input changes.

``` typescript
component.someInput = 'foo';
component.ngOnChanges({ someInput: { currentValue: 'foo' } } as any);
fixture.detectChanges();
```

### Testing component outputs

To test component outputs, simply spy on the output emit function:

``` typescript
@Component(
  template: `<button (click)="onButtonClicked()">`
)
export class ComponentWithOutput {
  @Output() public buttonClicked: EventEmitter<void> = new EventEmitter();

  public onButtonClicked(): void {
    this.buttonClicked.emit();
  }
}
```

``` typescript
spyOn(component.buttonClicked, 'emit');

expect(component.buttonClicked.emit).not.toHaveBeenCalled();

fixture.debugElement.query(By.css('button')).nativeElement.click();

expect(component.buttonClicked.emit).toHaveBeenCalled();
```

### Testing components with the `OnPush` change detection

If the component you are testing is using the `OnPush` change detection, `fixture.detectChanges()` will not work. To fix this, you can override the CD only during testing:

``` typescript
beforeEach(async(() => {
  TestBed.configureTestingModule({
    declarations: [ComponentWithInputComponent]
  })
  .overrideComponent(ComponentWithInputComponent, {
    set: { changeDetection: ChangeDetectionStrategy.Default },
  })
  .compileComponents();
}));
```

### Testing components with content projection

Components that use content projection cannot be tested in the same way as components that use only inputs for passing the data to the component. The problem is that, during TestBed configuration, you can only configure the testing module. You cannot define the template which would describe how the component is used inside another component's template.

One of the easiest ways to test content projection is to simply create a wrapper component for testing purposes only.

`TranscluderComponent`:

``` typescript
@Component({
  selector: 'app-transcluder',
  template: `
  <h1>
    <ng-content select="[title]"></ng-content>
  </h1>

  <div class="content">
    <ng-content select="[content]"></ng-content>
  </div>
  `
})
export class TranscluderComponent { }
```

`TranscluderModule`:

``` typescript
@NgModule({
  declarations: [TranscluderComponent],
  imports: [CommonModule],
  exports: [TranscluderComponent],
})
export class TranscluderModule { }
```

`TranscluderComponent` tests:

``` typescript
@Component({
  selector: 'app-transcluder-testing-wrapper',
  template: `
  <app-transcluder>
    <span title>{{ title }}</span>

    <a href="google.com" content>{{ linkText }}</a>
  </app-transcluder>
  `
})
class TranscluderTestingWrapperComponent {
  @ViewChild(TranscluderComponent) public trancluderComponent: TranscluderComponent;
  public title = 'Go to Google';
  public linkText = 'Link';
}

describe('TranscluderComponent', () => {
  let component: TranscluderTestingWrapperComponent;
  let fixture: ComponentFixture<TranscluderTestingWrapperComponent>;

  beforeEach(async(() => {
    TestBed.configureTestingModule({
      imports: [TranscluderModule],
      declarations: [TranscluderTestingWrapperComponent]
    })
    .compileComponents();
  }));

  beforeEach(() => {
    fixture = TestBed.createComponent(TranscluderTestingWrapperComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should display passed title', () => {
    const title: HTMLHeadingElement = fixture.debugElement.query(By.css('h1')).nativeElement;
    expect(title.textContent).toBe(component.title);
  });

  it('should display passed content', () => {
    const title: HTMLHeadingElement = fixture.debugElement.query(By.css('.content')).nativeElement;
    expect(title.textContent).toBe(component.linkText);
  });
});
```

### Trigger events from DOM instead of calling handlers directly

If you have some buttons with click handlers, you should test them by clicking on the elements instead of calling the handler directly.

Example:

``` html
<button class="login-button" (click)="onLogInClicked()">Log in</button>
```

Note: for simplicity, we will assume that the login action is synchronous.

``` typescript
// bad
it('should log the user in when Log in button is clicked', () => {
  expect(component.loggedIn).toBeFalsy();

  component.onLogInClicked();

  expect(component.loggedIn).toBeTruthy();
});

// good
it('should log the user in when Log in button is clicked', () => {
  expect(component.loggedIn).toBeFalsy();

  const logInButton = fixture.debugElement.query(By.css('.login-button')).nativeElement;
  logInButton.click();

  expect(component.loggedIn).toBeTruthy();
});
```

In this way, you are verifying that the button is actually present in the rendered DOM and that it is clickable. Otherwise, the test might pass even if the template was empty (no button to trigger the action).

### Testing HTTP requests

Your application will most likely be making requests left and right, so it is important to test how your app behaves when requests succeed and fail.

We will go through the whole process from service creation to testing success/failure states.

**Service setup**

We will create a simple `DadJokeService` which will be used for fetching jokes from `https://icanhazdadjoke.com`.

The service looks like this:

``` typescript
@Injectable({
  providedIn: 'root'
})
export class DadJokeService {
  static I_CAN_HAZ_DAD_JOKE_URL = 'https://icanhazdadjoke.com';
  static DEFAULT_JOKE = 'No joke for you!';

  constructor(private http: HttpClient) { }

  getJoke(): Observable<string> {
    const options = {
      headers: new HttpHeaders({
        'Accept': 'application/json'
      })
    };

    return this.http.get(DadJokeService.I_CAN_HAZ_DAD_JOKE_URL, options).pipe(
      catchError(() => {
        const noJoke: Partial<IJoke> = { joke: DadJokeService.DEFAULT_JOKE };

        return observableOf(noJoke);
      }),
      map((response: IJoke) => {
        return response.joke;
      })
    );
  }
}
```

You notice that our service depends on `HttpClient` which it injects using DI. It also catches any request errors and returns a "default" joke.

We also have an interface that defines how the response JSON is structured:

``` typescript
export interface IJoke {
  id: string;
  joke: string;
  status: number;
}
```

**Test setup**

Let's start by creating the most basic test which ensures only that our `DadJokeService` instantiates correctly.

If you've generated your service using Angular CLI, the `spec` file will probably look something like this:

``` typescript
describe('DadJokeService', () => {
  beforeEach(() => TestBed.configureTestingModule({}));

  it('should be created', () => {
    const service: DadJokeService = TestBed.get(DadJokeService);
    expect(service).toBeTruthy();
  });
});
```

Scaffolding tends to change from version to version, and this particular example has been generated using Angular CLI version 7.

We will modify this default `spec` file a bit, by moving the DadJokeService instance fetching from the `it` block into a `beforeEach` hook, right after we configure the `TestBed`:

``` typescript
describe('DadJokeService', () => {
  let service: DadJokeService;

  beforeEach(() => {
    TestBed.configureTestingModule({ });

    service = TestBed.get(DadJokeService);
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });
});
```

This transformation of the default `spec` file is something that we usually do for all services we test.

If you run the tests now, they will fail because our service injects `HttpClient`, and we have not provided it in our tests. Luckily, there is a module called `HttpClientTestingModule`, which you can import from `@angular/common/http/testing`. It provides a complete mocking back end for the `HttpClient` service. We just have to import it in our `TestBed` module:

``` typescript
TestBed.configureTestingModule({
  imports: [
    HttpClientTestingModule,
  ]
});
```

Our basic test now passes.

**Mocking HTTP requests in tests**

This is the fun part—we would like to test how our service behaves if a request succeeds or fails.

We will start by testing a successful request:

``` typescript
describe('DadJokeService', () => {
  let service: DadJokeService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [
        HttpClientTestingModule,
      ]
    });

    service = TestBed.get(DadJokeService);
    httpMock = TestBed.get(HttpTestingController);
  });

  it('should return the joke if request succeeds', () => {
    const jokeResponse: IJoke = {
      id: '42',
      joke: 'What is brown and sticky? A stick.',
      status: 200,
    };

    service.getJoke().subscribe((joke) => {
      expect(joke).toBe(jokeResponse.joke);
    });

    const mockRequest = httpMock.expectOne(DadJokeService.I_CAN_HAZ_DAD_JOKE_URL);

    mockRequest.flush(jokeResponse);
  });
});
```

We added a `httpMock` variable to our `describe` block. We assign it to an instance of `HttpTestingController`, which is provided by `HttpClientTestingModule`.

We also defined a `jokeResponse` variable which is formatted in the same way as a real response JSON would be.

Remember that the `getJoke` method returns an Observable over a string because it maps the response JSON using the `map` operator. We subscribe to `getJoke()` and, in the success callback, we assert that the value which our service returns is mapped correctly to be a string value of the `joke` property from the response JSON.

We call the `expectOne` method on the `httpMock` object and pass it a URL to which we are expecting a call to be made. No actual calls will be made during the test run. `expectOne` returns a `TestRequest` object on which we can call the `flush` method and pass the response body.

Since all this is executed synchronously, after we flush the response, a success callback (which we passed when subscribing) is called immediately and the assertion is checked.

Error catching can be tested in a similar manner:

``` typescript
it('should return a default joke if request fails', () => {
  service.getJoke().subscribe((joke) => {
    expect(joke).toBe(DadJokeService.DEFAULT_JOKE);
  });

  const mockRequest = httpMock.expectOne(DadJokeService.I_CAN_HAZ_DAD_JOKE_URL);

  mockRequest.error(null);
});
```

In our specific example, it was not important which particular error happened because our error catching implementation in the `catchError` operator has no logic which would determine different behavior depending on error code. For example, if we wanted to test how our error handling handles 500 server errors, we could do something like this:

``` typescript
mockRequest.error(new ErrorEvent('server_down'), { status: 500 });
```

This allows us to test a more complex error handling, which usually includes some logic for displaying different error messages depending on error code. As shown, that is completely doable using the `error` method of the `TestRequest` object.

### Testing helpers

If you have helper functions, testing them is basically the same as in any other application in any framework (even Vanilla JS). You do not need TestBed. You just need good old Jasmine, and you test your helpers as pure functions. If your helpers are not pure functions, you should really make them pure.
