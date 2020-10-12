# PROJECT SETUP

When starting a new React Native project there are two possible options / tools which can be used to set up your project.

 1. [Expo](https://expo.io/), which is generally used if you need quick & easy solution. Even though it can speed up the whole process of setup and development of RN app, it comes with set of options and [limitations](https://docs.expo.io/introduction/why-not-expo/) which can be a dealbreaker for your project needs.
 2. [React Native CLI](https://github.com/react-native-community/cli) represents official RN community tool for building and managing RN projects.


## Setting up the development environment ( React Native CLI)

[Official documentation](https://reactnative.dev/docs/environment-setup) contains everything needed to setup development environment depending on the OS you are using.

When creating a new RN project check CLI [commands](https://github.com/react-native-community/cli/blob/master/docs/commands.md) to get familiar with all options.

Generally, most of our projects use `typescript` and `npm` as package manager. This means that for creation & initialization of a new project u need to use command:

````sh
npx react-native init AwesomeProjectName --template react-native-template-typescript --npm
````

***Note:*** If previous command fails, it's possible that `eact-native-template-typescript` is deprecated or there are some major changes to CLI. In that case, please contact someone from Javascript team to update the chapter.

### Github repository
Since the creation of github repositories is limited to admins only, contact one of the Javascript team leads to create a repository for you.
Once you've got your repository make sure to update your `.gitignore`.
Don't forget to add `README.md` which should consist of:

 1. Application name and logo
 2. **Toc** (Table of Content)
 3. **Setup** - add basic description of how to initialize project and install dependencies
 4. **Development** - description of all commands and configurations needed to start developing and running the application on real device / simulator
 5. **Versioning manifest** - notes on semver versioning
 6. **Building** - how to build the app for both platforms
 7. **Notes** - project specifics (gotchas, known bugs in used libraries, todos etc.)

### Project structure

To get started with project structure please check our official [React handbook](https://infinum.com/handbook/books/frontend/javascript/react).
Since there are some semantic differences and domain specifics between React and React Native application some of the naming is different.
For example, in RN we don't have concept of pages. RN uses screens as it's top level views (building elements), therefore we will have `screens` directory instead of `pages`.
Also, navigation in RN represents more than a meere routing system. It makes the applications base and holds informations on the structure and it's configuration. Therefore, we use define `navigation` directory inside project root. Check [navigation chapter](todo) for detailed informations on this.

### Dependencies

Since RN uses the React ecosystem, we can reuse most of it's development dependencies like linters, prettier configurations etc.


## Platform specifics

### Android
Android uses Gradle as a build and dependency managment system. For that reason all android specifics configurations, like application version, minimal supported SDK, flavors etc. are configured inside application gradle file.
To get familiar with android (gradle) configuration please check our official [android handbook chapter](https://infinum.com/handbook/books/android/project-structure/gradle-build-system).

### iOS

More on iOS specifics and multiple environment setup can be found inside [iOS handbook chapter](https://infinum.com/handbook/books/ios/project-flow/custom-xcconfigs).

## Multiple environments setup

Before you start reading this chapter please check platform specific section above to get familiar with native platforms.
This setup should be used if you application has multiple env. in sense of environment variables, custom icons defined per env. and similar. One usecase: application has different staging and production API with each separate authentication keys, application icons, signin keys etc.

If this is the case you should use [react-native-build-config](https://github.com/ismaeldcom/react-native-build-config) library together with platform specific settings to setup your project.