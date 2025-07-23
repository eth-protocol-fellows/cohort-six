# Pureth (EIP-7919) proposal for Reth

## Introduction
The [Pureth proposal](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md) contains a bunch of EIPs aimed at making Ethereum accessible without the need of trusted RPCs. This is crucial to Ethereum's principles, as untrusted RPCs could make the whole network trustless for constrained devices which can't run a full node themselves. The proposal aims to make Ethereum accessible to resource-constrained devices in a trustless manner.
Upon talking with my mentor, Etan about this proposal, this proposal needs to be divided into two main parts. One is implementing the [2D logs filter structure](https://eips.ethereum.org/EIPS/eip-7745) and the other is SSZ types, mainly [SSZ Progressive List](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7916.md). I have chosen to take up implementing the 2D logs filter in Reth.

## 2D Logs Filter

My project will focus mainly on implementing the 2D logs filter (EIP-7745) in the Reth execution client.

### Motivation
The current Bloom Filter data structure is [highly inefficient](https://ethereum-magicians.org/t/eip-7668-remove-bloom-filters/19447) as it gives a lot of false positives and it takes a lot of computation. It is useless to the point where it should be removed and replaced by a more efficient structure. [Zsolt](https://github.com/zsfelfoldi) proposed a global log value index, which is mapped into a 2D filter map, which allows for efficient querying and efficient validity proofs.

### Importance
A correct implementation of this structure should result in large speedups of querying logs and should result in significantly lower false positive rates.

### Project Description
This project aims to implement the EIP-7745 logs filter structure in Reth according to the [EIP specifications](https://eips.ethereum.org/EIPS/eip-7745) and similar to [Geth's implementation](https://github.com/zsfelfoldi/go-ethereum/blob/proof-poc/core/filtermaps) of the structure.

### Specification
The Pureth meta focuses on the Execution layer mainly, with some changes to the consensus layer.

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
Some new data structures have to be defined for the Execution layer:
```rust
#[derive(Encode, Decode)]
struct LogIndex {
    epochs: Vec<LogIndexEpoch>,
    next_index: u64,
}

#[derive(Encode, Decode)]
struct LogIndexEpoch {
    filter_maps: Vec<Vec<FilterRow>>,
    log_entries: Vec<LogEntry>,
}

type FilterRow = ProgressiveByteList[(MAX_BASE_ROW_LENGTH * log2(MAP_WIDTH)).div_floor(8 * MAPS_PER_EPOCH), LAYER_COMMON_RATIO]


#[derive(Encode, Decode)]
struct LogEntry {
    log: Log,
    meta: LogMeta,
}

#[derive(Encode, Decode)]
struct LogMeta {
    block_number: u64,
    transaction_hash: B256,
    transaction_index: u64,
    log_in_tx_index: u64,
}


#[derive(Encode, Decode)]
struct Log {
    address: Address,
    topics: Vec<Bytes>,
    data: Vec<Bytes>,
}

#[derive(Encode, Decode)]
struct BlockDelimiterEntry{
    dummy_log: Log,        // zero address and empty lists
    meta: BlockDelimiterMeta,
}

#[derive(Encode, Decode)]
struct BlockDelimiterMeta{
    block_number: u64,
    block_hash: B256,
    timestamp: u64,
    dummy_value: u64,  // 2**64-1
}

```


These structures also require implementing the [Progressive List](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7916.md) SSZ structure, which for now can replaced by `List[type, 9999999]`, which would be trivial to change afterwards.

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
- Test with kurtosis,  hive and other tooling
- Test RPC endpoints and valid and invalid proofs

## Possible Challenges
- Complexity of log filter structure
- Complexity of proof generation and verification
- Lack of tests for log filter structure

## Goals
The end goal of this project is to make sure Reth has a working implementation of the log filter data structure and ensuring that it is implemented within the duration of the fellowship.

## Mentors
[Etan Kissling](https://github.com/etan-status) is the mentor for this project.

## Resources
- [Original EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md)
- [Pureth Guide](https://pureth.guide/)
- [Purified Web3 website](https://purified-web3.box/)
