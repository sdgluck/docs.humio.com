---
title: Bar Chart
aliases: ["/ref/charts/bar-chart"]
weight: 700
---

The _Bar Chart_ widget can be used to visualize results of the {{< function "groupBy" >}}
and {{< function "top" >}}.

## Usage

The _Bar Chart_ widget can be used to visualize results of the {{< function "groupBy" >}}
and {{< function "top" >}}. Unlike [Pie Charts]({{< ref "pie-chart.md" >}}), Bar Charts
can be used to show results multiple aggregates.
If you provide several aggregate functions to the `function` argument of {{< function "groupBy" >}}
it will display one result per function.

### Example: Displaying Several series per category

Assume we have a service producing logs like the ones below:

```
2018-10-10T01:10:11.322Z host=web01 reponseTime=10.2ms
2018-10-10T01:10:12.172Z host=web01 reponseTime=2.6ms
2018-10-10T01:10:14.122Z host=web02 reponseTime=11.5ms
2018-10-10T01:10:15.312Z host=web01 reponseTime=14.7ms
2018-10-10T01:10:16.912Z host=web03 reponseTime=10.8ms
```

We would like to make a bar char showing the maximum response time and
the average response time per host.

```humio
groupBy(host, function=[max(responseTime), avg(responseTime)])
```

If you select the Bar Chart Widget Type in the Widget Type dropdown,
this query will produce one bar chart with 3 categories (one per host)
and 2 series one per aggregate result (e.i. `_max`, and `_avg`).
