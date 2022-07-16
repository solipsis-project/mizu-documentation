# Mizu Tutorial

The purpose of this document is to explain how Mizu is used and how it can be useful to you by walking through a sample UI flow, where Mizu is used to create a decentralized pubsub for a blog, a reblog, a static website, and finally a multi-user art gallery complete with comments and favorites.

## How to use this tutorial

Each step contains multiple examples using the Mizu command line interface, demonstrating the concepts introduced in that section. You can follow along on your machine. We encourage you to make sure you understand the behavior in each section before moving on to the next one.

## Step 1: Publishing a message

All data in Mizu is stored in a distributed triplestore database. That is, you can think of the entire database as a set of object-key-value triples. All transactions on Mizu involve adding records to this database, and adding records is accomplished by publishing messages.

Upon publishing a message, it receives a CID that can be used to refer to the message in queries or subsequent messages.

We can use Mizu's command line interface for publishing messages, which takes a message from stdin and prints the CID to stdout. An example might look like this:

```
> $HelloWorld = echo '{ "text": "Hello, world!" }' | mizu publish
bafyreihedihs5xnwh52scar3h2irbzvb5cqjjsf4axjhcbntqowjlhthya
```

If you want to view a message that's already been published to the network, you can do that via the `mizu view` command, or via a HTTP gateway.

`mizu view message $HelloWorld`
or
`curl https://mizu.stream/message/$HelloWorld`

`mizu view message` will print the original contents of a message to stdout, while visititing https://mizu.stream/message/{MESSAGE} will contain the original message as an HTTP response.

So far, this isn't doing anything that couldn't be done with IPFS alone. To truly leverage the benefits of Mizu, we need to learn about queries.

## Step 2: Queries

A client can query the database to get all records that match a particular pattern.

Suppose we want to have a query that will return all of our own messages. How might we do that?

There are two different syntaxes for queries: json-rql syntax and SPARQL syntax. They're specified [here](./query_syntax). These examples use the json-rql syntax.

Consider the following example:

```
> echo '{ "author": "solipsis", "content": "roses are red" }' | mizu publish
bafyreierxqzf7k7a7nhbfdoq2hs4fnbrytfjlj4namu7bakzt55hz4uylu
> echo '{ "author": "solipsis", "content": "violets are blue" }' | mizu publish
bafyreibgpiqlk3y7xjbcse6ggm26zk7ovwqia5psmtqxp5gm4zohet3oua
> echo '{ "@select": "?message", "@where": { "@id": "?message", "author": "solipsis" } }' | mizu query
[
	{'?message':  'https://mizu.stream/message/bafyreierxqzf7k7a7nhbfdoq2hs4fnbrytfjlj4namu7bakzt55hz4uylu'},
	{'?message':  'https://mizu.stream/message/bafyreibgpiqlk3y7xjbcse6ggm26zk7ovwqia5psmtqxp5gm4zohet3oua'}
]
```

This will work even if the publish commands and the query command were run on different nodes!

However, there's an obvious problem with this approach: since our query will match against any message that has an "author" field set to the appropriate value, our query could easily return unwanted extra results. And to make things worse, there's nothing stopping anyone from publishing messages that match our query. If you run this command yourself, you may can many, many, additional results.

We want some way for nodes in the network to validate certain properties of messages, such as author, and then run a query on those properties. Reserved fields can do this for us.

## Step 3: Using $signatures to sign messages

If we want to make a query that only returns messages we published, we can use the ["$signatures" reserved field](./reserved_fields#signatures).

```
> echo '{ 
  "$signatures": [ 
    {
	  "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4",
	  "digest": "yagslalikjomkjqejkqqouyuriptfyijevbnds4d46abbcrzc6v2a53u3wyx7vajffhidviypkvkahtqc5uzw3ah464lbk4hzaogzeo5cht3mayktqvv225fuf7a6c3r2fhmrdhgdbh54vcfdnclq44qvd3weyxmmkeoq6owjap46ftvrbvj2t6kosm3qs6gbhl3pfmy43vodxk3ryp2hq2pecoh6uzj57sngqu4affuehs2eeh54ylxme6qbaamkyhcmjizzqpog2cdwrgg26ub7f7xvguf3mtgdkl6dlzgm3oa3aaa4cymk2v3lvlhhueo5qo774xqefmi2va5mcwzf6ruxr4qgddrni24trm7bo4ce3zpodaikr6chzcqsipjwrziz6nqiih4bmmnxnwc3m"
	}
  ],
  "content": "roses are red"
}' | mizu publish
bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44
```

In this message, the `$signatures` field contains one or more key-signature pairs that have signed the rest of the message. When the message is published to a node, or when a node receives the message from another node, it verifies the integrity of the signatures, and only accepts the message if every signature is valid.

Note that in practice you wouldn't hand-craft the `$signatures` field, your client will sign the messages for you.

Now if we run a query similar to the one above, we'll only get a single result, since this is the only message on the network signed with this key.

```
> echo '{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }' | mizu query
[
	{'?message':  'https://mizu.stream/message/bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44'},
]
```

Note that the query doesn't try to validate the key; it only checks that the key is in the message in the expected place. The validation was already done when the message was received by the running node (or when it was published, if published by the running node).

Also note the second `$` in the field `$$signatures` in the query. When a query wants to match on a reserved field, it escapes the field name by prepending an additional `$`. This makes it so that if the query gets included within a message, Mizu won't interpret the field in the query as a field that requires validation.

Why would a query get included in a message? So that it can be more easily shared.

```
> echo '{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }' | mizu publish
bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i
> curl 'https://mizu.stream/message/bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i'
'{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }'
> curl 'https://mizu.stream/query/bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i'
[
	{'?message':  'https://mizu.stream/message/bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44'},
]
```

Notice how the two URIs in the above example use different [actions](./concepts#action). The `message` action returns the original message, while the `query` message interprets the message as a query and returns the results. You can share this query URI with anyone and they'll be able to get all of your published messages that match the query. Anyone can use this URI akin to an RSS feed to subscribe to your published content. A more complicated query could even return metadata like post titles and descriptions, and sort the results to put the most recent posts first. An RSS-like app could render this query in a more human-readible way.

But unlike RSS, which assumes that the feeds have a specific structure and should be rendered a specific way, a Mizu query could return data in almost any structure, used for almost any purpose. But what if the message the contains the query also contains metadata that dictates how the results of the query should be displayed and interacted with (such as html and stylesheets)?

We call a message that contains both queries and markup language for rendering those queries, a Stream.

Streams are described in more detail in the [Streams Tutorial](./streams_tutorial).

## Step 4: Creating your own key, and auto-signing

We'd much rather have the Mizu command line interface create keys and digests for us than have to manually insert them into our messages. In this step, we're going to generate a key and configure the client to sign our messages with it. This will make the examples much simpler.

For more information, check out [Client Configuration](./client_configuration)

TODO: Describe how to configure the client to auto-sign messages.

## Step 5: Using $ref

In the previous example, we had a stream whose associated query matched against all messages signed by a specific key. This is fine for streams that have a single author, but doesn't work for multi-author streams and isn't very readible. When possible, we'd much rather have messages that declare what stream they belong to. (We say "when possible" bacause some streams, like reblogs, may contain content that wasn't originally made for that stream.)

So we're going to create a stream definition that requires that, in addition to the messages being signed, messages must reference the original stream definition. We can accomplish this by using the ["$ref" reserved field](./reserved_fields#signatures).

```
> $Stream = echo '{
	"title": "Aspiring Mizu Stream",
	"query": {
		"@select": "?posts",
  		"@where": {
  			"stream": { "$ref": "#" },
			"posts": "?posts",
  			"$$signatures": {
  				"key": `mizu config get key`
  			}
  		}
	},
	"stylesheet": "...",
	"html": "..."
}' | mizu publish
bafy...
```

We no longer include the full cid in the example because its value will depend on the key that was used to sign the message. However, we assign the result to a variable so that we can reference it in later commands.

When an object in the Mizu data model has only a single field named `$ref`, that object is a reference to another Mizu URI. A Mizu reference can be full (contain a path) or patial (only contain a fragment). Here we use the partial reference `#`, which means "the root of the current message." Note that we can't use an absolute reference here: an absolute reference would require knowing this message's path, but the message's path is a hash of its content! We won't know what it is until after we publish the message.

Here's a message that matches this query:

```
> $Message1 = echo '{
	"stream": { "$ref": "https://mizu.stream/message/$StreamURI" },
    "posts": ["Roses are red"]
}' | mizu publish
```

Before running the query, the client will replace the partial reference in the query with the corresponding absolute reference, so the reference in the message will match.

```
> curl https://mizu.stream/query/$Stream/query#?message
[
	{ "?posts": "Roses are red." }
]
```

Giving a message the abilty to explicitly tag itself as belonging to a specific stream or query is an important stepping stone to building applications on Mizu.

At this point, there are several ways that we can address the "Roses are red" string from our most recent message.

- https://mizu.stream/message/$Message1/#/posts (or `mizu view message $Message1 blog`)
- https://mizu.stream/update/$Message1/#/posts (or `mizu view update $Message1 blog`)
- https://mizu.stream/query/$Stream/query/#/%3Fposts (or `mizu view query $Stream/query ?post`)

Right now, all of these URLs will resolve to the same data: [ "Roses are red" ]. However, there's an important difference between them.

"message/$Message1/#/posts" (using the "message" action) means "give me the object named 'posts' from raw message $Message1. This value is immutable, and can be referenced by future messages.

"update/$Message1/#/posts" (using the "update" action) means "give me the object named 'posts' as it was set by $UPDATE1, merged with any previous messages." This value is immutable, and can be referenced by future messages. Right now, there are no previous messages, so this is identical to the message action, but we'll see how they differ soon.

"query/$Stream/query/#/%3Fposts" (using the "stream" action) means "give me the object named '?posts' from the query result ('%3F' is just an escaped '?'), as it exists in the present, based on all messages that matched the query." This value is mutable, so it can't be referenced by future messages.

If we publish another update, then the values of these three URLs will diverge. We'll see this happen in the next step.

If a query returns a lot of results, then caching the result could take up a lot of space. Worse, sharing the result with other peers in the network, or getting the most up to date result from the network, could be expensive. This is because when a node asks its peers for messages matching a query, either the requester needs to communicate which messages it already has, or the peers need to send every message it knows about. This can add up to a lot of traffic. Is there an efficient way for nodes in the network to determine the minimum set of data that they need to exchange?

There is! And the `$prev` reserved field can come to the rescue.

## Step 6: Using $prev to create a DAG of messages


# TODO: Extend this tutorial.