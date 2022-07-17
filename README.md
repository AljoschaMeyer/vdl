# Valuable Description Language

A language for defining subsets of the [valuable values](https://github.com/AljoschaMeyer/valuable-value), suitable for validation, documentation, test data generation, etc. Slightly opinionated, vdl is intended to define sensible subsets, not to be able to represent every oddly defined API on the planet.

**Status: deliberately not a full spec yet, but hopefully an interesting read already**

Some comparable/related efforts (for other data formats):

- [JSON Schema](https://json-schema.org/)
- [CDDL](https://datatracker.ietf.org/doc/html/rfc8610)
- [GraphQL Type System](https://spec.graphql.org/October2021/#sec-Type-System)
- [clojure.spec](https://clojure.org/guides/spec)

## Overview

VDL allows you to specify certain subsets of the [valuable values](https://github.com/AljoschaMeyer/valuable-value). Consider, for example, an API that expect an object that maps the key `"name"` to a utf-8 string, and the key `"age"` to an integer. With VDL, you can define the set of all such values in a compact, unambiguous, and machine-friendly manner.

In the same spirit as the valuable value specification, we start by developing an abstract model of the VDL before giving a specific (and ultimately replaceable) syntax. Throughout this document, a *value* refers to a valuable value. A *type* is a [set](https://en.wikipedia.org/wiki/Set_(mathematics)) of values.

VDL is concerned with *type descriptions*, objects that can be mapped to types. An implementation of VDL can take a value and check whether it is of the type that is described by some type description.

VDL does not support naming of type descriptions, hence they are neither recursive nor generic type descriptions. Those are part of VDLM, the *valuable description language with modules*. VDLM does not exist outside the head of the author yet, but inside that head, it is already fully developed. Feel free to reach out if you are very curious about this. For now, I am focusing on quickly sketching a larger valuable value ecosystem by omitting recursion everywhere (in particular, VVM, the *valuable values with modules* does not yet exist either outside my head).

Another aspect of useful type systems (in particular when thinking about query languages and/or mutation languages for valuable values) that the specification deliberately omits for simplicity sake are read-only and write-only access to component types. Again, I have plans for those, but keep them out of this very basic specification.

### Type Descriptions

Type descriptions are defined inductively.

#### Base Cases

- Every *value* is a type description. The corresponding type is the set singleton set containing exactly that value.
- A *range* is a type description, based on the [subvalue relation](https://github.com/AljoschaMeyer/valuable-value#subvalues). A range consists of one or two values `s` and `t`, and is either *exclusive*, *inclusive*, or open:
  - An exclusive range contains all values `v` such that `s <= v < t`. We write `s..=t`.
  - An inclusive range contains all values `v` such that `s <= v <= t`. We write `s..t`.
  - An open range contains all values `v` such that `s <= v`. We write `s...`.
- `F32` is a type description. The corresponding type is the set of all floats that can be exactly represented as IEEE 754 single precision floats (32 bit).
- `Utf8` is a type description. The corresponding type is the set of arrays that contain only integers between zero (inclusive) and 256 (exclusive) that form valid utf-8.

These are technically the only base cases we need, but VDL specifically defines some further type descriptions. All of these could also be represented through some other type descriptions, but it is convenient to have shorthands for these, and implementations would special-case them anyways:

- `Empty`: maps to the empty type.
- `Value`: maps to the type of all values.
- `Bool`: maps to the type `{false, true}`.
- `U8`, `U16`, `U32`, `U63`: map to the set of integers from `0` (inclusively) to `2^8`, `2^16`, `2^32`, and `2^63` (exclusively) respectively.
- `I8`, `I16`, `I32`, `I64`: map to the set of integers representable in Two's complement with 8, 16, 32, 64 bits respectively.
- `F64`: maps to the set of floats.

#### Inductive Cases

The inductive cases allow combining multiple type descriptors into a new type.

- *Unions*: If `td_0` is a type description that maps to the type `T_0`, and `td_1` is a type description that maps to the type `T_1`, then the *union description* of `td_0` and `td_1`, written as `td_0 || td_1`, maps to the [union](https://en.wikipedia.org/wiki/Union_(set_theory)) of `T_0` and `T_1`.
- *Intersections*: If `td_0` is a type description that maps to the type `T_0`, and `td_1` is a type description that maps to the type `T_1`, then the *intersection description* of `td_0` and `td_1`, written as `td_0 && td_1`, maps to the [intersection](https://en.wikipedia.org/wiki/Intersection_(set_theory)) of `T_0` and `T_1`.
- *Homogeneous Arrays*: If `td` is a type description that maps to the type `T`, then the *array description* of `td`, written as `[td]`, maps to the set of arrays whose items all have type `T`.
- *Homogeneous Maps*: If `k_d` and `v_d` are type descriptions that map to the types `K` and `V` respectively, then the *map description* of `k_d` and `v_d`, written as `k_d=>v_d`, maps to the set of maps that map keys of type `K` to values of type `V`.
- *Records*: If `k_0, k_1, ..., k_k` are values, and `td_0, td_1, ..., td_k` are type descriptions that map to the types `T_0, T_1, ..., T_k`, then their *record description*, written as `{k_0: td_0, k_1: td_1, ..., k_k: td_k}`, maps to the type of maps that contain an entry `k_i: v_i` for each `i` from zero to `k` (inclusive) such that `v_i` is of type `T_i`. Note that the map may also contain any number of additional entries.
- *Products*: If `td_0, td_1, ..., td_k` are type descriptions that map to the types `T_0, T_1, ..., T_k`, then their *product description*, written as `(td_0, td_1, ... td_k)`, maps to the type of arrays `[v_0, v_1, ..., v_k]` such that `v_0` is of type `T_0`, `v_1` is of type `T_1`, and so on.
  - We write `(td; n)` for the product of `n` times the type description `td`.
- *Sums*: If `tag_0, tag_1, ..., tag_k` are values, and `td_0, td_1, ..., td_k` are type descriptions that map to the types `T_0, T_1, ..., T_k`, then their *sum description*, written as `(tag_0: td_0) + (tag_1: td_1) + ... + (tag_k: td_k)`, maps to the type of maps of the form `{tag_i: v_i}` (exactly one entry) for some `i` from zero to `k` (inclusive) such that `v_i` is of type `T_i`.

## Syntax

Yes, there will be a syntax eventually. Multiple, in fact: human-readable, compact, and canonic ones. Possibly a way of encoding VDL type descriptions as valuable values. The compact and canonic encodings could then be defined as the typed encoding (see below) obtained from the VDL meta type description for the type of all VV-encoded type descriptions.

## Typed Encodings

If encoder and decoder are both aware of the type of values they need to handle, they can use more efficient encodings than what untyped VV can provide. These encodings can be derived mechanically from a type description. That will likely be two different typed encodings: one that minimizes size (suitable for network transfer) and one more suitable for in-memory representations (something that a programming language with [algebraic datatypes](https://en.wikipedia.org/wiki/Algebraic_data_type) would use internally).

## Further Sketches

Briefly sketching the contents of future specifications that generalize VDL:

- VDLM (valuable description language with modules) generics are parametric polymorphism where all type variables must have the kind of simple types.
- VDLM has modules for organizing named type descriptions, and can optionally make use of a package manager.
- VDLM types can be recursive, and statically tracks whether recursive types represent a hierarchy of containment, acyclic graphs, or arbitrary graphs (essentially corresponding to rust's boxes, reference counted pointers, and garbage collected pointers).
- The extension with access capabilities (read-access and/or write-access) annotates the type definitions with those capabilities. An even more general language that is aware of a hierarchy of roles can define types and annotate them with access for those different roles; such a definition can be compiled into capability-annotated definitions for a particular roles.

Other stuff:

- A query language similar to GraphQL, which is simple with non-recursive types. The query language for VDLM can allow fully recursive queries when the recursion only happens in subvalues that are part of a containment hierarchy or an acyclic hierarchy. GraphQL cannot support recursive queries at all.
- A mutation language based on identical principles.
- A language combining query language and mutation language. Might look into time travel à la Capt'n Proto.
- Orthogonal to recursive extensions, computational extensions that perform computations that are guaranteed to terminate, based on the [functions of the vvvm](https://github.com/AljoschaMeyer/vvvm#built-in-functions). Combine this with the "time travel" for the combined query and mutation language for a pretty flexible yet efficiently implementable system.
