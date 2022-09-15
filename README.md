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

## 2.1 Principal

Principals MUST use [bytesprefix](https://ipld.io/docs/schemas/using/authoring-guide/#bytesprefix-unions-for-bytes) representation of [DID](https://www.w3.org/TR/did-core/)s. Primary [`did:key` method](https://w3c-ccg.github.io/did-method-key/) MUST use (more compact) key specific variant representation. Additional DID methods MAY use `DID` variant representation.

``` ipldsch
type Principal union {
  -- This specification defines principals in terms of `did:key`s (multicodec
  -- identifiers for the public key type followed by the raw bytes of the key).
  -- We represent those as raw public key bytes prefixed with public key 
  -- multiformat code.
  | secp256k1  "0xe7"
  | BLS12381G1 "0xea"
  | BLS12381G2 "0xeb"
  | Ed25519    "0xed"
  | P256       "0x1200"
  | P384       "0x1201"
  | P512       "0x1202"
  | RSA        "0x1205"
  
  -- To accomodate additional DID methods we represent those as UTF8 encoding
  -- of the DID omitting `did:` prefix itself. E.g. `did:dns:ucan.xyz` can be
  -- represented as [0xd1d, ...new TextEncoder().encode('dns:ucan.xyz')] bytes.
  | DID "0x0d1d"
} representation bytesprefix
```

## 2.2 Capability

``` ipldsch
type Capability struct {
  with Resource
  can Ability
  nb optional NonNormativeFields
}
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

### 2.2.3 `nb` Non-Normative Fields

The `nb` capability field is OPTIONAL. When present, it MUST contain any additional domain specific details and/or restrictions of the capability. 

``` ipldsch
type NonNormativeFields = { String: Any }
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

The signature MUST be computed by first encoding it as a [canonical JWT](#3-jwt-canonicalization), and then signed with the issuer's private key. Signatures MUST be encoded as the _varsig_ multiformat representation `<varint sig_alg_code><vairint sig_size><bytes sig_output>`. _Varsig_ multiformat codes SHOULD be [registered multicodec codes](https://github.com/multiformats/multicodec). Unregistered signature types MAY be added via the `NonStandardSignature` prefix.

``` ipldsch
type Signature union {
  -- Algorithms here are expected to be valid "varsig" multiformat codes.
  | NonStandard "0xd000"
  | ES256K      "0xd0e7" -- secp256k1
  | BLS12381G1  "0xd0ea" -- NB: G1 is a signature for BLS-12381 G2
  | BLS12381G2  "0xd0eb" -- NB: G2 is a signature for BLS-12381 G1
  | EdDSA       "0xd0ed"
  | ES256       "0xd01200"
  | ES384       "0xd01201"
  | ES512       "0xd01202"
  | RS256       "0xd01205"
  | EIP191      "0xd191"
} representation bytesprefix

-- UCAN-IPLD spec will register non standard signature algorithms here and
-- recommend registering them as valid "varsig" multiformat codes to remove
-- need for 0xd000 varint prefix. Implementatations MAY be also augmented
-- with additional non standard signature types.
type NonStandardSiganture union {
  -- Example: EIP1271 "0xd01271"
} representation bytesprefix
```

# 3 JWT Canonicalization

Per the core UCAN spec, all implementations MUST support JWT encoding. This provides a common representation that all implementations can understand. JWT canonicalization allows for an IPLD UCAN to be expressed as a JWT, retain the JWT signature scheme, and so on for compatibility, while retaining the ability to translate into other formats for storage or transmission among IPLD-enabled peers.

To canonicalize an IPLD UCAN to JWT, the JSON segments MUST fulfill the following requirements:

1. All `can` fields MUST be lowercase
2. All unused optional fields (such as `fct`) that are empty MUST be omitted
3. [`dag-json`](https://ipld.io/specs/codecs/dag-json/spec/) encoding MUST be used
4. The resulting JWT MUST be [base64url](https://datatracker.ietf.org/doc/html/rfc4648#section-5) encoded per [RFC 7519]
5. All segments MUST be joined with `.`s, per [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519)

UCAN canonicalization is signalled by CID. If no canonicalization is used, the CID MUST use the [raw multicodec](https://github.com/multiformats/multicodec/blob/master/table.csv#L39). Canonicalized UCANs that wish to signal this encoding MUST use [any other CID codec](https://github.com/multiformats/multicodec/blob/master/table.csv), including but not limited to `dag-json` and `dag-cbor`.

## 3.1 Non-IPLD Validator CID Handling

Validators that have not implemented this specification MUST be provided JWT-encoded UCANs. These validators will be unable to validate the CID in the proofs field. This is not strictly a problem in a semi-trusted scenario, as UCAN only depends on the existence (not the specific CID) of a valid proof for the capabilities being claimed. The security risk is for a malicious peer to provide very long but ultimately invalid proof chains as a denial-of-service vector. This is the case for any validator that does not check the CID hash upon receipt.

# 4 Acknowledgments

Many thanks to [Joel Thorstensson](https://github.com/oed) and [Sergey Ukustov](https://github.com/ukstv) at [3Box Labs](https://3boxlabs.com/) for their feedback on the encoding, and considerations on how it would interact with [SIWE](https://eips.ethereum.org/EIPS/eip-4361).

Thanks to [Philipp Krüger](https://github.com/matheus23) for his feedback on canonicalization signalling.
