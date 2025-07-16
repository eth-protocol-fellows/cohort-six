# Benchmarking zkVMs on the Ream consensus client

Benchmark proof generation and validation of zero-knowledge virtual machines (zkVMs) on state transition functions

## Motivation

What problem is your project is solving? Why is it important and what area of the protocol will be affected?

Ream client is a consensus client that will be put in production once the beam chain replaces beacon chain. The project is related to consensus layer and the beam chain. In particular, the beam chain introduces SNARK proofs on every operation on the state transition functions. The computational costs reduce since the rest of the validator nodes only need to verify the proofs instead of recomputing the new block. This way, it will be feasible to validate and stake from devices with limited resources e.g. smart watches.

This project provides two main benefits to the Ream team.
- It will be the first systematic study of the zkVM market that will be useful when Beam chain specs become available and the team starts building.
- It helps identifying design decisions that are incompatible with zkVMs and possible bottlenecks.
## Project description

What is your proposed solution?

The project will build on top of the work done by the Ream team. Currently, the implementation includes risc0 and sp1 zkVMs. At the moment,
- Only one state transition function can be benchmarked on each run (not the whole block body).
- Benchmarks include only cycle counts and execution time.

The first part of the project involves expanding the tests to other zkVMs. I am working on openVM and Utsav has results on Jolt and possibly Zisk. Afterwards, there are many directions that we could take:
1. Expand current benchmarks to more zkVMs
1. Include time of proof generation and time of validation in the set of benchmarks
1. Implement a generalised framework such as the [ere interface](https://github.com/eth-act/ere) that allows testing of various zkVMs.

This is a collective decision to make but I consider that creating meaningful benchmarks is the highest priority. Maybe, the work necessary for a framework extends beyond the current fellowship but option 2. above seems feasible.

# Specification

How will you implement your solutions? Give details and more technical information on the project.

Implementation of the first part of the project is straightforward. The state transition functions to be benchmarked should be the same as in the examples of [sp1](https://github.com/ReamLabs/consensp1us) and [risc0](https://github.com/ReamLabs/consenzero-bench) so that comparison makes sense. On the openVM side there are worked out examples on how to prove a guest program in their [github repo](https://github.com/openvm-org/openvm/tree/main/examples). This will conclude the first part.

The second part is more complicated. We should start by measuring proving time and validation time. So far, proving takes too much time on a typical laptop. In some cases this could be done on the network of some zkVM companies. The endgoal, though, would be to run on home devices. Thus, it requires optimizations on the computationally intensive functions. If time allows, we will consider creating a modular testing framework to automate some of these processes.


## Roadmap

Finish openVM implementation by mid August and spend the rest of the time on improving benchmarks.
## Possible challenges

What are the limitations and issues you may need to overcome?

- Each zkVM presents different design that affect our implementation. It is possible that we face substantial overhead in optimising our state transition function in order to speed-up proving and validation because there are 4 different zkVMs.
- Unexpected bugs or obstacles on the zkVM side. So far, I encountered an issue with the Quickstart guide of openVM and Unnawut had to put effort to make risc0 work $2^{40}$ lists. See [Resources](#resources)
## Goal of the project

What does success look like? Describe the end goal of the project, scope, state and impact for the project to be considered finished and successful.

The main goal of the project is to enable Ream team to make informed decisions about the design of the client and the use of zkVMs. An advancement to the current status would be getting cycle counts and execution time for sp1, risc0, openVM, Jolt, Zisk. A major advancement would be the inclusion of time measurements for proof and validation.

I consider any of the two outcomes to be a successful fellowship.
## Collaborators

### Fellows

Utsav Sharma
### Mentors

Unnawut
## Resources

Provide links to repositories, PRs and other resources which constitute the project.

- Current implementations for sp1 and risc0: <https://github.com/ReamLabs/consensp1us> and <https://github.com/reamlabs/consenzero-bench>
- Utsav's Jolt code: <https://github.com/x-senpai-x/consenJolt>
- openVM quickstart issue and solution: <https://github.com/openvm-org/openvm/issues/1816>
- risc0 working with $2^{40}$ lists: <https://hackmd.io/@reamlabs/Bk9PF7WJge>
