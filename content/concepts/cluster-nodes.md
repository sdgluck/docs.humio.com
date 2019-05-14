---
title: Cluster Nodes
aliases: ["/ref/cluster-nodes"]
---

Humio can run as distributed system in a cluster.

Root users can access the Cluster Node Administration Page in the UI from the
account menu.

### Common Node Tasks

- [Adding a cluster node]({{< ref "/administration/adding-a-node.md" >}})
- [Removing a cluster node]({{< ref "/administration/removing-a-node.md" >}})
- [Updating node's Humio version]({{< ref "updating-humio.md" >}})

## Node Roles

All nodes in the cluster run the same software that can be configured to assume a
combination of different logical roles:

- [Arrival Node]({{< ref "#arrival-node" >}})
- [Digest Node]({{< ref "#digest-node" >}})
- [Storage Node]({{< ref "#storage-node" >}})

A node can have any combination of the three roles, and all play a
part in Humio's data ingest flow.

Below is a short overview of the node types. For more detailed explanation
refer to the [data ingest flow documentation page]({{< ref "ingest-flow.md" >}}).

It can be beneficial to specialize to only assume one role since that allows
you to better tune cluster performance.

### Arrival Node {#arrival-node}

An _arrival node_ is a node that is responsible for servicing:

- The User Interface (web)
- HTTP API
- Parsing Incoming Data

The arrival node receives data from external systems and parses it
into [events]({{< ref "events.md" >}}) and passes the data on to the [digest nodes]({{< ref "#digest-node">}}).

The _arrival node_ role is a logical role in the sense that there is no special
configuration option that "flags" node as "Arrival Node", it simply means
that incoming traffic is directed at this node (maybe by means of a load balancer,
DNS, or similar), and that it is "a node that is not configured to store data or execute queries".

Arrival nodes are usually the only nodes exposed to external systems and are placed
behind a load-balancer.

### Digest Node {#digest-node}

A _digest node_ is a node responsible for:

- Constructing segment files for incoming events (the internal storage format in Humio)
- Executing the Real-Time part of searches.

Once a segment file is completed it is passed on to [storage nodes]({{< ref "#storage-node">}}).

Digest nodes are designated by adding them to the cluster's [Digest Rules]({{< ref "digest-rules.md" >}}).
Any node that appears in the digest rules is a Digest Node.

### Storage Node {#storage-node}

A _storage node_ is a node that saves data to disk. They are responsible for:

- Storing Events (Segment files constructed by [digest nodes]({{< ref "#digest-node">}}))
- Executing the historical part of searches (the most recent results are handled by digest nodes)

The the data directory of a storage node is used to store the segment files.
Segment files make up for the bulk of all data in Humio.

Storage nodes are configured using the cluster's [Storage Rules]({{< ref "storage-rules.md" >}}).
Any node that appears in the storage rules is considered a Storage Node. A node that was previusly
in a storage rule can still contain segment files that are used for querying.

The Storage Rules are used to configure data [replication]({{< ref "storage-rules.md#replication" >}}).

## Node Identity {#uuid}

A cluster node is identified in the cluster by it's UUID. The UUID is
automatically generated the first time a node is started. The UUID is stored
in `$HUMIO_DATA_DIR/cluster_membership.uuid`. When moving/replacing a node you
can use this file to ensure a node rejoins the cluster with the same identity.
