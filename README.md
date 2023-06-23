# UCAN IPLD Schema v0.2.0

## Editors

* [Irakli Gozalishvili], [DAG House]

## Authors

* [Hugo Dias], [DAG House]
* [Irakli Gozalishvili], [DAG House]
* [Brooklyn Zelenka], [Fission Codes]

## Dependencies

* [IPLD]
* [UCAN Canonicalization]
* [UCAN Delegation]
* [multicodec]
* [multidid]
* [varsig]

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119].

# 0 Abstract

[Interplanetary Linked Data (IPLD)][IPLD] is a consistent, highly general data model for content addressed linked data forming any DAG. This data model provides a convenient pivot between many serializations of the same information. Being a chained token model, UCANs can be very naturally expressed via IPLD. However, JWTs are not fully deterministic due to whitespace and key ordering. This specification defines an IPLD Schema for UCAN, and a canonical JWT serialization for compatibility with all other clients.

# 1 Motivation

The [UCAN Delegation] specification is defined as a JWT to make it easily adoptable with existing tools, and as a common format for all implementations. There are many cases where different encodings are preferred for reasons such as compactness, machine-efficient formats, and so on. This specification outlines a format based on IPLD that can be deterministically encoded and decoded between many serialization formats, while still being able to encode as JWT for compatibility.

# 2 IPLD Schema

Unlike a JWT, the IPLD encoding of UCAN does not require separate header, claims, and signature fields.

```ipldsch
type UCAN struct {
  v String

  iss Principal
  aud Principal
  s Signature

  cap Capabilities

  -- All proofs are links, however you could still inline proof
  -- by using CID with identity hashing algorithm
  prf [&UCAN]

  fct [Fact]
  nnc optional String

  nbf optional Int
  exp          Int
} representation map {
  field fct default []
  field prf default []
}
```

## 2.1 Principal

Principals MUST use the [multidid] representation of [DID]s.

## 2.2 Capabilities

Capabilities are a map ____________.

``` ipldsch
type Capabilities = { Resource : { Ability: [ Caveats ] } }
```

### 2.2.1 Resource

The resource pointer MUST be formatted as a [URI].

``` ipldsch
type Resource = String
```

### 2.2.2 Ability

Abilities MUST be lowercase, and MUST be namespaced with `/` delimeters. Abilities SHOULD have at least one path segment.

``` ipldsch
type Ability = String
```

### 2.2.3 Caveats

The inclusion of caveats is OPTIONAL. When present, this map MUST contain any additional domain specific details and/or restrictions of the capability. 

``` ipldsch
type Caveats = { String: Any }
```

## 2.3 Fact

Facts MUST be structured as maps with string keys.

``` ipldsch
type Fact { String: Any }
```

## 2.4 Proof

As of UCAN 0.9, all proofs MUST be given as links (CIDs). Note that UCANs MAY be inlined via the [identity multihash](https://github.com/multiformats/multicodec/blob/master/table.csv#L2) (`0x00`) CID.

``` ipldsch
&UCAN
```

## 2.5 Signature

The signature MUST be computed by first encoding it as a [canonical JWT], and then signed with the issuer's private key. Signatures MUST be encoded as the _varsig_ multiformat representation `<varint sig_alg_code><vairint sig_size><bytes sig_output>`. <!---------------- FIXME      _Varsig_ multiformat codes SHOULD be [registered multicodec codes](https://github.com/multiformats/multicodec). Unregistered signature types MAY be added via the `NonStandardSignature` prefix 
  -->

# 3 Acknowledgments

Many thanks to [Joel Thorstensson](https://github.com/oed) and [Sergey Ukustov](https://github.com/ukstv) at [3Box Labs](https://3boxlabs.com/) for their feedback on the encoding, and considerations on how it would interact with [SIWE](https://eips.ethereum.org/EIPS/eip-4361).

Thanks to [Philipp Krüger](https://github.com/matheus23) for his feedback on canonicalization signalling.

<!-- Links -->

[Brooklyn Zelenka]: https://github.com/expede
[DAG House]: https://dag.house
[DID]: https://www.w3.org/TR/did-core/
[Fission Codes]: https://fission.codes
[Hugo Dias]: https://github.com/hugomrdias
[IPLD]: https://ipld.io/
[Irakli Gozalishvili]: https://github.com/Gozala
[RFC2119]: https://datatracker.ietf.org/doc/html/rfc2119
[UCAN Canonicalization]: https://github.com/ucan-wg/canonicalization/
[UCAN Canonicalization]: https://github.com/ucan-wg/canonicalization/ 
[UCAN Delegation]: https://github.com/ucan-wg/spec
[URI]: https://www.rfc-editor.org/rfc/rfc3986
[`dag-json`]: https://ipld.io/specs/codecs/dag-json/spec/
[`did:key`]: https://w3c-ccg.github.io/did-method-key/
[bytesprefix]: https://ipld.io/docs/schemas/using/authoring-guide/#bytesprefix-unions-for-bytes
[identity multihash]: https://github.com/multiformats/multicodec/blob/master/table.csv#L2
[multicodec]: https://github.com/multiformats/multicodec
[multidid]: https://github.com/ChainAgnostic/multidid 
[varsig]: https://github.com/ChainAgnostic/varsig/
