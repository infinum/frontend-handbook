<div class="markdown-output__summary">
  or the proper way to use BEM in your javascripts
</div>

*Guides are not rules and should not be followed blindly. Use your head and think.*

Syntax: `js-<identifier>`

JavaScript specific classes are used to reduce the risk of breaking JavaScript behavior and/or functionality. A JavaScript specific class should **never have any styling on it**, as it is used only as a selector for JavaScript. Any element that is needed to be selected directly from code should have a JavaScript specific class.

*Example:*

```html
<a href="/login" class="navigation__button navigation__button--login js-login"></a>
```

## Is utility classes

Syntax: `is-<state-name|action-name>`

Is classes should be used to make dynamic changes to the appearance of blocks or elements that are controlled through JavaScript. For example when you have a block that you will be showing and hiding through JavaScript, you would add an "is-shown" class to the block when it is shown.
Is classes unlike js classes should have styling, but they shouldn't be used without JavaScript, as modifiers exist for the static styling.

*Example:*

```html
<a href="/login" class="__button __button--login js-login is-shown is-hovered"></a>
```
