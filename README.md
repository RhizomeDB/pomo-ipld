# PomoIPLD Schema v0.1.0

## Editors

- [Quinn Wilton], [Fission Codes]
- [Brooklyn Zelenka], [Fission Codes]

## Authors

- [Quinn Wilton], [Fission Codes]
- [Brooklyn Zelenka], [Fission Codes]

## Dependencies

- [IPLD]

# Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

# 0. Abstract

[Interplanetary Linked Data (IPLD)](https://ipld.io/) is a consistent, highly general data model for content addressed linked data forming any DAG. This data model provides a convenient pivot between many serializations of the same information.

# 1. Motivation

TODO

# 2. Schemata

## 2.1 Fact

A PomoDB "fact" is an ordered 4-tuple ("quad") consisting of an [Entity ID], [Attribute], [Value], and [Causes]:

### 2.1.1

``` ipldsch
type Fact struct {
  e EntityID
  a Attribute
  v Value
  c [&Fact]
} representation tuple
```

### 2.1.2 Entity ID

``` ipldsch
type EntityID = Bytes
```

### 2.1.3 Attribute

``` ipldsch
type Attribute
  = Integer -- e.g. Normal indices
  | Float   -- e.g. Fractional indices
  | Utf8
  | Bytes
```

### 2.1.4 Value

``` ipldsch
type Value
  = Boolean
  | Integer
  | Double
  | Utf8
  | Cid
  | Bytes
```

### 2.1.5 Capsule

``` ipldsch
type FactCapsule struct {
  fc Fact (rename "pomodb/v0.1/fact")
}
```

## 2.2 Tables

``` ipldsch
type PomoStore struct = Set<Link<Fact>>
```

# 3. FAQ

## 3.1 Why a tuple instead of a map?

- Consistent layout
- reading order
- EDN

# 0 Abstract




Being a chained token model, UCANs can be very naturally expressed via IPLD. However, JWTs are not fully deterministic due to whitespace and key ordering. This specification defines an IPLD Schema for UCAN, and a canonical JWT serialization for compatibility with all other clients.

# 1 Motivation

The [core UCAN specification](https://github.com/ucan-wg/spec) is defined as a JWT to make it easily adoptable with existing tools, and as a common format for all implementations. There are many cases where different encodings are preferred for reasons such as compactness, machine-efficient formats, and so on. This specification outlines a format based on IPLD that can be deterministically encoded and decoded between many serialization formats, while still being able to encode as JWT for compatibility.

# 2 IPLD Schema

Unlike a JWT, the IPLD encoding of UCAN does not require separate header, claims, and signature fields.

```ipldsch
type UCAN struct {
  v String

  iss Principal
  aud Principal
  s Signature

  att [Capability]
  -- All proofs are links, however you could still inline proof
  -- by using CID with identity hashing algorithm
  prf [&UCAN]
  exp Int

  fct [Fact]
  nnc optional String
  nbf optional Int
} representation map {
  field fct default []
  field prf default []
}
```




























<!-- Links -->

[Brooklyn Zelenka]: https://github.com/expede
[Fission Codes]: https://fission.codes
[Quinn Wilton]: https://github.com/QuinnWilton

