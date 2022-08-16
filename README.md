# UCAN IPLD Schema

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

