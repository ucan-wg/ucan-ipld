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
  -- Any additional domain specific details and/or restrictions of the capability
  caveats { String: Any }
}
```

## 2.3 Fact

``` ipldsch
type Fact { String: Any }
```

## 2.4 Resource

-- The resource pointer in URI format
type Resource = String

## 2.5 Ability

-- Must be all lower-case `/` delimeted with at least one path segment
type Ability = String

## 2.6 DID

DIDs MUST be encoded as [DID](https://www.w3.org/TR/did-core/)s. The [`did:key` method](https://w3c-ccg.github.io/did-method-key/) is RECOMMENDED as it is self-contained.

did:key encoded as multicodec tagged public key

| Type    | Multicodec Prefix |
| ------- | ----------------- |
| Ed25519 | `0xed`            |
| RSA     | `0x125`           |
| P-256   | ``                |

0xed       Ed25519
0x1205     RSA
other DIDs could also be used by encoding it in UTF8 string

``` ipldsch
type DID = Bytes
```

## 2.7 Proof

-- All proofs are links, however you could still inline proof
-- by using CID with identity hashing algorithm

## 2.8 Signature

The signature MUST be computed by first encoding as a [base64url JWT].

1. Deriving JWT header & payload using DAG-JSON (to achieve for hash consitency)
2. Formating it into base64 string
3. Signing via issuers private key

``` ipldsch
type Signature = Bytes
```

# 3 JWT Encoding

Per the core UCAN spec, all implementations MUST support JWT encoding.

Raw multicodec
