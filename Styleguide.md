Based on the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript).

## [References](#references)

Avoid using `var`. If you're working with values that don't change, use `const`. In other cases, use `let`.

Note that both `let` and `const` are block-scoped.

```js
// const and let only exist in the blocks they are defined in.
{
  let a = 1;
  const b = 1;
}
console.log(a); // ReferenceError
console.log(b); // ReferenceError
```

Also, keep [temporal dead zones](http://jsrocks.org/2015/01/temporal-dead-zone-tdz-demystified/) in mind.

```js
// const and let don't exist before they're defined.
{
  console.log(a); // ReferenceError
  console.log(b); // ReferenceError
  let a = 1;
  const b = 1;
}
```

## [Objects](#objects)

Use computed property names when creating objects with dynamic property names.

```js
function getKey(k) {
  return `a key named ${k}`;
}

// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```

Use object method shorthand.

```js
// bad
const atom = {
  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  value: 1,

  addValue(value) {
    return atom.value + value;
  },
};
```

Use property value shorthand. Group your shorthand properties at the beginning of your object declaration.

```js
const anakinSkywalker = 'Anakin Skywalker';
const lukeSkywalker = 'Luke Skywalker';

// bad
const obj = {
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  lukeSkywalker: lukeSkywalker,
  episodeThree: 3,
  mayTheFourth: 4,
  anakinSkywalker: anakinSkywalker,
};

// good
const obj = {
  lukeSkywalker,
  anakinSkywalker,
  episodeOne: 1,
  twoJediWalkIntoACantina: 2,
  episodeThree: 3,
  mayTheFourth: 4,
};
```

Quote only properties that are invalid identifiers.

## [Arrays](#arrays)

Use array spreads `...` to copy arrays.

```js
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

To convert an array-like object to an array, use Array#from.

```js
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```

## [Destructuring](#destructuring)

Use object destructuring when accessing and using multiple properties of an object.

```js
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;

  return `${firstName} ${lastName}`;
}

// good
function getFullName(user) {
  const { firstName, lastName } = user;
  return `${firstName} ${lastName}`;
}

// best
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}
```

Use array destructuring.

```js
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

Use object destructuring for multiple return values, not array destructuring.

```js
// bad
function processInput(input) {
  // process the input and return necessary properties
  return [left, right, top, bottom];
}

// the caller needs to think about the order of return data
const [left, __, top] = processInput(input);

// good
function processInput(input) {
  // process the input and return necessary properties
  return { left, right, top, bottom };
}

// the caller selects only the data they need
const { left, right } = processInput(input);
```

When programmatically building up strings, use template strings instead of concatenation.

```js
// bad
function sayHi(name) {
  return 'How are you, ' + name + '?';
}

// bad
function sayHi(name) {
  return ['How are you, ', name, '?'].join();
}

// good
function sayHi(name) {
  return `How are you, ${name}?`;
}
```

## [Functions](#functions)

Never use `arguments`; opt to use rest syntax `...` instead.

```js
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

Use default parameter syntax rather than mutating function arguments.

```js
// really bad
function handleThings(opts) {
  // No! We shouldn't mutate function arguments.
  // Double bad: if opts is falsy, it'll be set to an object which may
  // be what you want but it can introduce subtle bugs.
  opts = opts || {};
  // ...
}

// still bad
function handleThings(opts) {
  if (opts === void 0) {
    opts = {};
  }
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}

Always put the default parameters last.

// bad
function handleThings(opts = {}, name) {
  // ...
}

// good
function handleThings(name, opts = {}) {
  // ...
}
```

## [Arrow functions](#arrow-functions)

Use arrow functions for anonymous function expressions.

```js
// bad
[1, 2, 3].map(function (x) {
  const y = x + 1;
  return x * y;
});

// good
[1, 2, 3].map((x) => {
  const y = x + 1;
  return x * y;
});
```

If the function body consists of a single expression, omit the braces and use implicit return.

```js
// good
[1, 2, 3].map((number) => `A string containing the ${number}.`);

// bad
[1, 2, 3].map((number) => {
  return `A string containing the ${number}.`;
});

// good
[1, 2, 3].map((number) => {
  const nextNumber = number + 1;
  return `A string containing the ${nextNumber}.`;
});
```

Ensure clarity between arrow functions and comparison operators.

```js
// bad
const itemHeight = (item) =>
  item.height > 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) =>
  item.height > 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = (item) => {
  return item.height > 256 ? item.largeSize : item.smallSize;
};
```

## [Modules](#modules)

Always use modules (`import`/`export`) over a non-standard module system. You can always transpile to your preferred module system.

```js
// bad
const AirbnbStyleGuide = require('./AirbnbStyleGuide');
module.exports = AirbnbStyleGuide.es6;

// ok
import AirbnbStyleGuide from './AirbnbStyleGuide';
export default AirbnbStyleGuide.es6;

// best
import { es6 } from './AirbnbStyleGuide';
export default es6;
```

## [Variables](#variables)

Group all your const declarations together and then group all your let declarations together for better readability.

```js
// bad
let index,
  total,
  projectName,
  data = fetchData(),
  isActive = true;

// bad
let index;
const data = fetchData();
let projectName;
const isActive = true;
let total;

// good
const isActive = true;
const data = fetchData();
let projectName;
let index;
let total;
```

## [Trailing commas](#commas)

Using an additional trailing comma in objects, arrays, and function parameters can lead to cleaner git diffs and improve code maintenance. Here's why:

1. Cleaner Git Diffs: Adding a trailing comma minimizes the number of lines changed when new elements are added, making it easier to review code changes.
2. Consistency: Ensures a consistent style in your codebase, which can improve readability.
3. Transpiler Support: Transpilers like Babel will remove trailing commas in the transpiled code, ensuring compatibility with older browsers.

**Example: Objects**

```js
// Bad - without trailing comma
const hero = {
  firstName: 'Florence',
  lastName: 'Nightingale'
};

// Good - with trailing comma
const hero = {
  firstName: 'Florence',
  lastName: 'Nightingale',
};
```

**Example: Arrays**

```js
// bad
const heroes = [
  'Batman',
  'Superman'
];

// good
const heroes = [
  'Batman',
  'Superman',
];
```

**Example: Function Parameters**

```js
// bad
function createHero(
  firstName,
  lastName,
  isHero
) {
  // ...
}

// good
function createHero(
  firstName,
  lastName,
  isHero,
) {
  // ...
}
```

## [Naming conventions](#naming-conventions)

Naming functions is a critical and often challenging aspect of programming. Clear and descriptive function names improve readability and maintainability of the code. Here are some best Practices for naming functions:

### Naming functions

**Descriptive and Specific:**
Function names should clearly describe what the function does. Use verbs to name functions that perform actions. Avoid vague names.

```js
// bad
function process() {
  // ...
}

// good
function calculateTotalPrice() {
  // ...
}
```

**Avoid Abbreviations:**
Use full words to avoid confusion.

```js
// bad
function calcTtl() {
  // ...
}

// good
function calculateTotal() {
  // ...
}
```

### Naming variables

**Use Clear and Descriptive Names:**

```js
// bad
let x = 10;
let y = 20;

// good
let width = 10;
let height = 20;
```

**Use Meaningful Context:**

Include context to avoid ambiguity.

```js
// bad
let temp = 98;

// good
let bodyTemperature = 98;
```

**Combining Best Practices**

```js
// bad
function calc() {
  let w = 10;
  let h = 20;
  return w * h;
}

// good
function calculateArea() {
  const width = 10;
  const height = 20;
  return width * height;
}
```
