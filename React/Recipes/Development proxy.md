## Motivation

When developing applications, you sometimes want to test your local changes against data from a different environment. For example, creating a production hotfix requires you to locally connect to the production API. Maybe even testing the solution on a different device, e.g. a [phone connected to your machine](https://developers.google.com/web/tools/chrome-devtools/remote-debugging). That API could have http-only cookies that cannot be manipulated on the client and require a proxy that can intercept some requests and do some modifications to them.

## The issue

NextJS already supports/recommends ways to proxy outgoing requests:

* [custom reverse proxy](https://github.com/vercel/next.js/tree/master/examples/with-custom-reverse-proxy)
* [redirects](https://nextjs.org/docs/api-reference/next.config.js/redirects)

The problem with the former is that it runs on a different server. When testing local changes on e.g. a phone, we will get unresolvable CORS issues.
The problem with the latter is that NextJS redirects don't provide a possibility to manipulate requests and/or responses. They just redirect.

## The fix

Prerequisites:

* Install [`http-proxy-middleware`](https://github.com/chimurai/http-proxy-middleware): `npm install http-proxy-middleware`
* NextJS [API Routes](https://nextjs.org/docs/api-routes/introduction) with the [Optional catch all API route](https://nextjs.org/docs/api-routes/dynamic-api-routes#optional-catch-all-api-routes)
* NextJS [API middleware](https://nextjs.org/docs/api-routes/api-middlewares) using [`http-proxy-middleware`](https://github.com/chimurai/http-proxy-middleware)
* a way to pick the proper URL to the proxy (`PROXY_ENV` in this case)

You can create a reusable `createProxy` in your code base, and then use it in your API routes:

```ts
// src/lib/proxy/index.ts

import { createProxyMiddleware, Options } from 'http-proxy-middleware';
import { NextApiRequest, NextApiResponse } from 'next';

interface ICreateProxyOptions extends Omit<Options, 'apiUrl'> {
	/**
	 * Used for disabling the proxy. By default it's enabled only in development.
	 *
	 * @default process.env.NODE_ENV !== 'development'
	 *
	 * @example
	 *
	 * createProxy('http://localhost:3000', {
	 *   disable: NODE_ENV === 'production'
	 * })
	 *
	 * @example
	 *
	 * createProxy('http://localhost:3000', {
	 *   disable: (req) => req.headers['x-skip-proxy'] === 'true'
	 * })
	 */
	disable?: boolean | ((req: NextApiRequest) => boolean);
}

export const createProxy = (apiUrl: Options['target'], options: ICreateProxyOptions = {}) => {
	const { disable = process.env.NODE_ENV !== 'development', ...proxyOptions } = options;

	const proxy = createProxyMiddleware({
		target: apiUrl,
		// changeOrigin: true ensures the Host header is rewritten to match the target
		changeOrigin: true,
		// Rewrite cookie domain to localhost so cookies work in development
		cookieDomainRewrite: 'localhost',
		// Remove /api prefix before forwarding to target
		pathRewrite: { '^/api': '/' },
		...proxyOptions,
		onProxyRes: (proxyRes: any, req: any, res: any) => {
			// You can manipulate the cookie here
			// This is necessary because production APIs often set secure cookies
			// that won't work over HTTP localhost

			if (!proxyRes.headers['set-cookie']) {
				return;
			}

			// For example you can remove secure and SameSite security flags so browser can save the cookie in dev env
			// Secure cookies require HTTPS, and SameSite=None requires Secure flag
			const adaptCookiesForLocalhost = proxyRes.headers['set-cookie'].map((cookie: string) =>
				cookie.replace(/; secure/gi, '').replace(/; SameSite=None/gi, '')
			);

			proxyRes.headers['set-cookie'] = adaptCookiesForLocalhost;

			if (proxyOptions.onProxyRes && typeof proxyOptions.onProxyRes === 'function') {
				proxyOptions.onProxyRes(proxyRes, req, res);
			}
		},
		onError: (err: any, req: any, res: any, target: any) => {
			console.error(err);

			if (proxyOptions.onError && typeof proxyOptions.onError === 'function') {
				proxyOptions.onError(err, req, res, target);
			}
		},
	}) as (req: NextApiRequest, res: NextApiResponse<unknown>) => void;

	return function handler(req: NextApiRequest, res: NextApiResponse<unknown>) {
		if (typeof disable === 'function' ? disable(req) : disable) {
			return res.status(404).json({ message: 'Not found' });
		}

		return proxy(req, res);
	};
};
```

Use the `createProxy` function in your API routes:

```js
// pages/api/[[...slug]].ts

import { createProxy } from '@/lib/proxy';

export default createProxy(`${process.env.NEXT_PUBLIC_BASE_URL}/api/v1`, {
	secure: false,
	pathRewrite: { '^/api/backend': '/' },
});

export const config = {
	api: {
		// Disable body parsing so we can stream the request body
		bodyParser: false,
		// Tell Next.js this route is handled externally
		externalResolver: true,
	},
};

```


## The implications

The only problem with this approach is that the defined NextJS route (`pages/api/[[...slug]]`) can be reached in production because NextJS doesn't support conditional removal of API routes. The `disable` option in `createProxy` handles this by checking if the current environment is NOT "development" and returning a 404. This ensures the proxy is effectively disabled in production.
