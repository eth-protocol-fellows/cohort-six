# Reth FOCIL Implementation

## Motivation

FOCIL (Fork-choice Enforced Inclusion Lists) [EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) is a proposal that introduces Inclusion List (IL) committees for each block. These committees propose lists of transactions that must be included in the next block. It’s a response to the increasingly centralized nature of block building, currently dominated by a handful of builders.

About 80% of blocks are built by just four builders, and approximately 3% of blocks actively censor transactions. This number has previously peaked at 85% and only dropped recently due to U.S. policy changes. But policies change, and centralization risks persist.

FOCIL helps defend Ethereum's credible neutrality, which is a core value and major advantage over traditional financial systems and other blockchains and provides censorship resistance. Ethereum's trustless nature breaks down if transaction censorship becomes possible, at that point it starts to look more like a traditional payment network, heavily influenced by state policies. And thus just another traditional finance protocol, like Swift.

## Project Overview

The goal of this project is to implement the FOCIL spec in the Reth execution client, enabling it to participate in interop testing with other clients. This involves updating and extending the current spec as outlined in [ethereum/execution-specs#1214](https://github.com/ethereum/execution-specs/pull/1214), and ensuring Reth plays well with other clients.

The implementation builds on [prior work](https://github.com/jacobkaufmann/reth/tree/prague-focil) by Jacob Kaufmann, rebasing and upgrading it to align with the latest specs and dependencies. A key focus is creating a clean, well thought out integration that not only supports interop testing but also helps advance the EIP across the broader Ethereum ecosystem.


## Specification

### FOCIL Scope

#### Consensus Layer (CL)
- Adds **Inclusion List (IL) committees**: 16 validators per slot submit ILs.
- Handles **P2P gossip** of ILs between validators.
- Tracks **equivocations** (conflicting ILs) for slashing.
- Updates **fork choice** to make decisions based on observed IL's.

#### Execution Layer (EL)
- Enforces **ILs** during block building and validation.
- Builds **ILs** from the mempool.
- Adds new **Engine API methods**:
  - `engine_getInclusionListV1`
  - `engine_updatePayloadWithInclusionListV1`
  - `engine_newPayloadV5`

The EL side is relatively straightforward and I’m hoping it serves as a solid first EIP to implement in an EL client. Most of the complexity lives in the CL, where things are a lot less structured, handling p2p gossip, tracking equivocations, and juggling liveness issues, all within a tightly time constricted environment.
### Project Scope

#### Rebase and Update Initial Implementation

Rebase and modernize Jacob Kaufmann’s existing FOCIL branch to match current specifications and dependencies:

- Define and integrate the Inclusion List data type.
- Update payload structures to include ILs in payload attributes.
- Implement new JSON-RPC endpoints:
  - `engine_getInclusionListV1`
  - `engine_updatePayloadWithInclusionListV1`
  - `engine_newPayloadV5`
- Implement a basic Inclusion List (IL) validation strategy.

#### Address TODOs and Optimization Tasks

Finalize work items left in Jacob’s original implementation and complete remaining integration tasks:

- Implement the finalized FOCIL logic in OP Reth.
- Optimize the `new_payload()` logic for efficiency and clarity.
- Refactor code for readability and ensure idiomatic Rust practices.

#### Reth Testing

Add comprehensive test coverage:

- Unit tests for all new logic.
- Simulate payload flow and validate Reth's behavior against the spec.
- Integration tests for JSON-RPC endpoints and payload flows.
- Ensure test coverage for edge cases and spec deviations.

#### Extensive Interop Testing

Engage in cross-client interoperability testing:

- Contribute to and utilize [Jihoon’s local devnet](https://github.com/jihoonsong/local-devnet-focil/tree/main) for simulations.
- Identify any mismatches or gaps in conformance.
- Track performance and correctness using the [FOCIL Metrics Board](https://github.com/ethereum/beacon-metrics/pull/16).
- Document and share findings with the ecosystem.

## Roadmap

### Weeks 5–12 (Coding):
- Rebase Jacob’s changes onto current Reth
- Identify areas for improvement
- Update custom dependencies (`alloy`, `alloy-evm`, etc.)
- Ensure implementation works locally with full client code

### Weeks 13–20 (Testing & Fixing):
- Run interop tests with all CLI FOCIL implementations using Kurtosis
- Debug and refine as needed
- Write full test suite for FOCIL changes

### Weeks 21–25 (Metrics & Wrap-Up):
- Collect client performance data
- Benchmark against other clients and flag inefficiencies
- Write a summary report and prepare for the final presentation

## Possible Challenges

- Limited experience running local devnets for interop testing  
- Lack of a dedicated mentor if deeper technical issues arise  

## Collaborators

- [@pellekrab](https://github.com/pellekrab)  


## Fellows

No other Fellows are working on this client, but several are building FOCIL implementations for other clients.

## Mentors

No official mentor assigned. Reth has a strong open-source community, and I’m open to seeking mentorship as the project develops.

## Resources

- FOCIL HackMD: https://hackmd.io/@jihoonsong/ryTt1Fv7le  
- Working branch: https://github.com/PelleKrab/reth/tree/focil_impl  
- Jacob's original impl: https://github.com/jacobkaufmann/reth/tree/prague-focil  
