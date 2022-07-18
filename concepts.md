# Core Concepts

## Mizu URIs

A Mizu URI is made up of a path, the fragment, and the parameters. The format of a mizu URI is:

https://mizu.stream/{action]/{cid}/{path}[#{fragment}][?{parameters}]

Note that just because a Mizu URI begins with "https://mizu.stream/" doesn't mean that a DNS lookup for mizu.stream will necessarily occur, or that the https protocol will be used to resolve the URI. It's merely an identifier that allows Mizu URIs to be easily identified, chosen to coincide with the Mizu reference gateway at https://mizu.stream (which *does* use DNS and HTTPS to resolve requests).

### Cid

Every message published to Mizu is assigned a CID, which is a [multihash](https://multiformats.io/multihash/) of the message content. This portion of the Mizu URI identifies a message.

### Path

The path portion of a Mizu URI is used to identify the component of the message that is the target of the action. A large message might contain multiple queries, streams, or templates. 

### Action

An action that determines how to interpret the message specified by the path. There are two types of actions: static actions, which always resolve to the same response, and dynamic actions, which can resolve to different responses depending on the current state of the network and local database. The different allowed actions are:

#### Static actions
- `message`: Return the raw data of the message
- `update`: Interpret the message as a delta applied to a previous message

#### Dynamic actions
- `query`: Evaluate the message as a query, returning the result.
- `stream`: Load the message as a Stream (Mizu's name for a decentralized app).

### Fragment

Often, a Mizu action resolves to a structured object. We may want to identify only part of that object. The fragment tells us which part.

### Parameters

Additional paramaters that may affect how the message is evaluated. Not used for all URIs.

## Publishing Messages

The core unit of Mizu is the message. A message is simply a piece of structured data that is stored in the Mizu datastore. Mizu uses IPFS's [DAG-CBOR](https://ipld.io/docs/codecs/known/dag-cbor/) codec to store data: we choose this because (unlike JSON) a piece of data has a single canonical DAG-CBOR representation, and IPFS already has good support for it. User-facing endpoints accept and emit JSON, and parse from / render to JSON at the endpoint.

Some message fields start with a $. These are reserved fields, and each one has special semantics, either purely for message validation, or for one or more Mizu actions. Reserved fields are validated when a message is published or when received by a node: thus, all published messages can be assumed to obey certain constraints specified in this document. Names starting with a $ that are not listed here are reserved for later use and cannot be used in messages: clients must reject messages containing any such fields.

The most common reserved field is `$signatures`, which contains one of more key-digest pairs that sign the rest of the object. If any of the signatures don't correctly sign the object, then the message containing the object is invalid and will not be accepted by nodes in the network.

For the full list of reserved fields and their meanings, see [Reserved Fields](./reserved_fields)

## Streams

If all we're doing is publishing structured data to a distributed hash table, we don't need Mizu. We can already do all that and more with just IPFS.

What Mizu lets us do is run queries on that data, and present that data in a structured way, called Streams.

A stream is really nothing more than a packaged set of queries into the database, alongside metadata (like links to images, stylesheets, and other resources). This makes it extremely flexible.

The simplest stream is represented by just a single user (a public key), which means that only messages signed by the person with the corresponding private key belong in the stream. But more complicated streams are possible, such as:

A stream is added just like any other piece of data in the system, by publishing a message to the network:

For a detailed example of stream syntax, check out our [Query Tutorial](./tutorial) and [Stream Tutorial](./stream)

## Private Messages

You may want to use Mizu to organize data without making that data publicly available. This is possible and safe.

All content in Mizu has a URI, but that doesn't mean that the content is globally accessible, or that it ever leaves your machine. Think of Google Docs. Have you ever tried to share a Google Doc with someone, only for them to get an error that they don't have permission to view the document? The document had a URI that you shared, but the URI existing didn't mean that your data was publicly visible. Mizu works the same way.

Private messages are accomplished via Client Configuration Queries, described in the next section.

## Redactions, Blacklists, Privatelists, and Pinning

One of Mizu's principles is: You, and only you, have control over the data on devices you own. Programs that can modify or delete data stored on your device without your knowledge or consent is a violation of this principle. You alone are empowered to control what data is (and isn't!) on your device.

Once something exists on the public Internet, you should assume it's there forever. But there's still good reasons to want to remove content from your Stream. If some data is no longer needed to display your site, then removing it from your Stream can reduce the amount of storage required for subscribers to pin your Stream. Removing content from a Stream can also (but doesn't have to) act as a form of *redaction*: a request that nodes should not retain the removed data and to not propagate that data to their peers. But you should always keep in mind that there's no way to enforce what other people do on their machines: you can request that data be removed, but it's up to the client running on that machine to decide to obey.

Likeiwse, there may be content on a stream that you don't want to view, or that you don't want to be stored on your local node, or that you want to avoid forwarding to other nodes. Mizu should empower you to make those kinds of decisions.

Mizu implements all of these with the use of special Client Configuration Queries. These are queries that exist on your client and act as filters: and messages that match these queries are subject to special behavior by the client. The different Client Configuration Queries are:

- Pins Filter: Messages that match this filter are "pinned" and will not be removed from local storage, even if the client is trying to recover storage space.
- Private Filter: Messages that match this filter are flagged by Mizu as belonging only to you. You can separately configure what kinds of requests will permit the client to respond with these messages, including *no* requests, meaning that the message will never leave your machine.
- Block Filter: Messages that match this filter are never returned as a result of a local request. (Useful for flagging content that you don't wish to see, but you still want to help propogate.)
- Discard Filter: Messages that match this filter are never retained locally. 

When you subscribe to a stream, that stream gets added as a Pins filter, guaranteeing that your local copy of the stream will persist. Streams can tag a subquery as a "redaction filter". Adding a message reference to the redaction filter is how a publisher expresses content that they wish to be removed from the stream.

When subscribing to a stream that has a redaction filter, you're prompted to choose how your client should handle messages that match the redaction filter. Your options are:

- Discard
- Unpin and don't forward
- Don't forward
- Unpin
- Do nothing
  

