# Authors

- [Oliver Terbu](https://github.com/awoie) ([ConsenSys MESH](https://mesh.xyz))

# Status

**Version**: 0.1.1

:warning: This specification is still work in progress and the specification is subject to change. Don't use this for production use case cases. :warning:

# DID ENS Specification

The Ethereum Name Service (ENS) is a distributed, open, and extensible naming system based on the Ethereum blockchain.
ENS’s job is to map human-readable names like ’some.eth’ to machine-readable identifiers such as Ethereum addresses, other cryptocurrency addresses, content hashes, and metadata.

ENS has similar goals to DNS, the Internet’s Domain Name Service, but has significantly different architecture due to the capabilities and constraints provided by the Ethereum blockchain. Like DNS, ENS operates on a system of dot-separated hierarchical names called domains, with the owner of a domain having full control over subdomains.

Top-level domains, like ‘.eth’ and ‘.test’, are owned by smart contracts called registrars, which specify rules governing the allocation of their subdomains. Anyone may, by following the rules imposed by these registrar contracts, obtain ownership of a domain for their own use. ENS also supports importing in DNS names already owned by the user for use on ENS.

Because of the hierarchal nature of ENS, anyone who owns a domain at any level may configure subdomains - for themselves or others - as desired. For instance, if Some owns 'some.eth', she can create 'pay.some.eth' and configure it as she wishes.

ENS is deployed on the Ethereum main network and on several test networks.

The Ethereum community has established ENS names as their identifiers (see [Etherscan](https://etherscan.io/enslookup)) for web3 projects. This DID method specification has two purposes:
 1. to wrap existing ENS names as DIDs to be interoperable with applications relying on Decentralized Identifiers
 2. to define a canonical way to augment ENS names with DID capabilities such as services and verification methods.

## DID Method Name

The name string that shall identify this DID method is: ens.

A DID that uses this method MUST begin with the following prefix: did:ens. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

## Method Specific Identifier

ENS DIDs have the following format:

```
  DID     := did:ens[:<network>]:<name>
  network := mainnet | rinkeby | ropsten | goerli | ...
  name    := <ENS-name>
```

If the network is omitted, the network defaults to mainnet, so a `did:ens:some.eth` is equivalent to `did:ens:mainnet:some.eth`. However, the canonical form is `did:ens:mainnet:some.eth`.

> :warning: Issue [#2](https://github.com/veramolabs/did-ens-spec/issues/2): should we consider did:pkh as the canonical id?

ENS names are first normalized, using a process called UTS-46 normalization. This ensures that upper- and lower-case names are treated equivalently, and that invalid characters are prohibited.

### Examples

```
- did:ens:some.eth
- did:ens:my.some.eth
- ...
```

## CRUD Operations

ENS names can have TEXT records. This specification defines TEXT record names that will have an impact on DID resolution of ENS DIDs.

The following named TEXT records are defined:

- `org.w3c.did.service`

  OPTIONAL. A set of [services](https://www.w3.org/TR/did-core/#services) as per W3C DID Core specification. The service `id` property MAY be omitted. In that case the DID resolver will generate a canonical value for the specific service entry.

  > NOTE: the ENS Service will be automatically propagated as a service during DID resolution.

- `org.w3c.did.verificationMethod`
  
  OPTIONAL. A set of [verification methods](https://www.w3.org/TR/did-core/#verification-methods) as per W3C DID Core specification. Verification method `id` property values MUST be relative DID URIs, e.g., `#my-key-id-1234`. 
  
  >NOTE: the owner of the ENS name will be automatically propagated as a verification method during DID resolution.

- `org.w3c.did.verificationRelationship`
  
  OPTIONAL. A map of [verification relationship](https://www.w3.org/TR/did-core/#verification-relationships) as per W3C DID Core specification to a set of verification relationship identifiers (`id` property).
  
  >NOTE: the verification method that relates to the owner of the ENS name can be used for all verification relationships.

### CREATE

See [ENS](https://docs.ens.domains/dapp-developer-guide) on how to register ENS names.

> :warning: Issue [#4](https://github.com/veramolabs/did-ens-spec/issues/4): provide more details on ENS registration.

### READ

DID resolution will perform ENS resolution for a given ENS name. Additionally, the DID resolver will then retrieve the DID specific TEXT records for the ENS name and add default values for service endpoints and verification methods.

> :warning: Issue [#4](https://github.com/veramolabs/did-ens-spec/issues/4): provide more details on ENS resolution.

The default verification method will always include the owner of the ENS name as follows:

```json
{
  "id": "<DID-ENS>#<sha256-of-blockchainAccountId>",
  "type": "EcdsaSecp256k1RecoveryMethod2020",
  "controller": "<DID-ENS>",
  "blockchainAccountId": "<CAIP-10>"
}
```

The `id` of the default verification method is the concatenation of the ENS DID followed by the `#` and the hex representation of `sha256(blockchainAccountId)`. Additional verification methods that MAY be added MUST not use that verification method `id` and will be ignored in the DID Document.

The default service will always include the ENS service as follows:

```json
{
  "id":"<DID-ENS>#Web3PublicProfile-<sha256-of-blockchainAccountId>",
  "type": "Web3PublicProfile", 
  "serviceEndpoint": { 
    "profileService": "ENS",
    "ensName": "<ENS-name>",
    "network": "mainnet" 
  }
}
```

The `id` of the default service is the concatenation of the ENS DID followed by the `#Web3PublicProfile-` and the hex representation of `sha256(blockchainAccountId)`. Additional services that MAY be added MUST not use that service `id` and will be ignored in the DID Document.

If the DID specific TEXT records are malformed, the entire TEXT record will be ignored in the DID resolution process.

See ENS on how to resolve ENS names and how to resolve TEXT records for ENS names.

#### Example (no TEXT records)

For `did:ens:some.eth` (with no TEXT records added), the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/ens/v1",
    "https://w3id.org/casa/profile-services/v1"
  ],
  "id": "did:ens:some.eth",
  "canonicalId": "did:ens:mainnet:some.eth",
  "verificationMethod": [{
    "id": "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7",
    "type": "EcdsaSecp256k1RecoveryMethod2020",
    "controller": "did:ens:mainnet:some.eth",
    "blockchainAccountId": "eip155:1:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb"
  }],
  "service": [{
    "id":"did:ens:mainnet:some.eth#Web3PublicProfile-5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7",
    "type": "Web3PublicProfile", 
    "serviceEndpoint": { 
      "profileService": "ENS",
      "ensName": "some.eth",
      "network": "mainnet" 
    }
  }],
  "authentication": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "assertionMethod": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "capabilityInvocation": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "capabilityDelegation": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ]
}
```

#### Example (with keyAgreement)

For `did:ens:some.eth` with DID specific TEXT records added, the DID Document would look as follows:

```json
{
  "@context": [
    "https://www.w3.org/ns/did/v1",
    "https://w3id.org/ens/v1",
    "https://w3id.org/casa/profile-services/v1"
  ],
  "id": "did:ens:some.eth",
  "canonicalId": "did:ens:mainnet:some.eth",
  "verificationMethod": [{
      "id": "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7",
      "type": "EcdsaSecp256k1RecoveryMethod2020",
      "controller": "did:ens:mainnet:some.eth",
      "blockchainAccountId": "eip155:1:0xab16a96d359ec26a11e2c2b3d8f8b8942d5bfcdb"
    },
    {
      "id": "did:ens:mainnet:some.eth#zC9ByQ8aJs8vrNXyDhPHHNNMSHPcaSgNpjjsBYpMMjsTdS",
      "type": "X25519KeyAgreementKey2019", 
      "controller": "did:ens:mainnet:some.eth",
      "publicKeyMultibase": "z9hFgmPVfmBZwRvFEyniQDBkz9LmV7gDEqytWyGZLmDXE" 
    }
  ],
  "service": [{
    "id":"did:ens:mainnet:some.eth#Web3PublicProfile-5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7",
    "type": "Web3PublicProfile", 
    "serviceEndpoint": { 
      "profileService": "ENS",
      "ensName": "some.eth",
      "network": "mainnet" 
    }
  }],
  "authentication": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "assertionMethod": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "capabilityInvocation": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "capabilityDelegation": [
    "did:ens:mainnet:some.eth#5d3e82eaba0bf8991c38bd092fa5f5523b5b3bf13e06b4b29c0022a094a528d7"
  ],
  "keyAgreement": [
    "did:ens:mainnet:some.eth#zC9ByQ8aJs8vrNXyDhPHHNNMSHPcaSgNpjjsBYpMMjsTdS"  
  ]
}
```

The following TEXT records have to be set:

- `org.w3c.did.verificationRelationship`
- `org.w3c.did:verificationMethod`

> :warning: Issue [#3](https://github.com/veramolabs/did-ens-spec/issues/3): More details on TEXT records and more examples for other service endpoints and authentication relationships.

### UPDATE

See [ENS](https://docs.ens.domains/dapp-developer-guide) on how to add TEXT records.

> :warning: Issue [#4](https://github.com/veramolabs/did-ens-spec/issues/4): provide more details on ENS TEXT records.

### DELETE

See [ENS](https://docs.ens.domains/dapp-developer-guide) on how to delete ENS names or end the lease.

> :warning: Issue [#4](https://github.com/veramolabs/did-ens-spec/issues/4): provide more details on how the lease ends or ownership is transferred.

## Privacy Considerations

See [ENS](https://docs.ens.domains/dapp-developer-guide).

When any data (e.g. W3C Verifiable Credentials) is associated with ENS DIDs, sharing that data would also impose sharing the onchain data graph (e.g. transaction history, NFTs etc.) of the ETH account that owns the ENS name.

Using personal identifiable information as DID Method specific identifiers (e.g. alice.eth) discloses personal information every time the DID is shared with a counter party. This specification DOES NOT endorse the use of ENS names that correlate directly with real world human beings. 

> NOTE: The Ethereum community is already using ENS names for individuals (e.g. vitalik.eth).

## Security Considerations

See [ENS](https://docs.ens.domains/dapp-developer-guide).

ENS names are non-fungible and transferrable. When the owner of the ENS name changes, the authorative keys will also change. This needs to be considered when used in conjunction with verifiable data where the DID is embedded, e.g., W3C Verifiable Credentials.

> :warning: Issue [#1](https://github.com/veramolabs/did-ens-spec/issues/1): potentially addresses the above issue.
