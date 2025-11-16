# EPBS: EIP-7732 – Implementation Proposal

Enshrined Proposer Builder Separation (epbs) decouples execution payloads from consensus blocks.  The consensus block will undergo a lightweight validation before the attestation deadline whereas the execution payload will be processed independently.

## Motivation
 - Execution is removed from the consensus hot path
 - Execution could have up to 9 seconds for block validation, enabling a significantly increased gas limit
 - Notable increase in blob scaling
 - In-protocol block auctions allow proposers to gain exposure to MEV rewards without relying on external relays

## Project Description 
We will implement [EIP-7732](https://eips.ethereum.org/EIPS/eip-7732) in [Lighthouse](https://github.com/sigp/lighthouse) as it only contains CL modifications.  The focus will be on implementing the parts of the spec necessary for self-building a block without blobs since that is the target requirement for the Gloas `devnet-0` launch.

Scope:
- **CL Block/body changes**: Include signed execution bids and payload attestations in addition to remove the execution payload
- **Validator duties**: Implement PTC message production/aggregation and include aggregates in the next block
- **Networking**: Add gossip support for new libp2p topics

## Specification
1. `BeaconBlockBody` is gossiped as part of a `BeaconBlock` across the network.  It used to have an `execution_payload`, which contained the full transaction list.  Instead, it will now have a lightweight payload header signed by the builder, plus payload timeliness attestations from the previous slot.
```rust
 struct BeaconBlockBody {
    // remove execution_payload
     
    // new
    signed_execution_bid: SignedExecutionBid,   // builder’s bid + EL block hash
    payload_attestations: Vec<PayloadAttestation>,  // PTC aggregations from previous slot
     
   // existing fields below
}
```

2. The `BeaconState` will be extended to track fields signaling whether a payload was received by a builder and also track the status of builder to proposer payments:
```rust
pub struct BeaconState {
    // indicates if payload was locally received per slot
    pub execution_payload_availability: BitVector<SLOTS_PER_HISTORICAL_ROOT>,

    // gather the pending builder to proposer payments for each slot in the current epoch
    pub builder_pending_payments: Vector<BuilderPendingPayment, { 2 * SLOTS_PER_EPOCH }>,
    
    // track the pending builder to proposer payments from previous epochs that are ready to be processed by the EL
    pub builder_pending_withdrawals:
        List<BuilderPendingWithdrawal, BUILDER_PENDING_WITHDRAWALS_LIMIT>,
    
    // hash of the last fully executed payload
    pub latest_block_hash: Hash256,
    
    // slot at which the last payload was executed
    pub latest_full_slot: Slot,
    
    // withdrawals root processed in CL for the latest slot
    pub latest_withdrawals_root: Hash256,

   // existing fields below
}
```

3. A Payload Timeliness Committee (PTC) will consist of 512 validators per slot and attest to whether a builder revealed their payload timely within the slot.  The `PayloadAttestationMessage` will be gossiped from committee members and aggregated by the proposer as to include in the `BeaconBlockBody`:
```rust
pub struct PayloadAttestationData {
    // the HTR of the beacon block seen for the assigned slot
    pub beacon_block_root: Hash256,
    // the committee member's assigned slot
    pub slot: Slot,
    // indicates the execution payload was received by this validator
    pub payload_present: bool,
    // indicates the blob data was received by this validator
    pub blob_data_available: bool,
}
```

4. The block proposal flow will be modified such that the proposer will request a block, which will include a bid instead of a payload, and then broadcast a block including the bid and aggregated `PayloadAttestation`:
```rust
fn propose_block() {
    let bid = get_bid(slot);
    let body = BeaconBlockBody {
        // existing fields
        signed_execution_payload_bid: {block_hash, value, ...},
        payload_attestations: collect_previous_slots_ptc(),
    };
    
    sign_and_gossip_block(slot, body);
}
```

5.  Then, when receiving a new block gossiped by the proposer, the validator will process withdrawals and the `ExecutionPayloadBid` without having to wait for the EL to process transactions anymore before attesting to block validity:
```rust
fn process_block(state: BeaconState, block: BeaconBlock) {
    process_withdrawals(state);
    process_execution_payload_header(state, block); // see below
}
```
```rust
fn process_execution_payload_bid(state: BeaconState, block: BeaconBlock) {
    // Verify the header signature
    let signed_header = block.body.signed_execution_payload_header;
    assert!(
        verify_execution_payload_bid_signature(state, signed_header)
    );

    // Check that the builder is active, non-slashed, and has funds to cover the bid
    assert!(
        state.balances[builder_index]
            >= amount + pending_payments + pending_withdrawals + MIN_ACTIVATION_BALANCE
    );

    // Verify that the bid is for the current slot
    assert!(header.slot == block.slot);

    // Verify that the bid is for the right parent block
    assert!(header.parent_block_hash == state.latest_block_hash);
    assert!(header.parent_block_root == block.parent_root);

    // Record the pending payment
    state.builder_pending_payments[SLOTS_PER_EPOCH + header.slot % SLOTS_PER_EPOCH] =
        pending_payment;

    // Cache the signed execution payload header
    state.latest_execution_payload_header = header;
}
```

6. Before the PTC deadline during a slot, the proposer will release the self-built `ExecutionPayload` inside a `SignedExecutionPayloadEnvelope` including transactions, which will be passed from the the CL to the EL like today for validation.  The validator will now execute `process_execution_payload` upon receiving the `SignedExecutionPayloadEnvelope`:
```rust
fn process_execution_payload(
    state: BeaconState,
    signed_envelope: SignedExecutionPayloadEnvelope,
    execution_engine: ExecutionEngine,
) {
    // Verify the builder's signature
    // check that the payload corresponds to the BeaconBlock for this slot

    // Pass payload to the EL for validation just like today
    assert!(execution_engine.verify_and_notify_new_payload(
        NewPayloadRequest {
            execution_payload: signed_envelope.message.payload,
            versioned_hashes: versioned_hashes,
            parent_beacon_block_root: state.latest_block_header.parent_root,
            execution_requests: requests,
        }
    ));
}
```

7. New libp2p gossip topics:
    - `execution_payload_header` will be submitted by the builder and contain the bid
    - `execution_payload` will be sent by the builder to reveal the payload before the PTC deadline
    - `payload_attestation_message` will be gossiped by the PTC committee members


With these updates, Lighthouse will properly decouple the `ExecutionPayload` from the `BeaconBlock`, which is a clean separation and enables massive L1 scaling.  

## Roadmap (Weeks 6–21)

### Phase 1: Core Data Structures (Weeks 6–9)
**Deliverables:**
- SSZ serialization for all new containers, i.e. `SignedExecutionPayloadEnvelope`
- Extend `BeaconBlockBody` and `BeaconState` with EPBS fields
- Handle backwards compatibility of pre-gloas blocks that rely on deprecated `BeaconBlockBody` and `BeaconState` fields

**Success Criteria:** All new containers serialize/deserialize correctly and `lighthouse` compiles successfully.

### Phase 2: Block Processing (Weeks 10–13)
**Deliverables:**
- Add `process_execution_payload_bid` to verify builder signatures, check builder balance covers bid amount, validate slot and parent block hash, and record pending payment in `BeaconState`
- Add `process_payload_attestations` to verify ptc votes during block verification
- Modify `process_withdrawals` to handle both standard and builder pending withdrawals
- Add `process_builder_pending_payments` in epoch processing to move qualifying payments to `builder_pending_withdrawals` with proper exit queue and churn calculations

**Success Criteria:** Valid bids are accepted and payments recorded. PTC attestations properly verified

### Phase 3: Validator Client and PTC (Weeks 14–17)
**Deliverables:**
- Modify VC to beacon node request for block with `SignedExecutionBid` instead of full payload via beacon node API during block proposal
- Add PTC committee selection using balance weighted algorithm
- Add PTC message production for committee members to attest to payload and blob data availability
- For self-building, request and sign `SignedExecutionPayloadEnvelope` from beacon node before PTC deadline

**Success Criteria:** Proposers successfully request new gloas blocks. PTC members correctly selected.  Self-built payloads signed and broadcast before deadline.

### Phase 4: Networking (Weeks 18–21)
**Deliverables:**
- Add gossip topics `execution_payload_header` (builder bids), `execution_payload` (execution envelopes), and `payload_attestation_message` (PTC vote)
- Add request/response protocol handlers for retrieving execution payload envelopes by range and by root
- Create validation pipelines for bids, ptc votes, and envelopes

**Success Criteria:** All new gossip topics properly validated and propagated.



## Possible challenges
- Maintaining backward compatibility pre-fork boundary
    - Pipelined slot structure requires PTC validation and handling of new libp2p topics
    - CL managing builder payments is non-trivial
- Consensus spec changes are evergreen such as modifications to block processing, trustless payments, and ptc committee selection
- Some specs don't exist yet, such as epbs beacon api endpoints


## Goal of the project
The overarching goal is to implement EIP-7732 in Lighthouse so that execution payloads are decoupled from consensus blocks, enabling payload validation after the attestation deadline. We want a correct and maintainable implementation that integrates cleanly with existing Lighthouse components and moves the needles towards the `devnet-0` release.

## Collaborators

### Fellow 
[Shane](https://github.com/shane-moore)

### Mentor & Contributor
Lighthouse - [Mark](https://github.com/ethdreamer)

## Resources
- [Lighthouse Github](https://github.com/sigp/lighthouse)
- [EIP-7732](https://eips.ethereum.org/EIPS/eip-7732)
- [ePBS Consensus Specs](https://ethereum.github.io/consensus-specs/specs/gloas/beacon-chain/)
- [Beacon Api Specs](https://github.com/ethereum/beacon-APIs)
