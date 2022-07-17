# Mizu: RSS meets Git meets P2P file sharing built on top of [IPFS](https://ipfs.io), [libP2P](https://libp2p.io), and [json-rql](https://json-rql.org/)

Mizu is a decentralized datastore with pubsub features, allowing subscription to content feeds that still function even if the source of the content goes down. Mizu is even expressive enough to allow for the creation of decentralized apps for creating and interacting with content published on the Mizu network.

The core unit of data in Mizu is a Message, which is a row of a triplestore database, stored in a distributed hash table and shared with other nodes in the network on request.

Mizu is different from other distributed databases: instead of proactively trying to replicate the entire database on each node, only data that is specifically requested by other users is replicated. This makes Mizu more like IPFS but for mutable data: only the data you actually need is on your node.

For an example of what can be done with Mizu, check out [Building an Image Gallery in Mizu](./image_gallery_example).

For a more in-depth look at Mizu, check out [Concepts](./concepts), [Data Model](./data_model), [Tutorial](./tutorial), and [Streams Tutorial](./streams_tutorial).

To understand the similarities and differences between Mizu and other similar technologies, check out [Comparisons](./comparisons)

# Use Cases

## Online Use Cases

Mizu can be used to reduce hosting costs by leveraging peer to peer file sharing: subscribers to content share updates with each other. This allows the data to change and evolve and still distribute updates to all subscribers. Producers no longer need to be married to a centralized platform or their rules in order to gain the benefits of cheap file hosting.

## Offline Use Cases

A Mizu node can run entirely "offline", never sending any data into the network. This still offers all the advantages of Mizu's platform when operating on local private data. This still has many uses, such as storage and organizing of personal files, or creating one-click backups of websites.

## Private Swarm Use Cases

A node can also operate as part of a private swarm, with data sent encrypted over the network. This makes it possible to use Mizu to sync backups across multiple machines, or share data privately.
