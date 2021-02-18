## Introduction

Next.js supports some very cool features which can be used in different kinds of projects. For example, some projects with public pages can't be built with create-react-app (CRA) because it doesn't support server-side rendering, which makes SEO much harder. So, in projects that require SSR, a different stack must be used that would probably include something like Express on top of node, and custom Webpack and Babel configurations.

Also, any customization of Webpack or Babel inside CRA requires ejecting, which is a one-way operation. From that point onward, you are responsible for maintaining configuration files.

Multiple custom configuration files is something that we want to avoid.
Next.js solves above-stated problems and allows us to use a single stack for many different projects. Plus Next.js introduces some handy features, which come built-in:

- Code splitting
- SSR
- Page prefetching
- Dynamic import of modules
- Built-in Zero-Config TypeScript Support
- Integrated client-side routing
- Custom server with custom express routes
- static HTML export
