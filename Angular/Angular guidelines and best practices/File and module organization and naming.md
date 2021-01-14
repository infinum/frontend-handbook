If you have created the app using Angular CLI, there is already some structure in place. Building on that structure, you will probably want to add some more directories to help you organize your code.

# Keep the code organized by domain

As the project grows in size, it gets harder and harder to maintain a good file organization. For small projects, it might make sense to group all components in `src/app/components` and all models into `src/app/models`, etc. However, once the project gets large enough, this can get hard to navigate through and maintain. For this reason, we recommend grouping the files not by type, but by domain. This means, for example, that you might have one directory that has a mix of services, models, and interfaces that are all connected to a common domain entity.

## Services and models

Services should be placed in `src/app/services`.

Try to keep the code grouped by domain. For example, if you have a `Post` class, `IPost` interface, and a `PostService` used for managing the blog posts, you can have all these files inside one directory like so:

- `src/app/services/post/`
  - `post.service.ts`
  - `post.service.spec.ts`
  - `post.interface.ts`
  - `post.model.ts`
  - `post.model.spec.ts`

## Pages

Container components that are loaded directly via routing (by either `component` or `loadChildren`) should be placed in the `src/app/pages` directory.

## Shared components

If a component is used in multiple places, it should be extracted to the nearest common ancestor's components directory. Here are some examples:

- Post component is used in `posts-container` (list of posts) and `users-container` (user's list of posts) top-level routes:
  - `src/app/`
    - `components/`
      - `post/`
    - `pages/`
      - `users-container/`
      - `posts-container/`
- `PostsCount` component is used only inside the `users-container` top-level route, but it is used in both `user-details-container` and `users-list-container` sub-routes.
  - `src/app/`
    - `pages/`
      - `users-container/`
        - `components/`
          - `posts-count/`
        - `user-details-container/`
        - `users-list-container/`

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

```typescript
export const httpStatusCodeData = { [HttpStatus.OK_200]: { translationKey: 'http.success' } }
```

If the enum data is a bit more complex, defining an interface for it can be useful.

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

Global styles should be placed in the `src/app/styles` dir. The styles dir has a very similar structure as that described in the [SASS Styleguide](/books/frontend/SASS%20Styleguide/File%20organization), so please check it out.

## NgxFormObject

If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s, and validators in `src/app/forms`. If you have form components that are reused throughout the app, put them in `src/app/components/forms`.
