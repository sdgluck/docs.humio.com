---
title: "StatsD"
weight: 400
categories: ["Integration", "DataShipper"]
categories_weight: -80
featured-on: ["on-prem"]
---

The StatsD protocol is a great way of shipping raw metrics to Humio from a lot of [tools](https://github.com/etsy/statsd/wiki#client-implementations). Both UDP and TCP transport is supported.

{{% notice note %}}
Currently StatsD protocol is only supported in on-prem installations
{{% /notice %}}

## Configuring Humio

The StatsD format is very simple. In it's simplest form it looks something like this

```
<metricname>:<metricvalue>|<metrictype>
```

Start by [creating a new parser]({{< ref "parsers/creating-a-parser" >}}) with the following regex

{{< parser type="regex" datetimeformat="false" >}}
(?<metricname>\w+?):(?<metricvalue>[-+]?[\d\.]+?)\|(?<metrictype>\w+?)(\|@(?<metricsampling>[\d\.]+?))?
{{< /parser >}}

And no _Parse timestamp_ and _Parse key values_. Finally give it a name, i.e. "statsd".

Next, create an [ingest listener]({{< ref "sending-data-to-humio/ingest-listeners" >}}) with the statsd parser.

{{% notice note %}}
We strongly recommend using an UDP ingest listener for non-aggregated StatsD data
{{% /notice %}}