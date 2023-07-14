## Motivation

With `next/font`, you can automatically optimize both custom and standard fonts, reducing external network requests for enhanced performance and privacy.

One notable advantage of `next/font` is its built-in self-hosting capability for any font file. Leveraging the CSS `size-adjust` property ([mdn](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/size-adjust)), web fonts can be loaded optimally, eliminating any layout shift issues. Additionally, Next.js allows you to conveniently utilize [Google Fonts](https://fonts.google.com/) while prioritizing performance and privacy. During the build process, CSS and font files are downloaded and self-hosted alongside other static assets, eliminating the need for additional browser requests.

*More details can be found in the [Official NextJS Font Optimization documentation](https://nextjs.org/docs/basic-features/font-optimization).*

## Implementation

### Google Fonts

1. We import **ChakraProvider**, the **desired font *(e.g. Roboto)*** and our **theme** definition:

```javascript
// _app.tsx
import { ChakraProvider } from '@chakra-ui/react';
import { Roboto } from 'next/font/google';

import theme from '@/styles/theme';
...
```

2. We **define the font** with the desired [properties](https://nextjs.org/docs/app/api-reference/components/font):

```javascript
...
const roboto = Roboto({
	weight: ['400', '700'], // Required, unless using variable fonts
	style: ['normal', 'italic'], // Optional
	subsets: ['latin'], // Optional
	display: 'swap', // Optional
});
...
```

It is recommended to use [variable fonts](https://fonts.google.com/variablefonts#font-families). When using a variable font, we do **not** have to define the **weight** property.

> Good to know: Use an underscore (_) for font names with multiple words. E.g. Roboto Mono should be imported as Roboto_Mono.

3. We **extend our theme** by applying the font styles to our theme:

```javascript
...
theme.fonts.body = roboto.style.fontFamily;
theme.fonts.heading = roboto.style.fontFamily;
...
```

4. We use our modified theme and pass it to **ChakraProvider**:

```javascript
...
function App({ Component, pageProps, err }) {
	return (
		<ChakraProvider theme={theme}>
			<Component {...pageProps} err={err} />
		</ChakraProvider>
	);
}

export default App;
```

### Local Fonts

1. We import **ChakraProvider**, **localFont** and our **theme** definition:

```javascript
// _app.tsx
import { ChakraProvider } from '@chakra-ui/react';
import localFont from 'next/font/local';

import theme from '@/styles/theme';
...
```

2. We **define the font** with the desired [properties](https://nextjs.org/docs/app/api-reference/components/font):

```javascript
...
const gtHaptik = localFont({
	src: [
		{ path: '../assets/fonts/GT-Haptik-Regular.woff', weight: '400', style: 'normal' },
		{ path: '../assets/fonts/GT-Haptik-Bold.woff', weight: '700', style: 'normal' },
	],
});
...
```

3. We **extend our theme** by applying the font styles to our theme:

```javascript
...
theme.fonts.body = gtHaptik.style.fontFamily;
theme.fonts.heading = gtHaptik.style.fontFamily;
...
```

4. We use our modified theme and pass it to **ChakraProvider**:

```javascript
...
function App({ Component, pageProps, err }) {
	return (
		<ChakraProvider theme={theme}>
			<Component {...pageProps} err={err} />
		</ChakraProvider>
	);
}

export default App;
```

## Conclusion

By implementing font optimization techniques provided by Next.js, you can significantly improve the user experience of your website. The combination of efficient font loading and reduced external requests contributes to faster load times and improved privacy.