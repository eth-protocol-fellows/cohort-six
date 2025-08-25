# Lodestar: Backfill Sync & Historical Data Serving

## Motivation
Lodestar currently does not sync backwards the blocks prior to the checkpoint epoch after a checkpoint sync, so it neither retrieves nor serves historical blocks and blobs.
This leads to:
- Violation of expected peer behaviour, shrinking the pool of peers other nodes in the network can sync from.
- Lodestar’s low usefulness during large network outages, hurting client diversity and network resilience.
- Incompatibility with PeerDAS custody groups that rely on long-term blob availability for data availability sampling.

## Project description
Implement **Backfill Sync & Historical Data Serving**.
1. Walk backwards from the checkpoint sync point, downloading blocks, blobs and column proofs as per the respective fork.
2. Store them in archive data repositories and update `earliestAvailableSlot`.
3. Answer incoming p2p `BeaconBlocksByRange`, `BlobSidecarsByRange` and `DataColumnSidecarsByRange` requests for recent(spec-specified) epochs.

## Specification
1. **Primary BackfillSync Class**
   - `BackfillSync` as a service.
   - Event loop designed to be run in a Worker thread preventing blocking the main thread.

2. **Database**
   - Bucket for Backfill related data:
     ```
        backfill_state = 42, // stores Epoch -> EpochBackfillState data and backfill range singleton object
     ```
   - `BackfillStateRepository` Schema:
     ```ts
         type EpochBackfillState = { hasBlock: boolean; hasBlobs: boolean|null; columnIndices: number[]|null };
         type BackfillRange       = { beginningEpoch: number; endingEpoch: number };
     ```
   - Singleton key `BACKFILL_RANGE_KEY = -1` to track global progress.

3. **Peer management**
   - Listening to `NetworkEvent.peerConnected` and `NetworkEvent.peerDisconnected` events from the Backfill worker thread.
   - Maintain and manage a scored peer set (with storage map, pruning mechanism, size bounding, etc).
   - `BroadcastChannel` to avoid duplication of messages. (needs Investigation)

4. **ReqResp pipeline**
   - Check `BackfillStateRepository` for existing state of next epoch to get backfilled.
   - Create `beacon_blocks_by_range`, `blob_sidecars_by_range` and `data_column_sidecars_by_range` requests.
   - Gate requests by fork.
   - Validate responses and down-score bad peers.

5. **Persistence & Updating States**
   - Only persist data if  full epoch data is available:
     - `BlockArchive.put(block)`
     - `BlobArchive.put(blobs)`
     - `ColumnArchive.put(columns)`
   - Update `BackfillState` repository and `BackfillRange`.
   - Update `earliestAvailableSlot` on both the main and network threads.

## Roadmap
| Phase | Weeks | Deliverables |
|-------|-------|--------------|
| Core logic | 6-11 | Primary Backfill Class, DB Repositories, Peer Management, ReqResp Pipeline, State Management |
| Integration | 12-15 | Fully Integrate into node startup |
| Testing | 16-19 | Unit + integration tests, metrics, benchmarking |
| Optimization & docs | 20-22 | Final optimizations and documentation |

## Possible challenges
- **Cross-Thread State Management**: The most significant challenge is safely and efficiently managing the peer list and sync state across the network, main, and (future) backfill worker threads.
- **Resource Management**: The backfill process must be carefully rate-limited to prevent it from consuming too much CPU, disk I/O, or network bandwidth, which could impact the node's real-time, high-priority duties. It needs careful consideration considering trade-offs between performance and data availability. This point applies specifically to solo node runners/home-stakers.
- **Peer Reliability**: Need robust timeouts and scoring to defend against slow or malicious peers and potential DoS attacks.
- **Fork boundaries**: Backfill must select correct SSZ types across Deneb, Fulu and others.

## Goal of the project
A beacon node that, after a checkpoint sync, automatically backfills to genesis and serves any peer up to the spec-mandated retention depth. Success metrics:
- Passes new backfill test-vectors(unit and integration).
- Syncs mainnet in considerably low time on consumer hardware.
- Reduce down-scoring and disconnections from peers due to absence of sufficient historical data.
- Merge to `chainsafe/lodestar:unstable` before the v2.0 release.

## Collaborators
Fellows: [@vedant-asati](https://github.com/vedant-asati) \
Mentors: [@matthewkeil](https://github.com/matthewkeil) from the lodestar team

## Resources
- [Lodestar Backfill Issue](https://github.com/ChainSafe/lodestar/issues/7753)
- [Consensus specs](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/p2p-interface.md)
- [Prysm backfill implementation](https://github.com/OffchainLabs/prysm/blob/develop/beacon-chain/sync/backfill/service.go)# Lodestar: Backfill Sync & Historical Data Serving

## Motivation
Lodestar currently does not sync backwards the blocks prior to the checkpoint epoch after a checkpoint sync, so it neither retrieves nor serves historical blocks and blobs.
This leads to:
- Violation of expected peer behaviour, shrinking the pool of peers other nodes in the network can sync from.
- Lodestar’s low usefulness during large network outages, hurting client diversity and network resilience.
- Incompatibility with PeerDAS custody groups that rely on long-term blob availability for data availability sampling.

## Project description
Implement **Backfill Sync & Historical Data Serving**.
1. Walk backwards from the checkpoint sync point, downloading blocks, blobs and column proofs as per the respective fork.
2. Store them in archive data repositories and update `earliestAvailableSlot`.
3. Answer incoming p2p `BeaconBlocksByRange`, `BlobSidecarsByRange` and `DataColumnSidecarsByRange` requests for recent(spec-specified) epochs.

## Specification
1. **Primary BackfillSync Class**
   - `BackfillSync` as a service.
   - Event loop designed to be run in a Worker thread preventing blocking the main thread.

2. **Database**
   - Bucket for Backfill related data:
     ```
        backfill_state = 42, // stores Epoch -> EpochBackfillState data and backfill range singleton object
     ```
   - `BackfillStateRepository` Schema:
     ```ts
         type EpochBackfillState = { hasBlock: boolean; hasBlobs: boolean|null; columnIndices: number[]|null };
         type BackfillRange       = { beginningEpoch: number; endingEpoch: number };
     ```
   - Singleton key `BACKFILL_RANGE_KEY = -1` to track global progress.

3. **Peer management**
   - Listening to `NetworkEvent.peerConnected` and `NetworkEvent.peerDisconnected` events from the Backfill worker thread.
   - Maintain and manage a scored peer set (with storage map, pruning mechanism, size bounding, etc).
   - `BroadcastChannel` to avoid duplication of messages. (needs Investigation)

4. **ReqResp pipeline**
   - Check `BackfillStateRepository` for existing state of next epoch to get backfilled.
   - Create `beacon_blocks_by_range`, `blob_sidecars_by_range` and `data_column_sidecars_by_range` requests.
   - Gate requests by fork.
   - Validate responses and down-score bad peers.

5. **Persistence & Updating States**
   - Only persist data if  full epoch data is available:
     - `BlockArchive.put(block)`
     - `BlobArchive.put(blobs)`
     - `ColumnArchive.put(columns)`
   - Update `BackfillState` repository and `BackfillRange`.
   - Update `earliestAvailableSlot` on both the main and network threads.

## Roadmap
| Phase | Weeks | Deliverables |
|-------|-------|--------------|
| Core logic | 6-11 | Primary Backfill Class, DB Repositories, Peer Management, ReqResp Pipeline, State Management |
| Integration | 12-15 | Fully Integrate into node startup |
| Testing | 16-19 | Unit + integration tests, metrics, benchmarking |
| Optimization & docs | 20-22 | Final optimizations and documentation |

## Possible challenges
- **Cross-Thread State Management**: The most significant challenge is safely and efficiently managing the peer list and sync state across the network, main, and (future) backfill worker threads.
- **Resource Management**: The backfill process must be carefully rate-limited to prevent it from consuming too much CPU, disk I/O, or network bandwidth, which could impact the node's real-time, high-priority duties. It needs careful consideration considering trade-offs between performance and data availability. This point applies specifically to solo node runners/home-stakers.
- **Peer Reliability**: Need robust timeouts and scoring to defend against slow or malicious peers and potential DoS attacks.
- **Fork boundaries**: Backfill must select correct SSZ types across Deneb, Fulu and others.

## Goal of the project
A beacon node that, after a checkpoint sync, automatically backfills to genesis and serves any peer up to the spec-mandated retention depth. Success metrics:
- Passes new backfill test-vectors(unit and integration).
- Syncs mainnet in considerably low time on consumer hardware.
- Reduce down-scoring and disconnections from peers due to absence of sufficient historical data.
- Merge to `chainsafe/lodestar:unstable` before the v2.0 release.

## Collaborators
Fellows: [@vedant-asati](https://github.com/vedant-asati) \
Mentors: [@matthewkeil](https://github.com/matthewkeil) from the lodestar team

## Resources
- [Lodestar Backfill Issue](https://github.com/ChainSafe/lodestar/issues/7753)
- [Consensus specs](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/p2p-interface.md#beaconblocksbyrange-v1)
- [Prysm backfill implementation](https://github.com/OffchainLabs/prysm/blob/develop/beacon-chain/sync/backfill/service.go)
