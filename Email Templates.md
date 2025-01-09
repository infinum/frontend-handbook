## MJML

For creating email templates, we use [MJML](https://mjml.io/), a responsive email framework that simplifies writing HTML emails. MJML allows developers to write clean, reusable components that get compiled into fully responsive HTML emails.

### Why MJML?

1. Responsive by Default – MJML abstracts the complexity of email responsiveness.
2. Simple Syntax – Reduces the need for manually writing complex, nested HTML tables.
3. Cross-Email Client Compatibility – Works across Gmail, Outlook, Yahoo, and Apple Mail.
4. Reusable Components – Custom components speed up development and ensure consistency.
5. Fast Development – The built-in compiler instantly converts MJML into production-ready HTML.

### Getting Started with MJML

MJML can be used through an online editor, CLI, or Node.js.

#### MJML Online Editor

You can try MJML in the browser without installation [HERE](https://mjml.io/try-it-live)

#### Using MJML globally

Install MJML globally using pnpm:

```
pnpm i -g mjml
```

Now, you can convert MJML to HTML using:

```
mjml input.mjml -o output.html
```

#### Using MJML in a project

Install MJML as a local dependency:

```
pnpm i mjml
```

Use it in a JavaScript:

```
const mjml = require('mjml')

const htmlOutput = mjml(`
  <mjml>
    <mj-body>
      <mj-section>
        <mj-column>
          <mj-text>Hello World!</mj-text>
        </mj-column>
      </mj-section>
    </mj-body>
  </mjml>
`)

// Output compiled HTML
console.log(htmlOutput.html)
```

### MJML Structure

Each MJML file follows a simple structure, and provides built-in components that make email development faster. Here' basic example:

```
<mjml>
  <mj-head>
    <mj-title>My Email</mj-title>
  </mj-head>
  <mj-body>
    <mj-section>
      <mj-column>
        <mj-text>Hello World!</mj-text>
      </mj-column>
    </mj-section>
  </mj-body>
</mjml>
```

#### MJML Components

**Main parts**

| Tag          | Description                                               |
|--------------|-----------------------------------------------------------|
| `<mjml>`       | Root tag for the MJML template.                           |
| `<mj-head>`    | Contains metadata like `<mj-title>` and `<mj-preview>`.       |
| `<mj-body>`    | The main body of the email.                               |
| `<mj-section>` | Creates a full-width row inside `<mj-body>`.                |
| `<mj-column>`  | A column inside a section (used for structuring content). |
| `<mj-group>`   | Groups multiple columns together (useful for nested layouts). |
| `<mj-divider>` | Creates a horizontal divider.                                 |

**Typography Components**

| Tag    | Description                                              |
|--------------|----------------------------------------------------------|
| `<mj-text>`    | Adds text content.                                       |
| `<mj-title>`   | Sets the email title (appears in the browser tab).       |
| `<mj-preview>` | Provides a short preview of the email (seen in inboxes). |

**Media components**

| Tag           | Description                            |
|---------------|----------------------------------------|
| `<mj-image>`    | Adds an image with responsive scaling. |
| `<mj-carousel>` | Creates an image carousel.             |

**Button component**

Used to create call-to-action buttons.

```
<mj-button href="https://infinum.com">
  Click Me!
</mj-button>
```

**Table component**

```
<mj-table>
    <tr>
        <td>Tag</td>
        <td>Description</td>
    </tr>
    <tr>
        <td>`<mj-table>`</td>
        <td>Useful for tabular data.</td>
    </tr>
</mj-table>
```

### Styling in MJML

MJML allows inline styles and attributes for customization.

```
<mj-text font-size="18px" color="#555" padding="10px">
  Styled Text
</mj-text>
```

To apply global styles, use `<mj-style>` in the `<mj-head>`:

```
<mj-head>
  <mj-style inline="inline">
    .custom-class { color: red; font-size: 18px; }
  </mj-style>
</mj-head>

<mj-text class="custom-class">This is red text.</mj-text>
```

### Best Practices for MJML

1. Use sections & columns for Layouts
   * Always wrap content inside <mj-section> and <mj-column> to ensure responsiveness.

2. Limit the width of Emails
   * Emails should be 600px wide to look good on most email clients.

3. Use MJML-Specific attributes
   * Example: `padding`, `border-radius`, and `text-align` instead of writing CSS manually.

4. Ensure Accessibility
   * Use alt attributes for `<mj-image>`.

5. Use Inline Styles Sparingly
   * MJML helps with styling, so avoid excessive inline CSS.

6. Use the MJML Online Editor for quick testing
   * [MJML Playground](https://mjml.io/try-it-live) is useful for previewing templates.

### Common Issues & Debugging

| Issue                         | Solution                                                 |
|-------------------------------|----------------------------------------------------------|
| Email looks broken in Outlook | Use `mj-wrapper` to ensure proper rendering.               |
| Gmail clips email             | Keep HTML size under 102KB.                              |
| Custom fonts not loading      | Use Web-safe fonts (Arial, Helvetica, sans-serif).       |
| Images not showing            | Use absolute URLs (e.g., `https://example.com/image.png`). |
