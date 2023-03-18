---
title: "Reserved Fields"
permalink: reserved_fields.html
summary: A listing of all reserved fields, and how they're used to validate transactions.
---

# Reserved Fields

Some message fields start with a $. These are reserved fields, and each one has special semantics, either purely for message validation, or for one or more Mizu actions. Reserved fields are validated when a message is published or when received by a node: thus, all published messages can be assumed to obey certain constraints specified in this document. Names starting with a $ that are not listed here are reserved for later use and cannot be used in messages: clients must reject messages containing any such fields.

The current list of reserved fields includes:

### $signatures

Used for signing messages. The value of `$signatures` must be a list of objects with exactly two fields: `key` containing a multihash of the public key (as exported by the libp2p-crypto library), and `digest` containing the actual signature.

Both fields are base58btc-encoded strings. (This is done so that the message is human-readable when encoded as JSON, but it has the notable side-effect that when a Mizu message is serialized as dag-cbor, which happens when a message is stored in IPFS, we end up storing a binary encoding of a string that is itself a human-readable encoding of a hash.)

During publishing, the contents of the object, minus the `$signatues` field, are serialized as dag-cbor, and then verified against each provided signature. If the verification of any provided signature fails, the message is not published.

### $schema

Used for declaring that a message conforms to a provided schema. The value of this field must be either an absolute Mizu URI or an IPFS URI.

The linked schema itself declares what *kind* of schema it is. If the node does not support that kind of schema, or if the message does not conform to the provided schema, the message must be rejected.

### $ref

Used for including a reference to another piece of Mizu data. The value of `$ref` must be a static Mizu URI, and the object containing the `$ref` field must have no other fields.

### $include

Used for any action type other than 'message'. The value of `$include` must be a static Mizu URI. During processing, the message should be treated as if the referenced URI was inlined into the enclosing object. If the included object has field names that conflict with the field names of the enclosing message, the fields from the inlinded object are dropped and the fields from the enclosing object are used instead.

If two $includes in a message create a cycle, the message must be rejected. Note that it is not possible to construct a cycle across multiple messages, since an include to a different message must be an absolute URI, all absolute Mizu URIs contain CIDs, and CIDs cannot contain cyclical references.

### $prev

The $prev field is used to specify that one message has a "comes-after" relationship with another message. It must be a static Mizu URI. It's a hint that the former message will require the latter for full context, and that both messages are likely to appear in the same stream, such that they will need to be merged together. The 'update' action performs this merge and returns the result.

$prev interacts with the 'query' and 'stream' actions a bit differently. When a Mizu client evalutes one of these actions, it caches the result, and forms a dag out of the response messages, with the '$prev' field constituting an edge. It then identifies the set of response messages that are not referenced by any '$prev' fields and stores these as the "Stream Head." If the client later wants to evaluate the same stream again, it sends the list of Stream Heads in its request to other nodes. When a node with a more up to date cache receives the request, it can efficeintly compute which messages the requesting client is missing by comparing the received set of Stream Heads with the receiving client's own (along with its own dag).  

### $var

Sometimes we want to store a template that has some of its keys or values replaced before being interpreted. Values can be replaced by using "$var". "$var" can only be used as a field when it is the only field on the object. The value of this field must be a string. For all actions except "message", before the action is performed, all such objects are replaced with a string specified in the URI's parameters.

### $$

A common use case of messages in Mizu is to specify a query on the underlying datastore. One syntax for queries, the JSON-RQL syntax, allows you to represent the query as an object. However, we may want the query to match on reserved fields, which would require the query object to have fields with those same names, which would invoke the validation rules described in this section.

Our solution is to use field names beginning with $$, which escape the reserved field in the query: the query will match on the correct fields, without invoking validation for the message containing the query.
