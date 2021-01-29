*Guides are not rules and should not be followed blindly. Use your head and think.*

**Important:** This guide is made for Susy 2. For information about Susy 3, check out [Susy 3 Docs](http://oddbird.net/susy/docs/).

Susy is a Sass library that handles grid layouts, the math behind them, and contains some utilities. It has no inherent styling choices, as it is meant to be used only for the grid. Understanding the use of Susy mostly revolves around knowing its utilities and its configuration, so the guide will primarily focus on that.

## Warning

Before going through this guide, you will need some knowledge of Sass. For that, check out the [SASS reference](http://sass-lang.com/documentation/file.SASS_REFERENCE.html).

The guide is effectively split into two parts: Susy configuration and the actual guide. A more in-depth configuration explanation can be found in the documentation. If you are familiar with Susy, skip to the guide section.

## Configuration

First, we'll go through the configuration. The principles of using some of this might not be immediately apparent, but they will be explained in the guide section.

```scss
$susy: (
  flow: ltr,
  last-flow: to,
  math: fluid,
  column-width: false,
  output: float,
  container: auto,
  container-position: center,
  columns: 4,
  gutters: 0.25,
  gutter-position: after,
  global-box-sizing: content-box,
  debug: (
    image: hide,
    color: rgba(128, 128, 255, 0.25),
    output: background,
    toggle: top right,
  ),
  use-custom: (
    background-image: true,
    background-options: false,
    box-sizing: true,
    clearfix: false,
    rem: true,
  )
);
```

### Flow and last flow

`flow` is used to direct the flow of the columns. This will almost always be `ltr` (left to right), with the other option being `rtl` (right to left).

`last-flow` tells Susy where to float the last column. `from` will float to the left (assuming you've set the `ltr` flow) while `to` will float to the right (`to` is preferable). This isn't something you should touch unless you're sure that you need it.

### Math and column width

`math` is used to decide in which units and how to calculate column and gutter widths.

`fluid` math will output everything as percentages relative to the parent container. This is what you will usually use.
`static` math is used to output depending on the value in column-width (which has to be set for static math to be useful).

`column-width` can be set to either `false` (or `null`), and this is the default, as it only sets the outer maximum width for fluid widths, or a static unit, such as `200px` or `10em`.

### Output

`output` handles the way column positions are generated. `float` is the default and uses float to position elements, while `isolate` uses the negative margin trick. 
**Use float** as it's slightly more performant and much easier to spot errors and debug with. `isolate` is only useful when facing sub-pixel rounding errors, which it can avoid.

### Container and container position

`container` is used to set the max-width of the element with the applied container mixin. This can be either a relative or a static unit. If you're using static layouts, this is best left at `auto`, in order to avoid sub-pixel rounding errors.

The container position sets the position of the columns relative to the container. Center is usually preferred.

### Columns

`columns` determine the number of columns in the current layout. This is something you will probably use most often, as it is very versatile for creating any grid layout you need.

### Gutters and gutter position

`gutters` is the width of the gutter between columns. Its size is based on the column width. Therefore, `.25` means one quarter of the column width of a one-unit column.

`gutter-position` tells Susy where to put gutters. In case of `after`, gutters will be applied as margins on the right of the column, meaning the last column in a container should have no right margin. (This is solved in the span mixin whose usage will be explained in the guide section).

The possible values are:

 * `before`—margin on the left of the column
 * `after`—margin on the right of the column
 * `split`—half margin on the left and half on the right (this margins will not be removed unless explicitly overridden, this will mean columns will never go to the ends of the container)
 * `inside`—padding inside the column, also not removed on the ends of the container

### Border box sizing

`global-box-sizing` tells Susy the global default sizing so it can calculate the column size and gutters correctly.

_**This does NOT set box sizing globally.**_

There is a mixin for that, which will be explained in the guide section.

### Other

There are also [debug](http://susy.readthedocs.io/settings/#debug) and [custom options](http://susy.readthedocs.io/settings/#custom-support), which are not important for the purposes of this guide. You can read about them in the documentation.

[Susy documentation](http://susy.readthedocs.io/)
If something is not clear, check out the documentation for details.
