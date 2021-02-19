## Node logging

Infinum uses [Graylog](https://www.graylog.org/) to log all requests that are processed by our apps.

### Setup

Install the gelf-pro package via npm/yarn:

```npm install -s gelf-pro```

### Configuration

#### Node-only environment

Set the configuration on your app logger instance:

```Javascript
const log = require('gelf-pro');

log.setConfig({
  fields: {
    application: 'app-name'
  },
  adapterOptions: {
    host: 'host-name' // graylog host should be configured via secrets
  }
});
```

#### Isomorphic environment

If your code is running both in node and in a browser (sharing logic, server-side rendering, etc.), you need to make sure gelf is running only in node. For the client, we often use [loglevel](https://github.com/pimterry/loglevel) for better logging:

```Javascript
const isServer = typeof window === 'undefined';

let log;
if (isServer) {
  log = require('gelf-pro');

  log.setConfig({
    fields: {
      application: 'app-name'
    },
    adapterOptions: {
      host: 'host-name' // graylog host should be configured via secrets
    }
  });
} else {
  log = require('loglevel');

  // Set level to debug for development or to silent for production
  log.setLevel(process.env.NODE_ENV === 'development' ? 1 : 5);
}

module.exports = log;
```

**❗❗❗Note:** `gelf-pro` and `loglevel` don't have the same API, so you should only use `error`, `info`, and `debug` methods. In the future, we might create a gelf plugin for loglevel which would then abstract those differences.

### Usage

```Javascript
log.info('Hello world');
log.error('Something bad has happened')
```

### Documentation

Read more about logging here:
* [gelf-pro JS API](https://github.com/kkamkou/node-gelf-pro)
* [Graylog & GELF](http://docs.graylog.org/en/latest/pages/gelf.html)
* [loglevel](https://github.com/pimterry/loglevel)
