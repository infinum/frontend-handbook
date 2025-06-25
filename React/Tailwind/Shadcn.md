# ShadCN UI

ShadCN UI (often written simply as **shadcn/ui**) is a set of open-source React component templates that sit on top of [**Radix UI** primitives](https://www.radix-ui.com/primitives) and **Tailwind CSS** utilities. It is not a full-blown design system like Material UI or an abstract component library like Chakra. Instead it gives you copy-and-paste blueprints for accessible UI parts that remain **100 percent yours** after generation. Think of it as a scaffold: it bootstraps predictable, well-tested markup and behaviour, then gets out of your way so you can style or refactor without fighting an API surface.

## What Is ShadCN UI?

ShadCN UI is a project maintained by [@shadcn](https://x.com/shadcn) that curates Radix primitives into higher-level patterns - buttons, dialogs, dropdowns, tables, tabs, and more. Each component is provided as plain `.tsx` code plus matching Tailwind classes. There is **no package to import at runtime**. You run a CLI script once, commit the generated files to your repo, and thereafter treat them like any other local component. Updates are opt-in: rerun the script with the `add` command, accept the diff in a pull request, customise as you wish.

Key attributes:

- **Headless but helpful** - logic, state and accessibility baked in; visual style left to Tailwind
- **Ownership** - generated files live in your `components` folder; rename, inline, split - nothing breaks
- **Type-safe** - everything is written in TypeScript and ships Radix-level type definitions
- **Composable** - each primitive is compatible with [tailwind-variants](https://www.npmjs.com/package/tailwind-variants) or [class-variance-authority (CVA)](https://cva.style/docs) for themeable variants

## Why We Chose ShadCN UI

- **Tailwind alignment** - every snippet already uses Tailwind utilities, so our design tokens and responsive classes work out of the box
- **No runtime payload** - because components are local code, we avoid shipping extra JavaScript bundles or CSS files
- **Accessible by default** - Radix UI handles focus traps, keyboard navigation and ARIA attributes. QA can focus on business logic, not low-level a11y
- **Incremental adoption** - we can generate only the primitives we need, drop them into an existing codebase, and migrate gradually
- **Long-term maintainability** - owning the files means we are not blocked by upstream releases. If a bug appears we can patch it immediately

## Core Principles and Design Philosophy

1. **Copy, do not import** - you generate source once and version-control it. No vendor lock-in
2. **Radix first** - behaviour comes from Radix primitives ([Dialog](https://www.radix-ui.com/primitives/docs/components/dialog), [Popover](https://www.radix-ui.com/primitives/docs/components/popover), [ScrollArea](https://www.radix-ui.com/primitives/docs/components/scroll-area)) which have battle-tested accessibility
3. **Tailwind for visuals** - styling stays declarative and token-driven. There is no CSS-in-JS runtime or Shadow DOM
4. **Plain React** - no custom renderers, no wrappers around hooks; what you see is what you get
5. **Opt-in complexity** - advanced features (CVA variants, Framer Motion animations) are present but optional

## Installing ShadCN UI in a Next.js + Tailwind Stack

The project ships a small CLI you run once during setup. In a fresh repository:

1. Ensure Tailwind is already configured (our project starter includes it)
1. Initialise shadcn/ui:  
    `npx shadcn-ui@latest init`
   - The script prompts for your project path (`src`), components path (`components/ui`), styling solution (`Tailwind CSS`), and preferred alias
1. Generate your first component, for example Button:  
   `npx shadcn-ui@latest add button`

The script drops `button.tsx` into `components/ui`. It also updates `tailwind.css` with any missing plugin or theme extension. Commit the diff - you own these files.

## Project Structure and File Conventions

Our should follow similar layout:

```
app/
    components/
        ui/
            button.tsx
            dialog.tsx
    lib/
        tailwind/
            tailwind.css
```

- **components/ui** - atomic primitives straight from `shadcn/ui`. Keep each file self-contained
- **components/** (root) - higher-level composites that import primitives
- **lib/** - helper utilities, CVA variant factories, Framer Motion wrappers
- **lib/tailwind/tailwind.css** - houses Tailwind's `@import 'tailwindcss';` plus any custom layers

Generated files follow these conventions:

- **PascalCase filenames** (`alert-dialog.tsx`)
- **CVA** variant definitions placed near the top for quick customisation

## Working with Primitives

### Buttons

The generated Button uses CVA to expose `variant` (`default | destructive | ghost | link`) and `size` (`sm | lg`) props. Extend with new variants by editing the `cva()` call and updating the union type.

Example usage:

```tsx
<Button variant="destructive" size="lg" asChild>
  <Link href="/danger-zone">Delete account</Link>
</Button>
```

`asChild` comes from Radix Slot and lets any element inherit Button behaviour.

### Inputs and Forms

shadcn/ui provides `input.tsx`, `label.tsx` and `textarea.tsx` primitives. Combine them with `react-hook-form`:

```tsx
const { register } = useForm()
<Label htmlFor="email">Email</Label>
<Input id="email" type="email" {...register('email')} />
```

The visual style stays consistent because every field shares the same Tailwind classes.

### Modals and Dialogs

`dialog.tsx` wraps `@radix-ui/react-dialog` with overlay, content, title and description slots. It already traps focus and restores scroll position. Tailwind classes control z-index, backdrop blur and animations.

### Navigation Components

`menubar`, `dropdown-menu` and `navigation-menu` cover most header and sidebar patterns. Each file exposes compound sub-components so markup remains readable:

```tsx
<NavigationMenu>
  <NavigationMenu.List>
    <NavigationMenu.Item>
      <NavigationMenu.Link href="/pricing">Pricing</NavigationMenu.Link>
    </NavigationMenu.Item>
  </NavigationMenu.List>
</NavigationMenu>
```

### Data Display Components

`table.tsx`, `badge.tsx` and `progress.tsx` handle common dashboard widgets. Because styling is Tailwind, you can inject our design tokens (e.g. `bg-brand-600`) without fighting a theming API.

## Theming and Design Tokens

shadcn/ui relies entirely on Tailwind for colours, spacing and radii. Our starter defines tokens in `@theme`:

```css
@theme {
  --radius-lg: 0.75rem;
  --brand-600: 22 119 255;
}
```

Component classes reference tokens through `bg-brand-600` or `rounded-lg`. Variants inside CVA also read tokens, ensuring dark-mode and future re-branding remain single-source.

## Accessibility Defaults and Customization

Radix primitives guarantee:

- **Keyboard navigation** - Tab, Shift-Tab, Arrow keys
- **Focus management** - correct `aria-modal`, focus trap, and restore
- **Screen-reader labels** - each component ships required `role` and `aria-*` attributes

You can add extra labels inline:

```tsx
<Dialog title="Delete account" description="This action is irreversible">
```

Color contrast remains our responsibility. Use Tailwind's `text-brand-foreground` tokens and [eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y) to lint contrast.

## Composing and Extending Components

Because each primitive is local code, composition works like ordinary React:

```tsx
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/cn"; // tailwind-merge helper

export function IconButton(props: ButtonProps & { icon: ReactNode }) {
  const { icon, className, ...rest } = props;

  return (
    <Button className={cn("flex items-center gap-2", className)} {...rest}>
      {icon}
      {props.children}
    </Button>
  );
}
```

For styling, prefer **class-variance-authority (CVA)** over ad-hoc template strings. This keeps variants declarative:

```tsx
const alertVariants = cva("rounded-md border p-4", {
  variants: {
    intent: {
      info: "bg-blue-50 text-blue-700 border-blue-200",
      warn: "bg-yellow-50 text-yellow-700 border-yellow-200",
      error: "bg-red-50 text-red-700 border-red-200",
    },
  },
});
```

## Integrating Animations (Framer Motion)

shadcn/ui does not dictate motion. Wrap primitives with `motion()`:

```tsx
import { DialogContent } from '@/components/ui/dialog'
import { motion } from 'framer-motion'

const MotionDialog = motion(DialogContent)

<MotionDialog
    initial={{ scale: 0.9, opacity: 0 }}
    animate={{ scale: 1,   opacity: 1 }}
    exit={{    scale: 0.9, opacity: 0 }}
    transition={{ duration: 0.15 }}
>
    {children}
</MotionDialog>
```

Because styles are static Tailwind classes, no runtime CSS conflicts with Framer Motion transforms.

## Performance Considerations

- **Bundle size** - no extra JavaScript beyond Radix UI - tree-shake with ES modules
- **No CSS payload** - utilities are already in Tailwind CSS, so adding more primitives does not grow CSS
- **Code splitting** - components are ordinary React, so Next.js can lazy-load pages or sections
- **Server Components** - primitives are compatible because they render to plain HTML without client-side styling logic

## Testing ShadCN Components

- **Unit tests** - use Jest + React Testing Library. Query by role (`getByRole('dialog')`) to verify accessibility
- **Visual regression** - Storybook or Playwright - because variants are deterministic CVA classes, snapshots remain stable
- **Accessibility audits** - `jest-axe` or `@axe-core/playwright` catch contrast and ARIA regressions

## Common Pitfalls and How to Avoid Them

- **Forgotten Tailwind plugin** - the CLI adds `tailwindcss-animate`. If classes like `animate-in` are missing, re-run `shadcn-ui init`
- **Overwriting local edits** - the `add` command asks before overwrite; choose `skip` or `diff`. Keep a dedicated `components/ui/overrides` folder for heavy customisations
- **Naming collisions** - ensure generated files use the same barrel export style as your project (`index.ts`)
- **Dark-mode mismatch** - declare `dark:` variants in CVA early; retrofitting later triggers double work
- **Radix version drift** - pin Radix to the version in `package.json` notes; major bumps can change prop contracts

## Migration Strategy from Other UI Libraries

1. **Audit primitives** - list which Chakra or MUI components you actually use
2. **Generate equivalents** - run `npx shadcn-ui add` for each
3. **Create wrappers** - keep the old prop API but proxy to shadcn primitives so call sites remain unchanged during migration
4. **Map tokens** - translate theme colours and radii into Tailwind config
5. **Remove legacy provider** - delete `ChakraProvider` or `ThemeProvider`, clean up leftover style resets
6. **Measure** - compare bundle size, FCP and Total Blocking Time before deleting the old dependency

## Frequently Asked Questions

- **Do I need to keep the CLI as a dev dependency?**  
  No, the CLI is optional after generation. You can remove it and re-add when you want new components.

- **Can I customise a component beyond recognition?**  
  Absolutely - **it is your file**. The only caveat is losing upstream updates, but because the diff is visible you can manually cherry-pick later.

- **Is shadcn/ui production ready?**  
  All underlying Radix primitives are stable. The glue code is thin. We use it in customer-facing apps without issue.

- **How does this differ from Radix directly?**  
  Radix gives low-level behaviour, shadcn/ui adds Tailwind classes, variants and opinionated composition so you skip repetitive boilerplate.

## Further Reading

- Documentation
  - [Radix UI docs](https://www.radix-ui.com/primitives)
  - [Tailwind Variants docs](https://www.tailwind-variants.org/)
  - [shadcn/ui docs](https://ui.shadcn.com/docs)
- Articles
  - [Design System in React with Tailwind, Shadcn/ui and Storybook](https://dev.to/shaikathaque/design-system-in-react-with-tailwind-shadcnui-and-storybook-17f)
