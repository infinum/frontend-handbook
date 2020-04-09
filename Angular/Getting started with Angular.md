![Angular](/img/angular.svg)

Angular is a big framework and there is a lot to cover when getting started. We have put together a short guide with links to resources which should help you.

Please read through this short getting started guide in its whole and then start opening the links on your journey to learn Angular 💪.

If you have a mentor guiding you through your learning process, you are encouraged to also discuss any topic with them.

## 1. Official documentation

[Official documentation](https://angular.io/docs) has a lot of information. There are some assumptions about prior knowledge required to understand the docs, so you should first get to know HTML, CSS, and modern JavaScript (+ TypeScript basics).

### 1.1. Tutorials

There are two tutorial-style sections in the official docs.

#### 1.1.2. [Getting Started with Angular: Your First App](https://angular.io/start)

This section covers all the steps you need to complete before you can start making your first app. It also covers the very basics of Angular's core features, including template syntax, component design, routing, dependency injection, working with forms and a simple deployment process.

This is a great resource for your first introduction to Angular and we recommend following all of the steps from this section. The tutorial provides a [starting point on StackBlitz](https://angular.io/generated/live-examples/getting-started-v0/stackblitz.html), an online integrated development environment. It is very important that you actually write code—do not just read or copy/paste it, write it.

#### 1.1.2. [Tour of Heroes App and Tutorial](https://angular.io/tutorial)

After you cover the basics, you can start with the famous Tour of Heroes App. This tutorial introduces some concepts and best practices which we use in everyday development. This time you will not use StackBlitz (online IDE), you will develop everything on your local machine. Just like before, it is important that you follow the tutorial and **write the code**.

Tour of Heroes tutorial includes links to various other sections of the documentation; for example, the [Dependency Injection guide](https://angular.io/guide/dependency-injection). While these links are important resources, we recommend you do not open those links quite yet because a lot of those resources are large documents and they might distract or overwhelm you. First finish the tutorial and then come back to those links.

### 1.2. Guides

After you finish the tutorial you will no doubt be left with questions. These guides,alongside your mentor, can help you find those answers.

Here are some of the most important guides:

- [Dependency Injection in Angular](https://angular.io/guide/dependency-injection)
- [Routing & Navigation](https://angular.io/guide/router)
- [Animations](https://angular.io/guide/animations)
- [Testing](https://angular.io/guide/testing)
- [Observables](https://angular.io/guide/observables)
- [The RxJS library](https://angular.io/guide/rx-library)

Most of these are quite large documents. Nobody expects you to read through all of the content of all of these documents. But you should know they are there, so you can come back to them when you are unsure of how some things work. Even the most experienced developers use the documentation pages on a regular basis to remind themselves of some parts of the framework which they might not have used as extensively in a while so they need a refreshed.

## 2. Infinum [Angular guidelines and best practices](https://handbook.infinum.co/books/frontend/angular/angular-guidelines-and-best-practices) Handbook

The official docs are a great source of reference information, guides and tutorials. After you finish all the official tutorials, taking a look at our very own [Angular Handbook](https://handbook.infinum.co/books/frontend/angular/angular-guidelines-and-best-practices). At first we recommend you skim-read it on a high-level, just to get to know which topics are covered. Of course feel free to read it completely if you have the time.

The Handbook covers some of the best practices and formatting preferences to which we try and adhere when possible. This includes:

- Scaffolding
- File organization
- Tooling
- Editor and editor extensions recommendations
- RxJS basics
- Ways to conceptually organize components
- Handling multiple deployment environments
- Formatting, naming and best practices in Angular templates and TypeScript
- Dependency injection links
- Angular universal links
- Two-way binding
- Working with forms
- Testing

One of the topics which is usually not covered in detail in most online tutorials is testing. A large portion of the Handbook is about [testing](https://handbook.infinum.co/books/frontend/angular/angular-guidelines-and-best-practices#testing) and this should come in very handy once you start working on an actual project where it will be required to write tests.

## 3. Various online resources

Completing Tour of Heroes and reading some of the guides from the official documentation should teach you enough to start working on a real project. To advance your knowledge from this point forward, there are many great websites, YouTube channels and blogs you can follow. As you bump into parts of the framework where it is not clear to you what is going on, there is a good chance that a quick web search will lead you to one of these sources:

- [Angular University](https://angular-university.io/) has tons of PDFs and video content
- Angular In Depth [[Web](https://indepth.dev/angular/), [Medium](https://medium.com/angular-in-depth)] usually covers more advanced topics, so it is probably not a great first resource for beginners, but it will definitely be of great value when you have some very specific questions
- Subscription-based learning portals like [egghead.io](https://egghead.io/) have courses covering a wide range of topics
- [Academind](https://www.academind.com/) covers Angular, TypeScript, NodeJS, vanilla JS, RxJS, React, Vue and various other topics. Some content is available on [YouTube](https://www.youtube.com/channel/UCSJbGtTlrDami-tDGPUV9-w/featured), and some on [Udemy](https://www.udemy.com/courses/search/?src=ukw&q=academind%20angular)

Feel free to start a course or simply start following some blogs to keep on top of things.

## 4. Get to know RxJS

There are many concepts which Angular shares with React, Vue and other component-based JavaScript libraries/frameworks. However, when it comes to Angular, there is one big player in the game who is usually not present when using other frameworks, and that is RxJS.

Writing good quality Angular applications both in terms of performance and code readability and reusability often requires having a good grasp of how RxJS works and how it can be utilized to create reactive data streams.

Luckily there are some good and free online tutorials available. We recommend watching Acedemind's [Understanding RxJS](https://www.youtube.com/watch?v=T9wOu11uU6U&list=PL55RiY5tL51pHpagYcrN9ubNLVXF8rGVi) YouTube playlist. There is also a cool website called [RxJS Marbles](https://rxmarbles.com/) where you can explore many different RxJS operators and see what they do in a visual way. [Egghead.io](https://egghead.io/) has some great courses as well, for example [RxJS Beyond the Basics: Operators in Depth](https://egghead.io/courses/rxjs-beyond-the-basics-operators-in-depth) and [Build an Angular Instant Search Component](https://egghead.io/courses/build-an-angular-instant-search-component).

We also recommend reading the official documentation pages related to Observables and RxJS:

- [Observables](https://angular.io/guide/observables)
- [The RxJS library](https://angular.io/guide/rx-library)
- [Observables in Angular](https://angular.io/guide/observables-in-angular)
- [Practical observable usage](https://angular.io/guide/practical-observable-usage)
- [Observables compared to other techniques](https://angular.io/guide/comparing-observables)

When reading/watching online tutorials please note that there have been some breaking changes in RxJS version 6—some of the APIs have changed (mostly the way we use operators), and many tutorials that were created earlier are now using older versions of RxJS (v5 or older). That being said, not much has changed conceptually, just the way you write some RxJS functionalities, so those tutorials should still be worth checking out.
