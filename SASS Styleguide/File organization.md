*Guides are not rules and should not be followed blindly. Use your head and think.*

In all web projects we usually have some kind of a css/sass folder, naming of which can sometimes be chosen. Usual names would be:

* Stylesheets
* Styles
* CSS
* SASS

If needed, brush up on sass partials and imports, as those are important to understand the file structure.
If the app is a Rails app, sprockets can be used instead of imports, differences will be noted when necessary.

Because of the BEM philosophy, files will usually follow the strucutre of components (that being blocks). An example of a structure would be:

* **vendor**
  * normalize.css
  * other-vendor.css
  * ...
* **overrides**
  * normalize.scss
  * other-vendor.css
  * ...
* **utils**
  * \_colors.scss
  * \_z-indexes.scss
  * \_shared-variables.scss
  * \_media.scss
  * \_placeholders.scss
  * \_mixins.scss
  * \_fonts.scss
  * \_animations.scss
* **components**
  * \_header.scss
  * \_nav-list.scss
  * \_article.scss
  * ...
* **pages**
  * \_homepage.scss
  * \_about-us.scss
  * \_some-other-page.scss
  * ...
* \_config.scss
* \_core.scss
* \_shame.scss
* application.scss

Only the main file (in example application.scss) should be a normal sass file, everything else should be a partial. This is so only one css file is generated, and when a file is not imported, it will not be unnecessarily compiled.

The application.scss file should only hold imports, no actual rules, styling, variables, mixins etc. can go in it. Usually it would look something like:

```css
@import 'vendor/**/*';
@import 'overrides/**/*';

@import 'config';
@import 'utils/**/*';

@import 'core';

@import 'components/**/*';
@import 'pages/**/*';

@import 'shame';
```

If everything is done correctly, load order shouldn't matter, so usage of wildcards is recommended in sass-rails projects. If using libsass it might not work (currently it doesn't) so files need to be imported individualy. A recommended way of solving the libsass problem would be using folder name files to load. For example structure:

* **folder**
  * \_first.scss
  * \_second.scss
* \_folder.scss
* application.scss

The `folder.scss` file would contain:

```css
@import 'folder/first';
@import 'folder/second';
```

And the `application.scss` file would contain:

```css
@import 'folder';
```

**If using sprockets**, `application.scss` file would contain the rails way of importing using require. As such variables and other defines won't carry over to files, so they would have to be @imported manually as needed.

### Vendor and overrides

The vendor folder should have css or scss files that are gotten from outside sources. As these files have nothing to do with the application styling, they should be loaded first.

Overrides are files which directly override vendor classes. For example bootstrap's `modal` class would be overridden in the `bootstrap.scss` inside of override folder with class `modal` .

Any global overrides for the vendor files should go into the override folder, with the same name as the vendor file, so they can easily be found when necessary.

### Core and config

The config file should contain any configuration for the imported vendors (Usual example would be susy), or other configuration for user defined mixins, functions  etc.

The core file should contain and globally defined styles for the application. That could mean setting a global font-size, adding global box-sizing and etc. The core file should not contain any configuration or mixins, variables and such, only styling (or includes of such).

### Components

The components folder contains blocks that are shared between pages. Those could be form definitions, button definitions and such. Each components has to be in its own file, with the name of the file matching that of the block name.

### Pages

The pages folder contains an scss file for each page on a site. The styles inside should be specific to that page, while still following the bem philosophy. Any blocks that would be shared between pages fit into the components folder in a separate file, and not inside a specific page. The name of the file should match the page handle.

### SHAME

`_shame.scss` file is to be used only for hackity hack code that really shouldn't belong in any other file. Usage is not recommended, but is sometimes necessary. When writing a rule in the shame file **ALWAYS** put a comment above it saying specifically why it was written, and what problem it solved. Don't be afraid to write multiple lines explaining it.

When you're working on a project with `_shame.scss` always check the file when you have strange behavior, or if you're removing something from it, fix the underlying issue it was trying to solve.

### Utils

* colors
* z-indexes
* shared variables
* media
* placeholders
* mixins

### Colors

The colors file should only contain color variables of the base and global kind.
All colors for blocks or elements should go into their local file on the top.

Base colors are actual color values used on the site, they will be used by both global colors and block-elements colors. 

When naming color variables avoid using names that correspond to the actual colors, better use some fictive names like `silver`, `wood`, `mercury`, etc...

Syntax and example:

Syntax: `base-[value]-color`

```scss
$base-rouge-color: #FF0000;
```

Only in base colors is the name of the color ok to use.

Global colors are color to be used for block or element level colors. They should contain base colors.

Syntax: `[primary, secondary, ternary ...]-<property>-color`

```scss
$primary-color: $base-red-color;
$primary-text-color: $base-light-gray-color;
```

Block element colors go into their respective files on the top, and are defined as such:

Syntax: `[block-name][__element-name]-<property>-color`

```scss
$some-block-background-color: $base-red-color;
$some-block__some-element-color: $primary-text-color;
```

Text is the only property that can be omitted, as its property name in css is color as well.

### Z-indexes

When using z-indexes, use z-index variables defined in z-indexes file.

```scss
$z-base: inital;
$z-modal: 10;
$z-loader: 20;
$z-modal-raised: 30;
$z-notification: 40;
$z-debug: 1000;

//For use in local situations only (inside of a specific block or element)
$z-1: 1;
$z-2: 2;
$z-3: 3;
$z-4: 4;
$z-5: 5;
```

### Shared variables

This file should only contain variables that will be shared across scss files, and doesn't include colors, z-indexes, media query or similiar.
Example:

```scss
$global-header-height: 250px;
```

Use global prefix so that when it is used in other files, it is clear where the definition is.

### Media queries

Media file should contain the variables and mixins used to define the media queries and breakpoints on the site.

**Usage of the media [mixin](https://github.com/infinum/media-blender) is highly recommended as it is explicit and will standardize media queries.** The readme in the repository explains configuration and usage of the mixin.

Otherwise, syntax is as follows.

[SASS reference on mixins](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins)

```scss
$breakpoint-tablet: '(max-width: 991px)';

@mixin tablet() {
  @media #{$breakpoint-tablet} {
    @content;
  }
}
```

### Placeholders

The placeholders file should contain globally shared pseudoclasses (or placeholders). Pseudoclasses in sass are classes beginning with %, which generate no css
output, but can be extended using sass directive @extend.
Example usage of a global placeholder.

* [SASS reference on pseudoclasses](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_)
* [SASS reference on extend](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend)
* [SASS reference on usage of pseudoclasses](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholders)

```scss
%container {
  @include container;
  width: $content-width-large;

  @include desktop {
    width: $content-width-desktop;
  }
  @include tablet {
    width: $content-width-tablet;
  }
  @include mobile {
    width: $content-width-mobile;
  }
}

//Or with media mixin
%container {
  @include container;
  width: $content-width-large;

  @include media(desktop) {
    width: $content-width-desktop;
  }
  @include media(tablet) {
    width: $content-width-tablet;
  }
  @include media(mobile){
    width: $content-width-mobile;
  }
}

````

### Mixins

The mixins file should contain any global mixin that can help organize sass better. If your project contains a lot of mixins it is reccomended that you put them in a separate files and groupe according to their purpose. Then put all those filed inside a new folder called **mixins**.
For instance adding clearfix.

```scss
@mixin clearfix() {
  &::after {
    content: ' ';
    display: table;
    clear: both;
  }
}
```

Example of structure:
* **utils**
  * **mixins**
    * \_buttons.scss
    * \_containers.scss
    * \_forms.scss
    * \_general.scss
    * \_sections.scss
    * \_sypography.scss



### Fonts

The fonts file should contain any and all font family declarations.

If the font has multiple weights, always use the same font family name, with different font-weight values. **Font weight values should always use named weights**. Same goes for using fonts in other scss code. Exceptions can be made if the font has more than 3 weights, or abnormal weight distribution.

The other part of the font file should contain font family pseudoclasses for using the fonts. The pseudoclasses should not contain any weight or size properties, those belong in the classes that use the font.

Name the font family pseudoclasses with primary, secondary and so on.
The primary should usually refer to the heading font, and the secondary to the standard (body text) font.

e.g.

```css
@font-face {
  font-family: 'some-font';
  src: url('some-font-regular.eot');
  src: url('some-font-regular.eot?#iefix') format('embedded-opentype'),
       url('some-font-regular.woff2') format('woff2'),
       url('some-font-regular.woff') format('woff'),
       url('some-font-regular.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

@font-face {
  font-family: 'some-font';
  src: url('some-font-bold.eot');
  src: url('some-font-bold.eot?#iefix')
  format('embedded-opentype'),
       url('some-font-bold.woff2') format('woff2'),
       url('some-font-bold.woff') format('woff'),
       url('some-font-bold.ttf') format('truetype');
  font-weight: bold;
  font-style: normal;
}

//primary font should refer to the heading fonts if such exist
%primary-font {
  font-family: 'some-font';
}

//secondary font is the body fonts
%secondary-font {
  font-family: 'some-other-font';
}

```
If you project uses postCss there are libs that will make all this font-face declarations for you.

### Animations

Animations file should contain any keyframes definitions in the project, as well as the mixins needed for using said keyframe. Avoid animating anything but transforms and opacity, as these are the least expensive operations.

Animations should be prefixed with the keyframe prefix, while mixins using it should be prefixed with animation.

```css
@keyframes keyframe-move-down {
  0% {
    transform: translate(0, 0);
  }

  100% {
    transform: translate(0, 100%);
  }
}

@mixin animation-move-down() {
  animation-name: keyframe-move-down;
  animation-duration: 3s;
}
```

### SASS style guide

For a reference of sass functionality refer to this page: [SASS reference](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)

*This should cover almost every case when writing sass, including file structures, imports, code organization and logic*

We use scss-lint and csscomb as our linter, and codestyle checker. Dotfiles for these can be found on the [Useful links guide](/books/frontend/Useful-links)
