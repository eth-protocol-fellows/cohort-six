# Lighthouse: Better state cache heuristics

Implementing adaptive memory management to prevent OOM failures during Ethereum network stress events

## Motivation

The February 2025 Holesky incident exposed a critical vulnerability in how consensus clients manage memory during long non finality periods or when state sizes vary. Lighthouse nodes experienced a huge memory growth from 20GB to over 100s of GB, causing huge validator failures due to OOM.

Currently, Lighthouse uses a fixed size LRU cache for beacon state that performs well during normal operations but becomes inefficient during network stress. The cache is bounded by count(default 32 states), which translates to ~500MB during normal conditions but balloons to multiple GBs as Beacon State size increases. This threatens validator liveness and network stability, particularly affecting solo stakers with limited hardware resources.


## Project description

This project implements a memory aware caching system that can dynamically adjust based on the memory consumption of the Beacon States rather than the item count. This builds upon Lighthouse's existing cache infrastructure while adding memory tracking and dynamic sizing capabilities.

During normal finality, the cache will maximize capacity within memory bounds. During non-finality periods when states grow larger, it will automatically reduce the number of cached states to prevent memory exhaustion while maintaining the most critical states according to existing logic.


## Specification

1. Milhouse integration for `MemorySize`: The Milhouse PR [#51](https://github.com/sigp/milhouse/pull/51) introduces a `MemorySize` trait, which provides a standardized interface for any structure, specifically the BeaconState to report it's memory consumption. We will work on top of it once we make sure this PR is closed and implemented.
2. Implement the MemoryTracker for Lighthouse in an existing file or create a new `memsize.rs` file to track the memory usage of the `BeaconState`.
3. Extend the `StateCache` in `state_cache.rs` to include a `max_cached_bytes`
4. Add new helper functions for insertion, trim and removal of the state into LRU and use Milhouse's `MemoryTracker` to track the latest memory size post these operations.

## Roadmap

### Phase 1: Foundation and Research
- Review PR [#6532](https://github.com/sigp/lighthouse/pull/6532) implementation
- Make sure the work on Milhouse [#51](https://github.com/sigp/milhouse/pull/51) is finalized and merged
- Prototype importing `MemoryTracker` and `memsize.rs` to ensure all BeaconState fields are covered
- Decide on `max_bytes` default and the overall implementation strategy

### Phase 2: Memory Tracking Infrastructure
- Extend `StateCache`  with `cached_bytes`(if required) and `max_cached_bytes` field(s)
- Implement helper traits for `BeaconState` and functions in `state_cache.rs` using MemoryTracker calls
- Add/Modify CLI flags and configs
- For tracking the cache memory, add logs and metrics

### Phase 3: Dynamic Capacity Adjustment
- Add more functions and logics to track/modify the caching memory of the `BeaconState`
- Add tests on top of the existing ones and modify them if required

### Phase 4: Testing and Benchmarking
- Simulate non finality and test out the implementation
- Test out the dynamic in memory caching in testnet if possible and time permits

## Possible challenges

Beacon states contain complex structures that make precise memory measurement challenging. We'll use conservative estimates and periodic calibration to ensure we don't exceed targets. Memory tracking could impact cache operation performance. Lastly, reproducing the exact problems of non finality is difficult, but setting synthetic benchmarks can help in testing out the changes.


## Goal of the project

The goal of this project is to make sure Lighthouse nodes can operate through extended non-finality periods without manual intervention or OOM failures.


## Collaborators

### Fellows 

[Poulav Bhowmick (Odinson)](https://github.com/PoulavBhowmick03)

### Mentors

[Dapplion](https://github.com/dapplion)

[Michael Sproul](https://github.com/michaelsproul)

## Resources

- [Lighthouse Repository](https://github.com/sigp/lighthouse)
- Related Issues: [#7449](https://github.com/sigp/lighthouse/issues/7449), [#7450](https://github.com/sigp/lighthouse/issues/7450)
- [Holesky Incident Analysis](https://blog.sigmaprime.io/pectra-holesky-incident.html)
- [Lighthouse's LRU cache implementation](https://github.com/sigp/lighthouse/tree/unstable/common/lru_cache)
- [In-memory caching](https://www.gridgain.com/resources/glossary/in-memory-computing-platform/in-memory-cache#:~:text=An%20in%2Dmemory%20cache%20removes,and%20improves%20online%20application%20performance)
- [Holesky Incident Postmortem and Client team's fixes](https://github.com/ethereum/pm/blob/master/Pectra/holesky-postmortem.md)
