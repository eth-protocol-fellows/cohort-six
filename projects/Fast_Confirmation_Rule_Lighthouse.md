# Fast Confirmation Rule

## Motivation and Overview

This project is a research prototype of the Fast Confirmation Rule specification in Lighthouse. The primary motivation is to help EF Protocol Consensus Team get a deeper understanding on how FCR works in one of the Consensus Layer Clients (apart from Prysm and Teku as they are already involved with FCR) and help improve the evolving specification.

### The Specifications

The [Fast Confirmation Rule](https://arxiv.org/pdf/2405.00549) is a novel algorithm designed to significantly reduce transaction confirmation times on Ethereum from the [current 13-19 minutes](https://www.circle.com/blog/exploring-confirmation-rules-for-ethereum) down to just 12-24 seconds (1-2 slots) in optimal conditions. This algorithm provides a complementary confirmation mechanism to Ethereum's existing FFG Finalization Rule, offering faster block confirmation while relying on network synchrony assumptions. Unlike the FFG Finalization Rule which works under asynchronous network conditions, the Fast Confirmation Rule assumes synchronous network conditions where attestations arrive within 8 seconds.

The Fast Confirmation Rule operates under two primary assumptions:

* *Network Synchrony*: LMD-GHOST votes sent by honest validators are delivered by the end of each slot (within 8 seconds)
* *Byzantine Threshold (β)*: A known maximum fraction of Byzantine stake across any set of committees, typically set to 20-30%

The algorithm works by checking whether a block's LMD-GHOST weight exceeds a threshold of `(committee_weight × 0.5) + (committee_weight × β)`. This ensures sufficient honest validator support to guarantee the block remains canonical.

## Roadmap

The primary goal of this project is to have a near-complete implementation of FCR in a Lighthouse fork as a research prototype, with a focus on performance.

#### Phase 1: Foundation and Core Architecture (Weeks 6-8)
**Goal**: Establish FCR module structure and basic integration with Lighthouse's fork choice
- **Key Deliverables**:
  - Create FCR module structure with feature-gated compilation
  - Implement core data structures
  - Add CLI arguments for FCR configuration
  - Establish basic integration hooks in fork choice pipeline
- **Success Criteria**: Lighthouse compiles with FCR enabled/disabled, basic hooks integrate without errors

#### Phase 2: Core FCR Logic Implementation (Weeks 9-11)
**Goal**: Implement the core FCR algorithm with LMD-GHOST and FFG support calculation
- **Key Deliverables**:
  - Implement Q-indicator calculation 
  - Add committee weight calculation with caching
  - Implement FFG support calculation with lazy evaluation
  - Create confirmation advancement logic
- **Success Criteria**: Core FCR logic works correctly, basic confirmation tests pass
- Prioritize correctness over optimization - performance tuning comes later

#### Phase 3: Integration and Engine API (Weeks 12-13)
**Goal**: Integrate FCR with execution layer and optimize performance
- **Key Deliverables**:
  - Integrate with Engine API for safe head selection
  - Optimize performance using Lighthouse's caching architecture
  - Implement fallback mechanisms when FCR unavailable
- **Success Criteria**: Execution layer receives FCR-confirmed safe heads, performance meets targets

#### Phase 4: Testing and Validation (Weeks 14-15)
**Goal**: Comprehensive testing and performance validation
- **Key Deliverables**:
  - Unit and integration tests for major FCR components
  - Edge case handling and Byzantine threshold sensitivity tests
  - Performance benchmarking and optimization

Points to note :

* *Iterative Development*: Each phase builds upon the previous one, with the flexibility to revisit earlier phases based on implementation discoveries.

* *Priority-Based*: Core functionality (Phases 1-2) takes precedence over optimization and testing. If time constraints arise, focus on getting the core algorithm working correctly.

* *Adaptive Timeline*: While the roadmap provides a general timeline, implementation complexity may require adjusting phase durations. The key is maintaining momentum on core deliverables.

* *Continuous Integration*: Regular integration with Lighthouse's main branch to ensure compatibility and catch issues early.

Ideal success metrics :

**Performance Targets**:
- Confirmation time: 12-24 seconds (1-2 slots) under optimal conditions
- Fork choice overhead: <50μs additional latency per `get_head()` call
- Memory overhead: <15% increase in fork choice memory usage
- Cache hit rate: >90% for committee weight and FFG support calculations

**Functional Requirements**:
- Correct Q-indicator calculation matching FCR specification
- Robust FFG integration preventing confirmation of unsafe blocks
- Proper epoch boundary handling and cross-epoch confirmation
- Safe fallback to finalized checkpoint when synchrony assumptions fail
- Feature-gated compilation with zero impact when disabled

**Research Outcomes**:
- Detailed performance analysis comparing tree-states vs. traditional caching
- Byzantine threshold sensitivity analysis for single-slot vs. multi-slot confirmation
- Network synchrony impact on confirmation reliability
- Memory usage patterns and optimization opportunities
- Comparison with Prysm's implementation approach and lessons learned

As this is a research prototype, the roadmap is subject to changes.

## Possible Challenges

The Fast Confirmation Rule demands sub-slot confirmation timing, making committee weight calculation performance absolutely critical. The FCR algorithm must rapidly compute the Q-indicator: `S/W > 0.5 + β`. Any delays in committee weight computation directly impact the algorithm's ability to provide fast confirmation within the required window.

The Ethereum validator set has grown dramatically, with [beacon states expanding from 5MB at genesis to 150MB today, and up to 500MB+ with caches on mainnet](https://blog.sigmaprime.io/tree-states-part1.html). Committee weight calculations must scale efficiently with this growth, particularly as:
* More validators participate in attestation committees
* Committee assignments become more complex across epoch boundaries
* Effective balance calculations involve larger datasets

Benchmarking should specifically measure:
* State Access Patterns: How tree-states traversal affects committee weight queries compared to linear memory access
* Cache Efficiency: Performance impact of committee weight caching strategies under the persistent data structure model
* Memory vs. CPU Trade-offs: Whether tree-states' memory savings offset computational overhead for frequent committee queries

**Key areas to benchmark:**
1.  **Committee Weight Calculation**: Measure the latency of `get_committee_weight_between_slots` under Lighthouse's tree-states architecture, which has O(log n) access patterns.
2.  **FFG Lazy Support Calculation**: Benchmark `get_ffg_support_lazy` to ensure it remains sub-millisecond even with a large number of head votes.
3.  **Overall `get_head` Latency**: Measure the total additional time spent in the `fork_choice` write lock due to the FCR hooks.

# Goal of the project

The overarching goal is to measure the performance of the FCR. More, specifically, measuring things like :
- Out of all blocks reorged in a give time span, how many of those were confirmed by the FCR
- How long does it take to confirm a block. Ideally, this should be measured for each slot number with in an epoch as slots at the beginning of an epoch would behave differently than those at the end.

To achieve the above we need a correct implementation of the FCR. Code performance matters to the extent that it is not to slow to execute as this would impact the measurements above.

We want a correct (and simple) implementation that's fast enough. 

As a stretch goal, the research prototype can be cleaned up and refactored to get it merged in the Lighthouse repository.

# Collaborators 

Mentors - *Roberto Saltini (EF Protocol Consensus Lead)*, Mikhail Kalinin (from Consensys) and *Michael Sproul (from Lighthouse)*
Permissioned Protocol Fellow - [Harsh Pratap Singh](https://harsh-ps-2003.bearblog.dev/)

Apart from me, no other permissioned/permissionless fellow is working on this project.

## Resources 

* [A more detailed proposal](https://hackmd.io/@harsh-ps-2003/SJSOZISVge)
* [FCR Paper](https://arxiv.org/pdf/2405.00549)
* [Circle Blog](https://www.circle.com/blog/exploring-confirmation-rules-for-ethereum)
* [Committee Weight Estimation Notebook](https://gist.github.com/saltiniroberto/9ee53d29c33878d79417abb2b4468c20)
* [Original FCR Consensus Specification](https://github.com/ethereum/consensus-specs/pull/3339)
* [New FCR Specification](https://github.com/mkalinin/confirmation-rule)
* [Prysm has already implemented an experimental version of the old Fast Confirmation Rule specification](https://github.com/OffchainLabs/prysm/pull/15164/files)
* [Prysm Challenges](https://github.com/OffchainLabs/prysm/pull/13604)
* [Prysm's Discussion](https://ethresear.ch/t/fast-confirmation-rule-on-safe-head-in-prysm/22167)
