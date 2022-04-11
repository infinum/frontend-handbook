[![infinum/next-passenger - GitHub](https://gh-card.dev/repos/infinum/next-passenger.svg)](https://github.com/infinum/next-passenger)

## The issue

We couldn't figure out how to run [zeit/next.js](https://github.com/zeit/next.js) on Passenger with Nginx. The issue we had was very simple but seemingly unfixable. If you take a look at a very basic but working example of passenger `location` block in nginx:

```text
location / {
	passenger_enabled on;
	passenger_app_type node;
	passenger_app_root /app;
	passenger_startup_file index.js;
}
```

You will notice the `passenger_startup_file` directive that **has** to point to a file and next.js has to be run with `next start` to run in production.

## Hopeful fix

The first thing we tried was setting `passenger_startup_file` to the `next` bin file:

```text
location / {
	passenger_enabled on;
	passenger_app_type node;
	passenger_app_root /app;
	passenger_startup_file ./node_modules/.bin/next;
}
```

If you try to run this it will work 🎉! But soon we found out that the app is very slow. Now - if you know how to work with next and you remember my statement that the run command is `next start` you will notice that `start` is missing! We've been running the app in development mode! Good thing we haven't released to production yet!

## The real fix!

```javascript
const path = require('path');

const nextPath = path.join(__dirname, 'node_modules', '.bin', 'next');

process.argv.length = 1;
process.argv.push(nextPath, 'start');

require(nextPath);
```

Now if you point passenger to this file it works!

```text
location / {
	passenger_enabled on;
	passenger_app_type node;
	passenger_app_root /app;
	passenger_startup_file index.js;
}
```

## How?

This, very much, is a hack... But it's important to understand how this hack works.

```javascript
const path = require('path');

// define the path to the next bin (luck has it that it's a node script)
const nextPath = path.join(__dirname, 'node_modules', '.bin', 'next');

// strips away all arguments until it's at `node`
process.argv.length = 1;

// redefining the arguments so it's like we are running `next start`
process.argv.push(nextPath, 'start');

// start the script/bin
require(nextPath);
```

## Proof of concept

You'll find and example next.js app and nginx/passenger setup here: [infinum/next-passenger](https://github.com/infinum/next-passenger). You'll find a docker image there to test this out with your setup.
