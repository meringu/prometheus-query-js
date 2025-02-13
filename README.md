
# Welcome to prometheus-query  👋
[![NPM version](https://img.shields.io/npm/v/prometheus-query.svg?style=flat-square)](https://npmjs.org/package/prometheus-query)
<a href="https://www.jsdelivr.com/package/npm/prometheus-query"><img src="https://data.jsdelivr.com/v1/package/npm/prometheus-query/badge" alt="jsDelivr Downloads"></img></a>
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#)

> A Javascript client for Prometheus **query** API.

## ✨ Features

- Thin & **minimal low-level HTTP client** to interact with Prometheus's API
- Works both on the **browser** and **node.js**
- **UMD compatible**, you can use it with any module loader
- Supports **query** and admin [APIs](https://prometheus.io/docs/prometheus/latest/querying/api/)

⚠️ This library does not export metrics. Please use [prom-client](https://github.com/siimon/prom-client) instead.

## 🚀 Install

### NodeJS

```sh
npm install prometheus-query
```

### Browser

```html
<script src="https://cdn.jsdelivr.net/npm/prometheus-query/dist/prometheus-query.umd.min.js"></script>

<script type="application/javacript">
	const pq = new PrometheusQuery(...);
</script>
```

## 💡 Quick start

```js
const PrometheusQuery = require('prometheus-query');

const pq = new PrometheusQuery({
    endpoint: "http://demo.robustperception.io:9090",
    baseURL: "/api/v1" // default value
});
```

### Instant query

```js
// last `up` value
const q = 'up{instance="demo.robustperception.io:9100",job="node"}';
pq.instantQuery(q)
    .then((res) => {
        const series = res.result;
        series.forEach((serie) => {
            console.log("Serie:", serie.metric.toString());
            console.log("Time:", serie.value.time);
            console.log("Value:", serie.value.value);
        });
    })
    .catch(console.error);
```

Output:

```txt
Serie: up{instance="demo.robustperception.io:9100", job="node"}
Time: Sun Feb 16 2020 18:33:59 GMT+0100 (Central European Standard Time)
Value: 1
```

### Range query

```js
// up during past 24h
const q = 'up';
const start = new Date().getTime() - 24 * 60 * 60 * 1000;
const end = new Date();
const step = 6 * 60 * 60; // 1 point every 6 hours

pq.rangeQuery(q, start, end, step)
    .then((res) => {
        const series = res.result;
        series.forEach((serie) => {
            console.log("Serie:", serie.metric.toString());
            console.log("Values:\n" + serie.values.join('\n'));
        });
    })
    .catch(console.error);
```

Output:

```txt
Serie: up{instance="demo.robustperception.io:9090", job="prometheus"}
Values:
Sat Feb 15 2020 18:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 00:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 06:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 12:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 18:21:47 GMT+0100 (Central European Standard Time): 1

Serie: up{instance="demo.robustperception.io:9091", job="pushgateway"}
Values:
Sat Feb 15 2020 18:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 00:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 06:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 12:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 18:21:47 GMT+0100 (Central European Standard Time): 1

Serie: up{instance="demo.robustperception.io:9093", job="alertmanager"}
Values:
Sat Feb 15 2020 18:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 00:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 06:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 12:21:47 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 18:21:47 GMT+0100 (Central European Standard Time): 1

Serie: up{instance="demo.robustperception.io:9100", job="node"}
Values:
Sat Feb 15 2020 18:20:51 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 00:20:51 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 06:20:51 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 12:20:51 GMT+0100 (Central European Standard Time): 1
Sun Feb 16 2020 18:20:51 GMT+0100 (Central European Standard Time): 1
```

### List series matching query

```js
const match = 'up';
const start = new Date().getTime() - 24 * 60 * 60 * 1000;
const end = new Date();

pq.series(match, start, end)
    .then((res) => {
        console.log('Series:');
        console.log(res.join('\n'));
    })
    .catch(console.error);
```

Output:

```txt
up{instance="demo.robustperception.io:9090", job="prometheus"}
up{instance="demo.robustperception.io:9091", job="pushgateway"}
up{instance="demo.robustperception.io:9093", job="alertmanager"}
up{instance="demo.robustperception.io:9100", job="node"}
```

### List all active alerts

```js
pq.alerts()
    .then(console.log)
    .catch(console.error);
```

Output:

```js
[
  Alert {
    activeAt: 2019-11-14T20:04:36.629Z,
    annotations: {},
    labels: { alertname: 'ExampleAlertAlwaysFiring', job: 'alertmanager' },
    state: 'firing',
    value: 1
  },
  Alert {
    activeAt: 2019-11-14T20:04:36.629Z,
    annotations: {},
    labels: { alertname: 'ExampleAlertAlwaysFiring', job: 'node' },
    state: 'firing',
    value: 1
  },
  Alert {
    activeAt: 2019-11-14T20:04:36.629Z,
    annotations: {},
    labels: { alertname: 'ExampleAlertAlwaysFiring', job: 'prometheus' },
    state: 'firing',
    value: 1
  },
  Alert {
    activeAt: 2019-11-14T20:04:36.629Z,
    annotations: {},
    labels: { alertname: 'ExampleAlertAlwaysFiring', job: 'pushgateway' },
    state: 'firing',
    value: 1
  }
]
```

## 🏋️‍♂️ Documentation

// @TODO

## ✅ Run tests

```sh
npm run test
```

## 🔐 Security advisory

If you open a Prometheus instance on Internet, it would be a good idea to block some routes.

Start by blocking `/api/v1/admin`. I'm pretty sure allowing only `/api/v1/query` and `/api/v1/query_range` will match your needs.

Also don't use Prometheus as a multitenant timeseries database!

At your own risk... 😘

## 🤝 Contributing

The Prometheus Query client is open source and contributions from the community (you!) are welcome.

There are many ways to contribute: writing code, documentation, reporting issues...

## Author

👤 **Samuel Berthe**

* Twitter: [@samuelberthe](https://twitter.com/samuelberthe)
* Github: [@samber](https://github.com/samber)

## 💫 Show your support

Give a ⭐️ if this project helped you!

[![support us](https://c5.patreon.com/external/logo/become_a_patron_button.png)](https://www.patreon.com/samber)

## 📝 License

Copyright © 2020 [Samuel Berthe](https://github.com/samber).

This project is [MIT](./LICENSE) licensed.
