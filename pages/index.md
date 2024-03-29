---
title: "Mizu"
keywords: landingpage
permalink: index.html
summary: A brief introduction to the Mizu platform.
---

NOTE: Mizu is unreleased software. These tutorials are an informal specification of how Mizu *should* function, not a description of currently implemented behavior.

If you want to get involved with Mizu:

- [Check out the project on GitHub](https://github.com/users/solipsis-project/projects/1/views/1)
- [Look at the code](https://github.com/solipsis-project/mizu-client)
- Join the Matrix space at [mizu:matrix.org](https://matrix.to/#/#mizu:matrix.org)
- [Join the Discord server](https://discord.gg/kEW32kWsaA)
- Drop me an email at the.solipsis.project@gmail.com

# Mizu: The decentralized database built on [IPFS](https://ipfs.io), [libP2P](https://libp2p.io), and [json-rql](https://json-rql.org/), designed for connecting publishers and subscribers.

Think RSS meets Git meets Bittorrent.

Like IPFS, users of the Mizu network can request static content based on its Content-ID (or CID). In addition, because Mizu is a database, applications built on Mizu can make database queries, asking their network peers for necessary records, including the most up-to-date information.

This allows for the creation of pubsub feeds (akin to RSS), where content subscribers can share updates with each other, increasing availability and reducing bandwidth costs for the publisher, all without forcing publishers to rely on centralized content hosting services. Mizu is even expressive enough to allow for the creation of decentralized apps for creating and interacting with content published on the Mizu network.

Mizu is different from other distributed databases: instead of proactively trying to replicate the entire database on each node, only data that is specifically requested by other users is replicated. This makes Mizu more like IPFS but for mutable data: only the data you actually need is on your node.

For an example of what can be done with Mizu, check out [Building an Image Gallery in Mizu](./image_gallery_example).

For a more in-depth look at Mizu, check out [Concepts](./concepts), [Data Model](./data_model), [Tutorial](./tutorial), and [Streams Tutorial](./streams_tutorial).

To understand the similarities and differences between Mizu and other similar technologies, check out [Comparisons](./comparisons)

# Use Cases

## Online Use Cases

Mizu can be used to reduce hosting costs by leveraging peer to peer file sharing: content subscribers share updates with each other. This allows the data to change and evolve and still distribute updates to all subscribers. Producers no longer need to be married to a centralized platform or their rules in order to gain the benefits of cheap file hosting.

## Offline Use Cases

A Mizu node can run entirely "offline", never sending any data into the network. This still offers all the advantages of Mizu's platform when operating on local private data. This still has many uses, such as storage and organizing of personal files, or creating one-click backups of websites.

## Private Swarm Use Cases

A node can also operate as part of a private swarm, with data sent encrypted over the network. This makes it possible to use Mizu to sync backups across multiple machines, or share data privately.
