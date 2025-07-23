# zkVMs for Beacon Chain Snarkification

**TLDR** : link to [slides proposal](https://docs.google.com/presentation/d/1Cq84ENguMO28ucNRmnSHLN7syIZn3Zumtt5FFOrLQ8M/edit?usp=sharing) for motivation on the project.

## Motivation

Beacon Chain runs a complex State Transition Functions(STFs) that validate and update the Beacon Chain state. These functions are computationally intensive, requiring significant amount of resources to verify. We plan to execute Ethereum’s Beacon Chain STFs on zkVMs to generate zero-knowledge proofs of correct consensus state transitions.

Benefits of using proofs instead of computing STF on-chain:
* Saves computation by verifying STFs with zk-proofs instead of re-running them.
* zkVMs offload computational processes to off-chain environments, reducing on-chain burden.
* Makes it easy for light clients to check Beacon Chain state securely.

Specifically, [Grandine team](https://github.com/eth-protocol-fellows/cohort-six/blob/master/projects/project-ideas.md#grandine-zkvms-for-beacon-chain-snarkification) are experimenting with zkVM to enable SNARK-based proving of Beacon's Chain state transition functions.

This project would help advancing on chain snarkification, and eventually "zk-proving" block in real-time, and validators could just validate the block by simply verifying the proof (In zero-knowledge proof, verifying a zk-proof is a lot simpler computationally then generating the proof).

## Project description

Grandine has already integrated SP1 and RISC Zero, the codebase is private as of now. Early benchmarks indicate support for networks with tens of thousands of validators. We will be scaling this work to support larger validator sets and exploring zkVMs.

State Transition Function is very computationally heavy, so when choosing a zkVM we need to be carefull to pick one. At present, proposed zkVMs include:

* [OpenVM](https://github.com/openvm-org/openvm) - "no-CPU" architecture adaption, continuation + recursion support, production-ready. Grandine integration with OpenVM is going to be handled by OpenVM core team, as confirmed by Saulius.

* [Zisk](https://github.com/0xPolygonHermez/zisk) - RISC-V architecture, continuation support, massive parallelism via minimal trace

Performance Considerations before choosing a zkVM:
* Recursive proving / Proof aggregation
* Continuation support
* Precompiles support
* GPU acceleration

We also propose exploring the following zkVMs for integration:
* [Jolt](https://github.com/a16z/jolt) - sum-check protocol optimization, assigned to Aman (?)
* [Pico](https://github.com/brevis-network/pico) - CircleSTARK implemenation, assigned to Jimmy Chu
* \[**TODO**\] [Another VM](tk), assigned to Ritesh Das

## Specification

There are three fellows involved in this proposal and each of us will be working on different zkVM.

Here we have an implementation structure for OpenVM.

OpenVM provide bunch of [examples](https://github.com/openvm-org/openvm/tree/main/examples) of guest programs. Project template will have a host and a guest program. Host is the "outside" environment responsible for setting up the execution environment for the zkVM, supplying inputs to the zkVM. Basically It will load ```BeaconState``` and ```BeaconBlock``` .

We will be implementing STF inside the the Guest program. The main job of the guest is to "perform the computation" that needs to be proven and to output results in a format that the host can verify.

Since the overall program for executing the STF is long, we will use continuations to split it into multiple segments and prove each segment separately. We can get motivation from [OpenVM Reth Benchmark](https://github.com/axiom-crypto/openvm-reth-benchmark) project which uses continuations to prove unbounded program execution by splitting the program into multiple segments and proving segments separately.

The proving workflow in OpenVM involves multiple stages, starting from compiling high-level STF logic to generating a zk proof. The following diagram illustrates the step-by-step process -

![image](https://i.postimg.cc/9QR9kKfv/Screenshot-2025-07-21-at-3-10-58-PM.png)

## Roadmap

Since few fellows are working on this project, we will be divinding the work among overselves. We got a draft plan - 

- [ ] STF implementation in OpenVM and Zisk
- [ ] Determining the optimal zkVM for Beacon Chain operations
- [ ] Optimization strategies tailored to zkVMs

## Possible challenges

* **Bugs or issues in zkVMs** - Since some zkVMs are still new, we might hit unexpected bugs or missing features during development.
* **Large trace size** - The Beacon STF is heavy and may create large traces that don’t fit in one proof, so we might need to split execution into smaller parts.

## Goal of the project

The goal is to implement the Beacon Chain’s STF inside a zkVM. Success means we can generate zero-knowledge proofs for correct state transitions, which can be verified efficiently without re-executing the whole STF. The project will be considered complete when: 
* The STF runs correctly in the zkVM and passes test vectors.
* Proofs are generated for multiple slots or epochs.
* Verification of these proofs is fast and works with real Beacon Chain data.

## Collaborators

### Fellows

* [Aman](https://github.com/0xprivateChaos) 
* [Jimmy Chu](https://github.com/jimmychu0807)
* [Ritesh Das](https://github.com/Dyslex7c)

### Mentors

* [Saulius Grigaitis](https://github.com/sauliusgrigaitis), Grandine core team

## Resources

* [Grandine STF](https://github.com/grandinetech/grandine/blob/develop/transition_functions/src/combined.rs)
* [OpenVM framework](https://github.com/openvm-org/openvm)
  * OpenVM guest program [examples](https://github.com/openvm-org/openvm/tree/main/examples)
  * OpenVM [book](https://book.openvm.dev/)
  * Continuation example ([OpenVM Reth Benchmark](https://github.com/axiom-crypto/openvm-reth-benchmark))
* [Zisk framework](https://github.com/0xPolygonHermez/zisk)
  * Zisk [docs](https://0xpolygonhermez.github.io/zisk/introduction.html)
* [Brevis Pico](https://github.com/brevis-network/pico)
* Ream work on zkVM benchmarking by Jun Song and unnawut
  * [consenzero](https://github.com/ReamLabs/consenzero)
  * [consenzero-bench](https://github.com/ReamLabs/consenzero-bench)
  * [consensp1us](https://github.com/ReamLabs/consensp1us)
