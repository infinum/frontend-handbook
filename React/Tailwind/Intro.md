# Introduction to Tailwind CSS

Tailwind CSS is a **utility-first styling framework**. Instead of shipping pre-styled components or encouraging semantic class names, Tailwind exposes thousands of small, composable classes, each mapping one-to-one with a single CSS declaration:

- `pt-4` → `padding-top: 1rem`
- `flex` → `display: flex`
- `bg-blue-600/90` → `background-color: rgba(37, 99, 235, 0.9)`

You place these classes directly in your JSX. When the build runs Tailwind's compiler, now called **Oxide** (rewritten in Rust for v4), scans every template, generates only the CSS you referenced, minifies it, and removes everything else. The output is a single static stylesheet with zero runtime JavaScript.

## A Three-Second Mental Model

1. **Everything is opt-in**  
   If you never write `rounded`, nothing gets rounded. No default design decisions are imposed.

2. **Markup is the source of truth**  
   Delete a component in React and its styles vanish automatically. There is no orphan stylesheet to clean up.

3. **Classes read like English**  
   `flex flex-col gap-6 lg:flex-row items-center bg-surface p-8` tells you the `layout, spacing, breakpoint variation, alignment, colour and padding` in one glance.

4. **Design tokens are code**  
   Colours, spacing steps, radii and shadows live in `tailwind.config.js` (or in v4 inside an `@theme` block). Every token becomes a real CSS custom property, readable by JavaScript, Framer Motion or even a Figma plugin that scrapes the DOM.

## How Tailwind Works Under the Hood

### The JIT Compilation Cycle

1. **Template scan** - Oxide reads your `.tsx`, `.mdx`, `.html` and any other file types you list. It tokenises class attributes, including the more exotic arbitrary value syntax like `max-w-[42ch]`.
2. **Class lookup** - Each token maps to a rule in Tailwind's internal AST. Tailwind also supports user-defined classes added through the `theme.extend` or `@apply` directives.
3. **CSS emission** - Only referenced rules are emitted. Tailwind groups selectors that share the same declaration to keep file size minimal.
4. **Minification and purging** - Duplicate declarations are collapsed, comments are stripped and the file is saved alongside the rest of your assets. Because unused CSS is never written in the first place, the purge step is effectively free.
5. **Incremental rebuilds** - Oxide tracks a dependency graph. Editing a single component triggers micro-rebuilds that typically complete in under 50 ms even for large mono-repos.

### Configuration in v4

```css
@import "tailwindcss";

@theme {
  --radius-lg: 0.75rem;
  --brand-500: 34 197 94; /* R G B */
}
```

No separate JavaScript config file is required, and those custom properties are accessible at runtime:

```ts
const brand = getComputedStyle(document.documentElement).getPropertyValue(
  "--brand-500"
);
```

## Comparing Tailwind to other styling solutions

Below comparison is focused on developer experience, performance and how well each option fits Next.js, React Server Components (RSC) and streaming.

### Global CSS and SCSS

- **What it is** - One or a handful of style sheets imported at `_app.tsx` or the root layout.
- **Impact** - Fast builds, no runtime overhead, but the global cascade introduces accidental regressions.
- **Why Tailwind wins** - Utilities are local and predictable, eliminating the “who overrode my h1?” problem.

### CSS/SCSS Modules

- **What it is** - Next.js transpiles `.module.css` or `.module.scss` files into unique class names.
- **Impact** - Small bundle, no runtime penalty, but lots of files and boilerplate selectors.
- **Why Tailwind wins** - You rarely leave the JSX file and unused CSS is stripped automatically.

### Styled JSX (Next.js default)

- **What it is** - Inline styles scoped to a component, compiled to CSS at build time, injected with `<style>` tags.
- **Impact** - Good isolation, but every component ships extra JS to inject its styles.
- **Why Tailwind wins** - No runtime injection means faster hydration and less JavaScript over the wire.

### Emotion / Styled-Components

- **What it is** - Tagged template literals that generate styles at runtime (or via Babel for partial extraction).
- **Impact** - Powerful conditional styling but increases bundle size and complicates the RSC boundary.
- **Why Tailwind wins** - Styles are static assets processed at build, so nothing crosses the server-client line.

### Stitches, Panda, Vanilla-Extract

- **What they are** - Compile-time CSS-in-JS libraries that aim for zero runtime.
- **Impact** - Predictable output, but you learn a custom API and maintain separate style files.
- **Why Tailwind wins** - Utilities use the universal language of class names; JSX shows the final UI instantly.

### UI Component Libraries (Chakra UI, Material UI)

- **What they are** - High-level accessible React components with theming and style props.
- **Impact** - Great productivity for simple apps, but extra JavaScript and difficult deep customisation.
- **Why Tailwind wins** - Tailwind plus **shadcn/ui** gives copy-paste recipes with zero runtime cost.

## Pros and Cons of Tailwind CSS

### Strengths

- **Small static CSS** - Multi-tenant SaaS apps often ship under 30 KB gzipped.
- **Fast refactors** - Styles live and die with their JSX.
- **Token driven** - Edit a brand color in one file and every component updates.
- **Performance** - No style tag insertion or flash-of-unstyled-content.
- **Ecosystem** - Plugins for forms, typography, aspect-ratio and many Tailwind-native component kits.

### Trade-offs

- **Class verbosity** - Long class strings need Prettier and ESLint rules.
- **Learning curve** - Utility syntax feels alien for a week or two.
- **Arbitrary values** - `ml-[17px]` undermines consistency if unpoliced.
- **No baked components** - You must build or adopt primitives like **shadcn/ui**.
- **Potential reset conflicts** - Tailwind should load after any CSS reset.

## Why We Replaced Chakra UI with Tailwind

| Concern         | Chakra UI                               | Tailwind CSS                      |
| --------------- | --------------------------------------- | --------------------------------- |
| Bundle size     | 40-80 KB CSS, 25-60 KB JS               | < 30 KB CSS, negligible JS        |
| Runtime cost    | Emotion creates class names at runtime  | None                              |
| Theming         | Tied to Chakra's token names and scales | Plain CSS variables, any naming   |
| Custom variants | `extendTheme` JS code                   | Combine utilities or use `@apply` |
| Server comps    | Experimental hydration path             | Works out of the box              |
| Refactors       | Prop explosion (`_hover={{ bg: ... }}`) | Pure HTML classes, quick edits    |

### Migration Journey - Recommended Workflow

1. **Token mapping** - export the current Chakra (or any other) theme to JSON (use `npx @chakra-ui/cli tokens`). Copy colours, spacing, radii and font scales into Tailwind's `@theme` block. Preserve token names one-for-one so existing design language stays intact.
2. **Parallel Tailwind setup** - [Install Tailwind CSS with Next.js](https://tailwindcss.com/docs/installation/framework-guides/nextjs)
3. **Component wrappers** - introduce thin legacy shims such as `<LegacyButton>` or `<LegacyBox>`. Each shim keeps the Chakra prop API but renders plain HTML with Tailwind utilities. This lets you migrate page-by-page without a disruptive big-bang refactor.
4. **Incremental replacement** - prioritize the highest-traffic or most visually shared primitives first (Button, Input, Modal). For each component:

   - replace Chakra props with Tailwind class strings
   - snapshot the result in Storybook or Playwright
   - verify accessibility states, focus rings and responsive behaviour

5. **Design-system parity check** - run a UI/UX audit once every Chakra component has a Tailwind counterpart. Compare variant, state and breakpoint coverage; update tokens or utilities until every design spec passes.

6. **Dependency removal** - remove `<ChakraProvider>` and `CSSReset`, delete Chakra from `package.json`, and run grep to locate any lingering `_hover` or `variant` props.

7. **Ship and measure** - deploy to staging and measure bundle size and Core Web Vitals against the previous build. Teams consistently see
   - **JS bundle shrink** ≈ 15-25 %
   - **First Contentful Paint** improve by 200-300 ms on a 3G Fast profile
   - **Developer velocity** increase thanks to clearer, inline class semantics

## Common Concerns and How We Answer Them

| **Question**                         | **Answer**                                                                                                  |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Isn't Tailwind hard to read?         | After a short ramp-up, seeing colour, spacing and typography inline is faster than jumping to a CSS file    |
| Will utility classes bloat the HTML? | Gzip/Brotli compress repeating prefixes well; file size impact is minimal                                   |
| Can designers work with Tailwind?    | Tokens live in one config file and are exported to Figma via the Tailwind Tokens plugin                     |
| What about dark mode or theming?     | Tailwind's `dark:` variant and CSS variables enable instant theme switching without re-rendering components |

## Further Reading

- Documentation
  - [Tailwind CSS documentation](https://tailwindcss.com/docs)
- Articles
  - [It is Possible to Mix Chakra UI with Tailwind CSS ?](https://www.geeksforgeeks.org/it-is-possible-to-mix-chakra-ui-with-tailwind-css/)
