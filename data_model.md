# Data Model

All data in Mizu is stored in a distributed triplestore database; that is, an unordered set of records, each of which is a triple of the form `subject, verb, object`.

It's possible to view a set of triples as an object, where the verb is a field name and the object is a field value.

Note that multiple records may have the same subject and verb but different objects. In this case, we can conceptually think of the object built from those records in one of two ways:

- An object that allows multiple fields with the same name.
- An object where all field values are sets (whose elements are not sets).

In the latter case, we may say that a field has a non-set value (a primitive or another object), but this is equivalent the field value being a set with one element.

Mizu uses two different encoding schemes: CBOR for binary messages and JSON for human-readable messages.

In both encoding schemes, Mizu uses arrays to represent sets, and objects with numerical keys to represent arrays.