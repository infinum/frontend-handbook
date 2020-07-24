# Best Practices

## How to separate styles and avoid big unmaintainable component

BAD:
```
└─ components
    └─ Header
        └─ index.tsx
```

```tsx
const Header = ({ progress }) => {
  return (
    <div
      css={{
        backgroundColor: 'white',
      }}
    >
      <div
        css={{
          maxWidth: '1024px',
          display: 'flex',
        }}
      >
        <div>
          LOGO
        </div>
        <div>
          <a href="#">Home</a>
          <a href="#">Gallery</a>
          <a href="#">About us</a>
        </div>
      </div>
    </div>
  )
}
```

GOOD:
```
└─ components
    └─ Header
        └─ components
        ⎮   ├─ Wrapper.tsx
        ⎮   ├─ Container.tsx
        ⎮   ├─ Logo.tsx
        ⎮   ├─ Navigation.tsx
        ⎮   └─ Link.tsxs
        └─ index.tsx
```

## When to use `style` prop instead of `css`?

BAD:
```tsx
const Progress = ({ progress }) => {
  return (
    <div
      css={{
        backgroundColor: 'red',
        height: '5px',
        width: `${progress}%`,
      }}
    />
  )
}
```

GOOD:
```tsx
const Progress = ({ progress }) => {
  return (
    <div
      css={{
        backgroundColor: 'red',
        height: '5px',
      }}
      style={{
        width: `${progress}%`,
      }}
    />
  )
}
```

TODO: explain why!


## Polymorphic components

### `as` prop

```tsx
import styled from '@emotion/styled'

const Button = styled.button`
  color: hotpink;
`

render(
  <Button
    as="a"
    href="https://github.com/emotion-js/emotion"
  >
    Emotion on GitHub
  </Button>
)
```
> You can find more details in [Emotion docs](https://emotion.sh/docs/styled#as-prop)

note the problem with TS
