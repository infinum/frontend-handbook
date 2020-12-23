A lot of bike-shedding can be avoided by using a tool that enforces the formatting rules. Luckily, such a tool exists and it is called `prettier`. We have detailed how to use `prettier` alongside some other tools as well—you can check it out [here](/handbook/books/frontend/javascript/code-quality)!

Even when using `prettier`, there can still be some areas in which `prettier` is unopinionated and it is up to us to decide what is the best way to name and format things. That is exactly what this chapter covers.

## In templates

For the purposes of future-proofing and avoiding conflicts with other libs, prefix all component selectors with something unique/app-specific. Specify the prefix in `angular.json` like this: `"prefix": "my-app"` (needless to say, pick a better prefix than `my-app`).

----

### Try to fit all inputs, outputs, and regular HTML attributes in one line. If there are simply too many of them, break them up

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

### Closing elements and angle brackets placement

Case without content:

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
```

Case with content:

```html
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

### If you are passing data to a component, use property binding only if necessary

```html
<!-- bad -->
<my-app-component [name]="'Steve'"></my-app-component>

<!-- good -->
<my-app-component name="Steve"></my-app-component>
```

----

### Prefix output handlers with `on`

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

### Name click handlers in a descriptive manner

```html
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

### Use the optional chaining operator

```html
<!-- bad -->
<div *ngIf="user && user.loggedIn"></div>

<!-- good -->
<div *ngIf="user?.loggedIn"></div>
```

----

### Add space around curly brackets used for interpolation

```html
<!-- bad -->
<div>{{userName}}</div>

<!-- good -->
<div>{{ userName }}</div>
```

----

### Use `ng-container` when possible to reduce the amount of generated DOM elements

```html
<!-- bad -->
<span *ngIf="something"></span>

<!-- good -->
<ng-container *ngIf="something"></ng-container>

<!-- good -->
<span *ngIf="something" class="i-need-this-class"></span>
```

----

### Use `*ngIf; else`

```html
<!-- bad -->
<ng-container *ngIf="something"></ng-container>
<ng-container *ngIf="!something"></ng-container>

<!-- good -->
<ng-container *ngIf="something; else anotherThing"></ng-container>
<ng-template #anotherThing></ng-template>
```

----

### Subscribe to observables using the `async` pipe. Avoid manual subscriptions

Component:

```typescript
// bad
@Component(...)
export class ArticlesList {
  public articles$ = new BehaviorSubject<Array<Article>>([]);
  public menu$ = new BehaviorSubject<Menu>(undefined);

  constructor(...) {
    this.articleService.fetchArticles().subscribe((articles) => {
      this.articles$.next(articles);
    });

    this.articleService.fetchMenu().subscribe((menu) => {
      this.menu$.next(menu);
    });
  }
  ...
}

// good
@Component(...)
export class ArticlesList {
  public articles$: Observable<Array<Article>> = this.articleService.fetchArticles();
  public menu$: Observable<Menu> = this.articleService.fetchMenu();
  ...
}
```

Template:

```html
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

### Use attributes for transclusion selectors

The `my-app-wrapper` component template:

```html
<div class="wrapper">
  <p>Wrapper content:</p>
  <ng-content select="[some-stuff]"></ng-content>
</div>
```

Some other component's template:

```html
<my-app-wrapper>
  <div class="some-item" some-stuff>Some stuff</div>
</my-app-wrapper>
```

----

### Order attributes and bindings

1. Structural directives (`*ngFor`, `*ngIf`, etc.)
2. Animation triggers (`@fade`, `[@fade]`)
3. Element reference (`#myComponent`)
4. Ordinary attributes (`class`, etc.)
5. Non-interpolated string inputs (`foo="bar"`)
6. Interpolated inputs (`[foo]="theBar"`)
7. Two-way bindings (`[(ngModel)]="email"`)
8. Outputs (`(click)="onClick()"`)

```html
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

### Prefer the `[class]` over `[ngClass]` syntax

```html
<!-- bad -->
<div [ngClass]="{ 'active': isActive }"></div>

<!-- good -->
<div [class.active]="isActive"></div>
```

## In code

### Prefix interfaces

Interfaces should be prefixed with `I`. This might be a polarizing decision, but you should do it because you might have cases where some class implements an interface, and you also have a stub class which implements the same interface.

```typescript
// bad
interface UserService { }

// good
interface IUserService { }

// if we prefix we can do this:
class UserService implements IUserService { }
class UserServiceStub implements IUserService { }
```

----

### Prefer interface over type

When defining data structures, prefer using interfaces over types. Use types only for those things for which interfaces cannot be used—unions of different types.

```typescript
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

```typescript
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

```typescript
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

### Always safe-guard `ngOnChanges`

```typescript
// bad
@Component(...)
class MyComponent {
  @Input() public input1: string;
  @Input() public input2: string;
  private upperCaseInput1: string;

  public ngOnChanges() {
    this.upperCaseInput1 = this.input1.toUpperCase();
  }
}
```

The problem with not checking which specific change was triggered is that whatever computation is done might be done unnecessarily. You should add checks so that the action is executed only when the related inputs change and ignore changes of other inputs. In above example `upperCaseInput1`'s value depends only on `input1`'s value, but the assignment will be executed even if only `input2` changes, which is unnecessary.

```typescript
// good
@Component(...)
class MyComponent {
  @Input() public input1: string;
  @Input() public input2: string;
  private upperCaseInput1: string;

  public ngOnChanges(changes) {
    if (changes.input1) {
      this.upperCaseInput1 = this.input1.toUpperCase()
    }
  }
}
```

----

### Extract ngOnChanges logic into methods

If `ngOnChanges` contains some logic, it is often a good idea to separate that logic into private methods to increase readability of `ngOnChanges` method.

```typescript
// bad
@Component(...)
class MyComponent {
  @Input() public user: UserModel;
  public userForm: FormGroup;

  constructor(private fb: FormBuilder) {}

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.user) {
      this.userForm = this.fb.group({
        name: this.user.name,
        ...
      });
    }
  }
}
```

```typescript
// good
@Component(...)
class MyComponent {
  @Input() public user: UserModel;
  public userForm: FormGroup;

  constructor(private fb: FormBuilder) {}

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.user) {
      this.updateUserForm();
    }
  }

  private updateUserForm(): void {
    this.userForm = this.fb.group({
      name: this.user.name,
      ...
    });
  }
}
```

We can even go one step further and make things "pure":

```typescript
// even better
@Component(...)
class MyComponent {
  @Input() public user: UserModel;
  public userForm: FormGroup;

  constructor(private fb: FormBuilder) {}

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.user) {
      this.userForm = this.buildUserForm(changes.user.currentValue);
      //   or
      // this.userForm = this.buildUserForm(this.user);
      //   it doesn't really matter much which one you choose since
      //   changes.user.currentValue === this.user
    }
  }

  private updateUserForm(user: UserModel): FormGroup {
    return this.fb.group({
      name: user.name,
      ...
    });
  }
}
```

----

### Remember that `ngOnChanges` is called on first change as well as all subsequent changes

In this example we want to react on input changes and call some action.

```typescript
//bad
@Component(...)
class MyComponent {
  @Input() public label: string;

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.label) {
      this.someActions()
    }
  }

  public ngOnInit() {
    this.someActions()
  }

  private someActions() { ... }
}
```

Problem with the above example is that someActions will be called twice, first in `ngOnChanges` and then in `ngOnInit`. Yes, this order seems a bit strange - `ngOnInit` is called **after** the first `ngOnChanges` call; but this is just how it is.

```typescript
//good
@Component(...)
class MyComponent {
  @Input() public label: string;

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.label) {
      this.someActions()
    }
  }

  private someActions() { ... }
}
```

Look at your use cases and check if you can remove `ngOnInit`. You almost never need `ngOnInit`. If there are some default input values which should be set, you can do it alongside input declaration (`@Input() label: string = 'default label'`). Any other reactions to input changes should be done in `ngOnChanges`.

----

### Consider different initial assignment options

There is often a need to assign some properties during component initialization. This can be done in multiple ways:

1. In component's constructor
2. During property declaration
3. In lifecycle hooks
4. Subscribing at initialization

When learning Angular, you hear a lot about different lifecycle hooks. Even when creating new components using the Angular CLI, you get a component with empty `constructor` and empty `ngOnInit` method. Because of this, it seems natural to use one of these for initialization.

**Example #1** - how a constructor might be used for initial value assignment:

```typescript
@Component(...)
class MyComponent {
  private foo: string;

  constructor() {
    this.foo = 'bar';
  }
}
```

This works and there is nothing wrong with this, but it can be written shorter like so:

**Example #2** - initial assignment alongside property declaration

```typescript
@Component(...)
class MyComponent {
  private foo: string = 'bar';
}
```

We did pretty much the same thing, but a bit shorter.

**Example #3** - order during initial assignment alongside property declaration

```typescript
@Component(...)
class MyComponent {
  private foo: string = 'bar';

  constructor() {
    this.foo = 'foo';
  }
}
```

Property assignments will be executed immediately after constructor execution has finished, so in this example the final value of `foo` will be `bar`.

We recommend setting initial values alongside property declaration since it is the shortest option.

However, there are some cases where you will have to use `ngOnInit` or `ngOnChanges`.

**Example #4** - when are `ngOnInit` and `ngOnChanges` necessary?

```typescript
// bad
@Component(...)
class PersonDetailsComponent {
  @Input() public birthDate: Date;
  public isLegalAge: boolean;

  constructor() {
    this.isLegalAge = new Date().getYear() - this.birthDate.getYear() > 18;
    // This check is not 100% correct, we are keeping it simple.
  }
}
```

If you try this, you will get an exception because `birthDate` will be undefined at component construction time. Input binding values are available only after `ngOnInit` and `ngOnChanges` are called. To make this work, you can use `ngOnInit`:

```typescript
// bad
@Component(...)
class PersonDetailsComponent implements OnInit {
  @Input() public birthDate: Date;
  public isLegalAge: boolean;

  ngOnInit() {
    this.isLegalAge = new Date().getYear() - this.birthDate.getYear() > 18;
  }
}
```

Using `ngOnInit` works and we get no exception (with the assumption that a valid `Date` object is passed down to our component via `birthDate` input).

We still have one problem with this solution. If input binding changes, `isLegalAge` will not be re-assigned. To solve this we simply switch to using `ngOnChanges`:

```typescript
// good
@Component(...)
class PersonDetailsComponent implements OnChanges {
  @Input() public birthDate: Date;
  public isLegalAge: boolean;

  ngOnChanges(changes: SimpleChanges) {
    if (changes.birthDate) {
      this.isLegalAge = this.checkIfLegalAge(changes.birthDate.currentValue);
    }
  }

  private checkIfLegalAge(birthDate): boolean {
    return new Date().getYear() - birthDate.getYear() > 18;
  }
}
```

Since this check is simple and returns a primitive value, using a getter here is also a valid option and it allows us to remove ngOnChanges completely and reduce the amount of code.

```typescript
// even better
@Component(...)
class PersonDetailsComponent {
  @Input() public birthDate: Date;

  public get isLegalAge(): boolean {
    return new Date().getYear() - this.birthDate.getYear() > 18;
  }
}
```

But be careful when using getters for values which are used in templates because they will be called very often. This is OK only if there is not too much heavy lifting inside getters and if the getter does not return a newly initialized object every time it is called.

### Avoid unnecessary creation of multiple observables and avoid subscriptions

Taking the learnings from all the previous examples and the fact that we can use services during initial assignment, we can greatly simplify initialization of observables and their usage in templates.

```typescript
// bad
@Component({
  template: `
  <ng-container *ngIf="user">
    {{ user.name }}
  </ng-container>
  `,
  ...
})
class MyComponent {
  public user: UserModel;
  private userSubscription: Subscription;

  constructor(
    private route: ActivatedRoute,
    private usersService: UsersService,
  ) {
    this.userSubscription = this.route.params.pipe(switchMap((params: Params) => {
      return this.usersService.fetchById(params.userId);
    })).subscribe((user) => {
      this.user = user;
    });
  }

  ngOnDestroy(): void {
    this.userSubscription.unsubscribe();
  }
}
```

This is bad because it will not work correctly with OnPush change detection and we have to take care of subscriptions manually - it is easy to forget to unsubscribe and we have to introduce `ngOnDestroy`.

```typescript
// still bad
@Component({
  template: `
  <ng-container *ngIf="user$ | async as user">
    {{ user.name }}
  </ng-container>
  `,
  ...
})
class MyComponent {
  constructor(
    private route: ActivatedRoute,
    private usersService: UsersService,
  ) {}

  public get user$(): Observable<UserModel> {
    return this.route.params.pipe(switchMap((params: Params) => {
      return this.usersService.fetchById(params.userId);
    }));
  }
}
```

This is a bit better since we no longer have to take care of unsubscribing, but the issue now is that during each template check, `user$` getter will be called. In _Example #4_ we had a case where using a getter was fine, but in this case it is not. Each time it is called it creates a new observable, template will subscribe to this new observable during each change detection, meaning that a new subscription will trigger another API call every CD cycle - which is unnecessary.

```typescript
// good
@Component({
  template: `
  <ng-container *ngIf="user$ | async as user">
    {{ user.name }}
  </ng-container>
  `,
  ...
})
class MyComponent {
  public user$: Observable<UserModel> = this.route.params.pipe(switchMap((params: Params) => {
    return this.usersService.fetchById(params.userId);
  }));

  constructor(
    private route: ActivatedRoute,
    private usersService: UsersService,
  ) {}
}
```

Finally the correct solution simply assigns user$ to an observable which is created only once. The template will have only one subscription and we will just react to updates in the data stream which happen when params change.

----

### Renaming RxJS imports

Some RxJS imports have very generic names, so you might want to rename them during import:

```typescript
import {
  of as observableOf
  EMPTY as emptyObservable
} from 'rxjs';
```

----

### Subscribe as late as possible and use the operators

If you are new to RxJS, you will probably be overwhelmed by the amount of operators and the "Rx-way" of doing things. To do things in the "Rx-way", you will have to embrace the usage of operators and think carefully when and how you subscribe.

A rule of thumb is to subscribe as little and as late as possible, and do minimal amount of work in subscription callbacks.

We will demonstrate this with an example. Imagine you have a stream of data to which you would like to subscribe and transform the data in some way.

```typescript
interface IUser {
  id: number;
  user_name: string;
  name: string;
  surname: string;
}
```

`/account/me` returns JSON:
```json
{
  "id": 42,
  "user_name": "TheDude",
  "name": "Jeff",
  "surname": "Lebowski"
}
```

```typescript
// bad
const user$: Observable<IUser> = http.get('/account/me');

user$.subscribe((response: IUser) => {
  console.log(response.user_name); // TheDude
});
```

```typescript
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

### No subscriptions in guards

Asynchronous guards are common, but they should not subscribe to anything; they should return an observable.

To demonstrate this on an example, imagine we have `AuthService` and the `getUserData` method which returns an observable. The observable will return the `User` model if the user is authenticated or emit an error if not.

We would like to implement a guard which allows only authenticated users to navigate to the route.

```typescript
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

### Be mindful of how and when data is fetched

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

Even though guards are really easy to use and have some advantages, there might be situations where the data has to be loaded through multiple requests or the loading might be slow. In those cases, it might be worth to consider using a container component for data loading instead of guards. The container component would use some service to make the requests and then show the data once the requests have been completed. In this way, it is possible to show partial data as it comes in, and implement empty state design for each chunk of data that is shown.

A good example where partial data loading via a container component can be preferred over all-at-once loading via guards is a dashboard-like page with multiple graphs where each graph shows data from one request. In such cases, it is probably better to let the container component handle data loading, and presentational components (graphs) should implement an empty state with some nice loaders.

Bottom line:

- Use resolve guards if route data can be loaded in one big chunk and is also shown all-at-once.
- Use container components if data is loaded in chunks and results should be shown as they come in.

----

### Ordering class members (including getters and lifecycle hooks)

Follow this guideline when ordering class members:

1. `@Input`s
2. `@Output`s
3. public, protected, and private properties
4. constructor
5. lifecycle hooks
6. public, protected, and private getters and setters
7. public, protected, and private methods

Example:

```typescript
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
