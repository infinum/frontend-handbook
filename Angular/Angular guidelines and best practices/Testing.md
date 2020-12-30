Writing good quality tests for critical functionalities is an important step in quality assurance. We do not recommend not writing any tests, nor do we recommend mindlessly trying to achieve 100% coverage. Applications with 100% coverage can still have bugs either because the tests are not good enough, or if the edge cases were not covered by the implementation in the first place.

There are many tips and tricks that go into testing Angular applications. This section will cover some of them.

## Unit vs. integration vs. end-to-end testing

Tests can usually be placed into one of the three categories: unit, integration, or end-to-end tests.

### Unit testing

Unit tests, as the name suggests, test units. A unit is some individual piece of software which can also have some external dependencies. In the context of Angular, units are components, services, guards, pipes, helper functions, interceptors, models and other custom classes, etc.

Unit testing in Angular comes out-of-the-box with Jasmine, as both the testing framework and assertion library. It is also possible to generate good coverage reports as HTML files which can be presented to management or some other interested parties.

### Integration testing

Integration testing includes multiple units which are tested together to check how they interact. In Angular, there is no special way of doing integration testing. There is a thin line between unit and integration tests, and that line is more conceptual than technical. For integration testing, you still use Jasmine, which comes with every project generated by Angular CLI.

What makes a test an integration test instead of a unit test? Well, it depends on how much you mock during testing. If component A which you are testing renders components B, C, and D, and depends on services S1 and S2, you have a choice:

1. You can test component A using mocks for components B, C, and D, and mocks for services S1 and S2. We could call that a unit test of component A since it is testing only component A's functionality, and it assumes that other components and services work as expected.
2. You can test component A using real components B, C, and D and real services S1 and S2. This will make the `TestBed` module configuration a bit heavier, and tests might run a bit slower. However, we could call this an integration test of the interaction between components A, B, C, and D and services S1 and S2.

So, it is really up to you to decide whether you want to test some component with all its dependencies mocked, or if you want to test interaction with other real components as well.

What you call those Jasmine tests—unit or integration—doesn't really matter, ♪ *anyone can see* ♪.

### E2E testing

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

## Utilizing TestBed

Dependency injection is very powerful during testing, as it allows you to provide mock services for testing purposes. You will sometimes want to spy on some private service's methods. This might tempt you to make the service public or to bypass TS by doing something like `component['myService']`. To avoid these *dirty* solutions, you can utilize `TestBed`.

Consider this example with a header component and user service which is used for logging in:

``` html
...
<button (click)="onLogInClick">Log in</button>
...
```

```typescript
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

```typescript
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

## Testing a service

We will test `SomeService` which injects `UserService`:

```typescript
@Injectable({ providedIn: 'root' })
export class SomeService {
  constructor(private userService: UserService) { }

  // ...rest of functionality
}
```

To make this test a "unit" test, we will mock all `SomeService`'s dependencies:

```typescript
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

```typescript
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

## Testing a component

When testing complex components that use multiple services and render other components, make sure to provide them with stub services and components.

In this example, we will test the `ComponentToBeTested` component which renders `SomeOtherComponent` and depends on `UserService`:

```typescript
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

```typescript
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

## Testing component inputs

If you want to test a component under different conditions, you will probably have to change some of its `@Input()` values and check if it emits `@Output()`s when expected.

To change the component input, you can simply:

```typescript
@Component(...)
class SomeComponent {
  @Input() someInput: string;
}
```

```typescript
component.someInput = 'foo';
fixture.detectChanges(); // To update the view
```

If you are doing stuff in `ngOnChanges`, you will have to call it manually since `ngOnChanges` is not called automatically in tests during programmatic input changes.

```typescript
component.someInput = 'foo';
component.ngOnChanges({ someInput: { currentValue: 'foo' } } as any);
fixture.detectChanges();
```

## Testing component outputs

To test component outputs, simply spy on the output emit function:

```typescript
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

```typescript
spyOn(component.buttonClicked, 'emit');

expect(component.buttonClicked.emit).not.toHaveBeenCalled();

fixture.debugElement.query(By.css('button')).nativeElement.click();

expect(component.buttonClicked.emit).toHaveBeenCalled();
```

## Testing components with the `OnPush` change detection

If the component you are testing is using the `OnPush` change detection, `fixture.detectChanges()` will not work. To fix this, you can override the CD only during testing:

```typescript
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

## Testing components with content projection

Components that use content projection cannot be tested in the same way as components that use only inputs for passing the data to the component. The problem is that, during TestBed configuration, you can only configure the testing module. You cannot define the template which would describe how the component is used inside another component's template.

One of the easiest ways to test content projection is to simply create a wrapper component for testing purposes only.

`TranscluderComponent`:

```typescript
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

```typescript
@NgModule({
  declarations: [TranscluderComponent],
  imports: [CommonModule],
  exports: [TranscluderComponent],
})
export class TranscluderModule { }
```

`TranscluderComponent` tests:

```typescript
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

## Trigger events from DOM instead of calling handlers directly

If you have some buttons with click handlers, you should test them by clicking on the elements instead of calling the handler directly.

Example:

``` html
<button class="login-button" (click)="onLogInClicked()">Log in</button>
```

Note: for simplicity, we will assume that the login action is synchronous.

```typescript
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

## Testing HTTP requests

Your application will most likely be making requests left and right, so it is important to test how your app behaves when requests succeed and fail.

We will go through the whole process from service creation to testing success/failure states.

### Service setup

We will create a simple `DadJokeService` which will be used for fetching jokes from `https://icanhazdadjoke.com`.

The service looks like this:

```typescript
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

```typescript
export interface IJoke {
  id: string;
  joke: string;
  status: number;
}
```

### Test setup

Let's start by creating the most basic test which ensures only that our `DadJokeService` instantiates correctly.

If you've generated your service using Angular CLI, the `spec` file will probably look something like this:

```typescript
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

```typescript
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

If you run the tests now, they will fail because our service injects `HttpClient`, and we have not provided it in our tests. Luckily, there is a module called `HttpClientTestingModule`, which you can import from `@angular/common/http/testing`. It provides a complete mocking backend for the `HttpClient` service. We just have to import it in our `TestBed` module:

```typescript
TestBed.configureTestingModule({
  imports: [
    HttpClientTestingModule,
  ]
});
```

Our basic test now passes.

### Mocking HTTP requests in tests

This is the fun part—we would like to test how our service behaves if a request succeeds or fails.

We will start by testing a successful request:

```typescript
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

```typescript
it('should return a default joke if request fails', () => {
  service.getJoke().subscribe((joke) => {
    expect(joke).toBe(DadJokeService.DEFAULT_JOKE);
  });

  const mockRequest = httpMock.expectOne(DadJokeService.I_CAN_HAZ_DAD_JOKE_URL);

  mockRequest.error(null);
});
```

In our specific example, it was not important which particular error happened because our error catching implementation in the `catchError` operator has no logic which would determine different behavior depending on error code. For example, if we wanted to test how our error handling handles 500 server errors, we could do something like this:

```typescript
mockRequest.error(new ErrorEvent('server_down'), { status: 500 });
```

This allows us to test a more complex error handling, which usually includes some logic for displaying different error messages depending on error code. As shown, that is completely doable using the `error` method of the `TestRequest` object.

## Testing helpers

If you have helper functions, testing them is basically the same as in any other application in any framework (even Vanilla JS). You do not need TestBed. You just need good old Jasmine, and you test your helpers as pure functions. If your helpers are not pure functions, you should really make them pure.

## Creating wrapper test components

TODO

## Typing mock services

TODO: ExtractPublic<T>