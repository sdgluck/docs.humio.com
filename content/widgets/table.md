---
title: Table
aliases: ["/ref/charts/table", "/ref/charts/table-view"]
weight: 300
---

The _Table_ widget can be used to display data in rows and columns.

## Input Format

The result of any Humio query can be displayed in a table. It is best used
with output that has a limited and predefined number of fields, unlike for instance
raw events which can produce a huge number of columns and slow down the UI.

## Usage

The table widget is best used with aggregate functions like {{< function "groupBy">}}
or {{< function "table" >}}. The {{< function "table" >}} can help sort the columns
since fields/columns will be displayed in the order that they are provided to the function.

### Example: A List of common errors

Assume we have a service producing logs like the ones below:

```
2018-10-10T01:10:11.322Z [ERROR] Invalid User ID. errorID=2, userId=10
2018-10-10T01:10:12.172Z [WARN]  Low Disk Space.
2018-10-10T01:10:14.122Z [ERROR] Invalid User ID. errorID=2, userId=11
2018-10-10T01:10:15.312Z [ERROR] Connection Dropped. errorID=112 server=120.100.121.12
2018-10-10T01:10:16.912Z [INFO]  User Login. userId=11
```

We want to figure out which errors occur most often and show them in a table
on one of our dashboards.

We can do a query like:

```humio
loglevel = ERROR |
groupBy(errorID, function=[count(as=Count), collect(message)]) |
rename(errorID, as="Error ID") |
table(["Error ID", message])
```
Counting the number of errors bucketed by their `errorId`. Since we also
want to show a human readable message in the table and not just the ID,
we include the function {{< function "collect" >}} which ensures that the value
of the `message` field makes it through the `groupBy` phase (which otherwise only
includes the series field (`errorId`) and the result of the aggregate function (`Count`)).

Since we want out table to look nice on the dashboard we rename the `errorID` field
to `Error ID` as this will be the header in out table.

Finally we use the {{< function "table" >}} to ensure the order of the columns.
