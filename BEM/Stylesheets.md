<div class="markdown-output__summary">
  or the proper way to use BEM in your stylesheets
</div>

*Guides are not rules and should not be followed blindly. Use your head and think.*

A simple form block using BEM can look like this:

```html
<form class="login-form">
  <div class="login-form__row">
    <label class="login-form__label">Username:</label>
    <input class="login-form__input login-form__input--username" />
  </div>
  <div class="login-form__row">
    <label class="login-form__label">Password:</label>
    <input class="login-form__input login-form__input--password" />
  </div>
</form>
```

Where the `login-form` class is the block name, the `login-form__row`, `login-form__label`, and `login-form__input` classes are elements, and the `login-form__input--username` and `login-form__input--password` classes are modifiers.

## Blocks

Syntax: `<block-name>`

### Block naming

A block name is a list of words separated with a dash describing the block and its use.

Take care not to use names that are too abstract or vague. Names such as `button` or `list` should be avoided as they can't be properly defined to fit everywhere. As such, you cannot reuse them properly. Names that are too specific are also not very useful, as they might prevent reuse on other parts of the page.

```html
<div class="my-block">
  <!--...-->
</div>
```

```css
.my-block {
  /* ... */
}
```

*Good names:*

```css
.blog-article,
.breadcrumbs,
.pagination,
.avatar-image,
.language-picker
```

*Bad names:*

```css
.list,
.main,
.content,
.footer-language-button,
.header-main-navigation-language-button
```

### Block modifiers

Syntax: `<block-name>--<block-modifier-name>`

A block modifier is a class that is used to modify the appearance of a block in certain contexts. A block modifier name is a list of names separated with a dash and prefixed with the block name and two dashes.

```scss
.my-block {
  /* ... */

  &.my-block--my-modifier {
    /* ... */
  }
}
```

```html
<div class="my-block my-block--my-modifier">
  <!--...-->
</div>
```

### Using blocks

Not everything is a block. This is not Minecraft. Just because an HTML element has children, that doesn't necessarily mean it is, or should be, a block. If an HTML element and its styling make no sense in the layout anywhere outside its position, it is not a block. If an HTML element cannot be reused, it is probably not a block. If an HTML element has no children, it is probably not a block. There are cases where this is not true, as each case is unique.

*A bad block example*

```html
<div class="main-navigation">
  <ul class="main-navigation__list link-list">
    <li class="link-list__item item">
      <a href="#" class="item__link">
    </li>
    <li class="link-list__item item">
      <a href="#" class="item__link">
    </li>
  </ul>
</div>
```

```scss
.main-navigation {
  /* ... */

  .main-navigation__list {
    /* ... */
  }
}

.link-list {
  /* ... */

  .link-list__item {
    /* ... */
  }
}

.item {
  /* ... */

  .item__link {
    /* ... */
  }
}
```

The "item" block is not really a block. Actually, it is meaningful only under the link-list (or even main-navigation) block. As such, it should be only an element, even though it has a child element.

*Fixed example*

```html
<div class="main-navigation">
  <ul class="main-navigation__list link-list">
    <li class="link-list__item">
      <a href="#" class="link-list__link">
    </li>
    <li class="link-list__item">
      <a href="#" class="link-list__link">
    </li>
  </ul>
</div>
```

```scss
.main-navigation {
  /* ... */

  .main-navigation__list {
    /* ... */
  }
}

.link-list {
  /* ... */

  .link-list__item {
    /* ... */
  }

  .link-list__link {
    /* ... */
  }
}
```

## Elements

Syntax: `<block-name>__<element-name>`

### Element naming

An element name is a list of words separated with a dash describing the element and its use and purpose under its block. It is prefixed with the block name and two underscores, or just two underscores as a shorthand. Using a full name is recommended, as overriding behavior might occur otherwise.

Unlike blocks, naming elements can be abstract and vague, as they are meaningful only under their parent block. Still, you should take care not to have names that describe the element's use in a block. There is no point in using the block name in the element name, as that would only result in redundancy.
The parent & symbol from Sass proves very useful when writing elements. A block with an element can be written in Sass as:

```scss
.my-block {
  /* ... */

  &__my-element {
    /* ... */
  }
}
```

Resulting in a CSS:

```scss
.my-block {
  /* ... */

}

.my-block__my-element {
  /* ... */
}
```

If you need it to nest inside of the block class, in Sass, you would have to use the & symbol before the element, or use string interpolation.

Example:

```scss
.my-block {
  /* ... */

  & &__my-element {
    /* ... */
  }
  /* OR */
  #{&}__my-element {
    /* ... */
  }
}
```

Results in:

```scss
.my-block {
  /* ... */

}
  .my-block .my-block__my-element {
   /* ... */
  }
```

Where the `&` character will interpolate into the block name. Use it.

Examples:

```html
<div class="my-block">
  <div class="my-block__my-element">
    <!--...-->
  </div>
</div>
```

```scss
.my-block {
  /* ... */

  .my-block__my-element {
    /* ... */
  }
}
```

*Good names:*

```css
.<block-name>__item,
.<block-name>__link,
.<block-name>__title,
.<block-name>__row,
.<block-name>__column
```

*Bad names:*

```css
.<block-name>__blog-article-title,
.<block-name>__header-list,
.<block-name>__element /* Too vague even for an element */
```

### Element modifiers

Syntax:

```html
<block-name>__<element-name>--<element-modifier-name>
```

An element modifier is a class that is used to modify the appearance of an element in certain contexts. An element modifier name is a list of names separated with a dash and prefixed with the element name and two dashes.

```scss

.my-block {
  /* ... */

  &__my-element {
    /* ... */

    &--my-modifier {
      /* ... */
    }
  }
}
```

```html
<div class="my-block">
  <div class="my-block__my-element my-block__my-element--my-modifier">
    <!--...-->
  </div>
</div>
```

### Using elements

Elements make sense only inside a block. Never should an element be without its block, either in HTML or Sass. When reusing the block, its elements don't necessarily have to be used in the same order, nor do they have to be used at all.

*Bad elements example*

```html
<div class="statistics-list">
  <div class="statistics-list__statistic statistic">
    <span class="statistic__plain-text">Over</span>
    <h1 class="highlighted">
      <span class="highlighted__text">22</span>
    </h1>
    <span class="statistic__plain-text">years of experience.</span>
  </div>
  <div class="statistics-list__statistic statistics-list__statistic--no-first statistic">
    <h1 class="highlighted">
      <span class="highlighted__text">6</span>
    </h1>
    <span class="statistic__plain-text">Countries</span>
  </div>
</div>
```

```scss
.statistics-list {
  /* ... */

  &__statistic {
    /* ... */

    &--no-first {
      /* ... */
    }
  }
}

.statistic {
  /* ... */

  &__plain-text {
    /* ... */
  }
}

.highlighted {
  /* ... */

  &__text {
    /* ... */
  }
}
```

The `highlighted` block is not really a block, it is an element, as it can only live inside of the statistic block. Also, the `highlighted__text` element it houses also makes sense only inside the statistic, so it is an element of the statistic, and not highlighted.

The `statistics-list__statistic--no-first` modifier doesn't make sense, as the modifier should be applied to the statistic block. The `statistics-list__statistic` element shouldn't worry about its content implementation, it should have styles that are based only on its position and appearance inside the "statistics-list" block.

*Fixed example*

```html
<div class="statistics-list">
  <div class="statistics-list__statistic statistic">
    <span class="statistic__plain-text">Over</span>
    <h1 class="statistic__highlighted">
      <span class="statistic__text">22</span>
    </h1>
    <span class="statistic__plain-text">years of experience.</span>
  </div>
  <div class="statistics-list__statistic statistic statistic--no-first">
    <h1 class="statistic__highlighted">
      <span class="statistic__text">6</span>
    </h1>
    <span class="statistic__plain-text">Countries</span>
  </div>
</div>
```

```scss
.statistics-list {
  /* ... */

  &__statistic {
    /* ... */

  }
}

.statistic {
  /* ... */

  &--no-first {
    /* ... */
  }

  &__plain-text {
    /* ... */
  }

  &__highlighted {
    /* ... */
  }

  &__text {
    /* ... */
  }
}
```

### Other notes

* An HTML element can be a block and an element, and can have any number of modifiers for both. However, it will never be 2 or more blocks, or 2 or more elements.
* **A block should never care about its position on the page, only about its appearance.** If a block needs to be positioned inside the block it is in, it is probably also an element of the parent block.
* A modifier should not be used to change the appearance of a block or an element in a dynamic way. An IS class is used for that. As the modifier contains the block name, it should not be selected through JavaScript, as well as any other block or element class.
