---
title: "Parser examples"
draft: true
---

## Regex

{{< parser type="regex" datetimeformat="false" >}}
(?<metricname>\w+?):(?<metricvalue>[-+]?[\d\.]+?)\|(?<metrictype>\w+?)(\|@(?<metricsampling>[\d\.]+?))?
{{< /parser >}}

## Regex with date format

{{< parser type="regex" datetimeformat="yyyy-MM-dd'T'HH:mm:ss[.SSS]XXX" >}}
(?<metricname>\w+?):(?<metricvalue>[-+]?[\d\.]+?)\|(?<metrictype>\w+?)(\|@(?<metricsampling>[\d\.]+?))?
{{< /parser >}}

## Regex with date format and timezone

{{< parser type="regex" datetimeformat="yyyy-MM-dd'T'HH:mm:ss[.SSS]" timezone="UTC" >}}
(?<metricname>\w+?):(?<metricvalue>[-+]?[\d\.]+?)\|(?<metrictype>\w+?)(\|@(?<metricsampling>[\d\.]+?))?
{{< /parser >}}

## JSON

{{< parser type="json" datetimeformat="false" />}}

## JSON with nested

{{< parser type="json" datetimeformat="false" parsenestedjson="true" />}}

## JSON with date format

{{< parser type="json" datetimeformat="yyyy-MM-dd'T'HH:mm:ss[.SSS]XXX" timestampfield="@timestamp" />}}

## JSON with date format and timezone

{{< parser type="json" datetimeformat="yyyy-MM-dd'T'HH:mm:ss[.SSS]" timestampfield="@timestamp" timezone="UTC" />}}

## Humio

{{< parser type="humio">}}
parseJson(line)
| parseTimestamp(timestamp)
| repo := "audit"
{{< /parser >}}

## Unknown

{{< parser type="unknown" >}}
Foo bar baz
{{< /parser >}}