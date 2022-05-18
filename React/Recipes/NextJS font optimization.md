## Motivation

Applications often need custom fonts which cannot be found within the system fonts of all the operating systems.
Because of that you either have the fonts locally in the project or use them via CDN (mostly used from Google Fonts).
Both approaches have their pros and cons, so NextJS took pros from each and implemented a good optimization.

Additionally, and related to the fonts, there were reports of issues with NextJS in some browsers where blank screens appeared when opening applications, even if the content is loaded ([reference](https://stackoverflow.com/questions/63055442/react-next-js-site-doesnt-load-properly-in-safari-blank-page)).
The issues were related to the fonts and how NextJS handled them when using them via CDN.
So the optimization also tackles this.

**Disclaimer**
- Font optimization currently supports only Google Fonts and Typekit (no self-hosted fonts can be optimized yet).


## Implementation

Since version `10.2`, NextJS has built-in web font optimization. In the optimization the fonts will be downloaded and inlined at the build time (as the `<style>` tag).

To support this add the following `link` tag to `_document.js`:

```jsx
// pages/_document.js

import Document, { Html, Head, Main, NextScript } from 'next/document';

class MyDocument extends Document {
    render() {
        return (
            <Html>
                <Head>
                    <link
                        href="https://fonts.googleapis.com/css2?family=Inter&display=optional"
                        rel="stylesheet"
                    />
                </Head>
                
                <body>
                    <Main />
                    <NextScript />
                </body>
            </Html>
        )
    }
}

export default MyDocument;
```

You can also opt-out of the optimization:

```javascript
// next.config.js

module.exports = {
  optimizeFonts: false,
}
```

More details can be found in the [NextJS Font Optimization documentation](https://nextjs.org/docs/basic-features/font-optimization).

## Conclusion

In case you are using custom fonts coming from a CDN, NextJS will (by default) download  the font files and serve them from the project directory, the same way it would be done if you manually added the custom font files yourself.