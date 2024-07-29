As you might already know, there are two approaches when working with forms in Angular—template-driven and reactive forms. There is a lot of great documentation on both:

- [Reactive Forms](https://angular.dev/guide/forms/reactive-forms)
- [Template-driven Forms](https://angular.dev/guide/forms)

The real question is which approach should you use? General reasoning is that template-driven forms should suffice if the form is simple enough (one or two elements). If the form is complex and has a lot of elements and validations, the reactive approach is easier to manage.

Going by that general reasoning, you might end up having a mix of reactive and template-driven forms in your application. For that reason, here at Infinum, we usually lean towards reactive forms. While they do have some initial overhead, forms often get more complex over time. If you start with using reactive forms, there is no need to switch from template-driven forms when they get harder to manage.

Reactive forms are also a bit easier to hook into RxJS pipelines.

## Typed forms
Typed forms help catch errors early in development, enhance code readability and maintainability, and support useful features like auto-completion. You do not have to do anything special to create your first typed `FormControl`, form controls are typed by default since Angular v15.

In majority of the cases Angular will correctly infer the type from the initial value. Only use explicit typings by using generics when the form control can accept more than one type.

```ts
private exampleControlTwo = new FormControl('');
// FormTestComponent.exampleControlTwo: FormControl<string | null>

private exampleControlTwo = new FormControl<string | number>('');
 // FormTestComponent.exampleControlTwo: <string | number | null>
```

In the above example `null` shows up as a possible type. This is because if we reset the `FormControl`, it will be reset to `null`. If we want to reset the control to its initial value, we must pass `nonNullable` flag.

```ts
private exampleControlTwo = new FormControl('', {nonNullable: true});
// FormTestComponent.exampleControlTwo: FormControl<string>
```

If both initial and generic types are not supplied, Angular won't be able to infer the type and the type will be set to `any`. Avoid doing this.

### Form value and raw value types
Angular does not come with built-in utility type for inferring the type of value property nor getRawValue method. Thanks to the talented individuals in our company, we have created a utility type that you can use. The helpers are part of the ngx-nuts-and-bolts package on the NPM. Take a look at official [documentation](https://infinum.github.io/ngx-nuts-and-bolts/docs/form-utils) to learn more.

### Form control bindings
Aim to bind form controls directly in the template using `formControl` instead of `formControlName` to catch errors as early as possible or not making one in the first place. Using this approach will throw an error if you try to access a control that does not exist, but you will also get auto-complete which further reduces the chance of making an error. You also protect yourself when making changes in the future, because renaming or deleting a FormControl will result in an error in the template. The same goes for accessing form controls in the components. Avoid using ‘get’ to catch errors earlier in the development process.

```ts
// Component
    private readonly exampleFormGroup = new FormGroup({
        exampleControlOne: new FormControl('exampleOne'),
        exampleControlTwo: new FormControl('exampleTwo', { nonNullable: true }),
    });

// Template
    <form [formGroup]="exampleFormGroup">
        <input formControlName="exampleControlThree" />
    </form>
    // Won't throw an error

    <form [formGroup]="exampleFormGroup">
        <input [formControl]="exampleFormGroup.controls.exampleControlThree" />
    </form>
    // Will throw an error in compile time
```

## NgxFormObject - our own library for working with forms!

Since we mostly use reactive forms, we've created a library, which makes things a bit easier. The library in question is [ngx-form-object](https://github.com/infinum/ngx-form-object). Check out the official [docs](https://infinum.github.io/ngx-form-object/docs) to find out how to use it.

Some of the benefits of using it:

- Definition of form attributes on the model level using decorators
- Support for single-values and relationships (Attribute, BelongsTo, HasMany)
- Introduces `FormObject` and `FormStore` for decoupling data and validation
- Form generation based on the model definition

We definitely recommend trying out `ngx-form-object` (*shameful self-promotion*) if you decide to use reactive forms. We've used it on many complex projects with huge forms and it has helped us a lot!

## Making your components work with forms transparently

Another topic that should be covered when talking about forms is using your own components with both reactive and template-driven forms. To use your component with forms just like you would use the `input` element, you will have to implement [ControlValueAccessor](https://angular.dev/api/forms/ControlValueAccessor) in your component. It isn't too straight-forward and might seem complex at first glance (what is `forwardRef` anyway, right?), but luckily, there are some great articles which we recommend reading:

- [Angular Custom Form Controls Made Easy](https://netbasal.com/angular-custom-form-controls-made-easy-4f963341c8e2)—quick introduction
- [Never Again be Confused when Implementing ControlValueAccessor in Angular Forms](https://blog.angularindepth.com/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms-93b9eee9ee83)—more in-depth, great article, recommend reading; but uses jQuery in the example—so démodé

You can implement both two-way binding and `ControlValueAccessor` if you want your component to be usable both inside and outside a form. However, you will usually not use the same component both inside and outside a form, so implementing just `ControlValueAccessor` or two-way binding should be enough. Which route you choose to go depends on the use case.
