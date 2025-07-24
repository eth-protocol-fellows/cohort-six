# SSZ Query Language with Merkle Proofs

## Motivation

Currently, consensus clients have limited ability to provide specific data fields in `BeaconState` along with their corresponding proofs. While [Ethereum Light Client](https://ethereum.org/en/developers/docs/nodes-and-clients/light-clients/) has introduced several paths to be proven, it lacks of standardization in how they generate and retrieve these proofs.

Retrieving an entire `BeaconState` using the [`/eth/v2/debug/beacon/states/{state_id}`](https://ethereum.github.io/beacon-APIs/#/Debug/getStateV2) is not practical. On mainnet, the size of the `BeaconState` [Slot 12145344](https://beaconcha.in/slot/12145344) is roughly **271MB**-large enough to burden the serving node and the client, and to be latency sensitive over typical network links. Moreover, the specification itself classifies the debug namespace as “*a set of endpoints intended for chain diagnostics and not for public exposure*.” ([Some](https://eth2book.info/latest/part3/containers/state/#beaconstate) argues `BeaconState` is a "God object")

A more scalable approach is to supply Merkle proofs or multiproofs. These techniques, already standard across blockchain workflows, allow data producers to transmit only the minimal cryptographic path needed to authenticate a specific field, while consumers avoid downloading the entire state. 

The inefficiency becomes evident when a verifier needs just one field that is not validators or balances:

<center>

| Field (Electra)  | Approx. size |
| ---------------- | ------------ |
| validators       | ~232MB       |
| balances         | ~15MB        |
| remaining fields | ~24MB        |
    
</center>

    
With Merkle (multi)proofs, a consumer can request the required leaf and the corresponding branch—typically only a few kilobytes—rather than the hundreds of megabytes dominated by the validators list. Related proposals, such as [partial SSZ](https://ethereum.stackexchange.com/a/84146) encoding, align well with the same objective: off-loading bandwidth and computation from both producers and consumers while preserving full verifiability.

Therefore, we need **a generic and standardized way** to fetch only necessary data with its accompanying proofs that any client can easily verify. This solution would not only dramatically enhance the "provability" of the Consensus Layer but also replace some scattered, ad-hoc implementations for specific field (e.g., `historical_summaries` in [Nimbus](https://github.com/status-im/nimbus-eth2/pull/6675)).

Looking ahead, the importance of SSZ as a core building block of Ethereum protocol will increase significantly. [Pureth (EIP-7919)](https://eips.ethereum.org/EIPS/eip-7919) suggests replacing RLP encodings with SSZ. Beam chain (or "Lean" chain) will [leverage](https://youtu.be/lRqnFrqpq4k?si=QQfF-Gwsbc8szS_B&t=1019) SSZ as its single serialization scheme.


## Project description

We propose **a new [Beacon API](https://github.com/ethereum/beacon-APIs) endpoint** that introduces an **SSZ Query Language (SSZ-QL)**, including the ability to generate **merkle proofs** for the queried data. This endpoint will offer a minimum feature set, but it will be sufficient for most use cases. (You can check out the API specification [here](https://hackmd.io/@junsong/rkAN9lIIxx).)

For a broader perspective, we're also aiming to develop a **full-featured SSZ-QL specification** that includes capabilities like filtering, range queries, and custom anchors, complete with Merkle proofs. This specification is intended for potential inclusion in the [`consensus-specs`](https://github.com/ethereum/consensus-specs/tree/dev/ssz). You can review the current draft [here](https://hackmd.io/@fernantho/rkjsksrIxg).


## Specification

A new endpoint( `/prysm/v1/beacon/states/{state_id}/query`) will be introduced which accepts `POST` HTTP method. (NOTE: `prysm` will be replaced to `eth` when it becomes official in [ethereum/beacon-APIs](https://github.com/ethereum/beacon-APIs).)

### `POST` request
    
Handles multiple queries with proof of inclusion. Response contains `proofs` field which is sorted **descending order** by the generalized index. You might check the [specification document](https://hackmd.io/@junsong/rkAN9lIIxx) for details.

> [!Warning]
> Examples below are tentative. For the latest version, refer to the [specification documents](##Resources).    
    
```json
{
    "query": [
        {
            "path": ".validators[100].withdrawal_credentials"
        },
        {
            "path": ".len(validators)"
        }
    ],
    "include_proof": true,
    "multiproof": false
}
```

*Code 1: Example response body*

When `"multiproof": false`, the server returns separate proof objects (one per requested field). 


```json
{
    "version": "electra",
    "execution_optimistic": false,
    "finalized": true,
    "data": {
        "result": [
            {
                "paths": [
                    ".validators[100].withdrawal_credentials"
                ],
                "leaves": [
                    "0xcf8e0d4e9587369b2301d0790347320302cc0943d5a1884560367e8208d920f2"
                ],
                "gindices": [
                    "1319413953332001"
                ],
                "proof": [
                    "0xcf8e0d4e9587369b2301d0790347320302cc0943d5a1884560367e8208d920f2"
                ]
            },
            {
                "paths": [
                    ".len(validators)"
                ],
                "leaves": [
                    "1000"
                ],
                "gindices": [
                    "76"
                ],
                "proof": [
                    "0xcf8e0d4e9587369b2301d0790347320302cc0943d5a1884560367e8208d920f2"
                ]
            }
        ],
        "root": "0xcf8e0d4e9587369b2301d0790347320302cc0943d5a1884560367e8208d920f2"
    }
}
```

*Code 2: Example response body for a multiproof request*
    
When `"multiproof": true`, it returns a single combined multiproof containing the shared helper hashes.

```json
{
    "version": "electra",
    "execution_optimistic": true,
    "finalized": true,
    "data": {
        "result": [
            {
                "paths": [
                    ".genesis_validators_root",
                    ".fork.current_version"
                ],
                "leaves": [
                    "0x4b363db94e286120d76eb905340fdd4e54bfe9f06bf33ff6cf5ad27f511bfe95",
                    "0x0500000000000000000000000000000000000000000000000000000000000000"
                ],
                "gindices": [
                    65,
                    269
                ],
                "proofs": [
                    "0x5730c65f00000000000000000000000000000000000000000000000000000000"
                ]
            }
        ],
        "root": "0x2d178ffec45f6576ab4b61446f206c206c837fa3f324ac4d93a3eece8aad6d66"
    }
}
```

---
    
### Generic Merkle Proof (GMP) generation

The current implementation of Merkle Proof Generation lacks of flexibility. The proofs are hardcoded and specific for each of the three implemented. 
We propose to refactor how proofs are generated in order to generate the **generic merkle proof** of any arbitrary data requested, following the methodology depicted in the [consensus-specs](https://github.com/ethereum/consensus-specs/blob/dev/ssz/merkle-proofs.md#merkle-multiproofs) and [remerkleable](https://github.com/protolambda/remerkleable). 

#### Compute Generalized Index (GI)

Convert each raw input (e.g. `.validators[100].withdrawal_credentials`) into a single GI.

#### Obtain branches to prove

For every target GI, derive (i) the *path indices* from that leaf up to—but excluding—the root, and (ii) the *branch indices* (the sibling nodes needed at each level). 

#### Compute branches

Hash every leaf to 32 bytes and gather its sibling hashes. If multiple leaves are requested, include only the hashes that cannot be recomputed-i.e. the minimal set required to assemble the compact multiproof.

#### Package the proof

Package the proof by sending back four things in one bundle: the values to be proven, the gindex of the leafs data and the indexed hashes of values that cannot be calculated from the index data and the anchor root.

#### Verify
Insert every leaf at its GI, plugs each helper hash where their GI indicates, hashes upward until reaching the root, and checks that the result equals the supplied anchor root. If they match, the data are proven to belong to the committed state.

![image](https://hackmd.io/_uploads/Bk6ogJ_Bll.png)
*Image 1: Merkle multiproof of validator not slashed, exited but not withdrawn*

Previous image shows how it looks when it comes to prove that a validator with a certain public key is **not slashed**, **exited** but yet to be **withdrawn**. GMP is provided with the `Validator` object with all the values. From them, GMP computes the GI of `pubkey`, `slashed`, `exit_epoch` and `withdrawable_epoch`. After that, it infers which are the needed hashes/proofs that allows us to reconstruct the root. As shown below, only 3 hashes are needed to prove all these four leaves:

- `h9` = `"withdrawal_credentials"`
- `h10`= `to_little_endian_bytes("effective_balance")`
- `h6` = `hash(hash("activation_elegibility_epoch"), hash("activation_epoch"))` 

For the verification, the algorithm just sets the values and hashes at the correct GI and consistently hashes all the way up to the root.
    
## Roadmap

### Implement SSZ Query for Arbitrary items (~Week 12)

- Parse `path` string into the data structure.
- Implement `POST` handler for the endpoint (`/prysm/v1/beacon/states/{state_id}/query`).
- Partially read SSZ encoded bytes.
    
### Implement Merkle Proof Generation (~Week 12)

- Visualize of merkle proofs using Jupyter Notebook ([fernantho/SSZ-QL-and-GMP](https://github.com/fernantho/SSZ-QL-and-GMP)).
- Generate merkle proofs for arbitrary items.
- Generate merkle [multiproofs](https://github.com/ethereum/consensus-specs/blob/dev/ssz/merkle-proofs.md#merkle-multiproofs) for any set of arbitrary items.
    
### Integrate two features (Week 13)
    
As this project consists of two subprojects, we need to dedicate a week to integrate them as one single feature.
    
### Wrap up and Documentation (Week 14~)

- Submit a PR to [ethereum/beacon-APIs](https://github.com/ethereum/beacon-APIs) for adding new endpoint.
- Make useful documents about SSZ and its merklelization.
- Prepare for the talk at DevConnect.
    
## Possible challenges

- The implementation must not overfit to `BeaconState`. The code must be generic enough to work with any other arbitrary SSZ object.
- Investigate DoS vector for adding a new endpoint. Based on the performance impact analysis, we might recommend adding an option to enable or disable the query feature.
- Manage properly the concurrency and prevent these queries to halt the normal functioning of Prysm client.


## Goal of the project

- **MUST** specify SSZ QL
- **MUST** add a new endpoint as a [Beacon API](https://github.com/ethereum/beacon-APIs)
- **MUST** generate merkle proofs of arbitrary items
- **MUST** demonstrate real-world queries, for example:
    - *Withdrawal credentials verification*
    - *MEV stealing detection*
    - *Validator slashing, exit epoch and withdrawable epoch*
    - *Validator partial withdrawal verification*
- **NICE TO HAVE** add *Verifying web3signer*. A flag that extends the data sent to the web3sginer any time there is a block proposal, allowing the block proposal signing duties recipient to verify certain fields such as `fee_recipient` and `graffiti`.


## Collaborators

### Fellows

- [Jun](https://github.com/syjn99) will mainly focus on **implementing the Beacon API** to expose SSZ-QL as a real-world feature.
- [Nando](https://github.com/fernantho) will focus on establishing a clear approach for **generating Merkle (multi)proofs**.

### Mentors

- [Bastin](https://github.com/Inspector-Butters)
- [Radek](https://github.com/rkapka)

## Resources
    
- Meta Issues
    - [SSZ Query Language #15343](https://github.com/OffchainLabs/prysm/issues/15343)
    - [Merkle proofs of everything #15344](https://github.com/OffchainLabs/prysm/issues/15344)
- Specification documents
    - [`postSszQuery` API specification](https://hackmd.io/@junsong/rkAN9lIIxx)
    - [SSZ-QL Specification](https://hackmd.io/@fernantho/rkjsksrIxg)
- Demonstration in Jupyter Notebook
    - [fernantho/SSZ-QL-and-GMP](https://github.com/fernantho/SSZ-QL-and-GMP)
- Initial project proposals
    - [Prysm: SSZ Query Language](https://github.com/eth-protocol-fellows/cohort-six/blob/master/projects/project-ideas.md#prysm-ssz-query-language)
    - [Prysm: Merkle Proofs of Everything](https://github.com/eth-protocol-fellows/cohort-six/blob/master/projects/project-ideas.md#prysm-merkle-proofs-of-everything)
- [Initial suggestion of SSZ-QL by Etan](https://hackmd.io/@etan-status/electra-lc#SSZ-query-language)
- [SSZ specification](https://github.com/ethereum/consensus-specs/tree/dev/ssz)
- [Sparse Merkle Multiproofs](https://www.wealdtech.com/articles/understanding-sparse-merkle-multiproofs/)