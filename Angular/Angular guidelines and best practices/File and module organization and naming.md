If you have created the app using Angular CLI, there is already some clear structure in place. Building on that structure, you will probably want to add some more dirs to help you organize your code. Here are some guidelines:

- Try to split up your application into modules which you can reuse. If, for example, you have to handle lazy loading of all images in the app, create an `Image` module and component in `app/components` and use it in other modules/components whenever you have to show an image. Export the declared component and import/provide dependencies which your component uses.

- Container components that are loaded directly via routing (by either `component` or `loadChildren`) should be placed in the `app/pages` dir. If a component is loaded because you have used it in a template, that component should be in `app/components`.
  - One little exception here: if there is a very specific component tied to some routing submodule which is used only in that submodule, it is OK to leave it in that submodule (but move it to `app/components` as soon as it is used in another routing submodule).

- Store all your animations in `app/animations/` and import them for use in component decorators. If you want to have some animation properties, such as a duration configurable, you can export the "animation factory" function which returns the actual `AnimationTriggerMetadata`.

- Classes that might be used throughout the application (e.g., generic `Dictionary` class) should be in `app/classes`.

- Entity models (`User`, `Post`, etc.) should be in `app/models`.

- If you are using some JS libs which do not have typings, add your own custom typing in `app/custom-typings`.

- Whenever you have an attribute whose value can be some value from a set of values, make an enum and store that enum in `app/enums`. A good example is an enum for HTTP status codes. If you need some metadata for the enum, create an enum data file which exports what is basically a dictionary with enum values used as keys, and use whatever you need for values. One such example would be: `export const httpStatusCodeData = { [HttpStatus.OK_200]: { translationKey: 'http.success' } }`. If the enum data is a bit more complex, defining an interface for it can be useful.

- If you are using `ngx-form-object`, place `FormObject`s, `FromStore`s, and validators in `app/forms`. If you have form components that are reused throughout the app, put them in `app/components/forms`.

- Put all route guards in `app/guards`.

- If you need some helper functions (like a rounding function which actually rounds correctly *cough* Math.round *cough* floating point *cough*), put the helper in `app/helpers` in its own dir. Make sure to write tests for it and simply export it in `app/helpers/index.ts`.

- Save all your common interface files to `app/interfaces`.

- If you have some custom pipes for use in templates, create their own dirs in `app/pipes`, similar to helpers. Declare, export, and provide the pipes in `app/pipes/pipes.module.ts`.

- Services should be placed in `app/services`, each service in its own dir.

- If you have any `NgModule`s which are not components (e.g., a module which imports all specific material modules that you need), put them in `app/shared-modules`.

- TypeScript `type` declarations should be placed in `app/types`.

- As defined in angular-cli, assets placed in `assets` will be served statically.

- Global styles should be placed in the `app/styles` dir. The styles dir has a very similar structure as that described in the [SASS Styleguide](https://handbook.infinum.co/books/frontend/SASS%20Styleguide/File%20organization), so please check it out.

- All environment files are placed in `environments`.

When it all comes together, your src folder might look something like this:
*(please note the postfixes in filenames, such as .guard, .interface, .model, etc.)*

- app/
  - animations/
      - fade.animations.ts
  - classes/
      - dictionary.ts
  - components/
      - header/
          - header.component.html
          - header.component.scss
          - header.component.spec.ts
          - header.component.ts
          - header.module.ts
      - modals/
      - forms/
  - custom-typings/
      - some-js-lib.d.ts
  - enums/
      - http-response-code.enum.ts
      - http-response-code-data.enum.ts
      - user-status.enum.ts
  - forms/
      - user/
          - user.form-object.ts
          - user.form-store.ts
  - guards/
      - authentication/
          - authentication.guard.ts
          - authentication.guard.spec.ts
  - helpers/
      - round/
          - round.helper.ts
          - round.helper.spec.ts
      - index.ts
  - interfaces/
      - user-data.interface.ts
  - models/
      - user.model.ts
  - pages/
      - home-container/
      - login-container/
      - users-container/
  - pipes/
      - order-by/
          - order-by.pipe.ts
          - order-by.pipe.spec.ts
      - pipes.module.ts
  - services/
      - user/
          - user.service.ts
          - user.service.spec.ts
  - shared-modules/
      - material/
          - material.module.ts
  - types/
      - sorting-function.type.ts
  - styles/
      - components/
          - _title.scss
      - overrides/
          - _button.scss
          - _card.scss
          - _media-blender.scss
      - themes/
          - _main-theme.scss
      - utils/
          - _colors.scss
          - _fonts.scss
          - _mixins.scss
          - _shared-variables.scss
          - _z-indices.scss
      - _core.scss
      - main.scss
- assets/
    - fonts/
        - CustomFont.eot
        - CustomFont.ttf
        - CustomFont.woff
        - CustomFont.woff2
    - images/
        - icons/
            - app-icon.svg
- environments/
    - environment.base.ts
    - environment.prod.ts
    - environment.staging.ts
    - environment.ts
