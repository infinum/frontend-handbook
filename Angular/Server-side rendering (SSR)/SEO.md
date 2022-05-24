SEO stands for “search engine optimization.” In simple terms, it means the process of improving your site to increase its visibility when people search for products or services related to your business in Google, Bing, and other search engines. The better visibility your pages have in search results, the more likely you are to garner attention and attract prospective and existing customers to your business. To read more about SEO you can read [this article](https://searchengineland.com/guide/what-is-seo).

## Meta tags
__Meta tags are invisible tags that provide data about your page to search engines and website visitors.__ In short, they make it easier for search engines to determine what your content is about, and are vital for SEO. 

Here is list of the most important meta tags: 

### Title
This is the text you’ll see in the SERP and at the top of your browser. Search engines view this text as the “title” of your page

### Description 
A brief description of the page. The meta description should provide an accurate description of the content of your page. It is usually the element that determines whether users will click on your page, which makes it important to spend time on its optimization.

### Canonical Tag
A canonical tag is an HTML link tag with the attribute `rel=canonical`.
It’s used to indicate that there are other versions of this webpage. By implementing the canonical tag in the code, your website tells search engines that this URL is the main page and that the engines shouldn’t index other pages.

### Alternative Text Tag
Search engines can’t read images, which are a crucial part of many websites. Alternative text (alt text) is a way around that issue.
You should add proper alt text to images, so that search engines know how to interpret them.

### Robots Meta Tag
The robots meta tag tells search engines to either index or non-index your web page.

The tag has four main values for the search engine crawlers:

- `FOLLOW` –The search engine crawler will follow all the links in that webpage
- `INDEX` –The search engine crawler will index the whole webpage
- `NOFOLLOW` – The search engine crawler will NOT follow the page and any links in that webpage
- `NOINDEX` – The search engine crawler will NOT index that webpage

`<meta name=”robots” content=”noindex, nofollow”>` Means not to index or not to follow this webpage.

`<meta name=”robots” content=”index, follow”>` Means index and follow this webpage.

### Open Graph Meta Tags and Twitter Cards
These tags make social media syncing easier. Open graph meta tags promote integration between Facebook, LinkedIn, Google, and your website.

Here is a list of important Open Graph tags:
- `<meta property="og:title" content="TITLE">`
- `<meta property="og:description" content="DESCRIPTION">`
- `<meta property="og:url" content="URL">`
- `<meta property="og:site_name" content="SITE_NAME">`
- `<meta property="og:image" content="IMAGE_URL">`
- `<meta property="og:type" content="TYPE">` (you can find [here](https://ogp.me/#types) list of all types)
- `<meta property="og:locale" content="LOCALE">`

Twitter will use Open Graph meta tags if twitter ones are not set. You don't need to set twitter meta tags except if you want a different contest for the twitter cards.

- `<meta name="twitter:card" content="summary  | summary_large_image">`

### Header Tags
You can use header tags to change font sizes and signify information hierarchy on a page.
The heading elements go from H1 to H6. H1 is the largest and most important level, and H6 is the smallest and least important.

## SEO service
In this example here we will cover service example for the SEO:

```typescript
export interface ISEOContent {
	title: string;
	description: string;
	url: string;
	siteName: string;
	image?: string;
	imageWidth?: string;
	imageHeight?: string;
	imageType?: string;
	type?: string;
	twitterCard?: string;
}

@Injectable({
	providedIn: 'root',
})
export class SeoService {
	public readonly defaultDescriptionTranslationKey = 'seo.description';
	public readonly siteNameTranslationKey = 'seo.siteName';
	public readonly defaultImageUrl = urlJoin(
		this.environmentVariablesService.get(EnvironmentVariable.APP_URL),
		'assets',
		'images',
		'seo.jpg'
	);
	public readonly defaultImageWidth = '1200';
	public readonly defaultImageHeight = '630';
	public readonly defaultImageType = 'image/jpeg';
	public readonly defaultType = 'website';
	public readonly defaultTwitterCard = 'summary_large_image';

	constructor(
		private readonly titleService: Title,
		private readonly metaService: Meta,
		private readonly translocoService: TranslocoService,
		private readonly environmentVariablesService: EnvironmentVariablesService<EnvironmentVariable>
	) {}

	public setSEO({
		title,
		description,
		url,
		siteName,
		image,
		imageWidth,
		imageHeight,
		imageType,
		type,
		twitterCard,
	}: ISEOContent): void {
		// First remove all meta tags to eliminate checking for exiting ones.
		// Also benefit from deleting all is not to add meta tag when it is not provided for that page.
		this.removeAllMetaTags();

		// Set new meta tags
		this.setTitle(title, siteName);
		this.setDescription(description);
		this.setUrl(url);
		this.setSiteName(siteName);
		this.setImage(image);
		this.setImageWidth(imageWidth, image);
		this.setImageHeight(imageHeight, image);
		this.setImageType(imageType, image);
		this.setType(type);
		this.setLocale();
		this.setTwitterCard(twitterCard);
	}

    public createTranslatedSeoContent(
        title?: TranslateParams | undefined,
        description?: TranslateParams | undefined
    ): Observable<{ title: string; description: string; siteName: string }> {
        const title$ = this.createTranslatedTitleObservable(title);
        const description$ = this.createTranslatedDescriptionObservable(description);
        const siteName$ = this.createTranslatedSiteNameObservable();

        return combineLatest([title$, description$, siteName$]).pipe(
            map(([title, description, siteName]) => ({ title, description, siteName }))
        );
    }

    public createTranslatedTitleObservable(title?: TranslateParams | undefined): Observable<string> {
        return this.translocoService.selectTranslate(title ?? '');
    }

    public createTranslatedDescriptionObservable(description?: TranslateParams | undefined): Observable<string> {
        return this.translocoService.selectTranslate(description ?? this.defaultDescriptionTranslationKey);
    }

    public createTranslatedSiteNameObservable(): Observable<string> {
        return this.translocoService.selectTranslate(this.siteNameTranslationKey);
    }

	private setTitle(title: string | undefined, siteName: string): void {
		const titleContent = title ? `${title} | ${siteName}` : siteName;

		this.titleService.setTitle(titleContent);
		this.metaService.addTag({
			property: 'og:title',
			content: titleContent,
		});
	}

	private setDescription(description: string): void {
		this.metaService.addTag({
			name: 'description',
			content: description,
		});
		this.metaService.addTag({
			property: 'og:description',
			content: description,
		});
	}

	private setUrl(url: string): void {
		this.metaService.addTag({
			property: 'og:url',
			content: url,
		});
	}

	private setSiteName(siteName: string): void {
		this.metaService.addTag({
			property: 'og:site_name',
			content: siteName,
		});
	}

	private setImage(image: string | undefined): void {
		if (!image) {
			this.setDefaultImage();
			return;
		}
		this.metaService.addTag({ property: 'og:image', content: image });
	}

	private setImageWidth(imageWidth: string | undefined, image: string | undefined): void {
		if (imageWidth && image) {
			this.metaService.addTag({ property: 'og:image:width', content: imageWidth });
		}
	}

	private setImageHeight(imageHeight: string | undefined, image: string | undefined): void {
		if (imageHeight && image) {
			this.metaService.addTag({ property: 'og:image:height', content: imageHeight });
		}
	}

	private setImageType(imageType: string | undefined, image: string | undefined): void {
		if (imageType && image) {
			this.metaService.addTag({ property: 'og:image:type', content: imageType });
		}
	}

	private setDefaultImage(): void {
		this.metaService.addTag({ property: 'og:image', content: this.defaultImageUrl });
		this.metaService.addTag({ property: 'og:image:width', content: this.defaultImageWidth });
		this.metaService.addTag({ property: 'og:image:height', content: this.defaultImageHeight });
		this.metaService.addTag({ property: 'og:image:type', content: this.defaultImageType });
	}

	private setType(type: string | undefined): void {
		this.metaService.addTag({ property: 'og:type', content: type ?? this.defaultType });
	}

	private setLocale(): void {
		this.metaService.addTag({
			property: 'og:locale',
			content: languageToLocale(this.translocoService.getActiveLang() as Language),
		});
	}

	private setTwitterCard(type: string | undefined): void {
		this.metaService.addTag({ name: 'twitter:card', content: type ?? this.defaultTwitterCard });
	}

	private removeAllMetaTags(): void {
		this.metaService.removeTag('property="og:title"');
		this.metaService.removeTag('name="description"');
		this.metaService.removeTag('property="og:description"');
		this.metaService.removeTag('property="og:url"');
		this.metaService.removeTag('property="og:site_name"');
		this.metaService.removeTag('property="og:image"');
		this.metaService.removeTag('property="og:image:width"');
		this.metaService.removeTag('property="og:image:height"');
		this.metaService.removeTag('property="og:image:type"');
		this.metaService.removeTag('property="og:type"');
		this.metaService.removeTag('property="og:locale"');
		this.metaService.removeTag('name="twitter:card"');
	}
}
```

For using this SEO service it is best to create `guard` which will call this service before accessing the page. You can define route data and read it in the `guard`:

```angularjs
{
    path: 'homepage',
    data: {
        [RouteData.Title]: 'homepage.seo.title',
        [RouteData.Description]: 'homepage.seo.description',
    },
    canActivate: [SeoContentGuard],
    runGuardsAndResolvers: 'always',
}
```

This is example if your application is using translation, so you will provide translation keys.

`runGuardsAndResolvers: 'always'` is used to be sure that guard will be fired on navigation back.

To be able to use translations there is function `createTranslatedSeoContent` in the service which is using the `TranslocoService` to get translations. In the last step lets crate the guard:

```typescript
@Injectable({
	providedIn: 'root',
})
export class SeoContentGuard implements CanActivate {
	constructor(
		private readonly environmentVariablesService: EnvironmentVariablesService<EnvironmentVariable>,
		private readonly seoService: SeoService
	) {}

	public canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
		const titleTranslateKey = route.data[RouteData.Title];
		const descriptionTranslateKey = route.data[RouteData.Description];
		const url = `${this.environmentVariablesService.get(EnvironmentVariable.APP_URL)}${state.url}`;
		const image = route.data[RouteData.Image];
		const imageWidth = route.data[RouteData.ImageWidth];
		const imageHeight = route.data[RouteData.ImageHeight];
		const imageType = route.data[RouteData.ImageType];
		const type = route.data[RouteData.Type];
		const twitterCard = route.data[RouteData.TwitterCard];

		const translatedSeoContent$ = this.seoService.createTranslatedSeoContent(
			titleTranslateKey,
			descriptionTranslateKey
		);

		return translatedSeoContent$.pipe(
			take(1),
			map(({ title, description, siteName }) => {
				this.seoService.setSEO({
					title,
					description,
					siteName,
					url,
					image,
					imageWidth,
					imageHeight,
					imageType,
					type,
					twitterCard,
				});

				return true;
			})
		);
	}
}

```

If your application is not using translations just skip `createTranslatedSeoContent` function and provide content directly.

## Canonical url service

```typescript
@Injectable({
	providedIn: 'root',
})
export class CanonicalUrlService {
	private readonly renderer: Renderer2;

	constructor(
		private readonly environmentVariablesService: EnvironmentVariablesService<EnvironmentVariable>,
		private readonly rendererFactory: RendererFactory2,
		@Inject(DOCUMENT) private readonly document: Document
	) {
		this.renderer = rendererFactory.createRenderer(null, null);
	}

	public createCanonicalUrl(url: string): void {
		const link = this.renderer.createElement('link');

		link.setAttribute('rel', 'canonical');

		const existingLink = this.document.head.querySelectorAll('[rel="canonical"]')[0];

		existingLink
			? this.document.head.replaceChild(link, existingLink)
			: this.renderer.appendChild(this.document.head, link);

		link.setAttribute(
			'href',
			urlJoin(this.environmentVariablesService.get(EnvironmentVariable.APP_URL), cleanUrl(url)) // Provide url without params
		);
	}
}
```

Use this service to set canonical url. Import and use it in SEO guard or some other guard.
