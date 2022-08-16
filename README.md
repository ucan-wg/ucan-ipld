# UCAN IPLD Schema v0.1.0

## Editors

* [Irakli Gozalishvili](https://github.com/Gozala), DAG House

## Authors

* [Irakli Gozalishvili](https://github.com/Gozala), DAG House
* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)

# 0 Abstract

The core UCAN specification is defined as a JWT to make it easily adoptable with existing tools, and as a lingua franca for all implementations. There are many cases where different encodings are preferred for reasons such as compactness, machine-readable formats, and so on. This specification outlines a format based on IPLD that can be deterministically encoded and decoded between formats, while still being able to encode as JWT for compatibility.

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 1 Introduction

## 1.1 Motivation

[Interplanetary Linked Data (IPLD)](https://ipld.io/) is a consisent, highly general data model for content addressed linked data. UCAN fits this model very nicely, but JWTs are not determinstic due to whitespace and key ordering.

# 2 IPLD Format

## 2.1 Container

Unlike the JWT format, the IPLD encoding of UCAN does not require separate header, claims, and signature fields.

```ipldsch
type UCAN struct {
  version String

  issuer DID
  audience DID
  signature Signature

  capabilities [Capability]
  proofs [&UCAN]
  expiration Int

  facts [Fact]
  nonce optional String
  notBefore optional Int
} representation map {
  field facts default []
  field proofs default []
}
```

## 2.2 Capability

``` ipldsch
type Capability struct {
  with Resource
  can Ability
  extension optional Extension
} representation map
```

### 2.2.1 Extensions

Extended capability fields contain any additional domain specific details and/or restrictions of the capability.

``` ipldsch
type Extension = { String: Any }
```

## 2.3 Fact

``` ipldsch
type Fact { String: Any }
```

## 2.4 Resource

The resource pointer MUST be formatted as a [URI](https://www.rfc-editor.org/rfc/rfc3986).

``` ipldsch
type Resource = String
```

## 2.5 Ability

Abilities MUST be lowercase, and MUST be namespaced with `/` delimeters. Abilities SHOULD have at least one path segment.

``` ipldsch
type Ability = String
```

## 2.6 DID

DIDs MUST be encoded as [DID](https://www.w3.org/TR/did-core/)s. The [`did:key` method](https://w3c-ccg.github.io/did-method-key/) is RECOMMENDED as it is self-contained.

``` ipldsch
type DID = String
```

## 2.7 Proof

As of UCAN 0.9, all proofs MUST be given as links.

Note that UCANs MAY be inlined via the [identity multihash (`0x00`)](https://github.com/multiformats/multicodec/blob/master/table.csv#L2) CID.

``` ipldsch
&UCAN
```

## 2.8 Signature

The signature MUST be computed by first encoding as a [base64url JWT](#3-jwt-canonicalization), and then signed with the issuer's private key.

``` ipldsch
type Signature = Bytes
```

# 3 JWT Canonicalization

Per the core UCAN spec, all implementations MUST support JWT encoding. To canonicalize the JWT, the JSON segments MUST be encoded per the [JSON Canonicalization Scheme (JCS)](https://www.rfc-editor.org/rfc/rfc8785), encoded as unpadded [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5), and joined with `.`s.

## 3.1 Deterministic Flag

When serialized to JWT, a deterministic encoding flag MUST be included in the JWT header `enc: "ipld"`.

FIXME decide on this vs `enc: "JCS"`

### 3.1.1 Example

``` javascript
{
  "ucv": "0.9.0",
  "typ": "JWT",
  "alg": "EdDSA",
  "enc": "ipld" // Canonicalized for IPLD
}
```

# 4 Acknowledgements

Many thanks to [Joel Thorstensson](https://github.com/oed) and [Sergey Ukustov](https://github.com/ukstv) at [3Box Labs](https://3boxlabs.com/) for their feedback on the encoding, and considerations on how it would interact with [SIWE](https://eips.ethereum.org/EIPS/eip-4361).
