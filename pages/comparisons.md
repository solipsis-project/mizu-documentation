---
title: "Comparisons"
permalink: comparisons.html
summary: How Mizu differs from, draws inspiration from, and builds on other tools in the decentralized application space.
---

# What Mizu is not

## Other decentralized application platforms

### Freenet (and Locutus)

Mizu's closest comparison is to Freenet, another peer-to-peer communications platform. (and its in-development successor, Locutus)

Mizu and Freenet/Locutus attempt to solve the same problem: building decentralized applications on top of a distributed data store. And in both systems, an application's key is a description of the application's behavior (or a content-hash of such a description.) Freenet shares Mizu's goal of making documents be long-lived without needing a centralized host. Freenet and Mizu also share the goal of having an API for the development of decentralized applications on the platform. But each system has different priorities and tradeoffs.

Freenet prioritizes user privacy and plausable deniability: that is,

- Users shouldn't have to expose their IP address to communicate with other users.
- Publishers and subscribers don't (and often can't) communicate directly, instead routing traffic through other nodes.
- It is impossible by design for the operator of a node to read traffic that passes through their node.

In order to protect user anonymity, both requests and responses are propogated through a "small-world" network where each user only knows a small number of other peeers.

However, there's a trade-off between protecting user anonymity this way and "not being a mule" for other users' data. Being able to publish content anonymously is an important goal, but if it requires users in the network to carry and make available traffic whose contents they can't read, users may feel a sense of liability. An important tenet of Mizu is that the content a user caches and propagates is the exact set of data that they *choose* to cache and propagate.

Despite these different priorities, There are things that Mizu can learn and borrow from Freenet and Locutus. Freenet's "darknet mode", where users only connect to an explicitly specified list of peers and all data must be passed along intermediate peers (albeit encrypted), is an intriguing alternative opt-in mode of operation, albiet one that is incompatible with the "anti-mule" guarentee listed above. We consider the possibility for this to exist as an alternate mode of operation for Mizu, but it's not a priority. 

Freenet has a tendency to "forget" data that is not retrieved regularly, as nodes purge old data to make space for new data. This is a consequence of Freenet's anonymity-first design, where files are "pushed" to the network and then no longer necessarily provided by their original host. Mizu tries to protect against this by borrowing the concept of "pinning" from IPFS: pinned resources will never be garbage collected by a node.

Lastly, Freenet is built on a couple of very simple primitives that are deceptively powerful but difficult to write apps for. Locutus, on the other hand, allows users to write arbitrary, Turing complete contracts which can do anything, but make the network much harder to secure. Mizu takes a middle ground between these two extremes: apps in Mizu make queries into a distributed database, using a deliberately not Turing complete query language. This makes Mizu's design more complicated than Freenet, but easier to write apps for, while also avoiding the pitfalls of compelling nodes to execute arbitrary code, like Locutus does.

(Also, as a noteworthy technical detail; Locutus takes a mutable imperative approach: the key describes the structure of the application's state and how that state is allowed be mutated over time. Mizu, on the other hand, takes a declaritive approach: the key describes how to compute the current state of the application by merging messages published to the network. These two approaches are equally expressible.)

### OrbitDB

[OrbitDB](https://orbitdb.org/), like Mizu, attempts to build a database on top of IPFS by building the database out of Conflict-Free Replicated Data Types (CRDTs).

It has a useful API for specifying Identities which can have varying degrees of read and write access to the databases.

There's already a publishing platform built on OrbitDB called [Vitriol](https://gitlab.com/vitriolum/vitriol-web) but it hasn't been updateds in 3 years and the reference web client appears to be down.

The main difference between OrbitDB and Mizu is that OrbitDB makes available specific data structures built on CRDTs (for example, an append-only log, or a key-value store), whereas in Mizu, developers achieve the same functionality by building on a single basic primitive: a triplestore DHT that clients can run queries on.

### I2P

I2P started as a fork of Freenet.

TODO: Understand how I2P differs from Freenet, and how I2P can inform the design of Mizu.

### Ethereum

Given that it's possible to make decentralized apps on Mizu, it may be tempting to compare it to Ethereum. But the two networks are fundamentally different in important ways.

An important part of Ethereum is the idea that there is a single agreed-upon order for transactions: that every client is able to reach a consensus on not just the order of the blocks in the chain, but on the order of transactions in those blocks. This is important to prevent double-spend attacks. More generally, if two transactions published on Ethereum are incompatible, there exists a mechanism for determining which transaction should appear on the blockchain.

This isn't a concern with Mizu because in Mizu, there's no such thing as an incompatible transaction, or a double spend. The datastore for Mizu is a Conflict-Free labeled graph. Transactions in Mizu are commutative, associative, and idempotent, meaning that they can be processed by the client in any order, even any number of times, and the resultant state of the datastore will be the same.

This has some consequences for Mizu. For starters, it means that certain types of apps are not possible on Mizu: particularly apps that rely on transferrable tokens such as cryptocurrencies or NFTs. Since there is no ordering of transactions in Mizu, there is no mechanism to prevent a situation where a user publishes a transaction relinquishing ownership of an asset... and then continues to make transactions based on their prior ownership of the asset. Any new client would have no way of verifying whether these new transactions were made before or after the relinquishment and is forced to accept these transactions as valid.

However, there's a major upside to not requiring consensus on transaction order: unlike blockchains, Mizu does not have transaction fees. Transaction fees in blockchains are necessary to reward miners who are not otherwise motivated to include stranger's data in their mined blocks. Mining in blockchains are necessary because the chain with the most work behind it is used to reach consensus about the order of instructions. But Mizu does not have miners, or mined blocks, or an order of instructions. Mizu does not need to incentivise nodes to replicate data, because the replication is being done by people who already want the data they're replicating to be available.

Mizu is built on the idea that if a content creator has an audience that willing to replicate their Stream, their Stream will be able to survive indefinitely in the network. It's better to think of Mizu as an improvement to bittorrent or IPFS than a blockchain: it just happens to be expressive enough to allow the creation of decentralized apps.

### ZeroNet

ZeroNet also allows mult-user sites, but requires the site owner to be online to add auth addresses to the list of signers for files. (This can be skipped with an authorization provider feature.)

## Other Distributed Hash Tables.

### IPFS

IPFS is a protocol for peer-to-peer file sharing, built on the libp2p library, that uses content addressable hashes to identify static content.

A Mizu node is also a full IPFS node, and Mizu data blobs that reference IPFS URIs are the defacto way to reference large static data within a Mizu message. However, IPFS only supports static content. Mizu is an attempt to build support for dynamic content on top of IPFS.

### Git and Dolt

Git is a peer-to-peer version control system that has many similarities to Mizu:

- Both represent the entire history of a mutable dataset.
- Both use a content-addressed hashtable to store its data.
- Mizu inherits Git's push/pull semantics: Changes propogate through Mizue nodes by explicit pushes and pull requests.
- Both can "branch", with different branches having a different view of the underlying data, and these branches can then be re-merged.

Mizu has even more similarities with Dolt, an SQL database with a Git-like API:

- Both represent a database that clients can run queries on.

However, Mizu differs from Git and Dolt in one critical way: Mizu cannot have merge conflicts. Merge conflicts happen when two branches have incompatible changes which must be manually resolved before merging. Because Mizu is built using Conflict-Free Replicated Data Types (CRDTs), there is always an intuitive, well-defined merge behavior for any two branches: manual merge conflict resolution is never required.

### Hypercore / Hyperdrive

TODO: Compare and contrast Mizu with Hypercore and its related protocols.

## Other Content Management tools

### Hydrus

Hydrus is a tool for organizing media collections. It uses a client-server model, supporting arbitrary plugins to run queries and download updates from various backends. This includes user-run repository servers, allowing for file-sharing and collaborative tagging and organizing.

Hydrus is designed for a specific type of content (append-only media collections), and it focuses on creating a robust experience for clients who manage this content. It's less of a platform and more of a tool for organizing data that users already have access to said data. The repository servers enable content sharing, but in a centralized manner: some centralized entity must shoulder the cost of operating the server.

Hydrus has a standard interface for defining import tools, making it easy for content from other platforms to be imported into Hydrus's ecosystem. If Mizu also supported this interface, it would make it easy to create Mizu streams that mirror content from other sites.

### Perkeep

Perkeep is a tool for creating personal data backups that eschews conventional directory structure for a content-addressed database: all content is tagged and added to the database where it can be queried. A perkeep repo is designed for use by a single user, and although repos can be cloned (in part or in whole), it's not really designed for networking or synchronization between peers.

One area Mizu may be able to draw from is Perkeep's efforts to tackle the "json canonicalization" problem of ensuring that an object blob (in this case, json) has a single canonical representation for signing and data-addressing. (https://perkeep.org/doc/json-signing/) If Mizu ends up needing to content-address json, we may be able to build on top of this work. However, the more likely solution for Mizu is to represent all structured data internally using a non-human readable format like IPFS's dag-cbor, and only parsing from / rendering to JSON at the endpoint.

### Bup

Bup (https://github.com/bup/bup) is a content-addressible datastores designed for single users to maintain private backups of their data. Bup is a content-addressible datastore built on top of git. It supports named versioned archives. Like Perkeep, Data is backed up to a single server and is not designed to be publicly accessible or sharded.


## Other p2p networks.

### Tor

Tor (The Onion Router) is a network, but not a platform: Tor can provide access to Internet connected servers in a private manner (including access to servers that are not otherwise accessible), but a Tor exit not always connects to a conventionally hosted server in the end. Using Tor does not remove a publisher's reliance on centralized hosting or costly self-hosting.

### Osiris

Osiris is a tool for creating peer-to-peer web portals, where each user of the portal hosts the entire site. Unfortunately, the project is closed-source and not under active development. It appeared to allow mutable content generated by portal users, but the exact capabilities are not clear.

## Other tools that may be similar to Mizu

The tools listed here may be able to inform the design of Mizu. However, more research is required.

### Maelstrom

### Blockstack

### satellite.earth

### m-ld

### GNUNet

### GUN

### [Quanta](https://quanta.wiki/)

### [Kindelia](https://github.com/Kindelia/Kindelia)
