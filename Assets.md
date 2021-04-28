## SVGs

### 1. Export

When we export an icon from Figma or Zeplin, we should export the icon container instead of the actual icon. _Check the image_
![image](https://user-images.githubusercontent.com/55184443/115124509-edfa2e80-9fc2-11eb-8386-fa29ad4c0548.png)
In this case, we will export `Menu` as SVG instead of the `Icon` element.

> With this, every icon we export will have the exact box dimensions, which is easier to maintain in the long run.

### 2. Optimization

Before adding a new icon to our codebase, we should optimize it. [`svgo`](https://github.com/svg/svgo) is a go-to tool for this purpose, with [SVGOMG](https://jakearchibald.github.io/svgomg/) as its hosted UI. [Here are the preferred settings](https://gist.github.com/kristian240/bf7be2570e7cc8074718484130d5ae2e) that should give the optimal output.

### 3. Add to codebase

After we exported and optimized the icon, we can add it to our codebase. We will add this new icon in one of two directories:

#### a. `src/assets/icons`

We should add an icon to this folder if it uses only one color and will be used in components like buttons and dropdowns. The `fill` property of the icon source should be changed to `currentColor`. This way its color can be changed by setting the `color: ${someColor}` style in CSS.

Example in code:

```jsx
import AddIcon from 'src/assets/icons/icon-add.svg';

const AddButton = () => (
  <Button>
    <Icon as={AddIcon} color='primary' />
  </Button>
);
```

> Make sure that the SVG element has only the `viewBox` property (without `width` and `height`). The context surrounding the icon should define its dimensions.

#### b. `public/images`

If the image is using complex colors gradients/multiple colors/... it probably belongs to this category.

> Before adding it, you can check if this image could be smaller in size using `png` or `jpg` image format. If this is the case, use that one instead of `SVG`. If you opt in for this, be careful if this icon will be used in large formats, `png` and `jpg` could strech and result as a pixelated image.

Example in code:

```jsx
const InstagramLink = () => (
  <Link>
    <Image src='/images/instagram-icon.svg' />
    Link to Instagram page
  </Link>
);
```
