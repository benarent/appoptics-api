# appoptics-api

[![npm version](http://img.shields.io/npm/v/librato-api.svg)](https://npmjs.org/package/librato-api)
[![Build Status](https://travis-ci.org/emartech/librato-api.svg?branch=master)](https://travis-ci.org/emartech/librato-api)
[![Coverage Status](https://coveralls.io/repos/github/emartech/librato-api/badge.svg?branch=master)](https://coveralls.io/github/emartech/librato-api?branch=master)
[![Dependencies Status](https://david-dm.org/emartech/librato-api.svg)](https://david-dm.org/emartech/librato-api)

#### This project is a WIP 

This package allows you to manage your AppOptics *metrics* backend configuration and query time series data, but it is not intended to submit metric data. There are other packages doing that
in a better way.

For a full description of the AppOptics API see the official
[AppOptics API](https://docs.appoptics.com/api/) documentation.

At the moment support for the following sections is implemented:
Authentication, Pagination, Metrics, Spaces, Charts, Alerts, Services, Sources.
Note the official API does not expose visual layout of spaces yet, only contents.

Explicit support for the following sections is missing:
Annotations, API Tokens, Jobs, Snapshots and Measurements Beta.
This is easy to fix, pull requests are welcome.

## Examples
```javascript
// the package is a ready to use client,
// using AppOpticsToken from the process environment
const AppOpticsApi = require('appoptics-api')

// it's also possible to create a client with custom config (all properties are optional)
const AppOpticsApi = require('appoptics-api').AppOpticsApi
const AppOpticsApi = new AppOpticsApi({
    serviceUrl: 'https://...',
    auth: { pass: '...' },
    logger: ...,
    request: ...
})

// all methods return Promises
AppOpticsApi.getMetrics().then(console.log)

// most methods support an options object which is passed to request-promise
AppOpticsApi.getMetrics({ qs: { offset: 200, limit: 100 } })

// iterates over pagination
AppOpticsApi.getAllMetrics()

// get a metric definition
AppOpticsApi.getMetric('router.bytes')

// retrieve one page of time series data for metric and time frame
AppOpticsApi.getMetric('router.bytes', { qs: { start_time: date1, end_time: date2 }})

// retrieve all pages of time series data for metric and time frame
AppOpticsApi.getAllMeasurements('router.bytes', { qs: { start_time: date1, end_time: date2 }})

// update metric definition
AppOpticsApi.putMetric('customers', { 'period': 3600 })

// use custom space finder (getSpace requires id)
AppOpticsApi.findSpaceByName('myspace')

// update chart definition in a space
AppOpticsApi.putChart(myspace.id, mychartId, mychart)

// not everything is explicitly supported yet, but generic api requests are easy to do
AppOpticsApi.apiRequest(['annotation', 'backup'], { method: 'PUT', body: { ... } })
```

## CLI Tool

This package installs a CLI tool named "appoptics" into your global or package bin-dir.

You have to export APPOPTICS_TOKEN for authentication to work.
```bash
export APPOPTICS_TOKEN='...'
appoptics help
appoptics list-metrics
...
```

### Warning

This tool is quite new and still a bit rough regarding command line parsing,
integrated help, etc. To see what it's doing it may be helpful to set LOG_LEVEL to verbose or debug.

### Configuration Directory Support

Apart from functions which model single API calls, the tool can take a local directory
containing json or js files in a certain structure and apply the contained elements to
a AppOptics account with the "update-from-dir" command. The repository contains an example directory
"example-config" which demonstrates this feature.

Note that all elements are referenced by
their name (or title for alerts), even if the API usually handles them with a numeric id.
This way generic configuration can be applied to multiple AppOptics accounts, but uniquness
of names etc. is assumed.

There is a simple templating feature to create serieses of similar metrics. The "show-config-dir"
command can be used to debug templating easily.

In general this will leave alone (not delete) server side elements which are not defined
in the config dir, but it can remove elements which are explicitly enumerated in the
outdated.json file.
