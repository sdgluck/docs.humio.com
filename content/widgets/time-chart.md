---
title: Time Chart
aliases: ["/ref/charts/time-chart"]
weight: 200
---

The _time chart_ widget can display bucketed time series data on a time line. It is
the most commonly used widget in Humio.

## Usage

The time chart widget expects a very specific input format that is produced by
it's companion query function {{< function "timechart" >}}. You are recommended to
read the function documentation for {{< function "timechart" >}}.

### Example 1: Metric Data

Say you have a service that periodically writes metrics to its logs. This could
be tools such as DropWizard or Micrometer or a system monitoring tool like MetricBeat.

In this case we will have JSON logs that could look something like this:

```json
{ "type": "metrics", "id": "1", "ts": "2018-11-01T00:10:11.001", "disk0": 11.21, "disk1": 21.14, "disk2": 12.01  }
{ "type": "metrics", "id": "2", "ts": "2018-11-01T00:10:13.106", "disk0": 11.21, "disk1": 21.14, "disk2": 12.01  }
{ "type": "metrics", "id": "3", "ts": "2018-11-01T00:10:18.771", "disk0": 10.57, "disk1": 20.41, "disk2": 11.91  }
{ "type": "metrics", "id": "4", "ts": "2018-11-01T00:10:18.772", "disk0": 9.15, "disk1": 19.12, "disk2": 10.07  }
```

where `disk0-2` represent some metrics that you would like to create a time chart
for.

```humio
type = metrics | timechart(function=[max(disk0, as="Disk 0"), max(disk1, as="Disk 1"), max(disk2, as="Disk 2")])
```

Notice that we provide several aggregate functions to the `function` parameter.
This is because we want to work on several fields on each input event.
In this example it creates three series in the resulting time chart - one for each
metric. We used the {{< function "max" >}} on each field. This means that when the
{{<function "timechart" >}} function buckets the data it uses larger number within
the bucket to represent the value of the series in the bucket. In other words,
imagine that event `id=3` and `id=4` in JSON events above end up in the same bucket
(Which is not an unreasonable assumption since their timestamps are only 1 ms apart!).

If we use {{< function "max" >}} we will get the largest value of the field, e.g.
`max(disk0)` of `id=3` and `id=4` would be `10.57` even though `id=4` occurs later
in the stream. Alternatively we could have used e.g. {{< function "avg" >}} which
would have given the average of the two values of `disk0`, in this case `9.86`.
Which aggregate function to use depends on what you what to visualize.

### Example 2: Log Levels

If you have logs that contain log levels like INFO, ERROR and WARN it can be
interesting to visualize them over time. Say you have logs like:

```
2018-10-10T01:10:11.322Z [INFO] User Logged in. userId=10, ...
2018-10-10T01:10:12.172Z [WARN] Invalid Login Attempt. userId=10, ...
2018-10-10T01:10:14.122Z [INFO] Database Query. timeMs=20 connection=12.10...
2018-10-10T01:10:15.312Z [INFO] Database Query. timeMs=10 connection=12.10...
2018-10-10T01:10:16.912Z [INFO] Database Query. timeMs=21 connection=12.10...
```

and you have a parser that extracts a field called `loglevel` from each line.
You can do something like:

```humio
timechart(loglevel)
```

This will count the number of occurrences of event that have a field called
`loglevel` and put them in a series in the time chart based on their value.
Based on the example data above this would create a time chart with 2 series,
`INFO` and `WARN`.

By default the {{< function "count" >}} function is used to calculate the value
of each bucket, but you can easily plot other values by specifying other functions
in the `function` property of the {{< function "timechart">}} function.
For instance if we use the {{< function "avg" >}} function on the field `timeMs`:

```humio
timechart(loglevel, function=avg(timeMs))
```

We can see the average time in milliseconds that a database query takes. Hint:
The {{< function "percentiles" >}} function is very useful as an aggregate function
in time charts when you wish to visualize response times like this.

## Further Reading

You are encouraged to read the documentation for the query function {{< function "timechart" >}}.
