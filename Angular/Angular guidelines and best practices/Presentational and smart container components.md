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
