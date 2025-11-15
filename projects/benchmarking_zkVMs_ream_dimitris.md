# Benchmarking zkVMs on the Ream consensus client

Benchmarking zero-knowledge proof generation and validation of state transition functions (STFs) for beacon chain and lean consensus.

## Motivation

Ream is a lean consensus client. During the first months of its development, a beacon chain consensus implementation was also developed mainly for testing and learning. This project follows the same path. First, we worked on proofs of beacon chain STFs, and then on STFs for lean consensus. Lean consensus introduces SNARK proofs for each operation on the chain. This reduces computational costs because it's only the block proposer that carries heavy computation to propose a new block. The rest of the validators only verify proofs. This makes it feasible to run a validator node on devices with limited resources, such as smartphones and smartwatches.

This project provides two main benefits to the Ream team:
- It is the first systematic study of zkVM performance in the early stages of lean chain development.
- It helps in designing the Ream client to be zkVM friendly.

## Existing work

The project builds on top of the work done by the Ream team on proving beacon chain STFs with [SP1](https://github.com/ReamLabs/consenSP1us) and [RiscZero](https://github.com/ReamLabs/consenzero-bench) zkVMs:

- Only one state transition function can be benchmarked on each run (not the whole block body).
- The benchmarks include only cycle counts and execution time but not proof generation or verification time, nor profiling results.
- The benchmarks only pick a few data points to compare; however, the results from SP1 and RISC Zero have not been directly compared.

In particular,

### SP1

The project structure is the following:
- `consensp1us/program/operations/src`: `main.rs` includes main logic of the program, reads the state transition function to be applied on the beacon block and commits `pre-state.<process-operation>` to the public output. Here, `process-operation` will be replaced by attestation, attester_slashing, block_header, etc. The zk proof will include this commit.
- `consensp1us/lib/src`: includes modules for reading files, providing beacon block fields as input, compression and serialization using snappy and SSZ respectively.
- `consensp1us/script`: contains scripts that run the program with or without proof and generates benchmarks.

### RiscZero

The project structure is:
- `consenzero-bench/host/src/bin`: includes code for setting up execution environment and the prover, doing the proving and proof verification, creating logs and benchmarks (cycle counts).
- `consenzero-bench/methods/guest/src`: contains the beacon state transition functions that will be proven by the zkVM.
- `consenzero-bench/lib/src`: the same library for modules as in SP1.

The structure is similar in both cases: `lib` provides functionality, there is a script which automates building, running, proving and validating, there is a module that sets up the zkVM environment for proving and finally there is the guest code i.e. the STFs that will run in the zkVM.

## Work during the fellowship

### zkVM on the beacon chain
In the first part of the project, we expanded it to introduce other zkVMs. I worked on OpenVM and Utsav worked on Jolt, Pico and ZisK as well as a unifying framework. Although there are differences among zkVMs, the main principle remains largely the same. We reused the `lib` modules, guest code and some scripts, but we built from scratch the host code. We also implemented elapsed time measurement and execution cycles. Utsav's framework is sufficient for our current purposes. We were advised to look at [ere interface](https://github.com/eth-act/ere), but it was not a priority back in August. It will be interesting to look into it in the future as an abstraction for the second part of the project.

### zkVM on lean consensus
Consensus is achieved using 3-slot-finality, specifically a simplified version called 3SF-mini. Proving is optional and is initialized with a CLI flag. We created a new service in Ream called `LeanProverService` that is called by the blockproposer concurrently with `ValidatorService`. We again prove the execution of STFs. For now, we generate a proof every 200 slots because proof generation extends beyond the slot that initialized it. It takes around 6 minutes on a 16GB MacBook to generate the first proof at slot 2, while the slot lasts 4 seconds. As time passes, the state gets larger, leading to 12, 14, ... minutes of proving, which results in concurrent provings taking place that leads to out-of-memory crash. With this setup, we managed to reach up to slot 1000.

# Specification
<!-- Use the correct tense and update what happened in the first part of the internship. ADD Utsav's contributions! -->
Implementation of the first part of the project was straightforward. The guest code was found in `consenzero-bench/methods/guest/src/main.rs` of RiscZero or `consensp1us/program/operations/src/main.rs` of SP1. The same was done with modules in `lib`. The host program was built according to the SDK instruction provided by each zkVM. In the case of OpenVM for example, the instructions are found in the SDK section of the [OpenVM book](https://book.openvm.dev/advanced-usage/sdk.html#using-stdin) specifying:

- the preambles that should be added to the guest program
- the code to be added in host to build, transpile and prove the guest

<!-- TODO-Utsav: Add your part about Jolt, Pico, Zisk  -->

We have not implemented GPU optimization or any other optimization for proof generation. Generating a proof for a single test of a single STF operation currently crashes our laptops with 16GB RAM due to memory exhaustion. On a desktop with a Ryzen 3400G and 32GB of RAM, it took about 10 hours to generate a proof with OpenVM for the test `attestation_invalid_correct_attestation_included_after_max_inclusion_slot`. This creates a major issue in running multiple tests with extensive coverage since we would need more than 60 hours of proving for a single test on 6 zkVMs. Additionally, different zkVMs are optimized for different scenarios: GPU versus CPU proving, high versus low number of execution cycles, and so on.
Our project is able to provide the following metrics: execution time, execution cycles (if implemented by the respective zkVM), and proof generation time for one or very few test cases.

The second part of the project is more complicated as it involves lean consensus. Lean consensus is under development with many questions still open. At the same time, as we have already experienced in the first part of the project, zkVMs are still slow for our use cases. These two factors impose some limitations on our project. We generate a proof every 200 slots and we did not manage to test validation because setting up a network with proving initialized created issues both inside Docker and locally. Currently we use RISC Zero, but we plan to abstract it soon so that all 6 zkVMs become available. Our design introduces proving as an additional feature to the Ream client, leaving the core logic untouched. Further details are found in the associated [draft PR](https://github.com/ReamLabs/ream/pull/835). We had to modify networking by adding a `BlockProof` message, a proof gossip topic, and proof verification logic. We also created the usual guest-host pair, added proof triggers, and two structs in ream/crates/common/consensus/lean/src/block.rs:
```rust
pub struct BlockProof {
    pub block: Block,
    pub proof: Proof,
}

pub struct Proof {
    /// RiscZero receipt containing the zkVM proof
    pub receipt: Receipt,
    pub method_id: [u32; 8],
}
```
<!-- I have to include details i.e. snippets about our new enum ? LeanProverService -->
## Roadmap
<!-- Add what our draft PR has along with Unnawut's suggestions -->
1. Ran the guest code from RISC Zero and SP1 repos inside OpenVM, Jolt, ZisK, and Pico.
2. Included proving and validation benchmarks in the above-mentioned zkVMs by mid-September.
3. Created a modular framework that incorporates all 6 zkVMs (OpenVM is temporarily excluded, we have to fix some dependency issues).
4. Learned about 3-slot finality.
5. Implemented proving and validation in lean consensus.
6. Successfully tested proof generation with validation test pending.

This concludes the fellowship.

## Possible challenges

- Every zkVM implements optimisations differently, including GPU optimisation. Taking into account fast and breaking changes even on minor updates it is hard for our current project to stay updated and ever harder to optimize. We should take a closer look at at [ere interface](https://github.com/eth-act/ere) and test if it handles automatically updates.
- Ream's codebase is also evolving which could create some issues. We don't think that this will be as serious though since our implementation operates as a separate service that only handles proving.
- Obtaining consistent metrics across zkVMs. Not all of them provide cycle counts for the execution of functions in the guest. Moreover, proof generation takes many hours which restricts us to testing very few cases.

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
- Dimitris' OpenVM implementation for the beacon chain: <https://github.com/DimitriosMitsios/openvm-ream>
- Utsav's framework that incorporates Jolt, Pico, Zisk, RiscZero, SP1: <https://github.com/x-senpai-x/Ream-ZKVM-Benchmarks>
- Our integration of RiscZero in lean chain: <https://github.com/DimitriosMitsios/ream/tree/leanprover>