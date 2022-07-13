# Mizu Tutorial

The purpose of this document is to explain how Mizu is used and how it can be useful to you by walking through a sample UI flow, where Mizu is used to create a decentralized pubsub for a blog, a reblog, a static website, and finally a multi-user art gallery complete with comments and favorites.

## How to use this tutorial

Each step contains multiple examples using the Mizu command line interface, demonstrating the concepts introduced in that section. You can follow along on your machine. We encourage you to make sure you understand the behavior in each section before moving on to the next one.

## Step 1: Publishing a message

All data in Mizu is stored in a distributed triplestore database. That is, you can think of the entire database as a set of object-key-value triples. All transactions on Mizu involve adding records to this database, and adding records is accomplished by publishing messages.

Upon publishing a message, it receives a CID that can be used to refer to the message in queries or subsequent messages.

We can use Mizu's command line interface for publishing messages, which takes a message from stdin and prints the CID to stdout. An example might look like this:

```> $HelloWorld = echo '{ "text": "Hello, world!" }' | mizu publish
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

```> echo '{ "author": "solipsis", "content": "roses are red" }' | mizu publish
bafyreierxqzf7k7a7nhbfdoq2hs4fnbrytfjlj4namu7bakzt55hz4uylu
> echo '{ "author": "solipsis", "content": "violets are blue" }' | mizu publish
bafyreibgpiqlk3y7xjbcse6ggm26zk7ovwqia5psmtqxp5gm4zohet3oua
> echo '{ "@select": "?message", "@where": { "@id": "?message", "author": "solipsis" } }' | mizu query
[
	{'?message':  'https://mizu.stream/message/bafyreierxqzf7k7a7nhbfdoq2hs4fnbrytfjlj4namu7bakzt55hz4uylu'},
	{'?message':  'https://mizu.stream/message/bafyreibgpiqlk3y7xjbcse6ggm26zk7ovwqia5psmtqxp5gm4zohet3oua'}
]```

This will work even if the publish commands and the query command were run on different nodes!

However, there's an obvious problem with this approach: since our query will match against any message that has an "author" field set to the appropriate value, our query could easily return unwanted extra results. And to make things worse, there's nothing stopping anyone from publishing messages that match our query. If you run this command yourself, you may can many, many, additional results.

We want some way for nodes in the network to validate certain properties of messages, such as author, and then run a query on those properties. Reserved fields can do this for us.

## Step 3: Using $signatures to sign messages

If we want to make a query that only returns messages we published, we can use the ["$signatures" reserved field](./reserved_fields#signatures).

```> echo '{ "$signatures": [ { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4", "digest": "yagslalikjomkjqejkqqouyuriptfyijevbnds4d46abbcrzc6v2a53u3wyx7vajffhidviypkvkahtqc5uzw3ah464lbk4hzaogzeo5cht3mayktqvv225fuf7a6c3r2fhmrdhgdbh54vcfdnclq44qvd3weyxmmkeoq6owjap46ftvrbvj2t6kosm3qs6gbhl3pfmy43vodxk3ryp2hq2pecoh6uzj57sngqu4affuehs2eeh54ylxme6qbaamkyhcmjizzqpog2cdwrgg26ub7f7xvguf3mtgdkl6dlzgm3oa3aaa4cymk2v3lvlhhueo5qo774xqefmi2va5mcwzf6ruxr4qgddrni24trm7bo4ce3zpodaikr6chzcqsipjwrziz6nqiih4bmmnxnwc3m"} ], "content": "roses are red" }' | mizu publish
bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44```

In this message, the `$signatures` field contains one or more key-signature pairs that have signed the rest of the message. When the message is published to a node, or when a node receives the message from another node, it verifies the integrity of the signatures, and only accepts the message if every signature is valid.

Note that in practice you wouldn't hand-craft the `$signatures` field, your client will sign the messages for you.

Now if we run a query similar to the one above, we'll only get a single result, since this is the only message on the network signed with this key.

```> echo '{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }' | mizu query
[
	{'?message':  'https://mizu.stream/message/bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44'},
]```

Note that the query doesn't try to validate the key; it only checks that the key is in the message in the expected place. The validation was already done when the message was received by the running node (or when it was published, if published by the running node).

Also note the second `$` in the field `$$signatures` in the query. When a query wants to match on a reserved field, it escapes the field name by prepending an additional `$`. This makes it so that if the query gets included within a message, Mizu won't interpret the field in the query as a field that requires validation.

Why would a query get included in a message? So that it can be more easily shared.

```> echo '{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }' | mizu publish
bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i
> curl 'https://mizu.stream/message/bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i'
'{ "@select": "?message", "@where": { "@id": "?message", "$$signatures": { "key": "bafzbeiaibfu7pu3k36mgbmymh5ae75hmn6jnmzo26x2pzis6zv7bxncge4" } } }'
> curl 'https://mizu.stream/query/bafyreidjms562ffchpdjwb4y6yyzojwll2mkcckvy6aemreg3smqotgh3i'
[
	{'?message':  'https://mizu.stream/message/bafyreigbi3qsut4smii736l6oq66ktaxlc3fuh2fesiyn7whi7kbiyjl44'},
]```

Notice how the two URIs in the above example use different [actions](./concepts#action). The `message` action returns the original message, while the `query` message interprets the message as a query and returns the results. You can share this query URI with anyone and they'll be able to get all of your published messages that match the query. Anyone can use this URI akin to an RSS feed to subscribe to your published content. A more complicated query could even return metadata like post titles and descriptions, and sort the results to put the most recent posts first. An RSS-like app could render this query in a more human-readible way.

But unlike RSS, which assumes that the feeds have a specific structure and should be rendered a specific way, a Mizu query could return data in almost any structure, used for almost any purpose. But what if the message the contains the query also contains metadata that dictates how the results of the query should be displayed and interacted with (such as html and stylesheets)?

We call a message that contains both queries and markup language for rendering those queries, a Stream.

Streams are described in more detail in the [Streams Tutorial](./streams_tutorial).

# TODO: Extend this tutorial.