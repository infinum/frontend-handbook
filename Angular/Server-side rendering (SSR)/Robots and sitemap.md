## Robots
If you want to influence the content on your website that search engines crawl and index, you have several options. Two of these options involve using robots meta tags and robots.txt. Whilst they may sound similar, they differ in important ways.

The robots.txt file is not suitable for safely excluding content from indexation. Incoming links may still cause content to be indexed under certain circumstances.

Google therefore advises that you use the robots.txt file to manage crawling traffic and prevent image, video and audio files from appearing in search results. You can read about this in more details on this [link](https://developers.google.com/search/docs/crawling-indexing/robots/intro#what-is-a-robots.txt-file-used-for).

Always set `noindex` and `nofollow` for the staging environment, and only allow indexing and following for production.

### What are robots meta tags?
Robots meta tags are snippets that you place in the head section of a page. They look like this:

`<meta name="robots" content="noindex" />`

You mark which search engine you want to address with the name attribute, and the content attribute indicates the desired action. In this example, the tag prevents the content from being indexed by all search engines.

We will create a function for setting robots depending on the environment type:

```typescript
export function initIndexing(
	environmentVariablesService: EnvironmentVariablesService,
	meta: Meta
): () => Promise<void> {
	return async () => {
		const isProduction = environmentVariablesService.get(EnvironmentVariable.APP_ENVIRONMENT) === 'production';

		meta.removeTag('name="robots"');
		meta.addTag({
			name: 'robots',
			content: isProduction ? 'index, follow' : 'noindex, nofollow',
		});
	};
}
```

And add this on app initiation in the `app.module.ts`:

```typescript
@NgModule({
    providers: [
        {
            provide: APP_INITIALIZER,
            useFactory: initIndexing,
            multi: true,
            deps: [EnvironmentVariablesService, Meta],
        },
    ]
})
```

### What is robots.txt?
A robots.txt file (Robots Exclusion Standard Protocol) is a text file that tells search engine crawlers which files or pages they can crawl.

The search engine or its crawler is identified in the robots.txt file with user-agent. `disallow` and `allow` commands can be used to specify which directories should and should not be crawled. You can also refer to the location of a sitemap in the robots.txt file:

```text
User-agent: *
Disallow: /
```

Instead of having a statically served robots.txt file that is read from a file, we can use express to return the content for the robots.txt file. The returned content can be different per-environment. Add the following snippet to `server.ts`:

```typescript
const nonIndexableRobotsContent = 'User-agent: *\nDisallow: /';
const indexableRobotsContent = 'User-agent: *\nDisallow:';
const isProduction = process.env[EnvironmentVariable.APP_ENVIRONMENT] === 'production';

server.get('/robots.txt', (req, res) => {
		res.type('text/plain');
		res.send(isProduction ? indexableRobotsContent : nonIndexableRobotsContent);
	});
```

## Sitemap
An XML sitemap is a file that lists a website’s important pages, making sure Google can find and crawl them all. It also helps search engines understand your website structure. You want Google to crawl every essential page of your website. But sometimes, pages end up without any internal links pointing to them, making them hard to find. A sitemap can help speed up content discovery.

Similar to the dynamic robots.txt file, we can also use express to build and serve the sitemap XML. This approach allows us to set some cache expiration time for the sitemap and even trigger re-build of the sitemap if the content changes dynamically. For this to work we will need to install these libraries:

 * [apicache](https://www.npmjs.com/package/apicache) for caching the response
 * [sitemap](https://www.npmjs.com/package/sitemap) for generating sitemap.xml file

```typescript
server.get('/sitemap.xml', cache('1 hour'), (req, res) => {
		res.header('Content-Type', 'application/xml');
        
		try {
			const smStream = new SitemapStream({ hostname: process.env[EnvironmentVariable.APP_URL] });

			(async () => {
				const routes = ['route1', 'route2'];

				for (const route of routes) {
					smStream.write(route);
				}

				smStream.end();
				
				smStream.pipe(res).on('error', (e) => {
					throw e;
				});
			})();
		} catch (e) {
			console.error(e);
			res.status(500).end();
		}
	});
```
This example can be used if there is no more than 50000 routes. For more details, please check out the [documentation](https://www.npmjs.com/package/sitemap).
