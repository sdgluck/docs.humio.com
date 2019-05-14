---
title: Pie Chart
aliases: ["/ref/charts/pie-chart"]
weight: 800
---

The _Pie Chart_ widget can be used to visualize results of the {{< function "groupBy" >}}
and {{< function "top" >}}.

## Usage

The _Pie Chart_ widget can be used to visualize results of the {{< function "groupBy" >}}
and {{< function "top" >}}. It can only show a single result at a time meaning that
if you provide several aggregate functions to the `function` argument of {{< function "groupBy" >}}
it will only display the _first_ aggregate result.

If you want to display multiple aggregate results in a single chart, you can use
the [Bar Chart Widget]({{< ref "bar-chart.md" >}}) which supports showing multiple
aggregate results.

### Example: Log Level Distribution

Assume we have a service producing logs like the ones below:

```
2018-10-10T01:10:11.322Z [ERROR] Invalid User ID. errorID=2, userId=10
2018-10-10T01:10:12.172Z [WARN]  Low Disk Space.
2018-10-10T01:10:14.122Z [ERROR] Invalid User ID. errorID=2, userId=11
2018-10-10T01:10:15.312Z [ERROR] Connection Dropped. errorID=112 server=120.100.121.12
2018-10-10T01:10:16.912Z [INFO]  User Login. userId=11
```

We would like to show a piechart of the relative distribution of events with
the respective log levels INFO, ERROR, WARN:

We can do this by selecting the Pie Chart Widget in the Widget Type dropdown
a query like:

```humio
groupBy(logLevel)
```
