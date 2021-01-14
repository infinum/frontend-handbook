> There are only two hard things in Computer Science: **cache invalidation** and naming things.
>
> — PHIL KARLTON

After the application is built and deployed, there comes a question of caching all the various files that are required by the app.

Application build files in `dist` directory include:

- fingerprinted JS and CSS chunks
- copied assets directory
  - individual assets files are _NOT_ fingerprinted
- index.html file

## Index file

The entry-point into a Single Page Application (SPA), like Angular, is the `index.html` file. During the build process, the source `src/index.html` is taken and updated with links to the generated JS and CSS chunks and placed into `dist/[project-name]/index.html`.

Fingerprinted links to JS and CSS chunks will be different for each build, and the `index.html` file will be different as well. Because of this, it is important that the `index.html` file is not cached permanently on the client and that the client requests cache revalidation. If the client has a stale `index.html` file, it will try loading stale JS and CSS chunks as well. Those might or might not be in its cache, and depending on this the client might get a completely outdated app or a broken app that is missing some chunks.

**Rule #1**: Do not cache the built `index.html` file. Revalidate it each time!

## JS and CSS chunks

JS and CSS chunks that are linked to from the `index.html` file are fingerprinted and because of that they can be cached indefinitely. If we create a new application build, the `index.html` file will be re-validated and the client will load new chunks.

**Rule #2**: Cache fingerprinted JS and CSS files with a long expiration time!

## Static assets

The tricky part with serving Angular applications is caching the static assets files. Some frameworks, depending on Webpack configuration and the way that the assets are used, fingerprint all the images and other static assets. In Angular, static assets like images and custom fonts are placed in the `src/assets/` directory. This directory gets copied without any modifications to the final application build directory - `dist/[project-name]/assets/`.

The files from the `assets/` directory do not get fingerprinted. Because of this, it is important to set up the correct caching mechanism for serving these files or else the client might be using stale files. There are two solutions:

1. Well, add fingerprinting to these files!
2. Utilize the transfer protocol caching mechanism

The first solution is actually a hack, just like fingerprinting JS and CSS chunks is a hack. Ideally, we want to rely on the transfer protocol to handle the caching, without the need for fingerprinting.

In the world of Web, application files are served via the HTTP protocol 99.99% of the time. It makes sense to use the built-in mechanisms offered by HTTP to handle caching of static assets. You can read more about the different caching mechanisms available in HTTP [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching).

For our use case of static assets serving, we must adopt cache re-validation using ETags. The short description of this method is that the Web server calculates the hash of the file being served and sends that hash to the client together with the file itself. The client can cache this file and when fetching the same file in the future, the client sends that hash value of the cached file in the request headers as well. Depending on whether the hashes match or not, the server can return 200 with new version of the file or 304 without any payload.

This caching mechanism could also be applied to JS and CSS chunks in a way that would not require fingerprinting those files, but since Angular already fingerprints those files, we will leave it as-is.

_One small note_: if you use relative paths in SCSS, for example for a background image, then that image will get fingerprinted. However, if you have an image element in your template, like `<img src="/assets/logo.png">`, the `logo.png` file will not be fingerprinted and you are at risk of the client using a stale version of the file!

**Rule #3**: Implement cache re-validation using ETags for all static assets!

## Cache re-validation implementation

Depending on how the application files are served, implementation details for setting up HTTP caching will be a bit different.

We will cover some of the common HTTP serving technologies, including static file serving and Server-side rendering.

### Static file serving

If the application files are served statically, the static file server must be configured to serve and cache assets files correctly.

#### Nginx example

The Nginx configuration file for the site should be updated to enable cache re-validation for the `assets/` directory:

```bash
    server {
        listen 80 default_server;
        root /var/www;
        location / {
            add_header Surrogate-Control "public, max-age=3600";
            add_header Cache-Control "public, max-age=3600";
            try_files $uri $uri/ /index.html;
        }
        location /assets {
            add_header Surrogate-Control "public, max-age=0";
            add_header Cache-Control "public, max-age=0";
            add_header Cache-Control must-revalidate;
        }
    }
```

#### S3 + CloudFront example

TODO

### Server-side rendering

If the application uses Server-side rendering, then the caching should be handled by the Node.js server.

When using Angular Universal, `server.ts` includes a piece of code in charge of serving the application client build files (this includes `index.html`, JS and CSS chunks, and files from `assets/`). This piece of code should be updated to force cache re-validation:

```typescript
server.get('*.*', express.static(distFolder, {
  maxAge: '0',
  etag: true,
}));
```

This change will actually force re-validation for `index.html`, JS and CSS chunks as well, not only the `assets/` directory. This is ok, you could even disable output hashing of chunk files.

Keep in mind that if there is a proxy in front of the Node.js SSR server, it should also be configured to handle these caching headers correctly.
