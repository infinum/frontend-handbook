## SVGs

### 1. Export

When we export an icon from the Figma or Zeplin, we should export the icon container instead of the actual icon. _Check the image_
![image](https://user-images.githubusercontent.com/55184443/115124509-edfa2e80-9fc2-11eb-8386-fa29ad4c0548.png)
In this case, we will export `Menu` as SVG instead of the `Icon` element.

> With this, every icon we export will have the exact box dimensions, which is easier to maintain in the long run.

### 2. Optimization

Before adding a new icon to our codebase, we should optimize it using [SVGOMG](https://jakearchibald.github.io/svgomg/) tool. [Here are settings](https://gist.github.com/kristian240/bf7be2570e7cc8074718484130d5ae2e) I use for a while now, and at the time, when I was configuring my settings, this would give me the best output. Feel free to reuse them.

### 3. Add to codebase

After we exported and optimized the icon, we can add it to our codebase. We will add this new icon in one of two directories:

#### a. `src/assets/icons`

We should add an icon in this folder if the icon uses only one color and will be used as an icon in buttons/dropdowns/... such as `StarIcon`, `ChevronIcon`, etc. One last change needed before we add it to the codebase is to change the `fill` color to `currentColor`. This way, a developer can use any color with a simple `color: ${someColor}` style in CSS.

Example in code:

```jsx
import AddIcon from 'src/assets/icons/icon-add.svg';

const AddButton = () => (
  <Button>
    <Icon as={AddIcon} color='primary' />
  </Button>
);
```

> Make sure that SVG element has only the `viewBox` property (without `width` and `height`). Context, where the icon is used, should provide `width` and `height` to the icon.

#### b. `public/images`

If the image is using complex colors gradients/multiple colors/... it probably contains in this category.

> Before adding it, you can check if this image could be smaller in size using `png` or `jpg` image format. If this is the case, use that instead of `SVG`.

Example in code:

```jsx
const InstagramLink = () => (
  <Link>
    <Image src='/images/instagram-icon.svg' />
    Link to Instagram page
  </Link>
);
```
