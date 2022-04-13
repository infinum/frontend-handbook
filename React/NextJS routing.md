In React you need to add a routing library to handle the routing, and you need to add the route components and handle the routes
for the new pages manually.

In NextJS the routing is handled in a simpler way, and there are few built-in tools to help you with that.

The explanation for the routing and everything it goes with it is based on the official NextJS routing documentation. 
For more information, see the [official NextJS routing documentation](https://nextjs.org/docs/routing/introduction).


## The `pages` folder

The `pages` folder contains all your pages (basically the files with the extracted React components) where each page is 
also a route.

E.g. the `hello-world.tsx` file in the root of the `pages` folder will be a `your.domain/hello-world` route.
More examples:
```javascript
pages/about.tsx => your.domain/about
pages/ABOUT.tsx => your.domain/ABOUT

pages/company/index.tsx => your.comain/company
pages/company/info.tsx => your.domain/company/info
pages/company/contact-us.tsx => your.domain/company/contact-us
```

These are also called the predefined pages/routes.
Note how the first two examples are case-sensitive.

### Dynamic routes

Usually we also want to achieve something like this for our pages:
```
your.domain/article/New-React-18
your.domain/article/How-To-Handle-Routing-In-NextJS
```
and luckily NextJS provides an easy way to support the dynamic routes.

To create a page that is dynamic you need to wrap the brackets around the page name:
```
pages/article/[articleId].tsx
// or pages/article/[article].tsx
// or pages/article/[id].tsx
```
Both of the above examples will be a match for this page, so the code for that page will be executed. On the page we can
then extract the path by using a built-in hook `useRouter` (more details about the hook later).

```jsx
// pages/article/[articleId].tsx

import { useRouter } from 'next/router';

const ArticlePage = () => {
    const { query } = useRouter()

    if (!query.articleId) {
        return <div>loading...</div>
    }

    return <div>{query.articleId}</div>;
}

export default ArticlePage;
```

The example above shows a statically optimized page made by
[Automatic Static Optimization (ASG)](https://nextjs.org/docs/advanced-features/automatic-static-optimization),
so to get a path you need to use `useRouter`.
[Server Side Rendering (SSR)](https://nextjs.org/docs/basic-features/data-fetching/get-server-side-props) and 
[Static Site Generation (SSG)](https://nextjs.org/docs/basic-features/data-fetching/get-static-props) can also give you the information from `useRouter`,
but they also propagate that info (and more) via context.

In these two examples below for SSR and SSG you can see how the values for the query id (`articleId`) are available on the first render in both `context`
and `useRouter` properties. Try to use the `context` prop over `useRouter` because it will be faster (no hook/function call).
```tsx
// SSR pages/article/[articleId].tsx

import { useRouter } from 'next/router';

const ArticlePage = ({ articleId }) => {
    const { query } = useRouter()

    if (!query.articleId) {
        return <div>loading...</div>
    }

    return <>
        <div>{query.articleId}</div>
        <div>{articleId}</div>
    </>;
}

export const getServerSideProps = async (context) => {
    return {
        props: {
            articleId: context.query.articleId
        }
    };
}

export default ArticlePage;
```

```tsx
// SSG pages/article/[articleId].tsx

import { useRouter } from 'next/router';

const ArticlePage = ({ articleId }) => {
    const { query } = useRouter()

    if (!query.articleId) {
        return <div>loading...</div>
    }

    return <>
        <div>{query.articleId}</div>
        <div>{articleId}</div>
    </>;
}

export const getStaticPaths = async () => {
    return {
        paths: [],
        fallback: 'blocking'
    }
}

export const getStaticProps = async (context) => {
    return {
        props: {
            articleId: context.params.articleId
        }
    };
}

export default ArticlePage;
```

![SSR or SSG path from context](/img/nextjs/routing/NextJS_path_in_context.jpg)


Prefer to name the dynamic page something different from `id`, and although it works, in the network tab you can easier 
see which page is loaded (e.g. `companyId` and/or `employeeId`).

![Naming dynamic routes](/img/nextjs/routing/NextJS_dynamic_route_naming.jpg)

### Catch all routes

We will continue the example for the articles from the above.

You can extend the dynamic routes to allow nested routes such as:
```
your.domain/article/New-React-18
your.domain/article/New-React-18/Introduction
your.domain/article/New-React-18/About-Suspense
your.domain/article/New-React-18/About-Suspense/Jumping-In-The-Implementation
```

To do that, rename your dynamic route to have three dots (`...`) in the brackets, e.g. `[...articleId].tsx`.

Now the above code for the article page will have implementation like this:
```jsx
// pages/article/[...articleId].tsx

import { useRouter } from 'next/router';

const ArticlePage = () => {
    const { query } = useRouter()

    if (!query.articleId) {
        return <div>loading...</div>
    }

    // now an array of all the subpaths
    // for `your.domain/article/New-React-18/About-Suspense/Jumping-In-The-Implementation` it is:
    // ['New-React-18', 'About-Suspense', 'Jumping-In-The-Implementation']
    return <div>{query.articleId.reduce((previousId, currentId) => `${previousId}/${currentId}`)}</div>;
}

export default ArticlePage;
```

### Optional catch all routes

Only difference between catch all and optional catch all is that optional catch all will take into consideration the
route without the parameter (main route; in the above examples `your.domain/article`).

To have optional catch all routes add additional brackets around catch all route - `[[...articleId]].tsx`.

```jsx
// pages/article/[[...articleId]].tsx

import { useRouter } from 'next/router';

const ArticlePage = () => {
    const { query } = useRouter()

    // this is how we handle a route without the parameters (main route)
    if (!query.articleId) {
        return <div>Articles in general</div>
    }

    // now an array of all the subpaths
    // for `your.domain/article/New-React-18/About-Suspense/Jumping-In-The-Implementation` it is:
    // ['New-React-18', 'About-Suspense', 'Jumping-In-The-Implementation']
    return <div>{query.articleId.reduce((previousId, currentId) => `${previousId}/${currentId}`)}</div>;
}

export default ArticlePage;
```

Usually, your main route/page looks different from the other routes/pages, so it is better to have this setup:
```
pages/article/index.tsx => a general page describing the articles, or showing the paginated articles; using predefined route
pages/article/[id].tsx => a page for the single article; using catch all page
```

### The pages and the routes examples

Here are some pages and the routes they can support (otherwise the 404 page is shown):

```
// for a single generic page (article in general); usually it will be articles instead
pages/article.tsx
query = {}
- your.domain/article

// for a single specific page (article)
query = {}
pages/article/hello.tsx
- your.domain/article/hello

// for a dynamic articles with only 1 subpath
pages/article/[articleId].tsx
e.g. query = { articleId: 'hello' }
- your.domain/article/hello
- your.domain/article/hi
- your.domain/article/how-are-you

// for a dynamic articles with infinite number of subpaths
pages/article/[...articleId].tsx
e.g. query = { articleId: ['how-are-you'] } 
or query = { articleId: ['how-are-you', 'i-am-fine'] } 
and similar
- your.domain/article/hello
- your.domain/article/hi
- your.domain/article/how-are-you
- your.domain/article/how-are-you/i-am-fine
- your.domain/article/how-are-you/i-am-fine/how-about-you
- your.domain/article/...

// for a dynamic articles with infinite number of subpaths where the main route (article) is also included
pages/article/[[...articleId]].tsx
e.g. query = {} 
or query = { articleId: ['how-are-you'] } 
or query = { articleId: ['how-are-you', 'i-am-fine'] } 
and similar
- your.domain/article
- your.domain/article/hello
- your.domain/article/hi
- your.domain/article/how-are-you
- your.domain/article/how-are-you/i-am-fine
- your.domain/article/how-are-you/i-am-fine/how-about-you
- your.domain/article/...

// for a dynamic articles with mandatory id and slug
pages/article/[articleId]/[slugId].tsx
e.g. query = { articleId: 'how-are-you', slugId: 'i-am-fine' }
- your.domain/article/how-are-you/i-am-fine

// for a dynamic articles with mandatory id and multiple slugs
pages/article/[articleId]/[...slugId].tsx
e.g. query = { articleId: 'how-are-you', slugId: ['i-am-fine'] } 
or query = { articleId: 'how-are-you', slugId: ['i-am-fine', 'how-about-you'] }
- your.domain/article/how-are-you/i-am-fine
- your.domain/article/how-are-you/i-am-fine/how-about-you

// for a dynamic articles with infinite number of subpaths where the main route ([articleId]) is also included; only difference than above is the query value
pages/article/[articleId]/[[...slug]].tsx
e.g. query = { articleId: 'how-are-you' } 
or query = { articleId: 'how-are-you', slugId: ['i-am-fine'] } 
or query = { articleId: 'how-are-you', slugId: ['i-am-fine', 'how-about-you'] } 
and similar
- your.domain/article/hello
- your.domain/article/hi
- your.domain/article/how-are-you
- your.domain/article/how-are-you/i-am-fine
- your.domain/article/how-are-you/i-am-fine/how-about-you
- your.domain/article/...
```

### Route precedence

```
Predefined > Dynamic > Catch all || Optional catch all
```

Examples:
```
Pages:
pages/article/NextJS.tsx (A)
pages/article/[articleId].tsx (B)
pages/article/[...articleId].tsx or pages/article/[[...articleId]].ts (C) // you can not have both at the same time

Routes:
your.domain/article -> C
your.domain/article/NextJS -> A
your.domain/article/NextJS2 -> B
your.domain/article/NextJS/2 -> C
```

### Other information

- Statically optimized pages by ASG will render the page twice: without the route parameters initially, 
and then after the hydration with the route parameters.
Even if there is a double render, this does not mean you should opt-out of automatic static optimization since it still beats
both SSR and SSG (the network time for a JS chunk is smaller for ASG).


## useRouter

To get the route information out of the `router` object and to manipulate the routing the 
[`useRouter`](https://nextjs.org/docs/api-reference/next/router) hook is used.

`useRouter` properties can be seen in the documentation, and more not obvious information is: 

- `query` property will be an empty object at first when accessing the page built with ASG. 
This is explained above in the `Other information` section, but we want to emphasize this again. To solve it, you can 
return nothing (null) when `isReady` prop from the `useRouter` returns `false`.
- Do not use `router.push` if you are handling with the external urls. Use `window.location` instead.
- The pages navigated via `router.push` are not prefetched, and you need to manually add a `router.prefetch` function call.
Also, the prefetch works only in the production build.
- Do not use `withRouter` HOC in React with hooks.
- At the moment there is no clean NextJS solution to handle the cancellation of the routes and their transition, but you
can check this [Route cancellation dirty solution](https://github.com/vercel/next.js/discussions/32231#discussioncomment-2421565)


## Link

The `Link` component is used for the internal client-side links. It renders the content differently based on the context.
Altough for plain text it will render an `a` element, always have the `a` element as the outermost element in `Link` children. 

```tsx
// all render <a href="/">Home</a>
<Link href="/">Home</Link> // notice only plain text; try to avoid this and always put the <a> tag
<Link href="/"><a>Home</a></Link>
<Link href="/" passHref><CustomLink>Home</CustomLink></Link> // notice passHref here

// notice forwardRef here
const CustomLink = forwardRef(({ children, ...rest }, ref) => {
	return (
		<a ref={ref} {...rest}>
			{children}
		</a>
	);
});
```

Prefer to use the NextJS `Link` component outside the implementation:
```tsx
// BAD
const Card = (...) => {
    return (
        <Link href={href}>{...}</Link>
    );
}

<Card href="/company/123" {...} />


// GOOD
const Card = (...) => {
    return (
        ...
    );
}

<Link href="/company/123">
    <a>
        <Card>{...}</Card>
    </a>
</Link>
```

By default, the links created with the `Link` component will prefetch the pages. `Link` uses 
[`Intersection Observer API`](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
to detect when to load with a `rootMargin` of `200px`. At the moment observer values can not be changed (checked in NextJS 12.1.4).
You can turn off this feature if needed by setting `false` for the prefetch` prop.

For the API reference, please check the official docs for the [`Link` component](https://nextjs.org/docs/api-reference/next/link).


## Base path

You can change the base path if your application needs to be under a sub-path of a domain.
To do that, change the `basePath` property in the `next.config.js` file.


## 404 pages

Routing to non-existing pages is working out of the box. 
Only if there is a `SSR` or a `SSG` then you need to handle it manually. To implement that, you should return: 
```
notFound: true
```
when you detect a non-existing page, or if for some reason you do not want to show the page content (e.g. the user is
not authorized).
It is also important to handle revalidation 
(for [`SSG`](https://nextjs.org/docs/basic-features/data-fetching/incremental-static-regeneration)) otherwise the routes 
will always show the 404 page.

```tsx
// SSR pages/article/[articleId].tsx

const ArticlePage = ({ article }) => {
    return (
        <>
            <div>{article.id}</div>
            <div>{article.title}</div>
        </>
    );
};

export const getServerSideProps = async (context) => {
    const articleId = context.params.articleId;
    let article = null;

    try {
        article = await fetch(`https://my.api.com/article/${articleId}`)
            .then(response => response.json())
            .then(data => data);
    } catch (error) {
        return {
            notFound: true,
        };
    }

    if (article.id !== articleId) {
        return {
            notFound: true,
        };
    }

    return {
        props: {
            article,
        },
    };
};

export default ArticlePage;
```

```tsx
// SSG pages/article/[articleId].tsx

const ArticlePage = ({ article }) => {
    return (
        <>
            <div>{article.id}</div>
            <div>{article.title}</div>
        </>
    );
};

export const getStaticPaths = async () => {
    return {
        paths: [],
        fallback: 'blocking'
    }
};

export const getStaticProps = async (context) => {
    const articleId = context.params.articleId;
    let article = null;

    try {
        article = await fetch(`https://my.api.com/article/${articleId}`)
            .then(response => response.json())
            .then(data => data);
    } catch (error) {
        return {
            notFound: true,
            revalidate: true, // boolean or a number in MS; only for SSG
        };
    }

    if (article.id !== articleId) {
        return {
            notFound: true,
            revalidate: true, // boolean or a number in MS; only for SSG
        };
    }

    return {
        props: {
            article,
        },
    };
};

export default ArticlePage;
```

## Internationalized Routing

Next.js has a built-in way for internationalized routing.
The [documentation](https://nextjs.org/docs/advanced-features/i18n-routing) shows how it can be handled, so we will only
mention another feature that can be built on top of the internationalized routing.

That feature in [i18n translated routes](https://github.com/kristian240/next-js-with-i18n-routes) repo is showing how we can also translate the
paths, and not just have the locales in the routes when dealing with other locales.
Check it out if you need something like:
```
// a route to the employee but translated
your.domain/en/employee/123
your.domain/de/mitarbeiter/123
your.domain/jp/shokuin/123
```



