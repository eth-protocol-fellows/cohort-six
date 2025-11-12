# Benchmarking zkVMs on the Ream consensus client

Benchmark zero-knowledge proof generation and validation of state transition functions (STFs) for beacon chain and lean consensus.

## Motivation

Ream is a lean consensus client. During the first months of its development, a beacon chain consensus implementation was also developed mainly for testing and learning. This project follows the same path. First, we worked on proofs of beacon chain STFs, and then on STFs for lean consensus. Lean consensus introduces SNARK proofs for each operation on the chain. This reduces computational costs because validators only verify proofs instead of recomputing new blocks. This makes it feasible to run a validator node on devices with limited resources, such as smartphones and smartwatches.

This project provides two main benefits to the Ream team:
- It is the first systematic study of the zkVM market that will be useful when Beam Chain specs become available and the team starts building.
- It helps identify design decisions that are incompatible with zkVMs and possible bottlenecks.

## Existing work 

The project builds on top of the work done by the Ream team. Currently, the implementation includes [sp1](https://github.com/ReamLabs/consensp1us) and [risc0](https://github.com/ReamLabs/consenzero-bench) zkVMs where:

- Only one state transition function can be benchmarked on each run (not the whole block body).
- Benchmarks include only cycle counts and execution time but not proof generation or verification time, nor profiling results.
- The benchmarks only pick a few data points to compare; however, the results of SP1 and RISC Zero have not been directly compared.

In particular,

### sp1

The project structure is the following:
- `consensp1us/program/operations/src`: `main.rs` includes main logic of the program, reads the state transition function to be applied on the beacon block and commits `pre-state.\<process-operation\>` to the public output. Here, `process-operation` will be replaced by attestation, attester_slashing, block_header, etc. The zk proof will include this commit.
- `consensp1us/lib/src`: includes modules for reading files, providing beacon block fields as input, compression and serialization using snappy and SSZ respectively.
- `consensp1us/script`: contains scripts that run the program with or without proof and generates benchmarks.

### risc0

The project structure is:
- `consenzero-bench/host/src/bin`: includes code for setting up execution environment and the prover, doing the proving and proof verification, creating logs and benchmarks (cycle counts).
- `consenzero-bench/methods/guest/src`: contains the beacon state transition functions that will be proven by the zkVM.
- `consenzero-bench/lib/src`: the same library for modules as in sp1.

The structure is similar in both cases: `lib` provides functionality, there is a script which automates building, running, proving and validating, there is a module that sets up the zkVM environment for proving and finally there is the guest code i.e. the state transition functions that will run in the zkVM.

## Work during the fellowship

### zkVM on the beacon chain
The first part of the project expanded to introduce other zkVMs. I worked on OpenVM and Utsav on Jolt, Pico and ZisK as well as a unifying framework. Although there are differences among zkVMs, the main principle remains largely the same. We reused the `lib` modules, guest code and some scripts, but we built from scratch the host code. We also implemented elapsed time measurement and execution cycles. Utsav's framework is sufficient for our current purposes. We were advised to look at [ere interface](https://github.com/eth-act/ere), but it was not a priority back in August. It will be interesting to look into it in the future as an abstraction for the second part of the project.

### zkVM on lean consensus
Consensus is achieved using 3-slot-finality and in particular a simplified version 3SF-mini. Proving is optional and is initialized with a CLI flag. We created a new service in Ream called `LeanProverService` that is called by the blockproposer concurrently with `ValidatorService`. We again prove the execution of STFs. For now, we generate a proof every 200 slots because proof generation extends beyond the slot that initialized it. It takes around 6 minutes on a 16GB MacBook to generate the first proof at slot 2, while the slot lasts 4 seconds. As time passes, the state gets larger, leading to 12, 14, ... minutes of proving, which results in concurrent provings taking place and leading to memory overflow. With this setup, we managed to reach up to slot 1000.

# Specification
<!-- Use the correct tense and update what happened in the first part of the internship. ADD Utsav's contributions! -->
Implementation of the first part of the project was straightforward. The guest code was found in `consenzero-bench/methods/guest/src/main.rs` of risc0 or `consensp1us/program/operations/src/main.rs` of sp1. The same was done with modules in `lib`. The host program was built according to the SDK instruction provided by each zkVM. In the case of OpenVM for example, the instructions are found in the SDK section of the [OpenVM book](https://book.openvm.dev/advanced-usage/sdk.html#using-stdin) specifying:

- the preambles that should be added to the guest program
- the code to be added in host to build, transpile and prove the guest

<!-- TODO-Utsav: Add your part about Jolt, Pico, Zisk  -->

We have not implemented GPU optimization or any other optimization for proof generation. Generating a proof for a single test of a single STF operation currently crashes our laptops with 16GB RAM. On an old PC with a Ryzen 3400G and 32GB of RAM, it took about 10 hours to generate a proof with OpenVM for the test `attestation_invalid_correct_attestation_included_after_max_inclusion_slot`. This creates a major issue in creating tests with extensive coverage that provide meaningful insights about various zkVMs. Additionally, different zkVMs are optimized for different scenarios: GPU versus CPU proving, high versus low number of execution cycles, and so on. 
Our project is able to provide the following metrics: execution time, execution cycles (if implemented), and proof generation time for one or very few test cases.

The second part of the project is more complicated as it involves lean consensus. Lean consensus is under development with many questions still open. At the same time, as we have already experienced in the first part of the project, zkVMs are still slow for our use cases. These two factors impose some limitations on our project. We generate a proof every 200 slots and we did not manage to test validation because setting up a network with proving initialized created issues both inside Docker and locally. Currently we use RISC Zero, but we plan to abstract it soon so that all 6 zkVMs become available. Our design introduces proving as an additional feature to the Ream client, leaving the core logic untouched. Further details are found in the associated [draft PR](https://github.com/ReamLabs/ream/pull/835). We had to modify networking by adding a `BlockProof` message, a proof gossip topic, and proof verification logic. We also created the usual guest-host pair, added proof triggers, and two structs: 
```rust
pub struct BlockProof {
    pub block: Block,
    pub proof: Proof,
}

pub struct Proof {
    /// risc0 receipt containing the zkVM proof
    pub receipt: Receipt,
    pub method_id: [u32; 8],
}
```
<!-- I have to include details i.e. snippets about our new enum ? LeanProverService -->
## Roadmap
<!-- Add what our draft PR has along with Unnawut's suggestions -->
1. Ran the guest code from RISC Zero and SP1 repos inside OpenVM, Jolt, ZisK, and Pico.
2. Included proving and validation benchmarks in the above-mentioned zkVMs by mid-September.
3. Created a modular framework that incorporates all 6 zkVMs (OpenVM is temporarily excluded).
4. Learned about 3-slot finality.
5. Implemented proving and validation in lean consensus.
6. Successfully tested proof generation with validation test pending.

This concludes the fellowship.

## Possible challenges

- Each zkVM presents a different design that affects our implementation. It is possible that we face substantial overhead when optimizing our state transition function to speed up proving and validation because there are multiple zkVMs.
- Unexpected bugs or obstacles on the zkVM side. So far, I encountered an issue with the OpenVM Quickstart guide, and Unnawut had to put effort into making RISC Zero work with $2^{40}$ lists. See [Resources](#resources). The issue with long lists will probably arise in OpenVM as well because the maximum memory address for an OpenVM program is $2^{29}$ bytes, as shown in the warning [here](https://book.openvm.dev/writing-apps/write-program.html).
- zkVMs are evolving rapidly, so we must account for the current pace of development and possibly change our project roadmap accordingly.
- Measuring proof generation time meaningfully across zkVMs is difficult because even for a single STF, proving time is very high. Additionally, a problem encountered with Jolt was that it was unable to generate a proof and the program was killed instead.
- Since several things might be entirely replaced in Beam Chain (such as BLS signatures), it might not be very fruitful to optimize those STFs involving heavy changes.

## Goal of the project

The main goal of the project is to enable the Ream team to make informed decisions about the design of the client and the use of zkVMs. An advancement to the current status would be getting cycle counts and execution time for SP1, RISC Zero, OpenVM, Jolt, and ZisK. A major advancement would be the inclusion of time measurements for proof and validation, providing concrete data for zkVM selection in Ethereum's consensus upgrade.
We consider that we reached both goals during the fellowship.

## Collaborators

### Fellows

Dimitrios Mitsios

Utsav Sharma

### Mentors

Unnawut

## Resources

- Current implementations for SP1 and RISC Zero: <https://github.com/ReamLabs/consensp1us> and <https://github.com/ReamLabs/consenzero-bench>
- Utsav's Jolt code: <https://github.com/x-senpai-x/consenJolt>
- OpenVM quickstart issue and solution: <https://github.com/openvm-org/openvm/issues/1816>