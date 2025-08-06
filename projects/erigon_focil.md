# Fork-Choice Enforced Inclusion Lists (FOCIL) For Erigon 

## Motivation

[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) is a potential candidate for the glamsterdame upgrade which focuses on improving the block building on Ethereum. Currently block building is centralized and not resistant to censorship. With this EIP ethereum expects to fix these issues and improve the block building capabilities of the chain.

FOCIL enforces some rules when the blocks are being built. It forces the block to include some transactions via IL's(inclusion lists). These inclusion lists are built by random committee members. These inclusion lists can be created using the ruling chosen by the client software. Once these lists are created their inclusion in the block are checked for the block to be accepted/rejected.

Block building is very centralized today and FOCIL can help mitigate it.

## Project description
This project focuses implementing FOCIL for erigon. [Erigon](https://github.com/erigontech/erigon) has execution client and a built in consensus client aswell. This project will implement it for both.

For implementing the required changes in both EL and CL. I will be following the exectuion specs and consensus specs for FOCIL implementation
* [Consensus spec](https://ethereum.github.io/consensus-specs/specs/_features/eip7805/beacon-chain/)
* [Execution spec](https://github.com/ethereum/execution-specs/pull/1214)


## Changes for the client

#### Execution Layer Changes:
* Build ILs from the mempool when requested by the consensus client.
* Enforces ILs during block building and validation.
  ##### New Engine API methods:
* `engine_getInclusionListV1` endpoint to retrieve an IL from the EL client.
* `engine_updatePayloadWithInclusionListV1` endpoint to update a payload with the IL that should be used to build the block. 
* `engine_newPayload`endpoint to include a parameter for transactions in ILs determined by the IL committee member.



#### Consensus Layer Changes:

##### Presets:
* `DOMAIN_IL_COMMITTEE`
* `IL_COMMITTEE_SIZE`
* `MAX_BYTES_PER_INCLUSION_LIST`

#### New Containers:

**Inclusion List**

 ```go
type InclusionList struct {
	Slot                       string   `json:"slot"`
	ValidatorIndex             string   `json:"validator_index"`
	InclusionListCommitteeRoot string   `json:"inclusion_list_committee_root"`
	Transactions               []string `json:"transactions"`
}
```

**Signed Inclusion List**

```go
type SignedInclusionList struct {
	Message   *InclusionList `json:"message"`
	Signature string         `json:"signature"`
}
```
#### P2P changes:

A new global topic for broadcasting `SignedInclusionList` objects.

```go
const GossipInclusionList = "inclusion_list"
```

## Roadmap

1. Complete the EL changes by mid August.
2. Complete the CL changes by the end of September.
3. Complete testing and fixes by the end of October.
## Possible challenges

* I haven't explored Erigon's consensus layer so figuring out my way to move around the codebase will be the challenge in my opinion.


## Goal of the project

The end goal of the project is to have a tested FOCIL implementation for Erigon's execution layer and consensus layer.

## Collaborators

### Fellows 

NA

### Mentors

Mark Holt and Giulio Rebuffo

## Resources
* [FOCIL consensus specs](https://github.com/ethereum/consensus-specs/tree/e678deb772fe83edd1ea54cb6d2c1e4b1e45cec6/specs/_features/eip7805)
* [EIP-7805](https://eips.ethereum.org/EIPS/eip-7805)
* [Erigon focil issue](https://github.com/erigontech/erigon/issues/16123)