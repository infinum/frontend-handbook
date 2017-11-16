## Node Logging

Infinum uses the [Graylog](https://www.graylog.org/) for logging all requests which are processed by our apps.

### Setup

Install gelf-pro package via npm/yarn:

```npm install -s gelf-pro```

### Configuration

Set configuration on your app logger instance:

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

### Usage

```Javascript
log.info('Hello world');
log.error('Something bad has happened')
```

### Documentation

Read more about logging here:
* [gelf-pro JS API](https://github.com/kkamkou/node-gelf-pro)
* [Graylog & GELF](http://docs.graylog.org/en/latest/pages/gelf.html)
