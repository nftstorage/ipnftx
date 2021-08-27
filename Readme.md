<!-- # Non-Fungible Token Description Framework (NFT-DF) -->

# InterPlanetary NFT Extensions (IPNFT)


## Simple Summary

A standard interface for non-fungible token _overlay_ data, expressed as a set of statements in form of `(entity, attribute, value)` tuples. 

## Abstract

The following standard allows for the implementation of a standard API for providing _overlay data_ for NFTs within smart contracts. This standard provides a basic functionality for publishing, tracking and indexing information associated with NFTs.

> Just as anyone can make statements about things in the physical world we aim to enable any actor on the network to make statements about specific NFTs on blockchain, regardless of their owner or contract origin. Just as in physical world individuals choose who's statements to consider any actors can choose which sources (contracts) and on what subject (attribute) to track / index.

We consider use case for NFTs in which they are entities that anyone on the network (regardless of their ownership) can publish statements about in form of (entity, attribute, value) forming open informational graph of [entity, attribute, values][EAV] similar to [RDF][]. Of course, the contract/owner of an NFT can use this protocol to provide data about their own NFT's, but the system is not limited to this in the same way that anyone on the Web can link to any website.

We aim at enable additional layers of functionality on top NFTs in an open ended, yet interoperable manner with little to no coordination.

## Motivation

A standard interface for overlay data for NFTs  allows extending NFTs in manner which:

1. Upholds NFTs immutablity guarantees. 
2. Allow arbitrary actors to extend NFTs.
3. Protocol Extensions require little or no coordination (multiformats).
4. Protocol Extensions are interoperable (multiformats) / conflict-free (append-only onchain events).


This standard is inspired by [Datomic Data Model][] and [Resource Description Framework (RDF)][RDF].

#### Examples

##### Associating CID with NFT

https://nft.storage service archives NFT assets on IPFS & filecoin, yet there is no good way to associate those [CID][]s to corresponding tokens.

##### Preview images

TODO: Scaled preview images for NFTs. 

##### IPNS

TODO: Map NFT to a CID to represent mutable content

## Specification

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in RFC 2119.

**Every NFT-AV contract must implement the NFTAV interface**

```solidity
interface ERC721 {
  /// Event adds or retracts relation between
  ///   - entity (NFT identified by `contract address`,
  ///     `uint256 tokenId` pair)
  ///   - attribute (multiformat value describing a
  ///     relation)
  ///   - value (value for the attribute)
  /// 
  /// A boolean `op` indicates whether relation is
  /// added or retracted
  event IPNFT(
    uint256 source_id, // NFT tokenId
    bytes entity, // multiformat MF(token_contract, token_id, net, chain)
    bytes attribute, // multiformat MF(cardinality, type, ...rest:MF)
    bytes value // multiformat CID or MF(RETRACT_OR_APPEND, CID)
  );
}
```

The `IPNFT` events are associate with a source NFT in their contract. This event describes
information about `entities`. When describeing information about *this* NFT the `entity` will be an empty (null) binary.


IPNFT can be used to publish and interlink metadata about entities represented as NFTs. 

TODO

## Indexing

We can create secure per-chain indexes of these events and roll all chains into a master index
for all of Web3. The events would look something like:

```
{ chainNamespace: 'eth',
  chainReference: 'mainnet',
  contractAddress: Binary,
  transaction: Binary,
  tokenId: 12987189739812,
  source: 89879798,
  entity: Binary,
  attribute: Binary,
  value: Binary
}
```

Note that every event has a `transaction` id provided by the chain.

You can then imagine a REST interface that caches access to this information to make it easily
available to web developers.

```
/ipnft/eth/mainnet/{contractAddress}/{tokenId}/{entity}/{attribute}
```

When the values are CID's we can do everything we do from gateways using these mutable identifiers.

```
/ipnft/eth/mainnet/{contractAddress}/{tokenId}/{entity}/{attribute}/_latest?format=jpeg
/ipnft/eth/mainnet/{contractAddress}/{tokenId}/{entity}/{attribute}/_latest?format=car
```

We can put the last transaction in a header so that you can query its history since a previous state.

```
/nftx/eth/mainnet/{contractAddress}/{tokenId}/{entity}/{attribute}/_since?transaction={transaction}
```

This would allow any number of new database to be written since this effectively globalizes a
cache of the change feed of any database written to this protocol, without these gateways
even needing to be aware of the underlying database protocol.

## Rationale

TODO

## The OPTIONAL description protocol extension

Each `IPNFT` event is a fact / statement that represent addition or retraction of a relation between an **entity**, an **attribute** and a **value**. This forms triples graph where `entity` denotes the resource (usually NFT in some chain), and the `attribute` denotes traits or aspects of the _entity_, and expresses a relation between the `entity` and a the `value`.

> While some indexes can choose to unify set of statements across (some or all) sources _(that are globally identifiable by `contract`, `source_id` pair)_, others may choose to namespace by a `contract` or `source_id` or both.
> 
> This allows both interoperability before standardisation and extensiblity without breaking compatiblity.

### Entity

An entity denotes the subject about which statement are made.

When `subject` is empty (0 bytes) it is considered to refec to the `source` token itself (as in `(emitting_contract, source)` pair). This is convinient when contract want to make statements about its own tokens.

However `subject` may encode information about other tokens, even in other chains. It is adviced to encode this information in [multiformat][]:

- **chain** - blockchain identifier
- **network** - network identifier, mainnet, testnet, etc...
- **contract** - contract address that issued token
- **token_id** - token identifier

// TODO: this should be a generic multiformat for identifying NFT's, similar to what ceramic and did do.

### Attributes

Every attribute is identified by a [multiformat][] value that follows (TCS+) (cardinality, type, subject) encodes it's semantics:

- **Cardinality**, specifying whether NFTs can have one or a set of values for the attribute.
- **Type**, the type allowed for an attribute's value. That is one of the following types:
    - boolean
    - integer
    - float
    - string
    - bytes
    - link
- **Predicate**, specifying rest of relation semantics. This can be just an arbitrary name or multiformat encoding more domain semantics.



### Attribute Values

NFT Attribute Values MUST be [multiformat][] 
that encode following:

- **OP** - Retraction or Addtion
- **Data** - Value that MUST correspond to the an attribute type. If attribute type is _link_ data MUST be a valid [CID][].

### References

- [Datoms][] from datomic
- [Resource Description Framework (RDF)][rdf]
- [InterPlanetary Name System (IPNS)][IPNS]
- [ERC-721]
- [multiformat]
- [did:key]
- [IPLD]
- [EAV]
- [CID]

### Notes



[did:key]:https://w3c-ccg.github.io/did-method-key/
[datoms]:https://docs.datomic.com/cloud/whatis/data-model.html#datoms
[rdf]:https://en.wikipedia.org/wiki/Resource_Description_Framework
[IPNS]:https://docs.ipfs.io/concepts/ipns/
[ERC-721]:https://eips.ethereum.org/EIPS/eip-721
[multiformat]:https://multiformats.io/
[UCAN]:https://whitepaper.fission.codes/access-control/cap-authz
[IPLD]:https://github.com/ipld/specs/blob/master/data-model-layer/data-model.md
[EAV]:https://en.wikipedia.org/wiki/Entity%E2%80%93attribute%E2%80%93value_model
[CID]:https://docs.ipfs.io/concepts/content-addressing/
[Datomic Data Model]:https://docs.datomic.com/cloud/whatis/data-model.html