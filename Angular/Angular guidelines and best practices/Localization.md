This chapter covers various best practices for localization and handling translations in your Angular application.

## Localization library

Currently, the preferred library of choice for handling localization in Angular is [Transloco](https://jsverse.github.io/transloco/).
`Transloco` allows you to define translations for your content in different languages and switch between them easily at runtime. It exposes a rich API to manage translations efficiently and cleanly. It provides multiple plugins that will improve your development experience. The setup is quite easy, and you can find it in the documentation, so we won't discuss it here.

## Usage

### transloco pipe

The easiest way to handle translation is through a pipe. To use it, simply import `TranslocoModule` in your component and pass the translation key to the pipe in your component template.

```html
<p>{{'common.exampleKey' | transloco}}</p>
```

### translate function

In some cases, we need to translate the keys in the component itself rather than in the template. Inject the service into your component and then use the translate function by passing in the key you wish to translate.

```ts
private translocoService = inject(TranslocoService)
private exampleTranslatedKey = this.translocoService.translate('common.exampleKey')
```

Note: Besides the ability to translate keys, the `TranslocoService` also provides many more functionalities, such as changing the selected language.

## Pluralization support

Besides just translating a single key, sometimes we need to handle singular and plural versions of a single word. While it might be simple for some languages like English, it can be quite complicated for languages like Croatian, German, etc. `Transloco` does not support this out of the box, but the [@jsverse/transloco-messageformat](https://jsverse.github.io/transloco/docs/plugins/message-format) plugin allows you to handle pluralization with ease.

## Installation

To enable the plugin, pass `provideTranslocoMessageformat()` to the `providers` array of either `appConfigFactory` or `TranslocoRootModule` if using modules.

## Language plural rules

To properly translate singular and plural forms, we need to first check the plural rules for the language we are writing our translation keys. The standard table with the rules can be found [here](https://www.unicode.org/cldr/charts/45/supplemental/language_plural_rules.html). Essentially, you need to check the needed categories for a specific language and create the translation keys accordingly. The following examples will be done for the English language, and they represent a small set of available features that will cover almost all cases. For additional functionalities, refer to the official plugin documentation.

### Usage

The categories for English are `one` for singular, and `other` for any value higher than one. To define the pluralization key, we must provide the following:

* A value variable, which is `minutes` in the following example.
* Whether the translations are plural or ordinal.
* A translation for each of the required categories. If any category is missing, the `other` category is used as a fallback.

### Plural

```json
{
 "minutes": "{minutes, plural, one {minute} other {minutes}}"
}
```

```html
<p>{{ "minutes" | transloco : { minutes: 1 } }}</p>
// minute


<p>{{ "minutes" | transloco : { minutes: 5 } }}</p>
// minutes
```

### Ordinal

For ordinals, we need to set the type to `selectordinal`

```json
{
 "minutesOrdinal": "{minutes, selectordinal, one {st} two {nd} few {rd} other {th}}",
}
```

```html
<p>1{{ "minutes" | transloco : { minutes: 1 } }}</p>
// 1st


<p>2{{ "minutes" | transloco : { minutes: 2 } }}</p>
// 2nd


<p>3{{ "minutes" | transloco : { minutes: 3 } }}</p>
// 3rd


<p>4{{ "minutes" | transloco : { minutes: 4 } }}</p>
// 4th


<p>21{{ "minutes" | transloco : { minutes: 21 } }}</p>
// 21st

```

### Interpolation

To reuse the value passed to the pipe to determine the translation used, we can use the `#` character, which will be substituted with the passed-in value at runtime.

```json
{
 "minutesOrdinal": "{minutes, selectordinal, one {#st} two {#nd} few {#rd} other {#th}}",
}
```

```html
<p>{{ "minutes" | transloco : { minutes: 1 } }}</p>
// 1st
```

### Interpolation in a sentence

In some cases, we are not just translating a single word, but rather a longer string, and only a part of it needs to be dynamically interpolated.

```json
{
"applesInBasket": "I have {apples, plural, one {# apple} other {# apples}} in the basket.",
}
```

```html
<p>{{ "applesInBasket" | transloco : { apples: 1 } }}</p>
// I have 1 apple in the basket.

<p>{{ "applesInBasket" | transloco : { apples: 5 } }}</p>
// I have 5 apples in the basket.
```

### Explicit handling of certain values

Some values require special translation. The plugin allows us to explicitly specify a custom translation for one or more of those values if needed. In the following example, we explicitly handle zero.

```json
{
"applesExactCount": "I {apples, plural, =0 {don't have apples} one {have # apple} other {have # apples}} in the basket.",
}
```

```html
<p>{{ "applesExactCount" | transloco : { apples: 0 } }}</p>
// I don't have apples in the basket.

<p>{{ "applesExactCount" | transloco : { apples: 1 } }}</p>
// I have 1 apple in the basket.

<p>{{ "applesExactCount" | transloco : { apples: 5 } }}</p>
// I have 5 apples in the basket.
```
