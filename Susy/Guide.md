
**Important:** this guide is made for Susy 2. For information about Susy 3, check out [Susy 3 Docs](http://oddbird.net/susy/docs/).

## Setting up

### Importing
After installing it in any way you need (gem, bower, npm), you need to import the main Susy file into your application.scss file.

```scss
@import 'susy';
```

The nice thing about Susy is that you can import it anywhere without the fear of extra CSS, because the only thing Susy offers are mixins and functions which won't compile into the final CSS.

### Box sizing
The next thing is to make sure the box sizing is set globally.
Use border-box sizing. This can easily be set up using a Susy mixin:

```scss
@include global-box-sizing(border-box);
```

This will go into the `_core.scss` file.

### Basic configuration
To set Susy configuration, you have to put the configuration object into the `$susy` Sass variable.

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

Use the `$layout` prefix for any Sass variable that holds a Susy configuration. The Susy configuration can be overridden on the fly when writing Sass, using the with-layout mixin - this is where the `$layout-no-gutter` variable could be used.

This goes into the `_config.scss` file.

## Using Susy
You will primarily use Susy through its mixins:

 * container
 * span
 * with-layout
 * break
 * gallery

This guide will go in-depth on using these correctly. There are also function variants of most of these, which will be covered briefly.

### Container
Container is set on elements the children of which are columns (defined by the span mixin). This will set the clearfix on the container element, as well as define the width if Susy is configured as static.

Usage:

```scss
.section {
  @include container();
}
```

### Span
The span mixin is used on elements that will behave as columns to the grid. A span element should always be underneath an element with a container defined in it.

Example usage:

```scss
.section {
  @include container();

  &__column {
    @include span(6);
  }
}
```

Span can take multiple parameters, the first one always being the number of column units the span takes. The maximum number of column units is defined in the columns configuration property. This can be overriden on a per-span basis by using the `of` syntax.

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

Lastly, regarding gutters: if using the gutter-position value `after`, you would need to set the last column to be the last in the span call. For instance:

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

In most cases, you want to use the gutter-position `before` or `after` - `after` being preferred.

### With layout
`with-layout` is a very useful mixin for generating specific context in which to use Susy mixins.

In the basic configuration section, there was an example layout configuration set in the `$layout-no-gutter` Sass variable.

```scss
$layout-no-gutter: (
  gutters: 0
);
```

If, for example, you need to generate columns with no gutters, by using the with-layout mixin you can generate a nested context with the new configuration as such:

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

In the previous example, this could have also been used to set the number of columns. For instance:

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
To be able to create rows, the mixin `break` is used. Break will force a clear on the columns appearing above, making it possible to create a row.

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

This would allow the first column to be in the row on its own (and take half of the container width) while the other two columns would go into the second row.

### Gallery
The `gallery` mixin is used when you need a varying number of same-height columns. As the name suggests, it makes creating a gallery grid easy.

The `gallery` mixin is very similiar to the span mixins, except there is no need to define the first or the last element (it allows for a dynamic number of items as such).

If items are not of the same height, a strange floating issue will occur - consider using break with span in that case.

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
While those are the meat of the Susy toolkit, there are some other useful functionality it offers.

### Push, Pull, Post
* `push` will move the span by the indicated number of column units (margin-left)
* `pull` will move a span backwards a number of column units (negative margin-left)
* `post` will move any span after this one a number of column units (margin-right)

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

With html

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
If, for some reason, you want to manually set widths of spans or columns, or just need the calculated widths, you can use the functions of the same name as mixins to get it.

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

For any other information, read the documentation.
