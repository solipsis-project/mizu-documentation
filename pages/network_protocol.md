---
title: "Network Protocol"
permalink: network_protocol.html
summary: A specification of how a Mizu node can receive updates from other nodes in the network.
---

(For more of the intent behind the design, and an gentler introduction to Merkle/Prolly trees, check out [my blog post on the topic](https://www.solipsis-project.com/mizu-network-primer/))

An application built on Mizu must define a set of **topics** that network users can query for updates on. For instance, in a blog application, this might include a topic for each author, listing that author’s posts, and a topic for each tag, including posts that contain that tag. Topics take the form of simple SPARQL queries.

Each node maintains a [Prolly Tree](https://docs.dolthub.com/architecture/storage-engine/prolly-tree) for each topic it's tracking, containing all of the records pertaining to that topic. By comparing Prolly Trees starting from the root, nodes are able to quickly and efficiently compute which records that they are missing and need to request from their peers.

If the user wants to make a more specific query, the client needs to choose a topic query that may be overbroad, and then do additional filtering on the results.

TODO: describe how an application defines its topics.

TODO: for additional user privacy, consider allowing users to query a topic using an opaque ID instead of a SPARQL query. This works so long as the requesting and providing nodes know which query the topic ID corresponds to.

The following is the description algorithm for pulling updates on a topic:
- A node (the requester) wants to poll the network for new records that match a topic.
- The requester uses libp2p’s pubsub module to send a request to other nodes that are also tracking the topic.
- Those other nodes (the responders) respond with the content hash of their prolly tree for that topic. (The nodes may also respond with other useful data, such as the number of records stored for that topic, - or the timestamp of when they last processed a new record.)
- The requester determines which nodes may have records that the requester has not yet seen, because the content hash is not in its hash table.
- The requester messages each of those nodes directly with a new request with the following parameters (based on `git`’s fetch protocol):
  - “have”: a list of content hashes that represent the requester’s most recently acquired records.
  - “need”: a list of content hashes that the responder should provide. (For the first request, this will just be the head of the responder’s prolly tree.)
- The responder attempts to compute the set of objects it must return: this includes the objects with the requested content hash, as well as any objects that those objects reference, and so on, until reaching one of the content hashes listed in the requesters “have” parameter. The responder is allowed to return extra objects that weren’t requested (like git, objects may be “packed” and the entire pack is transmitted.) If the number of objects calculated is too high, the responder may choose to limit the number of objects in the response, and wait for the requester to send an additional request. A responder may rate-limit their responses.
- The requester may make additional requests until it has every record in the responder’s prolly tree.

Some notes:
- The responding nodes do not keep an open connection with the requester: they respond to the request and then close the connection. The requester can send subsequent follow up requests if they need more records.
