---
title: Creating a Parser
weight: 200
aliases: ["/ref/creating-a-parser", "parsers/json-parsers", "parsers/regexp-parsers"]
---

A parser is a piece of code that transforms incoming data into [events]({{< ref "events.md" >}}).
Humio has built-in parsers for common log formats like `accesslog`. But if none of them fit your
data format or you want to extract more fields, do transformations on the data or assign datasources,
you can build your own parser.

In this guide we will go through the steps of creating a parser from scratch.

{{< figure src="/pages/parsers/code-view.png" width="100%" caption="Humio's parser editor showing a simple parser and two test cases." >}}

## Step 1. Creating a new parser

Go the the `Parsers` section of the repository you want to create the parser for.
Then click the `New Parser` button and give it a name. _The name is important since it is what the API uses to uniquely identify the parser._


## Step 2. Writing a parser

Once you have created your parser you will be presented with a code editor.

The programming language used for creating a parser is the same as you
use to write queries on the search page. The main difference between writing a
parser and writing a search query is that you cannot use aggregate functions
like e.g. {{< function "groupBy" >}} as the parser acts on one event at a time.

The input data is usually log lines or JSON objects, but could be any text format
like a stack trace or CSV.

When sending data to Humio the text string for the input is put in the field called `@rawstring`.
Depending on how data is shipped to Humio, other fields can be set as well.    
For example when sending data with Filebeat, the fields @host and @source will also be set. And it is possible to add more fields using the [Filebeat configuration](/integrations/data-shippers/beats/filebeat).

Let's write our first parser!

### Creating an event from incoming data

It is the parser's job to convert the data in `@rawstring` into an event.
That means the parser should:

1. Assign the special `@timestamp` and `@timezone` fields.
1. Extract additional fields that should be stored along with your event.

Let's take a look at a couple of parsers and try to understand how they work.


#### Example: Parsing Log Lines {#log-lines}

Assume we have a system producing logs like the following two lines:

```
2018-10-15T12:51:40+00:00 [INFO] This is an example log entry. id=123 fruit=banana
2018-10-15T12:52:42+01:30 [ERROR] Here is an error log entry. class=c.o.StringUtil fruit=pineapple
```

We want the parser to produce two events (one per line) and use the timestamp of each line as
the time at which the event occurred - i.e. assign it to the field `@timestamp` and `@timezone`.

To do this we could write a parser like:

```humio
// Create field called "ts" by extracting the first part of each
// log line using a regular expression. See https://docs.humio.com{{< ref "query-functions/_index.md#regex" >}}.
// The syntax `?<ts>` is called a "named group". It means whatever is matched will produce
// a field with that name - in this case a field named "ts".
/^(?<ts>\S+)/ |

// To set the timestamp for the event use the function parseTimestamp
// parseTimestamp uses the field ts we just extracted and parses the string value into a timestamp. It sets the the timestamp for the event by setting the field @timestamp.
// Note the timezone is also parsed and set using the field @timezone
// See https://docs.humio.com{{< ref "query-functions/_index.md#parseTimestamp" >}}.
parseTimestamp("yyyy-MM-dd'T'HH:mm:ss[.SSS]XXX", field=ts)
```

This parser assigns the `@timestamp` and `@timezone` fields, which is the minimum you we can
do to create events from the examples above. At this point we have a fully valid parser.

But the two log lines actually contains more useful information, like the `INFO` and `ERROR` log levels.
We can extract those by extending the regular expression:

```humio
//first the timestamp is extracted. Then the regex matches the loglevel. For example [INFO] or [ERROR]
/^(?<ts>\S+) \[(?<loglevel>[^\]]+)\]/ |
@timestamp := parseTimestamp("yyyy-MM-dd'T'HH:mm:ss[.SSS]XXX", field=ts) |
// The next line finds key value pairs and creates a field for each
kvParse()
```

The events will now have a field called `loglevel`.
 
At the bottom of the parser we also added the function [kvParse]({{< ref "query-functions/_index.md#kvParse" >}}).
This function will look for keyvalue pairs in the log line and extract them into fields, e.g. `id=123` `fruit=banana`.

#### Example: Parsing JSON {#json}

We've seen how to create a parser for unstructured log lines. Now let's create a
parser for JSON logs based on the following example input:

```json
{
  "ts": 1539602562000,
  "message": "An error occurred.",
  "host": "webserver-1"
}
{
  "ts": 1539602572100,
  "message": "User logged in.",
  "username": "sleepy",
  "host": "webserver-1"
}
```

Each object is a separate event and will be parsed separately, just like with
unstructured logs.

The JSON is accessible as a string in the field `@rawstring`. We can extract fields
from the JSON by using the {{< function "parseJson" >}} function.S
It takes a field containing a JSON string (in this case `@rawstring`)
and extracts fields automatically, like e.g:

```humio
parseJson(field=@rawstring) |
@timestamp := ts |
@timezone := "Z"
```

This will result in events with a field for each property in the input JSON,
e.g. `username` and `host`, and will use the value of `ts` as the timestamp. If the timestamp is a string it can be parsed using the {{< function "parseTimestamp" >}} function

## Next Steps

Once you have your parser script created you can start using
it by [assigning it to Ingest Tokens]({{< ref "assigning-parsers-to-ingest-tokens.md" >}}).

You can also learn about how parsers can help speed up queries by [assigning tags]({{< ref "tagging.md" >}}).

## Reference

### Named Capture Groups {#named-groups}

Humio extracts fields using _named capture groups_ - a feature of regular expressions
that allows you to name sub-matches, e.g:

```humio
/(?<firstname>\S+)\s(?<lastname>\S+)/
```

This defines a regex that expects the input to contain a first name and  a last name. It then extracts
the names into two fields `firstname` and `lastname`. The `\S` means
any character that is not a whitespace and `\s` is any whitespace character.
