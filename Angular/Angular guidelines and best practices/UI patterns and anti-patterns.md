## Tabs

There are a couple of things to consider when it comes to tabs. Depending on the design:

- tabs can be a simple in-memory component that switches between different tab contents, or
- tabs can actually be navigation links that just happen to look like tabs

Angular Material has support for both cases, as explained in the [component overview](https://material.angular.io/components/tabs/overview#tabs-and-navigation) and demonstrated in the [examples page](https://material.angular.io/components/tabs/examples).

There is often a requirement where a user might be looking at some page on some tab, they click on some details link and they want to go back and see the same tab being active. This is trivial if tabs are just navigation links, whereas it is a lot more work to do it with tabs as UI components, especially if you have nested tabs.

A general rule of thumb is that if there is a single tab bar near the top of the page and it changes the whole content below it, it should probably be a tab navigation bar. The main benefit of this is that the user will be able to navigate back and forth between the tabs just like they would navigate between any other pages. Think of the tab bar as a collection of links that are have their active states styled in such a way that it looks like tabs - that's exactly what they are in this use case. If you want, you can also make tab links do the navigation by replacing the current URL, just like you can when using routerLink on a regular anchor element.

You can use tabs as navigation links even if you have nested tabs. It is quite simple as long as there are no sets of tabs that are siblings. If you do sibling tabs, you would probably have to employ auxiliary routes or consider using tabs as UI components instead of navigation links.

### Examples of different use-cases

Angular material website inadvertently has some great examples of when you should use tabs as navigation link and when as UI components.

If you look at the documentation page for any of the Material components, you will usually see three tabs at the very top of the page that allow you to switch from Overview, API and Examples pages.

![Angular tabs example #1](/img/angular_tabs_example_1.png)

These tabs are navigation links that change the route when you click on them.

On Examples page, you can usually check out multiple code snippets and for each of the snippets you have a tab component for switching between HTML, TS or CSS.

![Angular tabs example #2](/img/angular_tabs_example_2.png)

This is a great example of how the user could refresh the page on examples page and remain on examples page because the main tab bar is using navigation links. If this main tab bar was in-memory UI component, page refresh would load the first tab or some alternative method would have to be implemented to remember active tab.

For HTML, TS and CSS snippets it is not that important to preserve the active tab state on page reload, so those are just simple UI tabs and not navigation tabs. State preservation on reload could be implemented, but it would not be that straight-forwards because there are multiple sets of snippets on the examples page.

It is up to discussion between the developers, UI/UX designers and product owners to decide upon when the tabs should be navigation links and when they should be UI components.

Overall, we recommend defaulting to using tabs as navigation links when possible because that allows for best UX and simplest implementation when it comes to navigation and active tab state preservation.


### Extra resources

If you would like to learn more about UI and UX design around tabs, including some implementation details with vanilla JS, we highly recommend checking out [Adam Argyle](https://twitter.com/argyleink)'s [article](https://web.dev/building-a-tabs-component/) and [video](https://www.youtube.com/watch?v=mMBcHcvxuuA) on the topic.
