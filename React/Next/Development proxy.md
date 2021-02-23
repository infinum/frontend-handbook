## Motivation

When developing applications, sometimes you want to test your local changes against data from a different environment. For example: creating a production hotfix requires you to locally connect to the production API. Maybe even testing the solution on a different device e.g. a [phone connected to your machine](https://developers.google.com/web/tools/chrome-devtools/remote-debugging). That API could have http-only cookies that cannot be manipulated on the client and require a proxy that can intercept a request and do some modifications on it.

## The issue

NextJS already supports/recommends ways to proxy outgoing requests:

- [custom reverse proxy](https://github.com/vercel/next.js/tree/master/examples/with-custom-reverse-proxy)
- [redirects](https://nextjs.org/docs/api-reference/next.config.js/redirects)

The problem with the former is that it runs on a different server. When testing local changes on e.g. a phone, we will get unresolvable CORS issues.
The problem with the latter is that NextJS redirects don't provide a possibility to manipulate requests and/or responses. They just redirect.

## The fix

Prerequisites:

- NextJS [API Routes](https://nextjs.org/docs/api-routes/introduction) with the [Optional catch all API route](https://nextjs.org/docs/api-routes/dynamic-api-routes#optional-catch-all-api-routes)
- NextJS [API middleware](https://nextjs.org/docs/api-routes/api-middlewares) using [`http-proxy-middleware`](https://github.com/chimurai/http-proxy-middleware)
- a way to pick the proper URL to the proxy (`PROXY_ENV` in this case)

```js
// pages/api/[[...slug]].ts
import { IncomingMessage } from 'http';
import { createProxyMiddleware } from 'http-proxy-middleware';
import { NextApiRequest, NextApiResponse } from 'next';

let apiUrl: string;

switch (process.env.PROXY_ENV) { 
  // pick API endpoint depending on the PROXY_ENV, assign to apiUrl 
  case 'production':
    apiUrl = 'https://production.example.com';
    break;
  case 'uat':
    apiUrl = 'https://uat.example.com';
    break;
  default:
    apiUrl = 'https://staging.example.com';
}

const proxy = createProxyMiddleware({
  target: apiUrl,
  changeOrigin: true,
  logLevel: 'debug',
  cookieDomainRewrite: 'localhost',
  onProxyRes: (proxyRes: IncomingMessage) => { 
    // You can manipulate the cookie here

    if (!proxyRes.headers['set-cookie']) {
      return;
    }

    // For example you can remove secure and SameSite security flags so browser can save the cookie in dev env
    const adaptCookiesForLocalhost = proxyRes.headers['set-cookie'].map((cookie) =>
      cookie.replace(/; secure/gi, '').replace(/; SameSite=None/gi, ''),
    );

    proxyRes.headers['set-cookie'] = adaptCookiesForLocalhost;
  },
  onError: (err: Error) => console.error(err),
});

export default function handler(req: NextApiRequest, res: NextApiResponse<unknown>) {
  // Don't allow requests to hit the proxy when not in development mode
  // NextJS doesn't allow conditional API routes
  if (process.env.NODE_ENV !== 'development') {
    return res.status(404).json({ message: 'Not found' });
  }
  // @ts-ignore
  return proxy(req, res);
}

export const config = {
  api: {
    bodyParser: false, // enable POST requests
    externalResolver: true, // hide warning message
  },
};
```

We can then run `PROXY_ENV=<insert-env> next dev` and then NextJS development instance will point to the appropriate API endpoint.

## The implications

The only problem with this approach is that the defined NextJS route (`pages/api/[[...slug]]`) can be reached in production since NextJS doesn't support conditional removal of API routes. We avoided the invocation of the proxy middleware by check if the current environment is NOT "development" and returned a 404 (can be changed to something more suitable).
