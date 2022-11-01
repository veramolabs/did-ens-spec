# Authors

- [Oliver Terbu](https://github.com/awoie)
- [Nick Reynolds](https://github.com/nickreynolds) ([Veramo](https://veramo.io))

# Status

**Version**: 0.2.0

:warning: This specification is still work in progress and the specification is subject to change. Don't use this for production use case cases. :warning:

# Abstract

This document defines a DID Specification called `did:ens` that utilizes the capabilities of the Ethereum Name Service (ENS) as well as existing `did:ethr` functionality to provide a DID Method that is secure, decentralized, supports DID CRUD operations such as rotating keys and declaring service endpoints, and links to the "username-like" identifiers of ENS.

Like `did:ethr`, this DID is self-soverign, compatible with any EVM network, partially resolvable on-chain, while minimizing (to some degree) the cost of updates to the DIDDocument. By linking this with ENS, we augment the capabilities of `did:ethr` with the decentralzied management of unique and *recognizable* identifiers.

# DID ENS Specification

The Ethereum Name Service (ENS) is a distributed, open, and extensible naming system based on the Ethereum blockchain.
ENS’s job is to map human-readable names like ’some.eth’ to machine-readable identifiers such as Ethereum addresses, other cryptocurrency addresses, content hashes, and metadata.

ENS has similar goals to DNS, the Internet’s Domain Name Service, but has significantly different architecture due to the capabilities and constraints provided by the Ethereum blockchain. Like DNS, ENS operates on a system of dot-separated hierarchical names called domains, with the owner of a domain having full control over subdomains.

Top-level domains, like ‘.eth’ and ‘.test’, are owned by smart contracts called registrars, which specify rules governing the allocation of their subdomains. Anyone may, by following the rules imposed by these registrar contracts, obtain ownership of a domain for their own use. ENS also supports importing in DNS names already owned by the user for use on ENS.

Because of the hierarchal nature of ENS, anyone who owns a domain at any level may configure subdomains - for themselves or others - as desired. For instance, if Some owns 'some.eth', she can create 'pay.some.eth' and configure it as she wishes.

ENS is deployed on the Ethereum main network and on several test networks.

Some in the Ethereum community have established ENS names as their identifiers (see [Etherscan](https://etherscan.io/enslookup)) for web3 projects. This DID method specification has two purposes:
 1. to wrap existing ENS names as DIDs to be interoperable with applications relying on Decentralized Identifiers
 2. to define a canonical way to augment ENS names with DID capabilities such as services and verification methods.

## DID Method Name

The name string that shall identify this DID method is: ens.

A DID that uses this method MUST begin with the following prefix: did:ens. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

ENS DIDs have the following format:

```
  DID     := did:ens:[<network>/<chainId>]:<name>
  network/chainId := mainnet | arbitrum | goerli | 0x1 | 0x5 ...
  name    := <ENS-name>
```

If the network is omitted, the network is assumed to be ethereum mainnet, so a `did:ens:some.eth` is equivalent to `did:ens:mainnet:some.eth`. However, the canonical form is `did:ens:mainnet:some.eth`.

ENS names are first normalized, using a process called UTS-46 normalization. This ensures that upper- and lower-case names are treated equivalently, and that invalid characters are prohibited.

### Examples

```
- did:ens:some.eth
- did:ens:my.some.eth
- ...
```

### Connection to `did:ethr`

In order to provide the most functionality with the least resource expeniture for implementers, we rely heavily on a connection between `did:ens` and `did:ethr` (which already has available implementations for resolving and managing DIDs). In particular, `did:ens` is treated as a "wrapper" around `did:ethr`. Since every ENS name resolves a document that includes a field `controller`, which corresponds to the ethereum address that controls the ENS name, we can simply treat the `did:ens` as a pointer to the `did:ethr` of the associated `controller`.

Because `did:ethr` resolution relies on the existence of a deployed `ethr-did-registry` contract, this DID method should only be considered resolvable on EVM chains that have *both* an `ENS Registry` and an `Ethr DID Registry` contract deployment.

## CRUD Operations

### READ

DID resolution will perform ENS resolution for a given ENS name, on the specified chain. Then, perform `did:ethr` resolution of the ENS name's `controller` Ethereum address (on the same chain).

The DID Document returned is simply the DID Document of the `did:ethr:<controller>`, with the top-level `id` field changed to `did:ens:<ensname>`

### CREATE / DELETE

The Create operation is implicit. If an ENS name can be resolved, and has a controller, then it has a `did:ens`, since the controller already has a `did:ethr`. In this way, 
`did:ens` DIDs are not "created", and cannot be destroyed (except possibly by setting the `controller` to the Null Address)

### UPDATE

The Update operation is largely handled by the `did:ethr` specification. When adding/removing a key or service for a `did:ens`, simply add/remove it from the associated `did:ethr`

It could also be considered part of the Update operation when the controller of an ENS name is changed, as that will change the DID Document resolved.

#### Example (no TEXT records)

For `did:ens:some.eth` (with no TEXT records added), the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/secp256k1recovery-2020/v2"
  ],
  "id": "did:ens:some.eth",
  "verificationMethod": [
    {
      "id": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb",
      "blockchainAccountId": "eip155:1:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb"
    }
  ],
  "authentication": [
    "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ],
  "assertionMethod": [
    "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ]
}
```

#### Example (with keyAgreement)

For `did:ens:some.eth` with DID specific TEXT records added, the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/secp256k1recovery-2020/v2"
  ],
  "id": "did:ens:some.eth",
  "verificationMethod": [
    {
      "id": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb",
      "blockchainAccountId": "eip155:1:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb"
    },
    {
      "id": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#keyAgreement",
      "type": "X25519KeyAgreementKey2019", 
      "controller": "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb",
      "blockchainAccountId": "eip155:1:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb"
    }
  ],
  "authentication": [
    "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ],
  "assertionMethod": [
    "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ],
  "keyAgreement": [
    "did:ethr:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#keyAgreement"  
  ]
}
```

For `did:ens:goerlie:some.eth` (ENS name on EVM chain other than mainnet)

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/security/suites/secp256k1recovery-2020/v2"
  ],
  "id": "did:ens:goerli:some.eth",
  "verificationMethod": [
    {
      "id": "did:ethr:0x5:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:ethr:0x5:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb",
      "blockchainAccountId": "eip155:5:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb"
    }
  ],
  "authentication": [
    "did:ethr:0x5:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ],
  "assertionMethod": [
    "did:ethr:0x5:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb#controller"
  ]
}
```

## Privacy Considerations

See [ENS](https://docs.ens.domains/dapp-developer-guide).

When any data (e.g. W3C Verifiable Credentials) is associated with ENS DIDs, sharing that data would also impose sharing the onchain data graph (e.g. transaction history, NFTs etc.) of the ETH account that owns the ENS name.

Using personal identifiable information as DID Method specific identifiers (e.g. alice.eth) discloses personal information every time the DID is shared with a counter party. Precaution should be taken when using this DID method to share sensitive information, but could be a good choice for verifiable data that is meant to be public (e.g. social network posts)

> NOTE: The Ethereum community is already using ENS names for individuals (e.g. vitalik.eth).

## Security Considerations

See [ENS](https://docs.ens.domains/dapp-developer-guide).

ENS names are non-fungible and transferrable. When the owner of the ENS name changes, the authorative keys will also change. This needs to be considered when used in conjunction with verifiable data where the DID is embedded, e.g., W3C Verifiable Credentials.

> :warning: Issue [#1](https://github.com/veramolabs/did-ens-spec/issues/1): potentially addresses the above issue.
