---
title: "Logstash"
weight: 200
categories: ["Integration", "DataShipper"]
categories_weight: -90
pageImage: /integrations/logstash.svg
---

Logstash is an established open source tool for collecting logs,
parsing them and outputting them to other systems.

You can use Logstash alongside Humio to process and analyze logs
together. In this scenario, you use Logstash as the log collection and
parsing agent and instruct it to send the data to Humio.

{{% notice tip %}}
__Humio supports the ElasticSearch bulk insertion API__  Just [point the Elastic outputter at Humio](#configuration).
{{% /notice %}}


The benefit of this approach is that you can take advantage of the
extensible architecture of Logstash to parse many kinds of data:

* You can install one of the many available plugins that can parse
  many well-known data formats.
* You can use the Grok language to build custom parsers for unusual
  data formats. Grok has many built-in patterns to help parse your
  data.

## Installation

To download Logstash visit the [Logstash downloads page](https://www.elastic.co/downloads/logstash).

{{% notice note %}}
You can find the complete documentation for Logstash at [the Reference page of the official Logstash website](https://www.elastic.co/guide/en/logstash/current/index.html).
{{% /notice %}}


## Configuration

Because Humio supports parts of the ElasticSearch insertion API, you
can use the built-in `elasticsearch` output in the Logstash
configuration.

The following example shows a very simple Logstash configuration that
sends data to Humio:

```
input{
  exec{
    command => "date"
    interval => "5"
  }
}
output{
  elasticsearch{
    hosts => ["https://$BASEURL/api/v1/ingest/elastic-bulk"]
    user => "$INGEST_TOKEN"
    password => "notused" # a password has to be set, but Humio does not use it
  }
}
```

{{< partial "common-rest-params" >}}


{{% notice warning %}}
Logstash uses 9200 as the default port if no port is specified. So if Humio is listening on the default ports 80 or 443 these ports should be explicitly put in the $BASEURL
{{% /notice %}}

In the above example, Logstash calls the Linux `date` command every
five seconds. It passes the output from this command to Humio.


### Field mappings

When you use the ElasticSearch output, Logstash outputs JSON
objects. The JSON for an event sent to Humio with the above
configuration looks like this:

```json
{
  "@timestamp": "2016-08-25T08:34:37.041Z",
  "message": "Thu Aug 25 10:34:37 CEST 2016\n",
  "command": "date"
}
```

Humio maps each JSON object into an Event. Each field in the JSON
object becomes a field in the Humio Event.

`@timestamp` is a special field in the Elastic protocol. It must be present, and contain the timestamp in ISO 8601 format (`yyyy-MM-dd'THH:mm:ss.SSSZ`).   
It is possible to specify the timezone (like +00:02) in the timestamp. Specify the time zone if you want Humio to save this information. 
Logstash adds the `@timestamp` field automatically. <br /><br />Depending on the configuration the timestamp can be the time at which Logstash handles the event, or the actual timestamp in the data. 
If the timestamp is present in the data you can configure logstash to parse it, for example, by using the date filter.  
Another option is to handle parsing the timestamp in Humio by connecting a parser to the ingest token.

### Adding Parsers in Humio
Humio can do further parsing/transformation of the data it receives by [connecting a parser to the ingest token]({{< ref "assigning-parsers-to-ingest-tokens.md" >}}). 
For more information on parsers, see [parsing]({{< relref "parsers/_index.md" >}}).


### Dropping fields

Logstash often adds fields like `host` and `@version` to events. You
can remove these fields using a filter and the `drop_field` function
in Logstash.


