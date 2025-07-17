## Pureth [EIP-7919](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md) Implementation for Nimbus-EL

### Collaborators

- [Vineet](https://github.com/vineetpant) 
- [Tamaghana](https://github.com/RazorClient)

We will coordinate and work parallely on separate EIPs for implementation, once core EIPs (7745 and 6404) are done , we will work on other EIPs for Log Reform and Remove old tx types categories in Pureth.

### LOG Reform for Pureth

#### Motivation

Since Pureth is Meta EIP and Log Reform is really importance part of Pureth which addresses major inefficiencies in current Logs.
Working on LOG Reform would solve following shortcomings :

- Smart contract wallets have to fetch and verify additional transaction receipts within a queried block range because Bloom filters produce false positives, leading to inefficient log retrieval.
- Fetching full receipts because of false positives is impractical for Light clients and mobile devices with resource constraints.
- Eth transfers from smart contracts don't emit logs which leads to problems in tracking transfers.

#### Why important ?

Working on Log Reform is important for following reasons

- Recent implementation from `Geth` has shown that new log indexer returns the results for `eth_getLogs` in milliseconds which other clients with BloomFilter might take few seconds.
- It will help lightClient and dapps to be more efficient by reducing the unnecessary data fetching.
- Reduce data overhead with compact `SSZ` encoding, enable trusless log retrieval with `Merkle proofs`.
- SSZ encoded logs will be available for all Eth Transfers(e.g., CALL, SELFDESTRUCT) and will be good for tracing.

#### Project description

We will start the work by focussing on the implemention of [EIP-7745:Two dimensional log filter data structure](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7745.md) in Nimbus EL :-

- Implement a two-dimensional SSZ index (by block and transaction) for precise, efficient log retrieval.
- Provide SSZ-encoded logs with Merkle proofs, verifiable against block headers.
- Remove the need for Bloom filters.
- Updated eth_getLogs with proofs

Once the work on EIP-7745 is complete, we will prioritize other log reform EIPs.

1. EIP-7708: ETH Transfers Emit a Log
   The purpose of this EIP is to make all Eth transfers emit logs because currently smart contracts don't emit logs for Eth transfers and this leads to problem in tracking the transfers sometimes.

2. EIP-7799: System Logs
   Some Eth balance changes are missing even with after the implementation of EIP-7708, So this EIP aims to add a new list to add all Block level logs so that `eth_getLogs` will be able to provide all the history.

#### Specification

Implementing EIP-7745 in [Nimbus-EL](https://github.com/status-im/nimbus-eth1) involves updating the RPC layer to build and query the SSZ log filter, generate Merkle proofs, and handle pattern matching.

1.  Define LogEntry and LogFilter structs

```nim
type LogEntry = object
address: Address
topics: List[Bytes32, 4]
data: Bytes
blockNumber: uint64
txHash: Bytes32
logIndex: uint64
type LogFilter = ProgressiveList[LogEntry]
```

- Use `ProgressiveList` for dynamic log growth

2. Build Two-Dimensional Log Index

The log index is constructed during receipt processing to enable efficient log retrieval. Each log entry is stored with its associated metadata, and logs are organized in a two-dimensional structure (by block and transaction). Additionally, Logs are stored in per-block Merkle trees, with roots linked to the block’s state or receipt trie for verifiable queries.

```nim
proc buildLogIndex(receipts: seq[Receipt], block: Block): LogFilter =
    for receipt in receipts:
        for log in receipt.logs:
            result.add(LogEntry(
                address: log.address,
                topics: log.topics,
                data: log.data,
                blockNumber: block.number,
                txHash: receipt.txHash,
                logIndex: log.index
            ))

    # Add a dummy log entry to mark the end of the block's logs
    result.add(LogEntry(
        address: Address(zeros(20)),
        topics: @[],
        data: @[],
        blockNumber: block.number,
        txHash: Bytes32(zeros(32)),
        logIndex: high(uint64)
    ))
```

3. Implement FNV-1a Hashing for Filtering

4. Pattern Matching and Verification

   - Implement `singleProver`, `matchAnyProver`, and `matchSequenceProver` (inspired by Geth PoC)
   - Use set operations for union (matchAnyProver) and intersection (matchSequenceProver)

5. Process Partial Proofs

   - Handle unproven gaps in filter rows
   - Treat unproven gaps as potential match ranges, ensuring exhaustiveness. Verify count field for partial proofs.

6. Update `eth_getLogs`

   - Modify to return SSZ-encoded LogFilter with proofs

7. Test with Provided Vectors

   - Implement binary proof parsing (little-endian) and validate against JSON test cases. Integrate with Nimbus’s test suite (tests directory).

8. Test with Kurtosis (Local testnet)
   - Send transactions (e.g., ERC-20, EIP-7708 logs), verify SSZ logs and proofs

#### Roadmap

The roadmap and implemention timeline can be broken as follows:-

#### Phase 1: 2 weeks

- Finalize the technical implementation details.
- Nimbus-EL architechture understanding.
- Basic Understanding of Nim language

#### Phase2: 7-8 weeks

- EIP-7745: Two-Dimensional Log Filter Data Structure.
  - Replace bloom filter with 2D log index
  - Test new logs against JSON RPCs
  - Test with Kurtosis

#### Phase3: 6-7 weeks

- For other Log reform EIPs

  - EIP-7708: ETH Transfers Emit a Log
    - Effort estimate
      - Should take 5 weeks (approx) to implement
      - Emit SSZ logs for opcodes like SELFDESTRUCT or CALL transfers.
      - Simulate tests for Eth transfer in smart contract wallet

  - EIP-7799: System Logs
    - Effort estimate
      - Shold take same effort as EIP-7708

#### Possible challenges

- Learning Nim and Nimbus Architecture
- Building a two-dimensional SSZ-based log index (ProgressiveList, EIP-7916) with block and transaction dimensions.
- Validating the implementation against test vectors (proof_test_vectors) and ensuring compatibility with Kurtosis testnet queries is time-consuming.

#### Goal of the project

- Trustless Log Retrieval

  - Support decentralization goals of Ethereum by reduding reliance on trusted JSON RPC providers.
  - Providing ssz encoded logs with Merkle proofs will enable light clients, dapps and smart contract wallets to verify correcness of logs against block headers.

- Eliminate Bloom Filter Inefficiencies

  - Replace bloom filter with 2D log Log index will reduce unnecessary data fetching due to false positives nature of bloom filters

- Improve scalability and efficiency of log queries `eth_getLogs`

### Mentors

[Etan](https://github.com/etan-status) is the mentor for Pureth EIP-7919 implementation

### Resources

| Resource                                | link                                                                   |
| --------------------------------------- | ---------------------------------------------------------------------- |
| Pureth EIP-7919                         | https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7919.md          |
| EIP-7745 : 2D log filter data structure | https://github.com/ethereum/EIPs/blob/master/EIPS/eip-7745.md          |
| EIP-7745 details by Zsolt               | https://github.com/zsfelfoldi/eip-7745/blob/main/proof_verification.md |
