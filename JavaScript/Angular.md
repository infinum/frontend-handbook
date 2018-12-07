# Angular Guidelines and Best Practices

*Guides are not rules and should not be followed blindly. Use your head and think.*

## Core Libraries and Tools

### [Angular CLI](https://github.com/angular/angular-cli)

If you are starting a new project, the best way to create a new project is by using Angular CLI. It will hide a lot of configuration behind the scenes (you will not have a webpack config exposed for editing), it does some optimizations when compiling your code and it offers some handy scaffolding tools.

Install Angular CLI globally with `npm install -g @angular/cli` and check out what you can do with `ng help`.

#### Creating a new project

Use Angular CLI for initial project setup:

```bash
ng new my-app
```

As of version 7 there will be a quick wizard-like setup prompting you about routing and styling. Choose with routing and SCSS styling.

#### Scaffolding

You can use Angular CLI for generating new modules, components, services, directives, etc.

Command is `ng generate [schematic-name]`, for example: `ng generate component`. There is also a shorthand for many generation commands, for example: `ng g c` === `ng generate component`

##### Creating new modules

To create a new module, use `ng g m MyModule`. If module will have routes, you can generate module with routing skeleton with `ng g m MyModule --routing`.

##### Creating new components

Create component with `ng g c MyComponent` or `ng g c my-component`. In both cases resulting component class name will be `MyComponent` and dir name will be `my-component`.

Usually when creating components you want to create component module as well. This makes the component more modular and ensures that everything that the component needs is provided by the component module. Component module should declare and export the component - think of it as a public API. Component module could also declare some other "internal" components, but it does not have to necessarily export those internal components.

Also remember that component can be declared only once (in one module), so it makes sense for reusable components to have their own module.

Example:

```bash
# create a module
ng g m Calendar

# create the component - this will also automatically declare the component in the above created module
ng g c Calendar

# make sure to export CalendarComponent in CalendarModule

# add some "internal" components which are not exported
ng g c WeekView
ng g c DayView

# WeekView and DayView do not have to be exported, as they are used only internally
```

```ts
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

##### Creating new services

Use `ng g s MyService` to generate new service. NOTE: unlike during component generation, service generation will not create a new dir for the service (although that might change in the future).

#### Ejecting

Do not do it, but if you **ABSOLUTELY HAVE TO** edit the webpack config manually, you can eject the application with `ng eject`. Unfortunately there is no way to extend webpack config.

#### Devserver proxy

If you have issues with CORS, for development purpuses it is OK to temporarily use Webpack Devserver proxy. You can do this without ejecting the webpack config, [as per instructions](https://angular.io/guide/build#proxying-to-a-backend-server).

#### Other commands

This handbook will not cover all `ng` commands, please check out the [wiki](https://github.com/angular/angular-cli/wiki).

### Typings

If you are using some JS library which is not written in TypeScript, you can usually install typings separately. Install typings as dev-dependency, for example:

```bash
npm install --save d3
npm install --save-dev @types/d3
```

### [RxJS](https://github.com/ReactiveX/rxjs)

Angular depends heavily on RxJS so getting to know RxJS is pretty important. It is used for all event handling and can even be used for application state management of some description.

Please check out some RxJS tutorials if you are not familiar with Rx.

#### Observers and Subscribers

When you want to observe a value, use some of the classes derived from `Observable`. Most use cases are covered with one of these:

- `Subject`
- `BehaviorSubject`
- `ReplaySubject`

All classed derived from `Observable` can be subscribed to in order to receive value changes. You can also use `operators` to manipulate how and when the value changes are sent to the subscribers.

There is a naming convention for `Observable` variables - postfixing with a dollar sign:

```typescript
// bad
const observable: Observable;

// good
const observable$: Observable;

// good
const mySubject$: Subject;
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
- `switchMap`

Avoid doing a lot of logic in `subscribe` callback. Subscribe as late as possible, and do all the necessary error catching and side-effects via operators.

You can also write your own operators. If you do so, it is strongly recommended to write them as [pure pipeable operator functions](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md#operator-as-a-pure-function).

#### async/await

You can use async/await to await completion of observables. Keep in mind that this will not work if the observable emits more than one value. For example, it is ok for things like HTTP requests.

To convert an observable to a promise, just call `obs$.toPromise()`.

### Recommended Editor

Recommended editor for Angular development is [VSCode](https://code.visualstudio.com/) as it has great integration with TypeScript and has some handy Angular plugins:

- [Angular Language Service](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template) - Intellisense in templates
- [Angular 7 Snippets](https://marketplace.visualstudio.com/items?itemName=Mikael.Angular-BeastCode)
- [TSLint](https://marketplace.visualstudio.com/items?itemName=eg2.tslint)
- [stylelint](https://marketplace.visualstudio.com/items?itemName=shinnn.stylelint)
- [Path Intellisense](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense)
- [Material Icon Theme](https://marketplace.visualstudio.com/items?itemName=PKief.material-icon-theme)

## File/Module Organization and Naming

If you created the app using Angular CLI, there is already some clear structure in place. Building on that structure, you will probably want to add some more dirs to help you organize your code.

Try to split up your application into components which you can reuse. If, for example, you have to handle lazy loading of all images in the app, create a `Image` module and component in `app/components` and use it in other components whenever you have to show an image. Export the declared component and import/provide dependencies which your component has.

Container components which are loaded directly via routing (by either `component` or `loadChildren`) should be placed in `app/pages` dir. If a component is loaded because you have used it in a template, that component should be in `app/components`

One little exception here: if there is some very specific component tied to some routing submodule which is used only in that submodule, it is OK to leave it in that submodule (but as soon as it is used in another routing submodule, move it to `app/components`.

Store all your animations in `app/animations/` and import them for usage in component decorators.

Classes which might be used throughout the application (e.g. generic `Dictionary` class) should be in `app/classes`.

Entity models (`User`, `Post`, etc.) should be in `app/models`.

If you are using some JS libs which do not have typings, add your own custom typing in `app/custom-typings`.

Whenever you have some attribute whose value can be some value from a set of values, make an enum and store that enum in `app/enums`. Good example is an enum for HTTP status codes. If you need some metadata for the enum, create an enum data file which exports what is basically a dictionary with enum values used as keys and use whatever you need for values. One such example would be: `export const httpStatusCodeData = { [HttpStatus.OK_200]: { translation: 'Success' } }`

If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s and validators in `app/forms`. If you have form components which are reused throughout the app, put them in `app/components/forms`.

Put all route guards in `app/guards`.

If you need some helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `app/helpers` in its own dir, make sure to write tests for it and simply export it in `app/helpers/helpers.ts`.

Save all your interface files to `app/interfaces`.

If you have some custom pipes for use in templates, create their own dirs in `app/pipes`, similar to helpers. Declare, export and provide the pipes in `app/pipes/pipes.module.ts`.

Services should be placed in `app/services`, each service in its own dir.

If you have any `NgModule`s which are not components (e.g. a module which imports all specific material modules that you need), put them in `app/shared-modules`.

TypeScript `type` declarations should be placed in `app/types`.

As defined in angular-cli, assets placed in `assets` will be served statically.

Global styles should be placed in `src/app/styles` dir. Styles dir has very similar structure as that described in SASS Styleguide, so please check it out.

All environment files are placed in `environments`.

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
    - use `changeDetection: ChangeDetectionStrategy.OnPush` and pass data down using `async` pipe

More on this philosophy can be found in various online articles. Some articles have framework-specific information, but the core concepts apply no matter which framework you are using.

## Environments

Environment files should be used for some global configuration changes which can differ between development and production builds. Keep in mind that these values are bundled in build at build-time and can not be changed after the build. This can be an issue if application is deployed using Docker - in which case you should use some SSR solution or script injection instead of defining variables via Angular's environment files.

Define interface for the environment: `/src/app/interfaces/environment.interface`:
```typescript
export interface IEnvironment {
  production: boolean;
}
```

Set base values `/src/environments/environment.base.ts`:
```typescript
import { IEnvironment } from 'app/interfaces/environment.interface';

export const baseEnvironment: IEnvironment = {
  production: false,
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

This is the production environment: `/src/environments/environment.prod.ts`:
```typescript
import { baseEnvironment } from './environment.base';

export const environment: IEnvironment = {
  ...baseEnvironment,
  production: true,
};
```

Make sure to update `angular.json` if you add/remove environment files.

After all is defined, run/build the app for different environments by specifying the environment in `ng` command: `ng build --env=prod` for production build; `ng build --env=staging` for staging build; `ng build` will use default `dev` environment.

## Formatting and Naming

### In Templates

For the purposes of future-proofing and avoiding conflicts with other libs, prefix all component selectors with something unique/app-specific. Specify the prefix in `angular.json` like so: `"prefix": "my-app",`.

----

#### Try to fit all inputs, outputs and regular HTML attributes in one line. If there is simply too many of them, break it up like so:

```html
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

<!-- maybe could be good? -->
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

#### Closing elements and angle brackets placement:

```html
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

#### If you are passing data to a component, use property binding only if necessary:

```html
<!-- bad -->
<my-app-component [name]="'Steve'"></my-app-component>

<!-- good -->
<my-app-component name="Steve"></my-app-component>
```

----

#### Prefix output handlers with `on`:

```html
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

#### Name click handlers in a descriptive manner:

```html
<!-- bad -->
<button (click)="onClick()">Log in</button>

<!-- good -->
<button (click)="onLogInClick()">Log in</button>

<!-- bad - too much -->
<button (click)="onLogInButtonClick()">Log in</button>

<!-- bad - use infinitive instead of past form -->
<button (click)="onLogInClicked()">Log in</button>
```

----

#### Use safe navigation operator:

```html
<!-- bad -->
<div *ngIf="user && user.loggedIn"></div>

<!-- good -->
<div *ngIf="user?.loggedIn"></div>
```

----

#### Add space around interpolation curly brackets:

```html
<!-- bad -->
<div>{{userName}}</div>

<!-- good -->
<div>{{ userName }}</div>
```

----

#### Use `ng-container` when possible to reduce the amount of generated DOM elements:

```html
<!-- bad -->
<div *ngIf="something"></div>

<!-- good -->
<ng-container *ngIf="something"></ng-container>

<!-- good -->
<div class="i-need-this-class" *ngIf="something"></div>
```

----

#### Use `*ngIf; else`:

```html
<!-- bad -->
<ng-container *ngIf="something"></ng-container>
<ng-container *ngIf="!something"></ng-container>

<!-- good -->
<ng-container *ngIf="something; else anotherThing"></ng-container>
<ng-template #anotherThing></ng-template>
```

----

#### Subscribe to observables with `async` pipe if possible:

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

`async` pipe is especially useful if `changeDetection` is `OnPush` since it subscribes, calls change detection manually on changes and unsubscribes when component is destroyed.

----

#### Use attributes for transclusion selectors:

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

----

#### Order attributes and bindings:

1. Structural directives (`*ngFor`, `*ngIf`, etc.)
2. Element reference (`#myComponent`)
3. Ordinary attributes (`class`, etc.)
4. Non-interpolated string inputs (`foo="bar"`)
5. Interpolated inputs (`[foo]="theBar"`)
6. Two-way bindings (`[(ngModel)]="email"`)
7. Outputs (`(click)="onClick()"`)

```html
<!-- bad -->
<my-app-component
  (click)="onClick($event)"
  [(ngModel)]="fooBar"
  class="foo"
  #myComponent
  [foo]="theBar"
  foo="bar"
  *ngIf="shouldShow"
  (someEvent)="onSomeEvent($event)"
></my-app-component>

<!-- good -->
<my-app-component
  *ngIf="shouldShow"
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

#### Prefer `[class]` over `[ngClass]` syntax:

```html
<!-- bad -->
<div [ngClass]="{ 'active': isActive }"></div>

<!-- good -->
<div [class.active]="isActive"></div>
```

### In Code

#### Prefix interfaces

Interfaces should be prefixed with `I`. This might be a polarizing decision, but the reason why is because you might have cases where some class implements an interface and you also have a stub class which implements the same interface.

```ts
// bad
interface UserService { }

// good
interface IUserService { }

// if we prefix we can do this:
class UserService implements IUserService { }
class UserServiceStub implements IUserService { }
```

#### Prefer interface over type

When defining data structures, prefer using interfaces instead of types. Use types only for things for which interfaces can not be used - unions of different types.

```ts
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

You might be tempted to use type for union of models. For example, in some CMS solution you might have `AdminModel` and `AuthorModel`. User can log in using either admin or author credentials.

```ts
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

This might seem fine at first, but it is actually an anti-pattern. What you should really do in cases like this is create an abstraction above `AdminModel` and `AuthorModel`:

```ts
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

#### Renaming rxjs imports

Some rxjs imports have very generic names so you might want to rename them during import:

```ts
import {
  of as observableOf
  EMPTY as emptyObservable
} from 'rxjs';
```

#### Ordering class members (including getters and life-cycle hooks)

TODO

## Dependency Injection

In Angular, DI is utilized heavily, primarily for providing instances of either primitive values or class instances.

Make sure to check out the [official DI guide](https://angular.io/guide/dependency-injection) and you can also have a look at [this repo](https://github.com/fvoska/angular-di-demo) for some examples.

As of Angular 6, make sure to use `{ providedIn: 'root' }` [for singletons](https://angular.io/guide/singleton-services#providing-a-singleton-service) whenever possible.

DI is very useful for testing purposes, as shown in next chapter.

## Testing

When unit testing, if the unit which you are testing is injectable, use `TestBed` for easier instance creation and easy providing of stub units. 'unit' in this context can be a component, service, pipe, etc - anything injectable.

### Testing a service example

```ts
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

```ts
@Injectable({ providedIn: 'root' })
export class SomeService {
  constructor(private userService: UserService)
}
```

```ts
let service: SomeService;

beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [
      { provide: UserService, useClass: UserTestingService },
    ],
  });

  service = TestBed.get(SomeService);
});

it('should create a service instance', () => {
  expect(service).toBeTruthy();
});
```

### Testing a component example

When testing complex components which use multiple services and render other components, make sure to provide them with stub services and components.

```ts
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

SomeOtherComponent folder structure should look like this:

- components/
  - some-other/
    - some-other.module.ts
      > `export class SomeOtherModule`

      > should declare and export `SomeOtherComponent`
    - some-other.component.ts
      > `export class SomeOtherComponent implements ISomeOtherComponent`
    - some-other.component.spec.ts
    - some-other.component.interface.ts
      > `export interface ISomeOtherComponent`

      > interface should include all `@Input()`s and `@Output()`s
    - some-other.testing.component.ts
      > `export class SomeOtherTestingComponent implements ISomeOtherComponent`

      > **important** - should have the same selector as `SomeOtherComponent`
    - some-other.testing.module.ts
      > `export class SomeOtherTestingModule`

      > should declare and export `SomeOtherTestingComponent`

Make sure to exclude `*.testing.*` files from code coverage reports.

Test should look like this:

```ts
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

To change components input, you can simply:

```ts
@Component(...)
class SomeComponent {
  @Input() someInput: string;
}
```

```ts
component.someInput = 'foo';
fixture.detectChanges(); // To update the view
```

If you are doing stuff in `ngOnChanges`, you will have to call it manually since in tests `ngOnChanges` is not called automatically during programmatic input changes.

```ts
component.someInput = 'foo';
component.ngOnChanges({ someInput: { currentValue: 'foo' } } as any);
fixture.detectChanges();
```

### Testing component outputs

To test component outputs, simply spy on output emit function:

```ts
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

```ts
spyOn(component.buttonClicked, 'emit');

expect(component.buttonClicked.emit).not.toHaveBeenCalled();

fixture.debugElement.query(By.css('button')).nativeElement.click();

expect(component.buttonClicked.emit).toHaveBeenCalled();
```

### Testing components with `OnPush` change detection

If the component you are testing is using `OnPush` change detection, `fixture.detectChanges()` will not work. To fix this, you can override the CD just during testing:

```ts
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

### Trigger events from DOM instead of calling handlers directly

If you have some buttons with click handlers, you should test them by clicking on the elements instead of calling the handler directly.

Example:

```html
<button class="login-button" (click)="onLogInClicked()">Log in</button>
```

Note: for simplicity, we will assume that login action is synchronous.

```ts
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

This way you are verifying that the button is actually present in the rendered DOM and that is is clickable. Otherwise it would be possible that the test passes even if the template was empty (no button to trigger the action).

### Utilizing TestBed

DI is very powerful during testing, as it allows you to provide mock services for testing purposes. You will sometimes want to spy on some private service's methods. This might tempt you to make the service public or to do bypass TS by doing something like `component['myService']`. To avoid these *dirty* solutions, you can utilize the `TestBed`:

```ts
export class HeaderComponent {
  constructor(private userService: UserService)

  onLogInClick() {
    this.userService.logIn();
  }
}
```

```ts
let component: HeaderComponent;
let fixture: ComponentFixture<HeaderComponent>;
let userService: UserService;

beforeEach(async(() => {
  TestBed.configureTestingModule({
    declarations: [HeaderComponent],
    providers: [
      { provide: UserService, useClass: UserTestingService },
    ],
  })
  .compileComponents();
}));

beforeEach(() => {
  fixture = TestBed.createComponent(HeaderComponent);
  component = fixture.componentInstance;
  userService = TestBed.get(UserService);
  fixture.detectChanges();
});

it('should log the user in when Log in button is clicked', () => {
  spyOn(userService, 'logIn');
  expect(userService.logIn).not.toHaveBeenCalled();

  component.onLogInClick();

  expect(userService.logIn).toHaveBeenCalledTimes(1);
});
```

### Testing HTTP requests

TODO: HttpClientTestingModule, HttpTestingController, expectOne, flush, errors, ...

### Testing asynchronous actions

TODO: using async/await in tests, fakeAsync and tick()

### Testing helpers

TODO: basically test them like plain JS functions like you would in any other application in any other framework - no need for TestBed - vanilla Jasmine.

## Angular Universal (Server-side Rendering)

When setting up server-side rendering, follow the [official guide](https://angular.io/guide/universal).

There are many tricks with SSR even after all the setup is done. You can check out [this repo](https://github.com/fvoska/angular-universal-demo) to see how many issues can be solved. This includes reading environment variables at runtime, so it is worth checking out.

## Routing and Lazy Loading

TODO

## Working with Angular Material

TODO

## Working with JSON API

TODO

## Working with Forms

TODO

## Two-way binding

TODO

## Inner Workings and Common Pitfalls

TODO
