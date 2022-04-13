## Motivation

Usually your applications need some kind of cooler fonts which can not be found within system fonts for all the operating systems.
Because of that you either have the fonts locally in the project or use them via CDN (mostly used from Google Fonts).
The both approaches has its pros and cons, so NextJS took only pros from each and implemented a good optimization.

Additionally, and related to the fonts, there were a reports of the issues with NextJS in some browsers where the blank screens appeared when opening the applications, 
even if the content is loaded (https://stackoverflow.com/questions/63055442/react-next-js-site-doesnt-load-properly-in-safari-blank-page).
The issues were related with the fonts and how NextJS handled them when using the fonts via CDN.
So the optimization also tackles this.

**Disclaimer**
- The font optimization currently supports only Google Fonts and Typekit (no self-hosted fonts can be optimized yet) 


## Implementation

Since version `10.2`, NextJS has built-in web font optimization. In the optimization the fonts will be downloaded and inlined
at the build time (as the `<style>` tag). This allows the external fonts to be within your project, as same as you added them in public folder manually.

Only code needed to support this is to add `link` tag in `_document.js`:
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

More details can be read in [NextJS Font Optimization documentation](https://nextjs.org/docs/basic-features/font-optimization).

## Conclusion

It is not wrong to have the fonts downloaded and used within the project, but this optimization from NextJS is even improving this by
having the latest font version on the build inlined in the pages.