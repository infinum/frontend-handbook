## Motivation

An ideal goal is to support all the platforms and all the browsers when developing a web application, 
but sometimes this is difficult or not profitable, especially today when a lot of devices can have 
their own ways of connecting to the Internet (including TVs and refrigerators).

Because of that you need some easy mechanism to just tell: "We are so sorry, but please install more 
sophisticated browser" - without your application crashing before it can even show the message.

Luckily, from Next.js version [10.1.0](https://github.com/vercel/next.js/releases/tag/v10.1.0) 
(`Add has route field`), you can use some specific property for redirects in `next.config.js` that 
can help you with that.

## The issue
Your application may cover a lot of different browsers, but still there are many commonly used browsers 
which probably can not run your application - either it does not look good or crashes. 

There might be issues with your own implementation or some 3rd party library, and you can not replace
any of it easily or without sacrificing something else (time, cost, functionality, etc.).

This problem is easier to fix if the application is not crashing before showing at least a popup, 
but it gets impossible if you have errors before JS is even executed.
This is where Next.js redirects and `has` property shines.

## The fix

There is more to talk about redirects, which can also be read in 
[Next.js redirects by header, cookie and query matching](https://nextjs.org/docs/api-reference/next.config.js/redirects#header-cookie-and-query-matching), 
but the most important is the `has` property which allows you to configure in what circumstances should your application be redirected when hitting some path.

So without further ado, here is the Next.js config which redirects requests from any
unsupported browser (user-agent header matches regex, in our case for Internet Explorer)
to the `not-supported.html` file located in the `public` folder.

```js
// next.config.js

module.exports = {
  // ...
  async redirects() {
    return [
      {
        source: '/', // all paths should be checked and redirected
        has: [
          {
            type: 'header', // we want user-agent and user-agent is part of the header
            key: 'user-agent',
            value: '.*(MSIE|Trident).*', // regex which checks if either MSIE or Trident is somewhere in user-agent string - we are excluding IE browser
          },
        ],
        permanent: true, // we want redirect to be permanent
        destination: '/not-supported.html', // this file is from public folder; it can be some other file on some other server
      },
    ];
  },
}
```

Once again, above code says: redirect all paths to `public/not-supported.html` if you come across 
words "MSIE" or "Trident" in user-agent header and make this permanent redirecton.

This way your app will not crash, or displayed because not a single JS from your application will be served prior 
to the redirection check.

`has` property can be used for more things than redirection based on the browser agent.

## The implications

The only problem that can occur is if you have wrong regex for browser detection so test your 
implementation.
Also double check if you want to exclude some browser since you are narrowing your audience (users).