[![Bonsai Asset Badge](https://img.shields.io/badge/Sensu%20InfluxDB%20Handler-Download%20Me-brightgreen.svg?colorB=89C967&logo=sensu)](https://bonsai.sensu.io/assets/sensu/sensu-influxdb-handler) TravisCI: [![TravisCI Build Status](https://travis-ci.org/sensu/sensu-influxdb-handler.svg?branch=master)](https://travis-ci.org/sensu/sensu-influxdb-handler)

# Sensu InfluxDB Handler

- [Overview](#overview)
- [Usage examples](#usage-examples)
- [Configuration](#configuration)
  - [Asset registration](#asset-registration)
  - [Asset definition](#asset-definition)
  - [Handler definition](#handler-definition)
  - [Check manifest](#check-manifest)
- [Installation from source and contributing](#installation-from-source-and-contributing)

## Overview

The Sensu InfluxDB Handler is a [Sensu Event Handler][3] that sends metrics to
the time series database [InfluxDB][2]. [Sensu][1] can collect metrics using
check output metric extraction or the StatsD listener. Those collected metrics
pass through the event pipeline, allowing Sensu to deliver the metrics to the
configured metric event handlers. This InfluxDB handler will allow you to
store, instrument, and visualize the metric data from Sensu.

Check out [The Sensu Blog][5] or [Sensu Docs][6] for a step by step guide!

## Usage Examples

Help:
```
Usage:
  sensu-influxdb-handler [flags]

Flags:
  -a, --addr string            the address of the influxdb server, should be of the form 'http://host:port', defaults to 'http://localhost:8086' or value of INFLUXDB_ADDR env variable
  -d, --db-name string         the influxdb to send metrics to
  -h, --help                   help for sensu-influxdb-handler
  -i, --insecure-skip-verify   if true, the influx client skips https certificate verification
  -p, --password string        the password for the given db, defaults to value of INFLUXDB_PASS env variable
      --precision string       the precision value of the metric (default "s")
  -u, --username string        the username for the given db, defaults to value of INFLUXDB_USER env variable

```

## Configuration

### Asset registration

Assets are the best way to make use of this handler. If you're not using an asset, please consider doing so! If you're using sensuctl 5.13 or later, you can use the following command to add the asset: 

`sensuctl asset add sensu/sensu-email-handler`

If you're using an earlier version of sensuctl, you can download the asset definition from [this project's Bonsai Asset Index page](https://bonsai.sensu.io/assets/sensu/sensu-influxdb-handler).


### Asset definition

```yml
---
type: Asset
api_version: core/v2
metadata:
  name: sensu-influxdb-handler_linux_amd64
  labels: 
  annotations:
    io.sensu.bonsai.url: https://bonsai.sensu.io/assets/sensu/sensu-influxdb-handler
    io.sensu.bonsai.api_url: https://bonsai.sensu.io/api/v1/assets/sensu/sensu-influxdb-handler
    io.sensu.bonsai.tier: Supported
    io.sensu.bonsai.version: 3.1.2
    io.sensu.bonsai.namespace: sensu
    io.sensu.bonsai.name: sensu-influxdb-handler
    io.sensu.bonsai.tags: ''
spec:
  url: https://assets.bonsai.sensu.io/b28f8719a48aa8ea80c603f97e402975a98cea47/sensu-influxdb-handler_3.1.2_linux_amd64.tar.gz
  sha512: 612c6ff9928841090c4d23bf20aaf7558e4eed8977a848cf9e2899bb13a13e7540bac2b63e324f39d9b1257bb479676bc155b24e21bf93c722b812b0f15cb3bd
  filters:
  - entity.system.os == 'linux'
  - entity.system.arch == 'amd64'
```

### Handler definition

```yml
---
api_version: core/v2
type: Handler
metadata:
  namespace: default
  name: influxdb
spec:
  type: pipe
  command: sensu-influxdb-handler -d sensu
  timeout: 10
  env_vars:
  - INFLUXDB_ADDR=http://influxdb.default.svc.cluster.local:8086
  - INFLUXDB_USER=sensu
  - INFLUXDB_PASS=password
  filters:
  - has_metrics
  runtime_assets:
  - sensu/sensu-influxdb-handler
```

### Check definition
```yml
---
api_version: core/v2
type: CheckConfig
metadata:
  namespace: default
  name: dummy-app-prometheus
spec:
  command: sensu-prometheus-collector -exporter-url http://localhost:8080/metrics
  subscriptions:
  - dummy
  publish: true
  interval: 10
  output_metric_format: influxdb_line
  output_metric_handlers:
  - influxdb
```

That's right, you can collect different types of metrics (ex. Influx,
Graphite, OpenTSDB, Nagios, etc.), Sensu will extract and transform
them, and this handler will populate them into your InfluxDB.

**Security Note:** The InfluxDB addr, username and password are treated as a security sensitive configuration options in this example and are loaded into the handler config as env_vars instead of as a command arguments. Command arguments are commonly readable from the process table by other unprivileged users on a system (ex: `ps` and `top` commands), so it's a better practice to read in sensitive information via environment variables or configuration files as part of command execution. The command flags for these configuration options are provided as an override for testing purposes.

## Installing from source and contributing

Download the latest version of the sensu-influxdb-handler from [releases][4],
or create an executable script from this source.

### Dependencies

Sensu-influxdb-handler uses [golang/dep](https://github.com/golang/dep) for managing its dependencies. You will need to install the latest stable version of dep in order to download/modify dependencies.

From the local path of the sensu-influxdb-handler repository:
```
dep ensure
```

### Compiling

From the local path of the sensu-influxdb-handler repository:
```
go build -o /usr/local/bin/sensu-influxdb-handler main.go
```

To contribute to this plugin, see [CONTRIBUTING](https://github.com/sensu/sensu-go/blob/master/CONTRIBUTING.md)

[1]: https://github.com/sensu/sensu-go
[2]: https://github.com/influxdata/influxdb
[3]: https://docs.sensu.io/sensu-go/5.0/reference/handlers/#how-do-sensu-handlers-work
[4]: https://github.com/sensu/sensu-influxdb-handler/releases
[5]: https://blog.sensu.io/check-output-metric-extraction-with-influxdb-grafana
[6]: https://docs.sensu.io/sensu-go/5.0/guides/influx-db-metric-handler/
