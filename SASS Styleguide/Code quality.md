**Don't ever use tag selectors if it can be avoided**. They are usually much slower than any class selector, and almost always have to be
overridden

Use of scss-lint is very recommended to force nesting, specificity, rule order and other rules in the following chapter.

[Scss-lint file](https://github.com/infinum/dotfiles/blob/master/code-linters/scss-lint.yml)

* [Nesting](#nesting)
* [Specificity](#specificity)
* [Rule order](#ruleOrder)
* [File length](#fileLength)

<a name="fileLength"></a>
### File length

An SCSS file should never be more than 200 lines long. Longer files are harder to read and find selectors and rules in. Optimal length would be around 100
lines per file. Very complicated blocks with more than 50 lines should also go to their own separate file, best under a folder with the name of the page it's on,
if it's not a component.

<a name="nesting"></a>
### Nesting

Nesting improves code readability, but must be taken into moderation. The rule is to nest no more than 3 times, with few exceptions.
Usual example of three level nesting is seen in a simple block element with a modifier.

```scss
.block {
  //level 1 nesting

  .block__element {
    //level 2 nesting

    &.block__element--modifier {
      //level 3 nesting
    }
  }
}
```

<a name="specificity"></a>
### Specificity

Specificity is a bit more complex than nesting. With specificity you have to take into account the generated css output. Each level of nesting
adds a level of specificity to a selector. Adding multiple selectors on a single line also adds to specificity, such as:

```scss
  //level 1 specificity
  .button {
    //properties...
  }

  //level 2 specificity
  .button.button-white {
    //properties...
  }
  //or
  .button {
    .button-white {
      //properties...
    }
  }

  //level 3 specificity
  .button.button-white.button-really-white {
    //properties...
  }
  //or
  .button {
    &.button-white {
      .button-really-white {
        //properties...
      }
    }
  }
```

**Specificity should be 4 levels maximum.**

<a name="ruleOrder"></a>
### Rule order

Rule order inside a selector are as follows:

* extends
* includes without @content
* properties
* nested properties
* includes with @content
* pseduoclasses (e.g. :hover)
* pseudoelements (e.g. ::after)
* parent selector modifiers (e.g. &.is-active)
* children element selectors

This will be enforced by the scss-lint.
