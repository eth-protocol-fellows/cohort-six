# Pureth (EIP-7919) proposal for Reth

## Introduction

The [Pureth proposal](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md) contains a bunch of EIPs aimed at making Ethereum accessible without the need of trusted RPCs. This is crucial to Ethereum's principles, as untrusted RPCs could make the whole network trustless for constrained devices which can't run a full node themselves. The proposal aims to make Ethereum accessible to resource-constrained devices in a trustless manner.
Upon talking with my mentor, Etan about this proposal, this proposal needs to be divided into two main parts. One is implementing the [2D logs filter structure](https://eips.ethereum.org/EIPS/eip-7745) and the other is SSZ types, which build on top of [SSZ Progressive List](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7916.md). I have chosen to take up implementing the 2D logs filter in Reth.

## 2D Logs Filter

My project will focus mainly on implementing the 2D logs filter (EIP-7745) in the Reth execution client.

### Motivation

The current Bloom Filter data structure is [highly inefficient](https://ethereum-magicians.org/t/eip-7668-remove-bloom-filters/19447) as it gives a lot of false positives and it takes a lot of computation. It is useless to the point where it should be removed and replaced by a more efficient structure. [Zsolt](https://github.com/zsfelfoldi) proposed a global log value index, which is mapped into a 2D filter map, which allows for efficient querying and efficient validity proofs.

### Importance

A correct implementation of the Pureth EIP should remove the need for external indexers and standardize a proof format to achieve verifiability for the onchain data received from RPCs. 
### Project Description

This project aims to implement the EIP-7745 logs filter structure in Reth according to the [EIP specifications](https://eips.ethereum.org/EIPS/eip-7745) and similar to [Geth's implementation](https://github.com/zsfelfoldi/go-ethereum/blob/proof-poc/core/filtermaps) of the structure.

### Specification

The 2D logs filter will require a couple of different components for a full implementation:

1. A `FilterMapProvider` trait which will be an interface over the actual backend.
2. A `MapRenderer` which runs in the background adding new logs, reverting old ones or incorrect ones and storing filled Filter Maps in the Database.
3. A `Matcher` struct which retrieves logs matching to a certain criteria, e.g. by address or topics or both.
4. A `Hasher` which can get the `log_index_root` hash.
5. A `LogsProver` which can prove the contents and existence of logs pertaining to a query.
6. A `Verifier` backend which can verify the proof sent.

#### Consensus Layer changes

The consensus block header format changes to the following:

```python
rlp([
    parent_hash,
    0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347, # ommers hash
    coinbase,
    state_root,
    txs_root,
    receipts_root,
    log_index_root, # previously logsBloom
    0, # difficulty
    number,
    gas_limit,
    gas_used,
    timestamp,
    extradata,
    prev_randao,
    0x0000000000000000, # nonce
    base_fee_per_gas,
    withdrawals_root,
    blob_gas_used,
    excess_blob_gas,
    parent_beacon_block_root,
])
```

For now, this can be done by making the logsBloom field's first 256 bits the log index root hash and then appending it with zeroes.

#### Execution Layer Changes

Some of the changes to the Execution layer will include:

```rust
/// Provider interface for accessing log index data structures.
pub trait FilterMapProvider {
    fn params(&self) -> &FilterMapParams;
    fn block_to_log_index(&self, block_number: BlockNumber) -> u64;
    fn get_filter_rows(
        &self,
        map_indices: &[u32],
        row_index: u32,
        layer: u32,
    ) -> Vec<FilterRow>;
    fn get_log(&self, log_index: u64) -> Option<Log>;
}

pub struct MapRenderer {
    pub params: FilterMapParams,
    pub current_map: FilterMap,
    pub map_index: u64,
    /// Tracks row levels filled in each map.
    pub row_fill_levels: Vec<HashMap<u32,u32>>
}

pub trait Matcher {
    /// Matches a single log hash.
    pub fn single(provider: Arc<dyn FilterMapProvider>,value: B256) -> Self;
    /// Matches a particular address OR any topic.
    pub const fn any(matchers: Vec<Self>) -> Self;
    /// Matches a particular address AND given topics.
    pub fn sequence(
        params: Arc<FilterMapParams>,
        base: Box<Self>,
        next: Box<Self>,
        offset: u64,
    ) -> Self;
}
```

Some structures also require implementing the [Progressive List](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7916.md) SSZ structure, which for now can replaced by `List[type, 9999999]`, which would be trivial to change afterwards.

The EIP also requires to implement proofs for the log values and their block ranges.

Also the RPC endpoints `eth_getLogs`, `eth_newFilter`, `eth_getFilterLogs`, `eth_getFilterChanges` have to be implemented for the new data structure.

### Roadmap

#### Milestone 0: Week 6

- Get familiar with Reth codebase and figure out the parts of the codebase to change

#### Milestone 1: Week 7-10

- Implement basic containers and structures in Reth
- Lay down a basic structure/framework to work with

#### Milestone 2: Week 10-15

- Implement function for inserting logs, retrieving logs and understand how the logs proof works in detail
- Test basic insertions and retrievals of logs

#### Milestone 3: Week 16-18

- Implement proof of inclusions, completeness proofs and block range proofs

#### Milestone 4: Week 19-22

- Test whether proofs work correctly and implement RPC endpoints to fetch logs and proofs

#### Milestone 5: Week 23 onwards

- Complete spec tests, local interop testing
- Test with kurtosis, hive and other tooling
- Test RPC endpoints and valid and invalid proofs

## Possible Challenges

- Complexity of log filter structure
    - In particular, the proving logic is a bit hard to grasp fully and the layering system is also a bit technical.
    - Solution is spending time trying to understand the proof logic and using AI tools to help with it.
- Lack of tests for log filter structure
    - Tests can be easier to write once the core logic is implemented.

## Goals

The end goal of this project is to make sure Reth has a working implementation of the log filter data structure and ensuring that it is implemented within the duration of the fellowship.

## Mentors

[Etan Kissling](https://github.com/etan-status) is the mentor for this project.

## Resources

- [Original EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md)
- [Pureth Guide](https://pureth.guide/)
- [Purified Web3 website](https://purified-web3.box/)
