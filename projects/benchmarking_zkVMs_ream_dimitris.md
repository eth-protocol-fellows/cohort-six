# Benchmarking zkVMs on the Ream consensus client

Benchmark proof generation and validation of zero-knowledge virtual machines (zkVMs) on state transition functions

## Motivation

Ream client is a consensus client that will be put in production once the Beam Chain replaces the Beacon Chain. The project is related to the consensus layer and the Beam Chain. In particular, the Beam Chain introduces SNARK proofs on every operation on the state transition functions. The computational costs reduce since the rest of the validator nodes only need to verify the proofs instead of recomputing the new block. This way, it will be feasible to validate and stake from devices with limited resources e.g. smart watches.

This project provides two main benefits to the Ream team.
- It will be the first systematic study of the zkVM market that will be useful when Beam Chain specs become available and the team starts building.
- It helps identifying design decisions that are incompatible with zkVMs and possible bottlenecks.

## Project description

The project will build on top of the work done by the Ream team. Currently, the implementation includes [sp1](https://github.com/ReamLabs/consensp1us) and [risc0](https://github.com/ReamLabs/consenzero-bench) zkVMs where:
- Only one state transition function can be benchmarked on each run (not the whole block body).
- Benchmarks include only cycle counts and execution time but not proof generation, verification time and profiling results.
- The benchmarks only pick a few data points to compare however the results of sp1 and risc0 have not been directly compared.

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

### Work during the fellowship

The first part of the project involves expanding the tests to other zkVMs. I am working on OpenVM and Utsav has results on Jolt and possibly ZisK. There are differences among zkVMs but the main principle remains largely the same. We will reuse the `lib` modules, guest code and some scripts but we will build from scratch the code of the host. Afterwards, there are many directions that we could take:
1. Expand current benchmarks to more zkVMs.
2. Include time of proof generation and time of validation in the set of benchmarks.
3. Implement a generalised framework such as the [ere interface](https://github.com/eth-act/ere) that allows testing of various zkVMs.

This is a collective decision to make but I consider that creating useful benchmarks is the highest priority. Maybe, the work necessary for a framework will extend beyond the current fellowship but option 2. above seems feasible.

# Specification

Implementation of the first part of the project is straightforward. The guest code can be found in `consenzero-bench/methods/guest/src/main.rs` of risc0 or `consensp1us/program/operations/src/main.rs` of sp1. The same will be done with modules in `lib`. The host program will be built according to the instructions found in the SDK section of the [OpenVM book](https://book.openvm.dev/advanced-usage/sdk.html#using-stdin). There are detailed instructions on
- the preambles that should be added to the guest program
- the code to be added in host to build, transpile and prove the guest

Once this is done for a single zkVM it would be easy to reproduce it on other zkVMs as well.
The state transition functions benchmarked by ReamLabs on SP1 and RISC Zero have been executed by Utsav on Jolt and he is waiting for a new feature Jolt is working on which will add functionality to track Guest's RISC-V cycle runs so that a detailed benchmarking report can be created and compared directly with SP1 and RISC Zero results.

The second part is more complicated. The main goal is to extend benchmarks to measure proving time and validation time of state transition functions. Currently, proving the execution of a single state transition function takes too much time. The identification of bottlenecks will provide useful information about zkVMs which will be necessary for Beam Chain implementation in the future (beginning of 2026).

The most direct method to proceed after completing the first part is to extend our implementation to include the new benchmarks in OpenVM, RISC Zero, SP1 and Jolt. There may be a more modular way to proceed by using the [ere interface](https://github.com/eth-act/ere). For the moment, the ere framework supports the 4 zkVMs but the current example for the guest program is a fibonacci sequence found in `ere/tests`. We need to include our guest program and then try to implement benchmarks. I cannot estimate the amount of work needed for this.

However looking at ere it seems we still need to add our own guest and host code that doesn't make ere much different than separately benchmarking in different repositories. Instead we could take a different approach and create a common parser to plot benchmarks from multiple zkVms on the same chart.

When we have sufficient benchmarking and profiling results, we would be able to comment on the current state of transition functions, identify bottlenecks and find ways to optimize them (snarkifying them in order to run inside zkVM environment efficiently) and implement recursive proving across different zkVM systems. Once this is done we should be able to combine all the state transition functions and instead run the whole block body inside the zkVM.

## Roadmap

1. Run the guest code as found in RISC Zero and SP1 repo inside OpenVM, Jolt, ZisK and Pico and any other zkVM if time allows.
2. Include proving and validation benchmarks in the above mentioned zkVMs by mid September.
3. Compare the results obtained and find key bottlenecks and ways to optimize the guest code along with the state transition functions themselves by end of September.
4. Implement new benchmarks of the remaining State Transition Functions preferably using a modular framework such as ere by end of October.
5. Each block contains multiple operations, and each type of operation is handled by a different STF therefore at the end each zkVM should be able to execute the entire block transition inside them.

This will conclude the fellowship.

## Possible challenges

- Each zkVM presents different design that affects our implementation. It is possible that we face substantial overhead in optimizing our state transition function in order to speed-up proving and validation because there are multiple zkVMs.
- Unexpected bugs or obstacles on the zkVM side. So far, I encountered an issue with the Quickstart guide of OpenVM and Unnawut had to put effort to make RISC Zero work with $2^{40}$ lists. See [Resources](#resources). The issue with long lists will probably arise in OpenVM as well because the maximum memory address for an OpenVM program is $2^{29}$ bytes, see the warning [here](https://book.openvm.dev/writing-apps/write-program.html).
- ZKVMs are evolving rapidly therefore we'd have to account in the current pace of development and maybe change our project roadmap accordingly.
- Measuring proof generation time meaningfully across zkVMs is hard because even for a single STF proving time is very high and a problem faced with Jolt was that it was unable to generate a proof and rather the program was killed.
- Since several things might be entirely replaced inside Beam Chain (such as BLS signatures) therefore it might not be very fruitful to optimise those STFs involving heavy changes.

## Goal of the project

The main goal of the project is to enable Ream team to make informed decisions about the design of the client and the use of zkVMs. An advancement to the current status would be getting cycle counts and execution time for SP1, RISC Zero, OpenVM, Jolt, ZisK. A major advancement would be the inclusion of time measurements for proof and validation providing concrete data for zkVM selection in Ethereum's consensus upgrade.
I consider any of the two outcomes to be a successful fellowship.

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
- RISC Zero working with $2^{40}$ lists: <https://hackmd.io/@reamlabs/Bk9PF7WJge>