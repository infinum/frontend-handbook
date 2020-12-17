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
