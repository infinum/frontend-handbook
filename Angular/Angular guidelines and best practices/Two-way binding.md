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
