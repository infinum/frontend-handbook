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

Container components that are loaded directly via routing (by either `component` or `loadChildren`) should be placed in the `src/app/pages` directory. If there are multiple levels of routing, the main `src/app` directory structure can be mimicked under some specific page directory.

If a component is used in multiple places, it should be extracted to the nearest common ancestor's components directory. To illustrate this with an example, consider the following routing setup:

- `/` - homepage, redirects to /posts
- `/posts` - list of posts (renders a list of post-overview components)
- `/posts/new` - new post form (renders post-form)
- `/posts/:id/edit` - edit post form (renders post-form)
- `/posts/:id/view` - post details view (renders post-overview component and post-content components)
- `/users` - list of users (renders a list of user-avatar components)
- `/users/:id/view` - user details view (renders user-avatar and a list of post-overview components)

Directory structure for such an app should look like this:

```
src/app/
  components/
    post-overview/
  services/
    auth/
    user/
    post/
  pages/
    users/
      components/
        user-avatar/
      pages/
        users-list/
          components/
            users-list/ ???
          users-list.component.ts # renders some header and users-list component
          users-list-routing.module.ts # eagerly-loads users-list.component
          users-list.module.ts
        user-details/
      users-routing.module.ts # lazy-loads users-list.module and users-details.module
      users.module.ts # imports users-routing.module
    posts/
      components/
        post-content/
        posts-list/
        post-form/
      services/
        social-sharing/ # service that is used only inside posts module
      pages/
        posts-index
        new-post
      posts-routing.module.ts # lazy-loads
      posts.module.ts # imports posts-routing.module
  app-routing.module.ts # lazy-loads users.module and posts.module
  app.module.ts # imports app-routing.module
```

## Animations

Store all your animations in `src/app/animations/` and import them for use in component decorators. If you want to have some animation properties customizable, such as a duration configurable, you can export the "animation factory" function which returns the actual `AnimationTriggerMetadata`.

## Custom classes

Classes that might be used throughout the application (e.g., generic `Dictionary` class) should be in `src/app/classes`. If a class is related to a particular domain entity, it can be placed alongside other files related to that same entity.

## Custom typings

If you are using some JS libs which do not have typings, add your own custom typing in `src/app/custom-typings`.

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

If you have some custom generic pipes for use in templates, create their own dirs in `src/app/pipes`. Each pipe should have its own module that declares and exports the pipe.

If a pipe is very specific and tied to a particular component, create it alongside that component instead of in the global pipes directory.

## Shared modules

If you have `NgModule`s which are not components (e.g., a module which imports all specific material modules that you need), put them in `src/app/shared-modules`.

## Custom types

Generic TypeScript `type` declarations should be placed in `src/app/types`. If the `type` is very specific and tied to a certain component/service/model, it is ok to place it alongside that component/service/model.

## Assets, global styles, and partials

As defined by the `angular-cli`, assets placed in `src/assets` will be served statically.

Global styles should be placed in the `src/app/styles` dir. The styles dir has a very similar structure as that described in the [SASS Styleguide](https://infinum.com/handbook/books/frontend/sass-styleguide/file-organization), so please check it out.

## NgxFormObject

If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s, and validators in `src/app/forms`. If you have form components that are reused throughout the app, put them in `src/app/components/forms`.
