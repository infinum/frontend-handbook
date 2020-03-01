# Common SSR errors

## Avoid usage of `:first-child` CSS selector with Emotion 10

If `:first-child` CSS selector is used in server-side rendered applications, React will show this warning in the console:
```
The pseudo class ":first-child" is potentially unsafe when doing server-side rendering. Try changing it to ":first-of-type"
```

Default server-side rendering in Emotion 10 renders the `<style>` tag inline with the component, instead of extracting everything inside `<head>`, similarly to "shadow CSS" in Web Components.
This approach enables streaming and requires no additional configuration, but does not work with nth child or similar selectors.

Here is an example of the `:first-child` selector:
```jsx
import styled from "@emotion/styled";

const Text = styled.p`
  color: gray;
  &:first-child {
    color: black;
  }
`;

export default () => (
  <div>
    <Text>Tilte</Text>
    <Text>Subtitle</Text>
  </div>
);
```

This is the DOM structure generated on the server side:
```html
<div>
  <style data-emotion-css="1fyxi0m">
    .css-1fyxi0m {
        color: gray;
    }

    .css-1fyxi0m:first-child {
        color: black;
    }
  </style>
  <p class="css-1fyxi0m">Tilte</p>
  <style data-emotion-css="1fyxi0m">
    .css-1fyxi0m {
        color: gray;
    }

    .css-1fyxi0m:first-child {
        color: black;
    }
  </style>
  <p class="css-1fyxi0m">Subtitle</p>
</div>
```
As the first child is `<style>` instead of `<p>` tag, `:first-child` selector won't work.

### Solution
Use `:first-of-type`, `:last-of-type` or `:nth-of-type` selectors.

> Read more about alternative SSR setup in the official documentation https://emotion.sh/docs/ssr
