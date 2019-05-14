---
title: Gauge
aliases: ["/ref/charts/gauge","/ref/charts/simple-gauge"]
weight: 600
---

The _Gauge_ Widget be used to display a single number. This is useful for displaying
e.g. the number of errors per day or the number of active connections to a system.

## Input Format

The Guage Widget can any data with at least one field containing a number.
It multiple rows are present in the input, the first element is used.
If multiple number fields are present, it is unspecified which field is used.

## Usage

The Gauge Widget is best used in conjunction with functions such as {{< function "sum" >}}, {{< function "count" >}},
or {{< function "avg" >}} which produce a single row with a single field, e.g. `_sum`.

Using the widget's _Style_ editor can configure the widget to assume different
colors for different value ranges.
You could for instance make the background turn yellow if the value is >100,
and red if it is >500.

### Example: Displaying Counting Errors

If we wanted to show the number of errors a system, we could count them using:

```humio
loglevel = /error/i | count()
```

This produces a single result with a field `_count`. If the Gauge widget is
selected, it will automatically select the first field it sees at use it as
the value to display.
