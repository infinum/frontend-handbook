## Motivation

Although this is not an issue with NextJS but React (check more details [here](https://github.com/facebook/react/issues/13838)),
you can encounter this if you have the static pages, and those static pages have some HTML entities in the page content -
e.g. ampersand (`&`) in query parameters.

The static pages will have the HTML entities being encoded, which will result in incorrect content. Probably it will be
decoded correctly on the client, but this might be an issue for the crawlers for your static content.

![Ampersand is encoded incorrect](/img/nextjs/nextjs_encoding_issue.jpg)

Because of this, there is a way to handle it by adding a custom decode in `_document` file.

Disclaimer:
- Check if this issue is still happening
- Check if need this fix
- Always be cautious when changing default NextJS configurations for a custom document ([custom document](https://nextjs.org/docs/advanced-features/custom-document#customizing-renderpage))

`To prepare for React 18, we recommend avoiding customizing getInitialProps and renderPage, if possible.`


## Implementation

If you do not have the `_document` file in the `pages` folder, create a file with the default content 
([copy all except gIP in _document](https://nextjs.org/docs/advanced-features/custom-document#customizing-renderpage)) and:
- install the `html-entities` package (https://github.com/mdevils/html-entities)
- add an import to the `html-entities` package
```javascript
import { decode } from 'html-entities';
```
- add the gIP implementation in the class
```jsx
static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);
    // based on https://github.com/vercel/next.js/issues/2006
    return {
        ...initialProps,
        html: initialProps.html.replace(
            /(href|src|srcSet)="([^"]+)"/g,
            (match, attribute, value) => `${attribute}="${decode(value)}"`
        ),
    };
}
```
- add or change regex properties to handle more cases

The above code executes for all the pages in NextJS, searches and replaces all `href`, `src` and `srcSet` attributes in HTML with the
decoded characters.

![Ampersand is now encoded correctly](/img/nextjs/nextjs_encoding_fixed.jpg)


## Conclusion

Use the solution above only if you encounter the issues. Do not add this by default in a new or an existing NextJS application.