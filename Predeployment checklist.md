*We advise that you follow these guidelines in order to achieve the optimal performance and keep the customer happy. Before deploying, check that you have added these functionalities. Add them if you haven't.*

## Social media optimization

### Facebook
Use the [Open Graph protocol](http://ogp.me/) to display the website as an object in a social graph (that is, display it with more detail on social networks). Use meta tags to describe the properties of an object of interest. Regarding Facebook, the four required properties for every page are:

* ``og:title``
  * The title of your object as it should appear within the graph, e.g., "The Rock".
* ``og:type``
  * The type of your object, e.g., "video.movie". Depending on the type you specify, other properties may also be required.
* ``og:image``
  * An image URL that should represent your object within the graph.
* ``og:url``
  * The canonical URL of your object that will be used as its permanent ID in the graph, e.g., "http://www.imdb.com/title/tt0117500/".

Along with these properties, we encourage you to use others, if applicable. For a more detailed overview of the tags, please consult the [comprehensive list of Facebook tags](https://developers.facebook.com/docs/sharing/webmasters).

### Twitter

Similar to the [Open Graph protocol](http://ogp.me/), [Twitter Cards](https://dev.twitter.com/cards/overview) are used to display the website on Twitter. There are various types of cards, the most basic one being the [Summary Card](https://dev.twitter.com/cards/types/summary). You should at least implement the Summary Card. Of course implement a more detailed one if possible. :)

## Icons and colors for the browser

###Icons
In order to add support for icons to the website, several meta tags have to be added. As the result, the icon may show up in many places, including the browser tab, recent app switch, the new (or recently visited) tab page, and more. It's necessary to provide icon support for all major **browsers** and **mobile platforms**. The [Real Favicon Generator](http://realfavicongenerator.net/) is a handy tool for this task.

### Colors
It's possible to color the browsers and elements of the platform according to the designer's wishes. For further info, check out what [Google Developers](https://developers.google.com/web/fundamentals/design-and-ui/browser-customization/theme-color?hl=en) have to say about this.

## Google Analytics

To start collecting the basic data from a website, it's necessary to set up Google Analytics. If the client hasn't done it already, set it up instead of them, and transfer the ownership later on.

In order to set up Analytics for a given page, please consult the following [guide](https://support.google.com/analytics/answer/1008015?hl=en).

## Google Search Console

Expanding on Google Analytics, establish the Search Console tools functionality for the site to be deployed. After setting them up, remember to add the client as the **owner** (not the admin). Make sure that we remain the owners of the site.

If you need help setting them up, [here you go, young sir or madam](https://support.google.com/webmasters/answer/6001104?hl=en).

_Side note: it's not possible to verify whether the tools have been set up properly before the actual deployment because Webmaster Tools require an actual link pointing to the website. Link everything up and verify it's functioning correctly immediately after deployment._

## Sitemap

In order to boost indexing efficiency, create a sitemap file. As we currently prefer using node on our projects, you should use a plugin for node. At the time of writing this guide, [this package was at its prime](https://www.npmjs.com/package/sitemap). At the time you will be reading this guide, be sure to check if that's still the case.

If for any reason you can't use node and its plugins, don't worry about it. You can use any substantial third-party option as long as it generates a sitemap.xml at the root of your project. As always, in times of need, consult [the documentation](https://support.google.com/webmasters/answer/183668?hl=en).

## HT<span></span>ML and semantic elements

Use semantic elements as much as possible. Even in the worst case scenario, it's possible to apply at least the `main` element. For an extensive list of elements and their meaning, follow [this link](https://developer.mozilla.org/en/docs/Web/HTML/Element) and read up.

## Cookies

Consensual cookies are good. Non-consensual cookies are illegal (well, at least in some countries). If you are using cookies, be sure to keep the user informed about their use and to obtain the user's consent.

Luckily enough, we have developed and are currently maintaining a gem for that. Add the [cookies_eu](https://github.com/infinum/cookies_eu) gem to your Gemfile and set it up. If you need help, consult the backend team.

## Environment-specific robots.txt

Letting crawlers crawl over your staging page is ill-advised, as it's bad for SEO. If the project you are working on has its staging environment, prevent crawlers from gathering data that hasn't been approved yet by creating environment-specific robots.txt files.

Similar to the former case, add the [roboto](https://github.com/LaunchAcademy/roboto) gem to your Gemfile and set it up. Feel free to check if a better-developed or better-maintained technology has become available in the meantime. You can always consult the backend team if you need assistance.

## Testing and optimising

As soon as you've deployed your site, you should check its speed, performance, and mobile usability.

You can check your site's performance with [Lighthouse](https://developers.google.com/web/tools/lighthouse/), [TestMySite](https://testmysite.withgoogle.com/), [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/), and with [WebPage Test](http://www.webpagetest.org/). All four tools sum up a list of potential problems and usually give advice on how to fix them.

You can use [Mobile-Friendly Test](https://search.google.com/search-console/mobile-friendly) to check mobile usability.
