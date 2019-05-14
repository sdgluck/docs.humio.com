---
title: Sankey
aliases: ["/ref/charts/sankey"]
weight: 400
---

The _Sankey_ Widget can render results as a 2 level sankey diagram. It is good at displaying
flows between entities e.g. network flows from one IP to another.

## Input Format

| Field         | Type    | Description                                                                                                     |
|---------------|---------|-----------------------------------------------------------------------------------------------------------------|
| `source`      | string  | The ID of the source node (Left Side). This value will also be used as the label of the node.                   |
| `target`      | string  | The ID of the target node (Right Side). This value will also be used as the label of the node.                  |
| `weight`       | number  | The value that is used to determine the size of the edge between `source` and `target`, the value is scaled automatically. This could e.g. be used to represent the traffic between to IP addresses. |

## Usage

The Sankey widget is most easily used with its companion query function {{< function "sankey" >}}. But
can easily be used simply by ensuring the input fields are named as expected.

### Example 1: Network Traffic

Here we are using the companion query function to visualize data flowing from
`src_ip` to `dst_ip`. We use the {{< function "sum" >}} to calculate the total
number of bytes sent - where `pkt_size` is a field containing the packet size.

```humio
sankey(source=src_ip, target=dst_ip, weight=sum(pkt_size))
```

### Example 2: Thread Usage

In some situations it might be easier to produce the input data by hand instead
of using the companion function.

```humio
rename(class, as=source) | rename(thread, as=target) | groupBy([source, target], function=count(as=weight))
```

In this case we want to visualize which classes use which threads in a service.
We need to rename the `class` and `thread` fields to match the expected input;
we do this using the {{< function "rename" >}} function and to produce `weight`
fields, we make sure that the function we use in the {{< function "groupBy" >}}
names it's result `weight`.
