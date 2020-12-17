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

![XKCD real programmers](/img/real_programmers.png)

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
