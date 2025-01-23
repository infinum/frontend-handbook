Dependency Injection (DI) is utilized heavily in Angular, primarily for providing instances of either primitive values or class instances.

DI is a complex topic that requires a separate discussion which would be out of the scope of this handbook. We recommend checking out the following resources:

- The [official DI guide](https://angular.dev/guide/di/dependency-injection) which explains many details
- This [repo](https://github.com/fvoska/angular-di-demo) for some examples
- This useful [infographic](https://christiankohler.net/angular-dependency-injection-infographic) that gives a good graphical overview of the whole DI mechanism

As of Angular 6, make sure to use `{ providedIn: 'root' }` [for singletons](https://angular.dev/guide/ngmodules/singleton-services) whenever possible. Alongside this, prefer to inject dependencies using the `inject` function over `constructor`.

```ts
// OK, but not preferred
constructor(private readonly userService: UserService){}

// Preferred way
private readonly userService = inject(UserService);
```

DI is very useful for testing purposes, as shown in a later chapter.
