This chapter covers various best practices when writing templates, business logic, components, services, etc. We also include some Angular-specific design patterns.

## A quick note on formatting

A lot of bike-shedding can be avoided by using a tool that enforces the formatting rules. Luckily, such a tool exists and it is called `prettier`. We have detailed how to use `prettier` alongside some other tools as well—you can check it out [here](/books/frontend/javascript/code-quality)!

Even when using `prettier`, there can still be some cases in which `prettier` is unopinionated and it is up to us to decide what is the best way to name and format things. Some such cases are covered by this chapter.

# Best practices

## Use strict mode

We recommend enabling strict mode when creating a new application (or enable it for already existing applications). This enables some additional checks for TypeScript and Angular templates. You can read more about it [here](https://angular.io/guide/strict-mode).

## Use a meaningful component prefix

The following examples in this chapter are using `my-app` component selector prefix. For the purposes of future-proofing and avoiding conflicts with other libs, prefix all component selectors with something unique/app-specific. Specify the prefix in `angular.json`.

## If you are passing data to a component, use property binding only if necessary

In the following example, property binding is unnecessary because we are assigning a string value.

```html
<!-- bad -->
<my-app-component [name]="'Steve'"></my-app-component>

<!-- good -->
<my-app-component name="Steve"></my-app-component>
```

## Prefix output handlers with `on`

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

## Name click handlers in a short and descriptive manner

```html
<!-- bad -->
<button (click)="onClick()">Log in</button>

<!-- bad - use infinitive instead of past form -->
<button (click)="onLogInClicked()">Log in</button>

<!-- bad - too many words -->
<button (click)="onLogInButtonClick()">Log in</button>

<!-- good -->
<button (click)="onLogInClick()">Log in</button>
```

## Use the optional chaining operator

```html
<!-- bad -->
<div *ngIf="user && user.loggedIn"></div>

<!-- good -->
<div *ngIf="user?.loggedIn"></div>
```

## Use `ng-container` when possible to reduce the amount of unnecessary DOM elements

```html
<!-- bad - span is unnecessary -->
<span *ngIf="something"></span>

<!-- good - span was unnecessary -->
<ng-container *ngIf="something"></ng-container>

<!-- good - span is necessary because of the class -->
<span *ngIf="something" class="i-need-this-class"></span>
```

## Use `*ngIf; else`

```html
<!-- bad - we have to type the same condition twice and check it twice -->
<ng-container *ngIf="something"></ng-container>
<ng-container *ngIf="!something"></ng-container>

<!-- good -->
<ng-container *ngIf="something; else anotherThing"></ng-container>
<ng-template #anotherThing></ng-template>
```

## Use attributes for transclusion selectors

When implementing transclusion with manual content selection via the `select` attribute of the `ng-content` element, we recommend using attribute selectors.

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

## Order/group attributes and bindings

1. Structural directives (`*ngFor`, `*ngIf`, etc.)
2. Animation triggers (`@fade`, `[@fade]`)
3. Element reference (`#myComponent`)
4. HTML attributes (`class`, etc.)
5. Non-interpolated string inputs and attributes (`foo="bar"`)
6. Interpolated inputs (`[bar]="theBar"`)
7. Two-way bindings (`[(ngModel)]="email"`)
8. Outputs (`(click)="onClick()"`)

```html
<!-- bad -->
<my-app-component
  (click)="onClick($event)"
  [(ngModel)]="fooBar"
  class="foo"
  #myComponent
  [bar]="theBar"
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
  [bar]="theBar"
  [(ngModel)]="fooBar"
  (click)="onClick($event)"
  (someEvent)="onSomeEvent($event)"
></my-app-component>
```

## `[class]` vs `[ngClass]` syntax

If you have to set just one class depending on some condition, `[class.class-name]="condition"` is the way to go. Example:

```html
<div [class.active]="isActive"></div>
```

If you need to set many classes, `[ngClass]="{ ... }"` is the way to go. Example:

```html
<div [ngClass]="{ 'active': isActive, 'main-item': isMainItem, 'accent': isAccent }"></div>
```

## Prefix interfaces

Interfaces should be prefixed with `I`. This might be a polarizing decision, but you should do it because you might have cases where some class implements an interface, and you also have a stub class that implements the same interface.

```typescript
// bad
interface UserService { }

// good
interface IUserService { }

// if we prefix we can do this:
class UserService implements IUserService { }
class UserServiceStub implements IUserService { }
```

## Prefer interface over type

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

## Ordering the class members (including getters and lifecycle hooks)

Follow this guideline when ordering class members:

1. `@Input`s
2. `@Output`s
3. properties
4. constructor
5. getters and setters
6. lifecycle hooks
7. methods

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

  public get computedProp(): string { ... }

  private get secretComputedProp(): number { ... }

  public ngOnInit(): void { ... }

  public ngOnChanges(): void { ... }

  public onSomeButtonClick(event: MouseEvent): void { ... }

  private someInternalAction(): void { ... }
}
```

## Always safe-guard `ngOnChanges`

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

## Extract ngOnChanges logic into methods

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
      this.createUserForm();
    }
  }

  private createUserForm(): void {
    this.userForm = this.fb.group({
      name: this.user.name,
      ...
    });
  }
}
```

We can even go one step further and make things _pure_:

```typescript
// even better
@Component(...)
class MyComponent {
  @Input() public user: UserModel;
  public userForm: FormGroup;

  constructor(private fb: FormBuilder) {}

  public ngOnChanges(changes: SimpleChanges) {
    if (changes.user) {
      this.userForm = this.createUserForm(changes.user.currentValue);
    }
  }

  private createUserForm(user: UserModel): FormGroup {
    return this.fb.group({
      name: user.name,
      ...
    });
  }
}
```

## Prefer using `ngOnChanges` over `ngOnInit`

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

Look at your use cases and check if you can remove `ngOnInit`. You almost never need `ngOnInit` (more on this in the following sub-chapter). If there are some default input values which should be set, you can do it alongside input declaration (`@Input() label: string = 'default label'`). Any other reactions to input changes should be done in `ngOnChanges`.

## Consider different initial assignment options

There is often a need to assign some properties during component initialization. This can be done in multiple ways:

1. In component's constructor
2. During property declaration
3. In lifecycle hooks
4. Subscribing at initialization

When learning Angular, you hear a lot about different lifecycle hooks. Even when creating new components using the Angular CLI, you get a component with empty `constructor` and empty `ngOnInit` method. Because of this, it seems natural to use one of these for initialization. We will explore what the options for assignment on initialization and what is the best way to do it.

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

We still have one problem with this solution. If input binding changes again (after the initial change), `isLegalAge` will not be re-assigned. To solve this we simply switch to using `ngOnChanges`:

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

To sump up, we recommend using ngOnChanges for computing values based on input changes. You can also use getters if they return primitive values and if the computation is quick and simple.

## Postfix observables with `$`

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

The `$` suffix is here to indicate and let you know it is something you can subscribe to, but it does not let you know if you can change the observable in case it is some type of a `Subject`, e.g. `ReplaySubject` or `BehaviorSubject`. This naming convention goes along with the `no-exposed-subjects` RxJS ESLint rule. The aforementioned rule will cause an error in case you have a publicly exposed` Subject`. The reasoning behind this is that subjects should never be publicly exposed unless converted to an observable using some pipeable operators or, for example, `asObservable` function.

```typescript
class ExampleComponent {
  private _isLoading$ = new BehaviorSubject(false); // for internal use for "state management"
  public isLoading$ = this._isLoading$.pipe(debounceTime(250)); // for use in template

  // to be called from some methods, for example during data fetching
  private updateLoadingState(state) {
    this._isLoading$.next(state);
  }
}
```

The potential downside of this convention/pattern is the name separation, in case of a naming conflict, the private member name should be prefixed with the `_`.

## Get to know the RxJS operators

To make good use of RxJS and develop the application in a reactive way, it is important to utilize the operators. RxJS operators allow you to modify the stream of data in multiple ways. Some common operators are:

- `map`
- `filter`
- `tap` (replaces `.do`)
- `delay`
- `debounceTime`
- `distinctUntilChanged`
- `catchError` (replaces `.catch`)
- `switchMap`

You can also write your own operators. If you do so, it is strongly recommended to write them as [pure pipeable operator functions](https://github.com/ReactiveX/rxjs/blob/master/doc/operator-creation.md#operator-as-a-pure-function).

Covering all the various operators is out of the scope of this handbook. Please refer to some of the learning resources mentioned earlier [[1](/books/frontend/angular/getting-started-with-angular/get-to-know-rxjs)] [[2](/books/frontend/angular/angular-guidelines-and-best-practices/core-libraries-configuration-and-tools#a-hrefhttpsgithubcomreactivexrxjs-target_blankrxjsa)].

## tap() vs subscribe()
When to use the `tap` operator in Rx flow? As per the [official documentation for the `tap` operator](https://rxjs.dev/api/operators/tap), it should be used when performing "side-effects". This means that the operator should be used when the user wants to "affect outside state with a notification without altering the notification". Although side-effects can be performed inside of the map operator, we should strive to write pure functions wherever possible and have a clear separation of responsibilities. This is one of the cases where the `tap` operator comes in handy.
Since the `tap` operator's callback is the same as the subscriber's `next` callback, nothing stops us from writing the same logic in either of those places. Well, nothing besides semantics. When writing Rx flows, we should convey as much information as we can just through our code, which means - use `tap` for performing side-effects and debugging/logging mostly, whereas subscribe represents the end of the flow which means that we have probably received some data which can then be used in application. This makes subscribe callback the correct place to do something with the data you've received through piped operators. This is also why an empty subscribe call is basically a code smell. Of course, there are exceptions when you cannot avoid using an empty subscribe(), but you should be mindful of it.

```ts
// Bad
source$.pipe(
  tap(() => {
    this.router.navigate(['some-route']);
  }),
).subscribe();
```
In the example above, we are navigating to a new location before we've reached the end of the flow which doesn't make too much sense. We could possibly add another operator in the flow which would make this approach even more wrong, especially if you forget to reorder operators so that the `tap` comes after all of the newly added mapping operators. 

```ts
// Better
source$.subscribe(() => {
  this.router.navigate(['some-route']);
})
```
The example above is more correct. Navigation and other, similar actions should be executed at the end of the Rx flow, which is in the subscribe's `next` callback.

## Observables and async/await

You can use async/await to await for the completion of observables. Keep in mind that this probably does not make too much sense for observables that emit more than one value, but it might be OK for things like HTTP requests.

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

## Avoid manual subscriptions and asynchronous property assignment

We recommend avoiding making manual subscription calls and asynchronous property assignment. The code example below has some comments indicating what is meant by "manual subscription" and "asynchronous property assignment".

It is best to leave subscription handling to the `async` pipe because it:

- Encourages [reactive programming](https://medium.com/front-end-weekly/why-should-we-use-reactive-programming-in-angular-2e2913297054)
- Handles unsubscribing automatically
- Works well with `OnPush` change detection
- Avoids the need to do asynchronous property assignment

Component:

```typescript
// bad
@Component(...)
export class ArticlesList {
  public articles: Array<Article>;

  constructor(...) {
    this.articleService.fetchArticles().subscribe((articles) => { // manual subscription
      this.articles = articles; // asynchronous property assignment
    });
  }
}

// good
@Component(...)
export class ArticlesList {
  public readonly articles$: Observable<Array<Article>> = this.articleService.fetchArticles();

  constructor(...) { }
}
```

Template:

```html
<!-- bad -->
<my-app-article-details
  *ngFor="let article of articles"
  [article]="article"
></my-app-article-details>

<!-- good -->
<my-app-article-details
  *ngFor="let article of articles$ | async"
  [article]="article"
></my-app-article-details>
```

## Avoid concatenation of translation keys

A common requirement for applications is that the application must be localized. Regardless of the library which is used for fetching the translations,,
we should avoid concatenation of translation keys. Concatenating translation keys makes managing of translation keys harder. It could prove very tiresome to keep the translations file in sync with translations that are used throughout the application, if you have to search for partial strings instead of searching for the whole string. One of the most tempting cases for using concatenation is when you iterate over some enum and match those values with the ones received from the backend service.

```ts
enum ApprovalStatus{
  Approved = 'Approved',
  Rejected = 'Rejected',
  NotApplicable = 'N/A',
}
```

```html
<mat-radio-button *ngFor="let approvalStatus of approvalStatusOptions">
  {{ 'enums.approvalStatus.' + approvalStatus | translate }}
</mat-radio-button>
```
In the example above, you also have to make sure that you create `approvalStatusOptions` structure which will be iterable and used in the `ngFor` directive.
In cases where enum keys don't match their values, there is also the question of storing translation keys in a way that will make the most sense. Should you use enum keys in the translations file, or should you use enum values corresponding to that key?

Another case that emphasizes the issue with this approach is when you need to add some new keys. You have to verify that you've added them to potentially multiple places and that everything will work together as intended.

A pattern that proved to work well for us is the "translatable enum" pattern which relies on using TypeScript's `Record` utility type and for which we've implemented `EnumPropertyPipe` that is available in the [ngx-nuts-and-bolts](https://infinum.github.io/ngx-nuts-and-bolts/docs/pipes/enum-property) library.

```ts
enum ITranslatableEnum {
  translationKey: string;
}

export enum ApprovalStatus {
  Approved = 'Approved',
  Rejected = 'Rejected',
  NotApplicable = 'N/A',
}

export const approvalStatusData: Record<ApprovalStatus, ITranslatableEnum> = {
	[ApprovalStatus.Approved]: {
		translationKey: 'enums.approvalStatus.approved',
	},
	[ApprovalStatus.Rejected]: {
		translationKey: 'enums.approvalStatus.rejected',
	},
  [ApprovalStatus.NotApplicable]: {
		translationKey: 'enums.approvalStatus.notApplicable',
	},
};

@Component({
  selector: 'demo-component',
  template: `
  <mat-radio-button *ngFor="let approvalStatus of approvalStatusOptions">
    {{ approvalStatus | enumProperty: approvalStatusData | translate}}
  </mat-radio-button>
  `
})
class DemoComponent{

  public readonly ApprovalStatus = ApprovalStatus;
  public readonly approvalStatusData = approvalStatusData;
}
```
By using this approach, your translation keys are all in one place, easing refactoring and keeping the translation file clean and up to date. You will also get compilation errors if a new enum is added without adding a corresponding entry with the translation key in the record. 
You can further expand on this pattern by adding additional "metadata" about the enum to the record. For example, a `sortingIndex` property that is taken into account when the enum values are rendered as dropdown options.

## Avoid unnecessary creation of multiple observables and avoid subscriptions

Taking the learnings from all the previous examples and the fact that we can use services during initial assignment, we can greatly simplify initialization of observables and their usage in templates.

In this example, we have to fetch the user by ID. The ID is read from route parameters.

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
// even worse
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

Here we no longer have to take care of unsubscribing, but the issue now is that during each template check, `user$` getter will be called. In _Example #4_ in one of the previous sub-chapters we had a case where using a getter was fine, but in this case it is not. Each time it is called it creates a new observable, template will subscribe to this new observable during each change detection, meaning that a new subscription will trigger another API call every CD cycle - which is unnecessary.

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

If the observable creation pipeline gets larger, you can move it to a private method like so:

```typescript
// good
@Component(...)
class MyComponent {
  public user$: Observable<UserModel> = this.createUserObservable();

  constructor(
    private route: ActivatedRoute,
    private usersService: UsersService,
  ) {}

  private createUserObservable(): Observable<UserModel> {
    return this.route.params.pipe(switchMap((params: Params) => {
      return this.usersService.fetchById(params.userId);
    }));
  }
}
```

## Subscribe as late as possible and use the operators

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

## Do not use observables if unnecessary

We recommend using `OnPush` change detection strategy. This means that some things that work with `Default` CD will no longer work when you start using `OnPush` CD. The best way to make things work again is to take a reactive approach when designing the component and utilize the Observables, operators and `async` pipe.

However, there are cases when you do not need an Observable. Sometimes, direct values work just as fine. Do not implement an Observable if it is not needed, it will just add some unnecessary overhead.

Here is an example where an observable is unnecessary:

```html
<!-- bad -->
{{ someNumber }}! = {{ factorialOfTheNumber$ | async }}
```

```typescript
// bad
@Component(...)
export class MyComponent implements OnChanges {
  @Input() public someNumber: number = 0;
  public readonly factorialOfTheNumber$ = new BehaviorSubject<number>(1);

  public ngOnChanges(changes: SimpleChanges): void {
    if (changes.someNumber) {
      this.factorialOfTheNumber$.next(calculateFactorial(changes.someNumber.currentValue));
    }
  }
}
```

```html
<!-- good -->
{{ someNumber }}! = {{ factorialOfTheNumber }}
```

```typescript
// good
@Component(...)
export class MyComponent implements OnChanges {
  @Input() public someNumber: number = 0;
  public readonly factorialOfTheNumber = 1;

  public ngOnChanges(changes: SimpleChanges): void {
    if (changes.someNumber) {
      this.factorialOfTheNumber = calculateFactorial(changes.someNumber.currentValue);
    }
  }
}
```

This can be a result of confusion about how `OnPush` CD works. If `factorialOfTheNumber$` is not an observable and is a direct value, it might be unclear if the template will be re-rendered or not. The answer is - yes, the template will be re-rendered correctly. While it is true that direct property assignment with `OnPush` CD can sometimes result in template not being re-rendered, that usually happens only when assigning in an asynchronous piece of code. If the `calculateFactorial` calculation is synchronous (as it is in this example), then we can assign the value directly, without the need to wrap it in an Observable.

Remember, any property assignments that are done synchronously in `ngOnChanges` will get rendered correctly. This is because CD is run synchronously after all the code in `ngOnChanges` is executed. This is true for both `OnPush` and `Default` CD.

## Consider not using ngOnChanges and use pure pipes

Building on the previous example with unnecessary Observables, we will go one step further and try to eliminate the `ngOnChanges` lifecycle hook altogether.

To do this, we have to understand how pipes work. Pipes can have multiple input values and one output value. Input values are mapped to the output value in the transformation method. Impure pipes will execute the transformation method on each change detection cycle. This is probably not optimal nor the desired behaviour in most situations. On the other hand, pure pipes will execute the transformation method only if some of the input values change. All pipes are pure by default, and that is great! You can read more about pipes [here](https://angular.io/guide/pipes).

We can not utilize the knowledge of pure pipes and update our component:

```html
{{ someNumber }}! = {{ someNumber | factorial }}
```

```typescript
@Component(...)
export class MyComponent implements OnChanges {
  @Input() public someNumber: number = 0;
}
```

```typescript
@Pipe({ name: 'factorial' })
export class FactorialPipe implements PipeTransform {
  public transform(value: number): number {
    return calculateFactorial(value);
  }
}
```

We have greatly reduced the amount of code in our component and have created a reusable piece of code in the form of a pure pipe. This pipe can be reused in other places, but it is also ok if it is used in only this one component's template. If it is not reused in other places, it is ok to leave the pipe implementation file alongside the component module file and declare it in the component's module.

You might ask a question "But what if I need to use the factorial value inside my component code?". The solution depends on how and when you need to use the value that is calculated by the pipe. If you need to access the value on some user interaction, you can pass the calculated value from the template to the action handler method. This is great because it keeps your methods pure.

```html
<ng-container *ngIf="someNumber | factorial as someNumberFactorial">
  {{ someNumber }}! = {{ someNumberFactorial }}

  <button (click)="saveFactorialValue(someNumberFactorial)">Save the factorial value</button>
</ng-container>
```

```typescript
@Component(...)
export class MyComponent implements OnChanges {
  @Input() public someNumber: number = 0;

  public saveFactorialValue(someNumberFactorial): void {
    // do something with someNumberFactorial
  }
}
```

Some notes:

- We wrapped everything in one `ng-container` to avoid calling the pipe twice
- Be careful in case that your pipe can return a valid falsy value, as in that case the `*ngIf` will not render the content; in such case you might consider using a [custom `*ngLet`](https://github.com/ngrx-utils/ngrx-utils#nglet-directive) structural directive

## Auto unwrap default exports when lazy loading

With Angular v15 it is possible to leverage default exports to shorten the syntax when lazy loading a module or a standalone component. Note that the CLI still generates modules and components without the default export so you will need to add that by yourself.

```ts
@Component({
  standalone: true,
})
export default class ExampleComponent { ... }

// Without default export
{
  path: 'lazy',
  loadComponent: () => import('./example-component').then(m => m.ExampleComponent),
}

// With default export
{
  path: 'lazy',
  loadComponent: () => import('./example-component'),
}
```

## Favouring canMatch guard

The [canMatch](https://angular.io/api/router/CanMatch) is the new type of guard introduced in Angular `v14.1`. It should be the preferred guard to use over `canActivate` or `canLoad`  which is deprecated in `v15`. `canMatch` has benefits of both worlds, it controls when the route can be used and as a side effect, whether we can download the code. In reality, this means that the chunk won't be loaded if the guard returns `false` but it will also be invoked/triggered every time the user tries to navigate. This is different from `canLoad`, which won't load the chunk, but once the guard returns `true`, it won't be called again which can cause some undesirable consequences, and `canActivate` which will be called every time, but that also means that chunk will be loaded in all cases. You can read more in the following [blog post](https://netbasal.com/introducing-the-canmatch-router-guard-in-angular-84e398046c9a) or in the [Angular team's PR description](https://github.com/angular/angular/pull/48180).

## No subscriptions in guards

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

## Be mindful of how and when the data is fetched

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

## The single observable pattern

Did you ever find yourself in the `ng-container` nesting hell? It might have looked something like this:

```html
<ng-container *ngIf="frameworkName1$ | async as frameworkName1">
  <ng-container *ngIf="frameworkName2$ | async as frameworkName2">
    <ng-container *ngIf="frameworkName3$ | async as frameworkName3">
      {{ frameworkName1 }} is way better than {{ frameworkName2 }}. I will not even talk about {{ frameworkName3 }}.
    </ng-container>
  </ng-container>
</ng-container>
```

There is a simple solution to this problem, and it is called the single data observable pattern. In short, this pattern takes advantage of the reactive programming paradigm introduced with RxJS to combine multiple observable streams into one, allowing us to reduce the amount of nested subscriptions in the template. In its purest form, it allows us to have only one subscription in the template (one `async` pipe). You can also utilize this pattern to create multiple combined observables instead of having many more individual subscriptions.

The example above could be refactored to use the single data observable pattern:

```ts
@Component(...)
export class MyComponent {
  private readonly frameworkName1$ = ...;
  private readonly frameworkName2$ = ...;
  private readonly frameworkName3$ = ...;

  public readonly frameworkNames$ = combineLatest([
    frameworkName1$,
    frameworkName2$,
    frameworkName3$,
  ]).pipe(map(([
    frameworkName1,
    frameworkName2,
    frameworkName3,
  ]) => {
    return {
      frameworkName1,
      frameworkName2,
      frameworkName3,
    }
  }))
}
```

```html
<ng-container *ngIf="frameworkNames$ | async as frameworkNames">
  {{ frameworkNames.frameworkName1 }} is way better than {{ frameworkNames.frameworkName2 }}. I will not even talk about {{ frameworkNames.frameworkName3 }}.
</ng-container>
```

We will not go into the details of this pattern in this handbook. To learn more about this pattern, please check out these videos:

- [Course Component Finished - Introduction to the Single Data Observable Pattern](https://angular-university.io/lesson/reactive-course-component-finished)
- [Reactive Angular - The Single Data Observable Pattern](https://angular-university.io/lesson/reactive-angular-the-single-data-observable-pattern)
- [Single Data Observable Pattern - Default Data Values](https://angular-university.io/lesson/reactive-single-data-observable-pattern-default-data-values)

## Avoid creating black-box components

The component's `Inputs` and `Outputs` are what we call the component's public API. There are many ways in which a component can be built and the definition of this public API for each specific component is completely up to the developer.

One of the approaches is to define the component's public API as a black box. This means that most or all of the implementation details are hidden and the component's user has a clearly defined and usually strict way of sending the data and configuration objects to the component, as well as getting some events back from the component. Component usually does some complex operations based on the input data and configuration.

One example of a black box component would be a tree component to which you pass the tree structure to the component as one root object that can have multiple levels of nested items. From the component's user's perspective, this seems like a nice and clear way of using a component, making this approach tantalizing.

Example how such a component would be used:

```html
<my-app-tree-view [rootNode]="navigationRoot">
</my-app-tree-view>
```

```typescript
interface ITreeNode {
  title: string;
  link: string;
  children?: Array<ITreeNode>;
}

@Component(...)
export class ParentComponent {
  public navigationRoot: ITreeNode = {
    title: 'Homepage',
    link: '/',
    children: [...]
  }
}
```

However, the black box approach is not very flexible and you could soon find yourself adding various configuration options and optional properties in the data objects (`ITreeNode` in the above example) as the new edge cases arise. This ends up with a bunch of if statements or similar logic in the component, growing in complexity and decreasing readability of the component's source code.

An alternative approach would be to decrease the amount of black-boxing and give the component's user a bit more choice in how the component is used. This can be achieved by good composition and content transclusion. The name for this alternative approach is glass box.

In the tree component example, this would mean that instead of having one large data object with nested children objects, we would leave the composition of the tree elements to the user.

Example of a glass box approach:

```html
<my-app-tree-node>
  <a routerLink="/">Homepage</a>

  <my-app-tree-node>
    ... child 1 ...
  </my-app-tree-node>

  <my-app-tree-node>
    ... child 2 ...
  </my-app-tree-node>
</my-app-tree-node>
```

This allows the parent component to add things like custom classes to the links or even make the links button instead of anchor elements. This makes for a more flexible component.

Please check out some of these real-world examples of good implementations of glass box components:

- [CDK Tree](https://material.angular.io/cdk/tree/overview)
- [Material tabs](https://material.angular.io/components/tabs/overview)

Here is also one interesting comparison between Material's and Nebular's implementation of a menu component:

- [Material menu](https://material.angular.io/components/menu/overview)
- [Nebular menu](https://akveo.github.io/nebular/docs/components/menu/overview#nbmenucomponent)

Material implementation takes the glass box approach, while Nebular leans more towards black-boxing. We believe this makes the Material implementation more flexible. For example, the material implementation allows you to have buttons or anchor elements inside the menu and to have custom components like icons inside each menu item. This is also possible with Nebular, but support for that had to be added in the menu component itself, by adding additional properties inside the menu data objects. This bloats the `NbMenuItem` interface which now includes all the various edge cases like icons, path matching, query params, badges, etc. If you need something similar to a badge but use your own custom component instead of the Nebular's badge, it might be hard to do it with this black box approach that Nebular took.

Please do not take this as us firing shots at Nebular. We still think that Nebular is a pretty good library, but there will always be some trade-offs when defining a component's public API. You could take 10 component libraries and none of them would have the best implementation of each and every component for each use-case of those components. Some will have some components implemented well for certain uses cases, while other component might not be as good.

Long story short, we recommend taking the glass box approach as it allows for more flexibility.
