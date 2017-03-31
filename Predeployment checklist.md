*In order to achieve optimal performance and keep the customer happy it is advised to follow these guidelines. Before deploying, check if you have added these functionalities, and add them if you didn't.*

## Social media optimisation

### Facebook
To be able to display the website as an object in a social graph (that is, display it with more detail on social networks) the [Open Graph protocol](http://ogp.me/) will be used. Meta tags are used to describe the properties of an object of interest. Regarding Facebook, the four required properties for every page are:

* ``og:title``
  * The title of your object as it should appear within the graph, e.g., "The Rock".
* ``og:type``
  * The type of your object, e.g., "video.movie". Depending on the type you specify, other properties may also be required.
* ``og:image``
  * An image URL which should represent your object within the graph.
* ``og:url``
  * The canonical URL of your object that will be used as its permanent ID in the graph, e.g., "http://www.imdb.com/title/tt0117500/".

Along with these properties, it's encouraged to use others, if applicable. For a more detailed overview of the tags, please consult the [comprehensive list of Facebook tags](https://developers.facebook.com/docs/sharing/webmasters)

### Twitter

Similar to the the [Open Graph protocol](http://ogp.me/), [Twitter Cards](https://dev.twitter.com/cards/overview) are used to display the website on the Twitter social network. There are various types of cards, the most basic being the [Summary Card](https://dev.twitter.com/cards/types/summary). At least the summary card should be implemented. Of course, if it's possible, implement a more detailed one =).

## Icons and colors for the browser

###Icons
In order to add support for icons to the website, several meta tags must be added. As a result, the icon may show up in many places, including the browser tab, recent app switch, the new (or recently visited) tab page, and more. It's necessary to provide icon support for all of the major **browsers** as well as for most of the **mobile platforms**. A handy tool for the task is the [Real Favicon Generator](http://realfavicongenerator.net/).

### Colors
In accordance with the designers wishes, it's possible to color browsers and elements of the platform. For further info, check out what [Google Developers](https://developers.google.com/web/fundamentals/design-and-ui/browser-customization/theme-color?hl=en) have to say about it.

## Google analytics

To start collecting basic data from a website it's necessary to set up Google analytics. If the client hasn't done it by himself already, set it up in his stead and transfer the ownership later on.

In order to set up analytics for a given page, please consult the following [Guide](https://support.google.com/analytics/answer/1008015?hl=en).

## Google Search Console

Expanding on Google analytics, establish Search Console tools functionality for the site to be deployed. After setting them up, remember to add the client as the **owner** (not the admin). Make sure that we remain the owners of the site.

If you need help setting them up, [here you go young sir or madam](https://support.google.com/webmasters/answer/6001104?hl=en).

_Side note: it's not possible to verify whether the tools have been set up properly before actual deployment because Webmaster tools require an actual link pointing to the website. Immediately after deployment, link everything up and verify it's functioning correctly._

## Sitemap

In order to boost indexing efficiency, create a sitemap file. Given the fact that we currently prefer using node on our projects, it's advised to use a plugin for node. At the time this guide was written, [this package was at it's prime](https://www.npmjs.com/package/sitemap). At the time you will be reading this guide, be sure to check if that's still the case.

If for any reason you can't use node and it's plugins, don't worry about it. You can use any substantial third party option, as long as it generates a sitemap.xml at the root of your project. As always, in times of need consult [the documentation](https://support.google.com/webmasters/answer/183668?hl=en)

## HT<span></span>ML and semantic elements

Use semantic elements as much as possible. Even in the worst case scenario, it's possible to apply at least the `main` element. For an extensive list of elements and their meaning, follow [this link](https://developer.mozilla.org/en/docs/Web/HTML/Element) and read up.

## Cookies

Consensual cookies are good. Non-consensual cookies are illegal (well, at least in some countries). If you are using cookies, be sure to keep the user informed about their use and to ascertain users consent.

Luckily enough, we have developed and are currently maintaining a gem for that. Add the [cookies_eu](https://github.com/infinum/cookies_eu) gem to your Gemfile and set it up. If you need help with this one, consult the backend team.

## Environment specific robots.txt

Letting crawlers crawl over your staging page is ill advised, as it's bad for SEO. If the project you are working on has its staging environment, prevent crawlers from gathering data that hasn't been approved yet by creating environment specific robots.txt files.

Similar to the former case, add the [roboto](https://github.com/LaunchAcademy/roboto) gem to your Gemfile and set it up. Feel free to check if a better developed or better maintained technology has become available. You can always consult the backend team if assistance is necessary.

## Testing and optimising

As soon as you've deployed your site you should check its speed, performance and mobile usability.

You can check your site's performance with [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) and
with [WebPage Test](http://www.webpagetest.org/). Both tools sum up a list of potential problems and usually advise you on
how to fix them.

To check mobile usability you can use [Mobile-Friendly Test](https://search.google.com/search-console/mobile-friendly).
