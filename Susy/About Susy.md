*Guides are not rules and should not be followed blindly. Use your head and think.*

Susy is a Sass library that handles grid layouting, the math behind it, and contains some utilities. It has no inherent styling choices, as it is meant to be used only for the grid. Understanding the use of Susy mostly revolves about knowing its utilities and its configuration, so the guide will first focus on that.

## Warning
Before going through this guide some knowledge of Sass is needed. For that check the [SASS reference](http://sass-lang.com/documentation/file.SASS_REFERENCE.html).

This guide is effectively split into two parts: Susy configuration and the actual guide. A more in-depth configuration explanation can be found in the documentation. If you are familiar with Susy, skip to the guide section.

## Configuration

Firstly we'll go through the configuration. The usage of some of this might not be immediately apparent, but will be explained in the guide section.

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
`flow` is used to direct the flow of the columns. This is almost always going to be `ltr` (left to right) with the other option being `rtl` (right to left)

`last-flow` tells Susy where to float the last column. `from` will float to the left (assuming `ltr` flow) while `to` will float to the right. (`to` is preferable). This isn't something you should touch unless you're sure that you need it.

### Math and column width
`math` is used to decide in which units, and how to calculate column and gutter widths.

`fluid` math will output everything as percentages such as to be relative to the parent container. This is what you will use the most.  
`static` math is used to output depending on the value in column-width (which has to be set for static math to be useful).

`column-width` can be set to either `false` (or `null`), and this is the default as for fluid widths it only sets the outer maximum width or a static unit such as `200px` or `10em`.

### Output
`output` handles the way columns positions are generated. `float` is the default and uses float to position elements, while `isolate` uses a negative margin trick.  
**Use float** as it's slightly more performant and much easier to spot errors and debug with. `isolate` is only useful when facing sub-pixel rounding errors, which it can avoid.

### Container and container position
`container` is used to set the max-width of the element with the applied container mixin. This can be either a relative or a static unit. If using static layouts, this is best left at auto, in order to avoid sub-pixel rounding errors.

Container position sets where the columns will be positioned relative to the container. Center is usually preferred.

### Columns
`columns` determine the number of columns in the current layout. This is something you will probably use the most, as it is very versatile for creating any grid layout you need.

### Gutters and gutter position
`gutters` is the width of the gutter between columns, and the size is based on the column width. Therefore `.25` means one quarter of the column width of a one unit column.

`gutter-position` tells Susy where to put gutters. In case of `after`, gutters will be applied as margins on the right of the column, meaning the last column in a container should have no right margin. (This is solved in the span mixin whose usage will be explained in the guide section)

The possible values are:

 * `before` - margin left on column
 * `after` - margin right on column
 * `split` - half margin on the left and half on the right (this margins will not be removed unless explicitly overridden, this will mean columns will never go to the ends of the container)
 * `inside` - padding inside the column, also not removed on ends of container

### Border box sizing
`global-box-sizing` sizing tells Susy the global default sizing, so it can calculate column and gutters correctly.

_**This does NOT set box-sizing globally.**_

There is a mixin for that which will be explained in the guide section.

### Other
There are also debug and custom options, which are not important for the purpose for this guide, and as such you can read about those in the documentation.

[Susy documentation](http://susydocs.oddbird.net/en/latest/)  
For anything that is unclear check the documentation for details.
