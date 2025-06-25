# Tailwind CSS - Best Practices, FAQ, and Recipes

This chapter gathers the tips, conventions, and tooling setup that keep our Tailwind codebase predictable and approachable for everyone from interns to principal engineers.

## FAQ

### Can I concatenate strings to build class names?

No. As soon as you write something like `'bg-' + color + '-500'`  
Tailwind's compiler cannot see the literal class names during the template scan, so it purges every potential match. Your colour vanishes in production and you spend an afternoon chasing why the button is transparent, this is very similar case to [Panda CSS Dynamic Styling](https://panda-css.com/docs/guides/dynamic-styling). Always pass complete strings to the compiler. When classes are conditional, wrap them with the `cn` helper or a CVA variant so the literals remain visible.

### Can Tailwind coexist with libraries like Material UI?

Yes. Tailwind is just CSS classes; it does not add a runtime. Two integration patterns work well:

- Simply use the utility classes in MUI components
  ```tsx
  <Button className="bg-brand-600 shadow-md" />
  ```
- Use Tailwind for layout, MUI for logic - Tailwind controls flex grids, spacing, and colors, while the MUI theme handles stateful styles (disabled, focus)

Important: make sure `tailwind.css` is imported after any baseline or component library stylesheet so its utilities win specificity ties.

### How do I handle dark mode?

Use Tailwind's `dark:` variant together with CSS custom properties:

```css
@theme {
  --color-sidebar-border: var(--color-slate-200);
  --color-sidebar-background: var(--color-slate-100);
}

@layer theme {
  .dark {
    --color-sidebar-border: var(--color-slate-800);
    --color-sidebar-background: var(--color-slate-900);
  }
}
```

You can also do it like this:

```css
:root {
  --acme-canvas-color: oklch(0.967 0.003 264.542);
}

[data-theme="dark"] {
  --acme-canvas-color: oklch(0.21 0.034 264.665);
}
```

Then in JSX:

```tsx
className = "bg-brand-500 dark:bg-brand-500/80";
```

Switching the dark class on `<html>` node flips every token instantly without component re-renders.

Check the [Colors documentation](https://tailwindcss.com/docs/colors) to learn more.

### How do I handle responsive design with Tailwind?

Tailwind CSS follows a mobile-first approach using breakpoint prefixes. You write base styles without a prefix and override them using responsive modifiers:

```tsx
<div class="p-2 md:p-4 lg:p-6">
  <!-- Small screens: 0.5rem, Medium: 1rem, Large: 1.5rem padding -->
</div>
```

You can also add hover:, focus:, and more within breakpoints:

```tsx
<button class="text-sm md:hover:text-lg">Click me</button>
```

For a deeper dive, check out Tailwind's [Responsive Design guide](https://tailwindcss.com/docs/responsive-design) - it covers customizing breakpoints, stacking multiple modifiers (like md:hover:), and other advanced patterns for building complex, fluid layouts.

### What is the `@utility` directive and when should I reach for it?

`@utility` lets you bundle several raw CSS declarations under a single Tailwind-style class without leaving your stylesheet. It is perfect for verbose properties such as `clip-path` or complex shadows that would otherwise require an arbitrary value every time.

```css
@utility {
  .clip-blob {
    clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
  }
}
```

You can now reuse `clip-blob` alongside built-in utilities with complete purge safety and no JavaScript.

### How does the group utility help with state-based styling?

Instead of repeating the same prefix on multiple siblings (`hover:`, `focus:`), wrap them in a parent that carries `group`:

```tsx
<button className="group inline-flex items-center gap-2 p-3 rounded-lg bg-brand-600">
  <Icon className="size-4 text-white group-hover:translate-x-0.5 transition-transform" />
  <span className="text-white group-hover:underline">Download</span>
</button>
```

Any child can now react to `group-hover`, `group-focus`, `group-aria-expanded`, etc., keeping class strings short and readable.

### My hover and focus class lists are gigantic. Is there a cleaner pattern?

Use CVA or the `cn` helper to build a map of state variants instead of stacking prefixes manually:

```tsx
const card = cva("rounded-lg p-4 transition", {
  variants: {
    intent: {
      default: "",
      hover: "hover:bg-brand-50 hover:shadow",
      focus: "focus-visible:ring-2 focus-visible:ring-brand-600",
    },
  },
});
```

Apply the variant you need with `card({ intent: 'hover' })`. Literal strings remain visible for the compiler and you avoid prefix noise.

### How do I configure Material UI so Tailwind utilities take effect?

In MUI v5 you must enable CSS custom properties:

```tsx
import { createTheme } from "@mui/material/styles";

const theme = createTheme({
  cssVarPrefix: "mui", // any prefix is fine
  cssVarEnabled: true, // critical: ensures Tailwind variables cascade
});
```

With `cssVarEnabled` set to true, Tailwind classes applied via `className` or `sx` override MUIs default palette and spacing. Remember to load `tailwind.css` after the MUI baseline styles so utility specificity wins.

### When is it acceptable to sprinkle !important?

Only when a third-party library injects an inline style or a high-specificity rule that you cannot control. Scope the override to the smallest selector possible, document the reason, and keep a TODO to remove it once the upstream lib allows custom classes.

```tsx
<!-- Third-party widget injects an inline red background we can’t control -->
<div className="p-4 text-white !bg-blue-500">
  The “!” prefix turns bg-blue-500 into bg-blue-500 !important,
  overriding the inline style without raising overall specificity.
</div>

<!-- You can combine it with other modifiers too -->
<div className="bg-gray-100 md:!bg-blue-500">
  On small screens: gray background
  On ≥ md screens: blue background !important
</div>
```

### I keep typing align-center instead of items-center. How do I memorise Tailwind's naming?

Tailwind mirrors **justify-** for main-axis alignment and **items-** for cross-axis. A quick mnemonic: “Flex items cross the axis”. Also make sure to turn on IntelliSense - misspelled utilities show a red underline and the correct suggestion in the hover tooltip.

If you really can't remember ANY Tailwind naming, just keep the [Tailwind Cheat Sheet](https://nerdcave.com/tailwind-cheat-sheet) opened, or get yourself the [Raycast Addon](https://www.raycast.com/vimtor/tailwindcss).

### Why is Tailwind IntelliSense considered mandatory?

- Autocompletes tokens and arbitrary values
- Flags mistyped utilities immediately
- Previews final CSS for any class
- Understands our `cn` and `cva` patterns through the custom regex in `.vscode/settings.json`

The plugin removes 90 percent of tab-out-and-Google moments, making new hires productive within hours, not days.

### Initial Tailwind setup feels slow. How can I streamline it?

Use the project starter that already includes:

- Tailwind v4 configured with `@theme` tokens
- Prettier and ESLint plugins pre-wired
- `cn` helper, CVA wrapper, and example primitives
- VS Code workspace settings for IntelliSense

Starting from template means you write your first utility within five minutes, bypassing the boilerplate phase entirely.

## Code Recipes

### The `cn` Helper

Create `lib/cn.ts` once per repo:

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Always call `cn` instead of template-string concatenation:

```tsx
<div
  className={cn(
    "grid grid-cols-1 gap-4",
    isSidebarOpen && "lg:grid-cols-[240px_1fr]"
  )}
/>
```

### Conditional Variants Without String Math

Bad pattern:

```tsx
className={`bg-${status}-50 text-${status}-700`}
```

Good pattern with CVA:

```tsx
const badge = cva(
    'inline-flex items-center rounded px-2 py-0.5 text-sm font-medium',
    {
    variants: {
        intent: {
        info:  'bg-sky-50  text-sky-700',
        warn:  'bg-yellow-50 text-yellow-700',
        error: 'bg-red-50 text-red-700'
        }
    }
    }
)

<span className={badge({ intent: status })}>{label}</span>
```

This preserves purge safety, keeps colors centralised, and eliminates template string bugs.

### Safelisting Dynamic Utilities

If you need to make sure Tailwind generates certain class names that don't exist in your content files, use [@source inline()](https://tailwindcss.com/docs/detecting-classes-in-source-files#safelisting-specific-utilities) to force them to be generated:

```css
@source inline("{bg-,text-,border-}{brand,info,warn,error}-{50,100,500,700}");
```

## Editor and Linting Setup

### Tailwind IntelliSense

1. Install VS Code extension `bradlc.vscode-tailwindcss`
2. Enable `.vscode/settings.json` so autocomplete works inside our `cn` helper

   ```json
   {
     /**
      * Tailwind CSS IntelliSense extensions settings
      * https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss
      */
     "files.associations": {
       "*.css": "tailwindcss" // Tells VS Code to always open *.css files in Tailwind CSS mode - this prevents you from using Stylelint in the project, but gives you autocomplete in @apply directives
     },
     "editor.quickSuggestions": {
       "strings": "on" // Enable quick suggestions inside strings
     },
     // Enables Tailwind CSS IntelliSense extension in "cva" and "cn" functions,
     // sorting is configured in Prettier settings
     "tailwindCSS.experimental.classRegex": [
       ["cva\\(((?:[^()]|\\([^()]*\\))*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
       ["cn\\(((?:[^()]|\\([^()]*\\))*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
     ]
   }
   ```

### ESLint Plugin for Tailwind

> ⚠️ Warning! As of June 2025, the Tailwind ESLint plugin is not compatible with Tailwind CSS v4. Follow [this issue](https://github.com/francoismassart/eslint-plugin-tailwindcss/issues/325) for more information.
>
> 📌 Note: The following example is for Tailwind CSS v3.

Add `eslint-plugin-tailwindcss`:

```bash
pnpm add -D -E eslint-plugin-tailwindcss
```

ESLint config snippet:

```js
plugins: ['tailwindcss']
extends: ['plugin:tailwindcss/recommended']
rules: {
'tailwindcss/classnames-order': 'error',
'tailwindcss/enforces-shorthand': 'warn'
}
```

This catches missing classes, wrong order, and unknown utilities.

### Prettier Plugin for Tailwind Sort

Prettier will group and sort class strings. Install `prettier-plugin-tailwindcss`:

```bash
pnpm add -D -E prettier prettier-plugin-tailwindcss
```

Prettier config file:

```js
plugins: ['prettier-plugin-tailwindcss'], // Enables Tailwind classes sorting
tailwindFunctions: ['cva', 'cn'], // Enables sorting of Tailwind classes in "cva" and "cn" functions, IntelliSense is configured in ".vscode/settings.json"
```

## Common Pitfalls and How to Dodge Them

- **Arbitrary values explosion** - every unique `shadow-[...]` makes a rule. Centralise rare values with `@apply` inside a utility class
- **Missing plugin utilities** - classes like `animate-in` need `tailwindcss-animate`
