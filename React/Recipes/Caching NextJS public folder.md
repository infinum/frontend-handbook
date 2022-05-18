## Motivation

The `Cache-Control` HTTP header allows setting the caching for the resources used by the browser.
It is always a good practice to check if some resources can be cached, so the user's browser can use them from the cache without consuming more data and without waiting on the resource to finish downloading.

More about `Cache-Control` can be read online (e.g. in [Mozilla docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control))

Next.js (up to `v12.1.4`) does not cache the `resources` in the `public` folder by the default, and usually a lot of the resources or even all can benefit some sort of the caching mechanism.
Because of that we need to add a small configuration in the project.

![By default NextJS does not cache public resources](/img/nextjs/nextjs_logo_not_cached.jpeg)

## Implementation

Next.js allows custom HTTP headers by adding the `headers` key in `next.config.js`.
You can easily add `Cache-Control` HTTP header by writing the code below:

```javascript
// An example of the `next.config.js` file - your content can be different

module.exports = {
    headers: async () => [
        {
            // list more extensions here if needed; these are all the resources in the `public` folder including the subfolders
            source: '/:all*(svg|jpg|png)',
            locale: false,
            headers: [
                {
                    key: 'Cache-Control',
                    value: 'public, max-age=31536000, stale-while-revalidate',
                },
            ],
        },
    ],
}
```

![Resources are cached now](/img/nextjs/nextjs_logo_cached.jpeg)

The configuration is adding `max-age` to 1 year so only after that time the browser will make a request to get the latest version of `svg`, `jpg` and `png` resources in `public` folder.

You can also mix different configurations such as:

- Caching only specific resources

```javascript
// An example of `next.config.js` file - your content can be different

module.exports = {
    headers: async () => [
        {
            // the `cached` folder in the `public` folder contains the resources (`jpg`s) we want to cache
            source: '/cached/:all*(jpg)',
            locale: false,
            headers: [
                {
                    key: 'Cache-Control',
                    value: 'public, max-age=31536000, stale-while-revalidate',
                },
            ],
        },
    ],
}
```
- Different caching values

```javascript
// An example of `next.config.js` file - your content can be different

module.exports = {
    headers: async () => [
        {
            // the `fresh` folder in the `public` folder contains the resources (`jpg`s) we want to cache for a short time
            source: '/fresh/:all*(jpg)',
            locale: false,
            headers: [
                {
                    key: 'Cache-Control',
                    value: 'public, max-age=5, stale-while-revalidate', // resource is refreshed after 5 seconds
                },
            ],
        },
        {
            // the `stale` folder in the `public` folder contains resources (`jpg`s) we want to cache for a longer time
            source: '/stale/:all*(jpg)',
            locale: false,
            headers: [
                {
                    key: 'Cache-Control',
                    value: 'public, max-age=31536000, stale-while-revalidate',
                },
            ],
        },
    ],
}
```

Also, more about caching values can be read online (e.g. in [Mozilla docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)).

## Conclusion

There is no issue to add some sort of caching for the resources that usually do not change, but investigate what kind of the caching mechanism suits best for your use case.