# Delayed Execution: EIP-7886 - Implementation Proposal

Delayed Execution splits block validation into two steps. When you receive block N, the execution layer only checks its header and verifies that it correctly references block N – 1’s execution results (state root, receipts, logs). Running block N’s transactions happens later, so the validator can vote on the block much faster.

A nice overview of the delayed execution proposal, EIP-7886, is detailed in https://nerolation.github.io/delayed-execution-docs/.

## Motivation
  - Reduce block validation and attestation latency
  - Significant throughput improvements across the network
  - Higher block gas limit

## Project Description 
We will implement this EIP-7886 across Lighthouse and Reth since it contains both CL and EL changes respectively. Then, we'll be able to hook up both layers in a Kurtosis devnet for testing and benchmarking.

Both the EL and CL will need to modify their `Header` structs to include the parent block's execution outcomes.

Reth Specific Changes:
- Update the live sync path and backfill sync path:
    -  validate the previous block's execution outcomes against the header
    -  take snapshots before running transactions
    -  deduct fees from senders upfront
    -  roll back tsx on gas mismatches

Lighthouse Specific Changes:
- Update both the block proposal and validation flows to support the new `parent_*` fields


## Specification
### EL Changes (Reth)
1. Block Header `struct` updates:
```rust
 struct Header {
    pre_state_root:            B256,  // was state_root
    parent_transactions_root:  B256,  // was receipt_root
    parent_receipt_root:       B256,  // was receipt_root
    parent_bloom:              Bloom, // was logs_bloom
    parent_requests_hash:      B256,  // was request_hash
    parent_execution_reverted: bool,  // tracks execution reversion
    // unchanged fields below 
 }
```

2. A new struct `DelayedExecutionOutcome` will be used to cache block execution results and can be used for both live and backfill sync.
```rust
pub struct DelayedExecutionOutcome {
    pub last_transactions_root:  B256,
    pub last_receipt_root:       B256,
    pub last_block_logs_bloom:   Bloom,
    pub last_requests_hash:      B256,
    pub last_execution_reverted: bool,
}
```
  - Note that in the live sync path, these execution outcomes will likely be stored in a `Hashmap` on the in-memory `Tree_State` as to have a cache of outcomes that can be used to validate the canonical chain and also for re-orgs.
  - For the backfill sync path, we can likely store these results between each block being executed in the `ExecutionStage` and overwrite the value after each block executes since backfill sync is a canonical linear chain from my understanding.

3. Before executing the block, the live sync's `on_new_payload` and the backfill sync's `ExecutionStage` will perform the following static checks past the fork boundary:
``` rust
fn validate_static(hdr, parent_outcome, current_state_root) {
  // pre_state_root must match the cached state root of block N‑1
  assert hdr.pre_state_root == current_state_root

  // all other parent_* fields vs outcome
  assert hdr.parent_transactions_root   == parent_outcome.last_transactions_root
  assert hdr.parent_receipt_root   == parent_outcome.last_receipt_root
  assert hdr.parent_bloom          == parent_outcome.last_logs_bloom
  assert hdr.parent_gas_used       == parent_outcome.last_gas_used
  assert hdr.parent_requests_hash  == parent_outcome.last_requests_hash
  assert hdr.parent_execution_reverted == parent_outcome.last_execution_reverted
    
// Additional checks per the EIP: (signatures, nonces, balances, inclusion gas ≤ header.gas_used, blob gas match, withdrawals root)
}
```
Note that once these checks are complete, the live sync path is free to send a `Valid` status if successful back to the CL.


4. Following static validation, the live sync path (`insert_block`) and backfill sync path (`ExecutionStage`) will then be responsible for the execution flow:
```rust  
fn execute(hdr) {
    // New (past fork boundary):
    // 1. Take a snapshot of the state before execution. 
    // Live sync and backfill sync path should be able to use
    //  revm's snapshot mechanism for this
    //  
    // 2. Pre-charge senders for maximum possible fees upfront
    // 
    // 3. Process the transactions sequentially. 
    // If block_output.block_gas_used + tx.gas > block_env.block_gas_limit
    //  or the resulting block_output.gas_used != hdr.gas_used, 
    //  set outcome.last_execution_reverted = true, 
    //  stop executing the transactions
    // 
    // 4. If outcome.last_execution_reverted = true, 
    // rollback the state to the snapshot, 
    // set gas_used = 0, 
    // and reset the execution outputs, i.e. receipts/logs. 
    // Else, save the state
    // 
    // 5. Save the gas_used, transactions_root, receipt_root,
    //  logs_bloom, and requests_hash to an in-memory cache 
    //  to be used when validating the next block's headers
}
```

5. `EngineApi` trait mods:
- Add `new_payload_v5`
    - accepts `ExecutionPayloadV4` with the additional fields for block validation.  We will have to modify the payload status to be returned after static validation.
- `engine_getPayloadV5`
    - returns `ExecutionPayloadV4` with new fields for block building
- extend `EngineTypes` to define a new`ExecutionPayloadEnvelopeV5` and related types. 
- add `EngineApiMessageVersion::V5` for the new hardfork.

6. Structs like `ForkTracker` will need to have a placeholder hard fork property for use as to apply the logic above only after the fork boundary. 

### CL Changes (Lighthouse)
1.  The `ExecutionPayload` struct is included in the `BeaconBlockBody`, so we will need to add a new variant to apply past the fork boundary that will include our new fields.
```
enum ExecutionPayload<E: EthSpec> {
    Eip7886(ExecutionPayloadEip7886<E>), // new variant
}
```

2.  Additionally, the evergreen `BeaconState` will need to be updated past the fork boundary to have its `ExecutionPayloadHeader` hold the new `parent_*` fields.

3.  The EL's block header is RLP encoded and then hashed to obtain the`block_hash`.  Since `block_hash` is stored in our `ExecutionPayload`, we'll need an `ExecutionBlockHeader` variant that includes the new fields.  The `block_hash` is later recalculated by the EL to check for validity.  The `block_hash` will also be persisted in the `BeaconState::ExecutionPayloadHeader`.

4.  Lighthouse will JSON serialize the `ExecutionPayload` to send to the EL via the engine api for block validation.  Therefore, we'll need a `new_payload_eip7886` helper to forward the payload to the engine.  Similarly, a `get_payload_eip7886` helper will return payloads for block building.

5.  Validator clients can request blinded blocks where only a payload header is sent. We'll need to extend this header:
```rust
pub struct BlindedPayload<E: EthSpec> {
    execution_payload_header_eip7886: ExecutionPayloadHeaderEip7886<E>,
    // existing variants
}
```

With these updates, Lighthouse will properly store parent execution output fields in the `BeaconState` and the `BeaconBlockBody`.  

## Roadmap

| Period                    | Goals                                                                                              |
|---------------------------|-----------------------------------------------------------------------------------------------------|
| July – mid August         |  Lighthouse - update `BeaconState` and `BeaconBlockBody` execution payloads     |
| Mid August – Mid October|  Header struct + engine api updates.  Then, update `ExecutionStage` + live sync path |
| Mid October – DevConnect   | Testing and benchmarking of proposed changes     |

## Possible challenges
- Maintaining backward compatibility pre-fork boundary will be complex


## Goal of the project
- Fully implements EIP-7886 across Lighthouse and Reth in order to benchmark performance gains in a Kurtosis devnet

## Collaborators

### Fellows 
N/A

### Mentors
- Reth - [Roman](https://github.com/rkrasiuk)
- Lighthouse - [Mark](https://github.com/ethdreamer)

## Resources
- [Lighthouse Github](https://github.com/sigp/lighthouse)
- [Reth Github](https://github.com/paradigmxyz/reth)
- [Delayed Execution Info Doc](https://nerolation.github.io/delayed-execution-docs/)
- [EIP-7886 Delayed Execution](https://eips.ethereum.org/EIPS/eip-7886)
- [EELS Spec](https://github.com/fradamt/execution-specs/tree/delayed-exec-basic)
