> There are only two hard things in Computer Science: **cache invalidation** and naming things.
>
> — PHIL KARLTON

After the application is built and deployed, there comes a question of caching all the various files that are required by the app.

Application build files in `dist` directory include:

- Fingerprinted JS and CSS chunks
- Copied assets directory
  - Individual asset files are _NOT_ fingerprinted
    - There are some edge cases where duplicate fingerprinted files could be generated, more on that in the following chapter
- index.html

## Paths to assets

### Too long, didn't read

Here is a quick summary of how you should define paths to assets in various use cases:

| Use case | Example |
| - | - |
| HTML / images | `<img src="./assets/…">` |
| XHR / fetch | `http.get('./assets/…")` |
| CSS `url()` | `url("^assets/…")` |

Keep reading for a detailed explanation of how to define paths to assets.

### Base href

Angular documentation about deploying applications contains [The base tag](https://angular.io/guide/deployment#the-base-tag) chapter, describing how hyperlinking works in conjunction with the base href tag. There is another chapter in the Routing subsection - [HTML5 URLs and the `<base href>`](https://angular.io/guide/router#html5-urls-and-the--base-href).

### Loading assets via HTML

To understand how `base` `href` affects assets loading, we will take a look at an example of image loading via the `<img>` element.

When developing locally, the application is hosted by the Webpack DevServer on `http://localhost:4200` and the base href is set to `/`. This means that when, for example, you load an image from the `assets` directory via `<img src="/assets/logo.svg">`, the image will be loaded from `http://localhost:4200/assets/logo.svg`.

A `src` path that is relative to the current component's source file will also work. For example, if you are working on `src/app/components/app.component.html` and you want to load the image that is in `src/assets/logo.svg` you can do it like so: `<img src="../../assets/logo.svg">` and it will work. Why and how this works might not be immediately clear. One might think that this works because it makes sense when looking at the `logo.svg` path relative to `AppComponent` in the source code directory structure. However, when looking at how the application is built, you realize that the source file directory structure (excluding the assets folder) does not really make any impact on the final build. `AppComponent` could be placed in any other directory and things would still work. Developers sometimes write such relative `src` paths because it's what the IDE/editor commonly offers as autocomplete options and that is expected - the autocompletion tools know only about the local source file structure and they are not aware of how the application is served.

The image with `src="../../assets/logo.svg"` will get loaded, but not because of the source file structure and the path relation between `app.component.html` and `logo.svg` files. It would also get loaded if the src was `../../../../../../../assets/logo.svg` or some even deeper relative path. At a glance, it is not completely clear why this would work. If we look at `src/app/components/app.component.html` and `../../../../../../../assets/logo.svg`, it is clear that this `logo.svg` file does not exist for the given path that is relative to the `app.component.html` file, and yet the image gets loaded. The reason why this works is because when working with relative paths the `img` `src` tag is relative to the `href` value from the `base` tag. In development, `base` `href` will be `/` and the `../../../../../../../assets/logo.svg` relative path will be resolved to `/assets/logo.svg` and the image will be loaded from `http://localhost:4200/assets/logo.svg`. This is why any amount of relative segments (`../`) will work and why `./assets/logo.svg`, `assets/logo.svg`, and `/assets/logo.svg` will all work just the same.

This might lead to the conclusion that it doesn't really matter if relative imports are used if it works. Not quite - there is an edge case where deep relative paths will not work, and that is if the `base` `href` is not `/`. This can be the case when the backend and the frontend are served from the same domain, but under different paths. For example, `https://app.com/web` and `https://app.com/api`. In such cases, the Angular application has to be build with `baseHref` and `deployUrl` builder options set to `/web/` (it is `/` by default). When running the local development server, you must also use `--serve-path=/web/`. It is important to include a trailing `/` for all of the options or it might not work correctly. Now, the relative path to logo.svg will be resolved differently, because it is no longer resolved relative to `base` `href` that is `/`, it is now resolved relative to `/web/`. When resolving `../../assets/logo.svg` relatively to `/web/`, you get `/assets/logo.svg` and the image will get loaded from `http://localhost:4200/assets/logo.svg` and this will not work. The image is now available on `http://localhost:4200/web/assets/logo.svg` (or `https://app.com/web/assets/logo.svg` in production).

Here is a quick table with a summary of what works and what does not work for different cases:

| `base` `href` | `<img>` `src` path | Resolved path | Actual URL on which the image is available | Works? |
| - | - | - | - | - |
| `/` | `../../assets/logo.svg` | `/assets/logo.svg` | `https://app.com/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/` | `../../../../../../../assets/logo.svg` | `/assets/logo.svg` | `https://app.com/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/` | `./assets/logo.svg` | `/assets/logo.svg` | `https://app.com/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/` | `/assets/logo.svg` | `/assets/logo.svg` | `https://app.com/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/` | `assets/logo.svg` | `/assets/logo.svg` | `https://app.com/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/web/` | `../../assets/logo.svg` | `/assets/logo.svg` | `https://app.com/web/assets/logo.svg` | <span style="color:red;">⨯</span> |
| `/web/` | `../../../../../../../assets/logo.svg` | `/assets/logo.svg` | `https://app.com/web/assets/logo.svg` | <span style="color:red;">⨯</span> |
| `/web/` | `./assets/logo.svg` | `/web/assets/logo.svg` | `https://app.com/web/assets/logo.svg` | <span style="color:green;">✔</span> |
| `/web/` | `/assets/logo.svg` | `/assets/logo.svg` | `https://app.com/web/assets/logo.svg` | <span style="color:red;">⨯</span> |
| `/web/` | `assets/logo.svg` | `/web/assets/logo.svg` | `https://app.com/web/assets/logo.svg` | <span style="color:green;">✔</span> |

Notes:

- All `<img>` `src` paths listed above are relative paths, but paths that start with `/` will ignore `base` `href` and resolve relative to the domain ([w3c docs](https://www.w3schools.com/tags/att_img_src.asp)). All other relative paths will take into account `base` `href` when resolving the path.

#### Conclusion

In summary, you must use `./assets/…` or `assets/…` when defining paths for assets loading via HTML in order to make it work in all the cases. We recommend using `./assets/…`.

### Loading assets via JS

The same rules apply when fetching assets via `fetch` or `XHR` because both of these APIs respect the `base` `href` value when resolving relative paths. For example, when using `transloco` for fetching translation files, you must use ```http.get<Translation>(`./assets/i18n/${lang}.json`)```. `assets/…` will also work, but `/assets` will not - just like with images.

#### Conclusion

Just as for images, the same conclusion applies for defining relative path when using `fetch`, `XHR`, `src`, and `href` - use `./assets/…`.

### Loading assets via SCSS

Loading images from the `assets` directory via the `url()` function is a bit different, because the SCSS transpiler processes SCSS with `postcss-loader` and the loader will throw an error if it can not find the file if it is defined by a relative URL in the `url()` function.

The processing that the compiler does on images (and some other assets) also means that they will be fingerprinted and, unless extra precautions are made, there will be duplicates in the final bundle - 1 original copy in `dist/assets` (e.g. for use in templates with `img` `src`) and 1 fingerprinted copy in `dist/` (e.g. for use in CSS with `url()`). This is [an issue](https://github.com/angular/angular-cli/issues/6599) with Angular CLI and it is not yet clear what the best solution is to avoid duplicate assets for both `/` and custom `base` `href` values. There are currently some hacky solutions, none of which are ideal, as described in [this StackOverflow thread](https://stackoverflow.com/a/62619147) ([+ another Angular issue on the topic](https://github.com/angular/angular-cli/issues/18013#issuecomment-649373940)).

If we wanted to set the `logo.png` image as `background-image` using `url()`, relative path is now relative with respect to the source file structure because it is processed by `postcss`. Transpilation will fail if the relative path leads to a file that does not exist. Absolute paths do not get processed and will always transpile. Here is an overview of what works and what does not work when loading an image (`src/assets/logo.png`) as `background-image` via `url()` from `src/app/app.component.scss`:

| `base` `href` | Path in `url()` in source SCSS | Path type | Path in `url()` in transpiled CSS | Works? | Duplicate file in `/dist`? |
| - | - | - | - | - | - |
| `/web/` | `^assets/logo.png` | Special | `assets/logo.png` | <span style="color:green;">✔</span> | - |
| `/` | `^assets/logo.png` | Special | `assets/logo.png` | <span style="color:green;">✔</span> | - |
| `/web/` | `~/src/assets/logo.png` | Relative | `/web/logo.d808b08ce47c5d0c53cd.png` | <span style="color:green;">✔</span> | `/logo.d808b08ce47c5d0c53cd.png` |
| `/web/` | `../assets/logo.png` | Relative | `/web/logo.d808b08ce47c5d0c53cd.png` | <span style="color:green;">✔</span> | `/logo.d808b08ce47c5d0c53cd.png` |
| `/web/` | `../../assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `../../../../../../../assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `./assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `~assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `~/assets/logo.png` | Relative | _does not transpile_ | <span style="color:red;">⨯</span> | - |
| `/web/` | `/assets/logo.png` | Absolute | `/assets/logo.png` | <span style="color:red;">⨯</span> | - |
| `/` | `/assets/logo.png` | Absolute | `/assets/logo.png` | <span style="color:green;">✔</span> | - |
| `/web/` | `/web/assets/logo.png` | Absolute | `/web/assets/logo.png` | <span style="color:green;">✔</span> | - |

Notes:

- Same rules will apply to any asset loading via `url()` (e.g. fonts).
- `d808b08ce47c5d0c53cd` is an example of a hash, different values will appear in reality.
- In certain cases, the logo image will be present two times in the final bundle:
  - `/dist/logo.d808b08ce47c5d0c53cd.png` and `/dist/assets/logo.png`
- `assets/logo.png` + `<base href="/web/">` would work and it would load the image from `https://app.com/web/assets/logo.png` if SCSS transpilation passed, but it does not. If you set this URL manually in devtools, you will notice that it does indeed work and takes into account the `base` `href` value.
- The last two entries from the table do work, but require source code changes if `base` `href` is changed.

#### Conclusion

Conclusion here is a bit different than for `<img>` and `fetch`/`XHR` paths. For SCSS URLs use `^assets/…` in order to avoid assets file duplication and make it work for different `base` `href` values without the need for modifying URLs in source files. Keep in mind that this solution could easily stop working if there are some breaking changes in Webpack or the Angular CLI. Handbook will be updated with a better solution once and if it is found.

## Caching

### Index file

The entry-point into a Single Page Application (SPA), like Angular, is the `index.html` file. During the build process, the source `src/index.html` is taken and updated with links to the generated JS and CSS chunks and placed into `dist/[project-name]/index.html`.

Fingerprinted links to JS and CSS chunks will be different for each build, and because of that the `index.html` file will be different as well. Because of this, it is important that the `index.html` file is not cached permanently on the client and that the client requests cache revalidation. If the client has a stale `index.html` file, it will try loading stale JS and CSS chunks as well. Those might or might not be in its cache, and depending on this the client might get a completely outdated app or a broken app that is missing some chunks.

**Caching Rule #1**: Do not cache the built `index.html` file. Revalidate it each time!

### JS and CSS chunks

JS and CSS chunks that are linked to from the `index.html` file are fingerprinted and because of that they can be cached indefinitely. If we create a new application build, the `index.html` file will be re-validated and the client will load new chunks.

**Caching Rule #2**: Cache fingerprinted JS and CSS files with a long expiration time!

### Static assets

The tricky part with serving Angular applications is caching the static asset files. Some frameworks, depending on the Webpack configuration and the way that assets are used, fingerprint all images and other static assets. In Angular, static assets, like images and custom fonts, are placed in the `src/assets/` directory. This directory gets copied without any modifications to the final application build directory - `dist/[project-name]/assets/`.

Files from the `assets/` directory do not get fingerprinted. Because of this, it is important to set up the correct caching mechanism for serving these files or else the client might be using stale files. There are two solutions:

1. Well, add fingerprinting to these files!
2. Utilize the transfer protocol caching mechanism

The first solution is actually a hack, just like fingerprinting JS and CSS chunks is a hack. Ideally, we want to rely on the transfer protocol to handle the caching, without the need for fingerprinting.

In the world of Web, application files are served via the HTTP protocol 99.99% of the time. It makes sense to use the built-in mechanisms offered by HTTP to handle caching of static assets. You can read more about the different caching mechanisms available in HTTP [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching).

For our use case of static assets serving, we must adopt cache re-validation using ETags. The short description of this method is that the Web server calculates the hash of the file being served and sends that hash to the client together with the file itself. The client can cache this file and when fetching the same file in the future, the client sends that hash value of the cached file in the request headers as well. Depending on whether the hashes match or not, the server can return 200 with the new version of the file or 304 without any payload.

This caching mechanism could also be applied to JS and CSS chunks in a way that would not require fingerprinting those files, but since Angular already fingerprints those files by default, we will leave it as-is.

_One small note_: if you use relative paths in SCSS, for example for a background image, then that image will get duplicated and fingerprinted. However, if you have an image element in your template, like `<img src="/assets/logo.png">`, the `logo.png` file will not be fingerprinted and you are at risk of the client using a stale version of the file! Please check out [Loading assets via SCSS](#loading-assets-via-scss) sub-chapters that covers this topic and explains how to avoid duplicated files.

**Caching Rule #3**: Implement cache re-validation using ETags for all static assets!

### Cache re-validation implementation

Depending on how the application files are served, implementation details for setting up HTTP caching will be a bit different.

#### Static file serving

If application files are served statically, the static file server must be configured to serve and cache asset files correctly.

Taking Nginx as an example of a static file server, the configuration file for the site should be updated to enable cache re-validation for the `assets/` directory:

```bash
    server {
        listen 80 default_server;
        root /var/www;
        location / {
            add_header Cache-Control "no-cache";
            try_files $uri $uri/ /index.html;
        }
    }
```

Options for the `Cache-Control` header can be a bit confusing. The name of the option `no-cache` suggests that there will be no caching, but that is not true. For more details, please check out this article about [Demistifying HTTP Caching](https://codeburst.io/demystifying-http-caching-7457c1e4eded?gi=2174675eaf73). A quick quote from the article:

>no-cache doesn’t mean “don’t cache”, it means it must revalidate with the server before using the cached resource.

#### Server-side rendering

If the application uses server-side rendering, then caching should be handled by the Node.js server.

When using Angular Universal, `server.ts` includes a piece of code in charge of serving the application client build files (this includes `index.html`, JS and CSS chunks, and files from `assets/`). This piece of code should be updated to force cache re-validation:

```typescript
server.get('*.*', express.static(distFolder, {
  etag: true,
  setHeaders: (res) => {
    res.set('Cache-Control', 'no-cache');
  }
}));
```

This change will actually force re-validation for `index.html`, JS and CSS chunks as well, not only the `assets/` directory.

Keep in mind that if there is a proxy in front of the Node.js SSR server, it should also be configured to handle these caching headers correctly.
