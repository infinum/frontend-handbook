# Vue

* [Official guide](https://vuejs.org/v2/guide/)
* [Official documentation](https://vuejs.org/v2/api/)

## Templating
For writing templates in Vue.js [directives](https://vuejs.org/v2/api/#Directives) are used.
* Usage of shorthands for directives is encuraged (`@` instead of `v-on`, `:` instead of `v-bind`, ...)
* Each and every attribute on an element should be written in a separate line
* The order of attributes is folllowing:
  1. v-* attributes (v-if, v-for, ...)
  2. non-binding attributes (type, name, ...) (attributes which values are not binded to a variable)
  3. :* attributes (v-bind)
  4. @ attributes (v-on)

```html
<template>
  <div>
    <input
      v-if="isInputShown"
      type="text"
      :name="inputName"
      :value="inputValue"
      @blur="onInputBlur"
    />

    <button
      :class="$style.submitButton"
      @click="onButtonClick"
    >
      Submit
    </button>
  </div>
</template>
```

## Components

When creating components all component info (template, logic and styling) is placed in one file.
Order of those blocks is:
1. `<template />`
2. `<script />`
3. `<style />`

Write small components so that a file is no longer than 300 LoC.
