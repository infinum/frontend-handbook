**Important:** This guide is written for Susy 2. For information about Susy 3, check out [Susy 3 Docs](http://oddbird.net/susy/docs/).

## Setting up

### Importing
After installing it in any way you need (gem, Bower, npm), you need to import the main Susy file into your application.scss file.

```scss
@import 'susy';
```

The great thing about Susy is that you can import it anywhere without fear of extra CSS, because the only thing Susy offers are mixins and functions which won't compile into the final CSS.

### Box sizing
The next thing is to make sure that box sizing is set globally.
Use border-box sizing. This can be easily set up using a Susy mixin:

```scss
@include global-box-sizing(border-box);
```

This will go into the `_core.scss` file.

### Basic configuration
To set up a Susy configuration, you have to put the configuration object into the `$susy` Sass variable.

For example:

```scss
$layout-default: (
  columns: 12,
  gutters: 0.25,
  gutter-position: after,
  global-box-sizing: border-box
);

$layout-no-gutter: (
  gutters: 0
);

$susy: $layout-default;
```

Use the `$layout` prefix for any Sass variable that holds a Susy configuration. The Susy configuration can be overridden on the fly when writing Sass, using the with-layout mixin—this is where the `$layout-no-gutter` variable can be used.

This goes into the `_config.scss` file.

## Using Susy
You will use Susy primarily through its mixins:

 * container
 * span
 * with-layout
 * break
 * gallery

This guide will go in depth on using these correctly. There are also function variants of most of these, which will be covered briefly.

### Container
A container is set on elements whose children are columns (defined by the span mixin). This will set the clearfix on the container element as well as define the width if Susy is configured as static.

Usage:

```scss
.section {
  @include container();
}
```

### Span
The span mixin is used on elements that will behave as columns to the grid. A span element should always be underneath an element with a container defined in it.

Example of use:

```scss
.section {
  @include container();

  &__column {
    @include span(6);
  }
}
```

A span can take multiple parameters, the first one always being the number of column units the span takes. The maximum number of column units is defined in the column configuration property. This can be overridden on a per-span basis by using the `of` syntax.

Example:

```scss
.section {
  @include container();

  &__column {
    @include span(2 of 4);
  }
}
```

This would generate a column half the width of the container (minus the gutter).

Finally, regarding gutters: if you're using the `after` gutter-position value, you have to set the last column to be the last in the span call. For instance:

```scss
.section {
  @include container();

  &__column-left {
    //if gutter-position was before last parameter of this span would be first (or alpha)
    @include span(2 of 4);
  }

  &__column-right {
    //can use omega instead of last
    @include span(2 of 4 last);
  }
}
```

In most cases, you want to use the `before` or `after` gutter-position, with `after` being preferred.

### With layout
`with-layout` is a very useful mixin for generating specific context in which to use Susy mixins.

In the Basic configuration section, there was an example layout configuration set in the `$layout-no-gutter` Sass variable.

```scss
$layout-no-gutter: (
  gutters: 0
);
```

If, for example, you need to generate columns with no gutters, you can generate a nested context with the new configuration by using the with-layout mixin:

```scss
@include with-layout($layout-no-gutter) {
  .section {
    @include container();

    &__column-left {
      @include span(2 of 4);
    }

    &__column-right {
      //last makes no sense here, as there are no gutters to remove
      @include span(2 of 4);
    }
  }
}
```

In the previous example, this could have been used to set the number of columns as well. For instance:

```scss
$layout-for-section: (
  gutters: 0,
  columns: 4
);

@include with-layout($layout-for-section) {
  .section {
    @include container();

    &__column-left {
      @include span(2);
    }

    &__column-right {
      @include span(2);
    }
  }
}
```

You can see how this makes Susy very flexible, without the need for custom overriding.

### Break
You use the `break` mixin to create rows. The break will force a clear on the columns appearing above, making it possible to create a row.

In the case of the following Sass and HTML:

```scss
.section {
  @include container();

  &__row {
    @include break();
  }

  &__column {
    @include span(6);
  }
}
```

```html
<div class="section">
  <div class="section__row">
    <div class="section__column"></div>
  </div>
  <div class="section__row">
    <div class="section__column"></div>
    <div class="section__column"></div>
  </div>
</div>
```

This allows the first column to be on its own in the row (and take half the container width), while the other two columns go to the second row.

### Gallery
The `gallery` mixin is used when you need a varying number of same-height columns. As the name suggest, it makes creating a gallery grid easy.

The `gallery` mixin is very similiar to the span mixin, except there is no need to define the first or last element. (It allows for a dynamic number of items as such).

If the items are not of the same height, a strange floating issue will occur—consider using a break with span in that case.

Usage:

```scss
.section
  @include container();

  &__gallery {
    @include gallery(3 of 12);
  }
}
```

## Other functionality
While these are the flesh of the Susy toolkit, there is some other useful functionality it offers.

### Push, pull, post
* `push` will move the span the indicated number of column units (margin-left)
* `pull` will move the span backwards a number of column units (negative margin-left)
* `post` will move any span after the current one a number of column units (margin-right)

Example:

```scss
.section
  @include container();

  &__span-1 {
    @include span(2 of 11);
    @include push(1 of 11);
  }

  &__span-2 {
    @include span(2 of 11);
    @include post(1 of 11);
  }

  &__span-3 {
    @include span(1 of 11);
  }

  &__span-4 {
    @include span(2 of 11);
    @include push(2 of 11);
  }

  &__span-5 {
    @include span(1 of 11);
    @include pull(2 of 11);
  }
}
```

With HTML

```html
<div class="section">
  <div class="section__span-1"></div>
  <div class="section__span-2"></div>
  <div class="section__span-3"></div>
  <div class="section__span-4"></div>
  <div class="section__span-5"></div>
</div>
```

Will result in:

```scss
//#1122#3#544
//where
//# is empty space
//1-5 is where the span divs are.
```

### Functions
If, for some reason, you want to manually set the width of spans or columns, or just need the calculated widths, you can use the functions of the same name as mixins.

Examples:

```scss
.container {
  //only makes sense with static layout
  width: container(12);
}

.span {
  width: span(6 of 12);
  margin-right: gutter(6 of 12);
}
```
Read the documentation if you need any other information.
