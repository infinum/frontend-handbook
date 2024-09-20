Next.js supports some very cool features which can be used in different kinds of projects. For example, some projects with public pages can't be built with create-react-app (CRA) because it doesn't support server-side rendering, which makes SEO much harder. So, in projects that require SSR, a different stack must be used that would probably include something like Express on top of node, and custom Webpack and Babel configurations.

Also, any customization of Webpack or Babel inside CRA requires ejecting which is a one-way operation, or using tools like [CRACO](https://craco.js.org/docs/) which are breaking the "guarantees" that CRA provides. From that point onward, you are responsible for maintaining configuration files.

Multiple custom configuration files is something that we want to avoid.
Next.js solves above-stated problems and allows us to use a single stack for many different projects. Also, Next.js introduces some handy features, which come built-in:

* Code splitting
* Hybrid Static & Server Rendering
* Automatic static optimization
* Page prefetching
* Dynamic import of modules
* Built-in Zero-Config TypeScript Support
* Automatic, internationalized routing
* Image optimization
* Middleware
* API Routes
* SEO Optimization
