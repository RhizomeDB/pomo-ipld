# PomoIPLD Schema v0.1.0

## Editors

- [Quinn Wilton], [Fission Codes]
- [Brooklyn Zelenka], [Fission Codes]

## Authors

- [Quinn Wilton], [Fission Codes]
- [Brooklyn Zelenka], [Fission Codes]

## Dependencies

- [IPLD]
- [PomoDB]

# Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

# 0. Abstract

[PomoDB] represents data in a consistent, content addressable data format for Datalog facts. This specification describes the IPLD encoding.

# 1. Motivation

Interplanetary Linked Data ([IPLD]) is a consistent, highly general data model for content addressed linked data forming any DAG. This data model provides a convenient pivot between many serializations of the same information.

PomoDB was originally designed with [IPFS] — and by extension IPLD — in mind. Content addresses depend on the codec of some data. As such, the exact layout of the data as IPLD is important. This specification gives the IPLD encoding.

# 2. Schema

## 2.1 Fact

A PomoDB "Fact" is an ordered 4-tuple ("quad") consisting of an [Entity ID], [Attribute], [Value], and [Caused By].

### 2.1.1 Fact

``` ipldsch
type Fact struct {
  entityId  EntityID
  attribute Attribute
  value     Value
  causedBy  [&Fact]
} representation tuple
```

Note that the `representation tuple` flattens the struct to a positional array. For example, the following concrete [DAG-JSON] representation parses to the IPLD representation below it (given in Rust).

``` js
// DAG-JSON
[123, "name/last", "Monroe", [{"/": "bafyreiaajfbxfnbbdbhvxmowe6t63ytsimv4daiitv5gkqetwrpww5zmsy"}]]
```

``` rust
// Rust
Fact {
  attribute: "name/last",
  caused_by: [Link("bafyreiaajfbxfnbbdbhvxmowe6t63ytsimv4daiitv5gkqetwrpww5zmsy")],
  entity_id: 123,
  value: "Monroe",
}
```

### 2.1.2 Entity ID

Restricting an entity ID to 128-bits is RECOMMENDED.

``` ipldsch
type EntityID = Bytes
```

### 2.1.3 Attribute

Attributes MUST be represented as one of the following:

``` ipldsch
type Attribute
  = Integer -- e.g. Normal indices
  | Float   -- e.g. Fractional indices
  | String
  | Bytes
```

### 2.1.4 Value

Values MUST be given as one of the following:

``` ipldsch
type Value union {
  | Boolean
  | Integer
  | Float
  | String 
  | Link
  | Bytes
} representation kinded
```
 
Note that all floating point values MUST be representable as 64-bit (double-precision) floats as defined in [IEEE 754-2019] when deserialized. `NaN`s MUST NOT be used.

### 2.1.5 Caused By

Links to other facts MUST be placed in an array in the `causedBy` field.

``` ipldsch
type CausedBy = [&Fact]
```
 
## 2.1.6 Capsule

An OPTIONAL capsule type to clarify the enclosed data MAY be used.

``` ipldsch
type FactCapsule struct {
  c Fact (rename "pomodb/v0.1/fact")
}
```

## 2.2 Store

As PomoDB is intended to operate over many Facts, storing and referencing collections is important. Below are two strategies for representing groups of Facts.

### 2.2.1 Hash Map

A collection of Facts is called a "Store". This structure is simply a content-addressed blockstore indexing Facts by their CID.

``` ipldsch
type Store = {CID : Fact}
```

### 2.2.2 Set

Stores contain too much information for transmission and encrypted storage. In these cases, a flat set called a Collection MAY be used:

``` ipldsch
type Collection = [Fact]
```

# 3 Canonicalization Considerations

IPLD cleanly canonicalizes data, though differently per codec. However, the same data MAY have multiple CIDs due to differences in encoding, hash algorithm, and so on. Strictly speaking, this in no way poses a problem for PomoDB: the same fact entered into the store twice is trivial for operations that only depend on the graph structure of the store.

Certain aggregate functions (e.g. counts, sums, averages) and stateful queries (e.g. graph colorings) depend on a node being present no more than once per graph. Deduplication is thus imperative for many use cases. It is RECOMMENDED that all facts added to a store have a canonical CID. This MAY be of any CID configuration. To reduce the amount of recomputation, using the following parameters are RECOMMENDED:

| Parameter    | Recommended Setting |
|--------------|---------------------|
| CID Version  | [CIDv1]             |
| [Multicodec] | [DAG-CBOR]          |
| [Multihash]  | [SHA2-256]          |

Note that the [multibase] of a CID is defined by the codec and CID version.

<!-- Links -->

[Attribute]: #213-attribute
[Brooklyn Zelenka]: https://github.com/expede
[CIDv1]: https://docs.ipfs.tech/concepts/content-addressing/#cid-versions
[Capsule Type]: https://notes.brooklynzelenka.com/Capsule+Types
[Capsule]: #216-capsule
[Caused By]: #215-caused-by
[DAG-CBOR]: https://ipld.io/specs/codecs/dag-cbor/spec/
[DAG-JSON]: https://ipld.io/docs/codecs/known/dag-json/
[Entity ID]: #212-entity-id
[Fact]: #211-fact
[Fission Codes]: https://fission.codes
[IEEE 754-2019]: https://en.wikipedia.org/wiki/IEEE_754
[IPFS]: https://ipfs.io
[IPLD]: https://ipld.io
[Multicodec]: https://github.com/multiformats/multicodec
[Multihash]: https://github.com/multiformats/multihash
[PomoDB]: https://github.com/RhizomeDB/spec
[Quinn Wilton]: https://github.com/QuinnWilton
[RFC 2119]: https://www.rfc-editor.org/rfc/rfc2119
[SHA2-256]: https://en.wikipedia.org/wiki/SHA-2
[Store]: #22-store
[Value]: #214-value
[multibase]: https://github.com/multiformats/multibase
