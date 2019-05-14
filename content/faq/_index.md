---
title: "FAQ"
weight: 1000
aliases: ["appendix/faq", "/ref/faq"]
---

## Is Humio cloud only? Is it possible to use Humio on premises?

Humio is a flexible log management solution. We have customers that use our
cloud solution, on-prem solutions, and hybrids of both options. Consider what
works best for your application and organization and we are happy to help you
ind the set up to best suit your needs.

## If we use other logging like Elastic Stack, is it easy to move over to Humio?

Moving to Humio is easy! We have several common integrations to bring your
logs into Humio and we even have a guide on
[moving from Elastic Stack to Humio]({{< ref "moving-from-elastic-stack.md" >}}) -
it's as easy as following a few steps to getting your logs flowing.

## Is Humio container ready?

Humio was built with containerization in mind! With integrations and
existing setup for Kubernetes, Humio is a solution focused on modern
deployment solutions.

## What common log shipping solutions does Humio use?

While this list is not exhaustive, Humio recommends [Beats]({{< ref "integrations/data-shippers/beats/_index.md" >}}),
[Logstash]({{< ref "logstash.md" >}}), or [Ryslog]({{< ref "rsyslog.md" >}}) for
shipping your logs. We can take advantage of other solutions, but these are the
most common we've experienced.

## Does Humio integrate with any notification systems?

Humio integrates with several common notification methods
including [email, Slack, and external services like OpsGenie]({{< ref "alerts/_index.md" >}}).
If you need Humio to work with your particular notification system, contact our support team!

## What happened to "Dataspaces"

"Repository" is the new term. What used to be a "dataspace" in Humio is now a [Repository]({{< ref "repositories.md" >}}).

The HTTP API includes the path `/api/v1/dataspaces/$REPOSITORY_NAME/` to be compatible with existing clients.
In this context, the `$REPOSITORY_NAME` variable is the name of the repository. (It used to be the name of the dataspace).

## Can I run Humio on IPv6-only, IPv4-only or both?

Humio runs on either or both IP versions, depending on what you specify using `HUMIO_JVM_ARGS`. By default the process binds on both IPv4 and IPv6.

If you use the Docker images provided by Humio for Kafka and Zookeeper, or run the "humio/humio" image that includes both of them,
you need to make sure those processe also get the same options regarding IP protocol as the Humio process.

IPv4 Only:

```
HUMIO_JVM_ARGS=-Djava.net.preferIPv4Stack=true
KAFKA_OPTS=-Djava.net.preferIPv4Stack=true
ZOOKEEPER_OPTS=-Djava.net.preferIPv4Stack=true
```

IPv6 Only:

```
HUMIO_JVM_ARGS=-Djava.net.preferIPv6Addresses=true
KAFKA_OPTS=-Djava.net.preferIPv6Addresses=true
ZOOKEEPER_OPTS=-Djava.net.preferIPv6Addresses=true
```

## How do I detect when a host (log source) stops sending logs

In Humio you can detect when a host or some other log source stops
sending logs using the `now()` function and `groupby`:

```
groupby(host, function=max(@timestamp, as=@timestamp))
| missing:=(now()-@timestamp)>(5*60*1000)
| missing=true
```

The above query shows a line for each host that we have not heard from
in the last 5 minutes (timestamps in Humio are in milliseconds). You
should run the query as a live search in a time interval that is
longer than you "missing" threshold - when the last log from a log
sources is older than your search time interval the log source will
disappear from the result.

## A menu item in the Humio UI appears to be missing

We have had reports of browser extensions such as "uBlock Origin" and
other plugins that modify the page mistakenly by removing or hiding parts of
the UI. Try disabling the plugin(s) on the pages of Humio if you
suspect something is wrong. And please notify the Humio team when it
happens, as we can likely change the specific element in the Humio
page to not get filtered.

## The Humio UI is responding slower than expected

We have had reports of browser extensions such as "LastPass" for
Chrome and other plugins that scan and/or modify the page slow down
the UI a lot: The extensions think they need to inspect the full page
for every change to the page, and the Humio UI changes the page
content a lot... Please disable all browser plugin(s) on the pages of
Humio.

## Can I set the license key via the API

Yes, via our [GraphQL API]({{< ref "/api/graphql">}}) (you need to
be a root user). Below is a curl example to get you going.

Example:

```
HUMIO_BASE_URL=<example: https://example.com>
API_TOKEN=<you can find this in your account details>
LICENSE=<your license string>

curl ${HUMIO_BASE_URL}/graphql \
-H "Authorization: Bearer ${API_TOKEN}" \
-H "Content-type: application/json" \
-d @- << EOF
{"query":"mutation {updateLicenseKey(license: \"${LICENSE}\") {__typename}}"}
EOF
```

## How is timezones handled when sharing queries with people in different timezones

The browser sends its timezone to the server and that determines
cutoff for say `timechart(span=1day)`. The timezone is not embedded
in the URL, so if a query is shared across timezones the day cutoff
will differ.

The `bucket()` and `timechart()` functions lets you specify an
explicit timezone (`timechart(..., timezone=Z)`). This will overrule
the timezone of the browser. Note, that the x-axis of time charts is
still shown in local time.

## Can I send multiline events to Humio

Yes! Humio does support receiving events with multiple lines.
What Humio does **not** support is correlating multiple events into a single multiline event, which means that it is up to the log shipper to detect wether an event spans across multiple lines.

Filebeat has support for detecting [multiline events]({{< ref "/integrations/data-shippers/beats/filebeat.md#multiline-events">}}).
