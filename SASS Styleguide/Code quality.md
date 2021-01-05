**Don't ever use tag selectors if they can be avoided**. They are usually much slower than any class selector, and almost always have to be overridden.

It is strongly recommended to use scss-lint to force nesting, specificity, rule order, and other rules in the following chapter.

[Scss-lint file](https://github.com/infinum/dotfiles/blob/master/code-linters/scss-lint.yml)

* [Nesting](#nesting)
* [Specificity](#specificity)
* [Rule order](#ruleOrder)
* [File length](#fileLength)

<a name="fileLength"></a>
### File length

An SCSS file should never be more than 200 lines long. Longer files are harder to read and find selectors and rules in. Optimal length would be around 100
lines per file. Very complicated blocks with more than 50 lines should go in their own separate file. It would be best to put it under a folder with the name of the page it's on,
if it's not a component.

<a name="nesting"></a>
### Nesting

Nesting improves code readability, but it must be used in moderation. The rule is to nest no more than three times, with few exceptions.
The usual example of three-level nesting is found in a simple block element with a modifier.

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

Specificity is a bit more complex than nesting. With specificity, you have to take into account the generated CSS output. Each level of nesting adds a level of specificity to a selector. Adding multiple selectors on a single line also adds to specificity. For example:

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

**There should be no more than four levels of specificity.**

<a name="ruleOrder"></a>
### Rule order

The rule order inside a selector is as follows:

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
