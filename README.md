# UCAN IPLD Schema v0.1.0

## Editors

* [Irakli Gozalishvili](https://github.com/Gozala), DAG House

## Authors

* [Hugo Dias](https://github.com/hugomrdias), DAG House
* [Irakli Gozalishvili](https://github.com/Gozala), DAG House
* [Brooklyn Zelenka](https://github.com/expede), [Fission](https://fission.codes)

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 0 Abstract

[Interplanetary Linked Data (IPLD)](https://ipld.io/) is a consistent, highly general data model for content addressed linked data forming any DAG. This data model provides a convenient pivot between many serializations of the same information. Being a chained token model, UCANs can be very naturally expressed via IPLD. However, JWTs are not fully deterministic due to whitespace and key ordering. This specification defines an IPLD Schema for UCAN, and a canonical JWT serialization for compatibility with all other clients.

# 1 Motivation

The [core UCAN specification](https://github.com/ucan-wg/spec) is defined as a JWT to make it easily adoptable with existing tools, and as a common format for all implementations. There are many cases where different encodings are preferred for reasons such as compactness, machine-efficient formats, and so on. This specification outlines a format based on IPLD that can be deterministically encoded and decoded between many serialization formats, while still being able to encode as JWT for compatibility.

# 2 IPLD Schema

Unlike a JWT, the IPLD encoding of UCAN does not require separate header, claims, and signature fields.

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

## 2.1 DID

DIDs MUST be encoded as [DID](https://www.w3.org/TR/did-core/)s. The [`did:key` method](https://w3c-ccg.github.io/did-method-key/) is RECOMMENDED as it is self-contained.

``` ipldsch
type DID = String
```

## 2.2 Capability

``` ipldsch
type Capability struct {
  with Resource
  can Ability
  extension optional Extension
} representation map
```

### 2.2.1 Resource

The resource pointer MUST be formatted as a [URI](https://www.rfc-editor.org/rfc/rfc3986).

``` ipldsch
type Resource = String
```

### 2.2.2 Ability

Abilities MUST be lowercase, and MUST be namespaced with `/` delimeters. Abilities SHOULD have at least one path segment.

``` ipldsch
type Ability = String
```

### 2.2.3 Extensions

The extended capability field is OPTIONAL. When present, it MUST contain any additional domain specific details and/or restrictions of the capability. 

``` ipldsch
type Extension = { String: Any }
```

## 2.3 Fact

Facts MUST be structured as maps with string keys.

``` ipldsch
type Fact { String: Any }
```

## 2.4 Proof

As of UCAN 0.9, all proofs MUST be given as links (CIDs). Note that UCANs MAY be inlined via the [identity multihash (`0x00`)](https://github.com/multiformats/multicodec/blob/master/table.csv#L2) CID.

``` ipldsch
&UCAN
```

## 2.5 Signature

The signature MUST be computed by first encoding it as a [canonical JWT](#3-jwt-canonicalization), and then signed with the issuer's private key.

``` ipldsch
type Signature = Bytes
```

# 3 JWT Canonicalization

Per the core UCAN spec, all implementations MUST support JWT encoding. This provides a common representation that all implementations can understand. JWT canonicalization allows for an IPLD UCAN to be expressed as a JWT, retain the JWT signature scheme, and so on for compatibility, while retaining the ability to translate into other formats for storage or transmission among IPLD-enabled peers.

To canonicalize an IPLD UCAN to JWT, the JSON segments MUST be encoded per [`dag-json`](https://ipld.io/specs/codecs/dag-json/spec/), encoded as unpadded [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5), and joined with `.`s.

UCAN canonicalization is signalled by CID. If no canonicalization is used, the CID MUST use the [raw multicodec](https://github.com/multiformats/multicodec/blob/master/table.csv#L39). Canonicalized UCANs that wish to signal this encoding MUST use [any other CID codec](https://github.com/multiformats/multicodec/blob/master/table.csv), including but not limited to `dag-json` and `dag-cbor`.

## 3.1 Non-IPLD Validator CID Handling

Validators that have not implemented this specification MUST be provided JWT-encoded UCANs. These validators will be unable to validate the CID in the proofs field. This is not strictly a problem in a semi-trusted scenario, as UCAN only depends on the existence (not the specific CID) of a valid proof for the capabilities being claimed. The security risk is for a malicious peer to provide very long but ultimately invalid proof chains as a denial-of-service vector. This is the case for any validator that does not check the CID hash upon receipt.

# 4 Acknowledgments

Many thanks to [Joel Thorstensson](https://github.com/oed) and [Sergey Ukustov](https://github.com/ukstv) at [3Box Labs](https://3boxlabs.com/) for their feedback on the encoding, and considerations on how it would interact with [SIWE](https://eips.ethereum.org/EIPS/eip-4361).
