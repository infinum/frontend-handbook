If you have created the app using Angular CLI, there is already some structure in place. Building on that structure, you will probably want to add some more directories to help you organize your code.

# Keep the code organized by domain

As the project grows in size, it gets harder and harder to maintain a good file organization. For small projects, it might make sense to group all components in `src/app/components` and all models into `src/app/models`, etc. However, once the project gets large enough, this can get hard to navigate through and maintain. For this reason, we recommend grouping the files not by type, but by feature/domain. This means, for example, that you might have one directory that has a mix of services, models, and interfaces that are all connected to a common feature/domain.

## Services and models

Services should be placed in `src/app/services`.

Try to keep the code grouped by domain. For example, if you have a `Post` class, `IPost` interface, and a `PostService` used for managing the blog posts, you can have all these files inside one directory like so:

```

src/app/services/post/
├── post.service.ts
├── post.service.spec.ts
├── post.testing.service.ts
├── post.interface.ts
├── post.model.ts
└── post.model.spec.ts
```

## Pages and shared components

Components that are loaded directly via routing (by either `component` or `loadChildren`) should be placed in the `src/app/pages` directory. If a component is used in multiple places, it should be extracted to the nearest common ancestor's components directory. If there are multiple levels of routing, the main `src/app` directory structure can be mirrored under some specific page directory. In other words, you can have components, services, pages, etc. directories inside some specific page directory. This helps separate the codebase on a feature/domain-basis.

As an example, please consider the following routing setup:

- `/` - homepage, redirects to `/posts`
- `/users` - list of users (renders a list of `user-avatar` components)
- `/users/:id` - user details view (renders `user-avatar` and a list of `post-overview` components)
- `/posts` - list of posts (renders a list of `post-overview` and `user-avatar` components)
- `/posts/new` - new post form (renders `post-form`)
- `/posts/:id` - post details view (renders `post-overview`, `user-avatar` and `post-content` components)
- `/posts/:id/edit` - edit post form (renders `post-form`)

Directory structure for such an app should look like this:

```
src/app/
components/ # these components are used in both posts and users feature pages
│   ├── post-overview/ # renders basic info about a post (without full content)
│   └── user-avatar/ # renders username and profile image
services/ # these services are used in both posts and users feature pages
│   ├── auth/ # session management, not important for this example
│   ├── user/
│   │   ├── user.service.ts # has methods for data fetching
│   │   ├── user.service.spec.ts
│   │   ├── user.testing.service.ts
│   │   ├── user.interface.ts
│   │   └── user.model.ts
│   └── post/ # basically the same as user.service
pages/
│   ├── users/
│   │   ├── pages/
│   │   │   ├── users-index/
│   │   │   │   ├── components/
│   │   │   │   │   └── users-list/ # renders a list of user-avatar components
│   │   │   │   ├── users-index.component.ts # renders some header/footer and users-list component
│   │   │   └── user-index/
│   │   │       ├── components/
│   │   │       │   └── user-details/ # renders user-avatar component, additional user info and a list of user's posts (post-overview)
│   │   │       ├── user-index.component.ts # renders some header/footer and users-details component
│   └── posts/
│       ├── components/ # these components are used only in posts feature/domain page
│       │   └── post-form/ # form used for both editing and creation
│       ├── services/
│       │   └── social-sharing/ # service that is used only inside posts page
│       ├── pages/
│       │   ├── posts-index/
│       │   │   ├── components/
│       │   │   │   └── posts-list/ # renders post-overview and user-avatar component for each post
│       │   │   ├── posts-index.component.ts # renders some header/footer and posts-list component
│       │   ├── post-index/
│       │   │   ├── components/
│       │   │   │   └── post-details/ # renders post-overview and user-avatar components and renders the post content
│       │   │   ├── post-index.component.ts # renders some header/footer and posts-details component
│       │   ├── new-post/
│       │   │   ├── new-post.component.ts # renders some header/footer and post-form component
│       │   └── edit-post/
│       │       ├── edit-post.component.ts # renders some header/footer and post-form component
```

## Animations

Store all your animations in `src/app/animations/` and import them for use in component decorators. If you want to have some animation properties customizable, such as a duration configurable, you can export the "animation factory" function which returns the actual `AnimationTriggerMetadata`. Animations are usually quite generic and there is usually not that many of them so it's ok to keep them in the global `animations` directory.

## Custom classes

Classes that might be used throughout the application (e.g., generic `Dictionary` class) should be in `src/app/classes`. If a class is related to a particular domain entity, it can be placed alongside other files related to that same entity.

## Custom typings

If you are using some JS libs which do not have typings or you want to extend some native typings with some custom properties, add your own custom typings in `src/app/custom-types.d.ts`. Example `custom-types.d.ts` file:

```ts
interface Window { // extend window with `env` object
	env?: Record<string, string>;
}

declare module 'json-to-pretty-yaml' { // add typings for vanilla JS lib
	function stringify(o: any): string;
}
```

## Enumerations

Similar as with services and models, you can place enums alongside other files that are related to the same domain entity.

If an enum is used across multiple top-level modules, you can place the enum in `src/app/enums`. A good example is an enum for HTTP status codes.

If you need some metadata for the enum, create an enum data file which exports what is basically a dictionary with enum values used as keys, and use whatever you need for values. One such example would be:

`src/app/interfaces/translatable-enum.interface.ts`

```typescript
export interface ITranslatableEnum {
  translationKey: string;
}
```

`src/app/enums/http-status.enum.ts`

```typescript
export enum HttpStatus {
  200_OK: 200,
  ...
}

export const httpStatusCodeData: Record<HttpStatus, ITranslatableEnum> = {
  [HttpStatus.200_OK]: {
    translationKey: 'http.success'
  }
}
```

## Guards

Put all generic route guards in `src/app/guards`.

## Helpers

If you need some generic helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `src/app/helpers` in its own dir. Make sure to write tests for it and simply export it in `src/app/helpers/index.ts`.


## Pipes

If you have some custom generic pipes for use in templates, create their own dirs in `src/app/pipes`.

If a pipe is very specific and tied to a particular component, create it alongside that component instead of in the global pipes directory.

## Shared modules

If you have `NgModule`s which are not components (e.g., a module which imports all specific material modules that you need), put them in `src/app/shared-modules`. Keep in mind that having large shared modules is usually unwanted because it makes dependency management hard in the case where shared modules import many other modules, avoid it if possible.

One of the remaining use cases for modules is when you have a group of components that are meant to be used together. In such cases, you can create a module to export them. This way, to use those components, you only need one import instead of multiple.

## Custom types

Generic TypeScript `type` declarations should be placed in `src/app/types`. If the `type` is very specific and tied to a certain component/service/model, it is ok to place it alongside that component/service/model.

## Assets, global styles, and partials

As defined by the `angular-cli`, assets placed in `src/assets` will be served statically.

Global styles should be placed in the `src/app/styles` dir. The styles dir has a very similar structure as that described in the [SASS Styleguide](https://infinum.com/handbook/books/frontend/sass-styleguide/file-organization), so please check it out.

## NgxFormObject

If you are using [ngx-form-object](https://infinum.github.io/ngx-form-object/), place `FormObject`s, `FromStore`s, and validators in `src/app/forms`. If you have form components that are reused throughout the app, put them in `src/app/components/forms`.
