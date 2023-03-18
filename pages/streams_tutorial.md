---
title: "Streams Tutorial"
permalink: streams_tutorial.html
summary: "The specification for Streams: apps built on Mizu."
---

<div class="notyet">

Some of the examples describe behavior that is not yet implemented. Those examples are in red boxes like this one. They should be treated as an informal specification of how Mizu will behave once complete, but it is not currently possible to replicate the behavior of these examples.

</div>



## Subscriptions

<div class="notyet">

Publishing records and running queries is neat and all, but so far everything we've discussed could be done more easily on a traditional local database. Where Mizu really shines is in how it enables data to be shared with others via the use of Subscriptions and Streams.

A subscription is a subset of the Mizu database, containing all the records necessary to resolve a query, organized in such a way that two clients in the network can quickly and efficeintly exchange missing records. Effectively, a Subscription tracks a query, caching the results and making it easier to receive new data that matches the query and share that data with others..

You can create a subscription with the `mizu sub add` command. The command accepts a query as its only mandatory parameter, which will be published to Mizu if it isn't already.

Once a track is created, it's assigned a CID which is the same as the CID of the associated query. Updates to the query can be manually requested and sent to other network nodes using the `mizu sub pull` and `mizu sub push` commands. These commands behave similarly to `git pull` and `git push`: the command specifies a remote, target, or a remote target is set in the mizu's configs.

But what if you don't know the address of a client that can provide you with the most up-to-date records for a query? Fortunately, network nodes are able to find each other over the libp2p network. The `mizu sub update` command uses libp2p to find other clients with the same subscription and request updates from them.

Finally, `mizu sub remove` can be used to remove a subscription.

</div>

## Streams

<div class="notyet">

Subscriptions are a powerful tool for caching and propagating specific sets of records, but they're not user friendly. That's where Streams come in.

A stream is an object, published into the Mizu datastore, that contains:

- One or more queries.
- A JSX-like template string describing how to embed the results of the associated queries in a webpage.

A Mizu Gateway server can render a stream as a webpage. This webpage may contain calls to the Mizu web API, allowing users to interact with application by publishing additional messages to the datastore.

</div>