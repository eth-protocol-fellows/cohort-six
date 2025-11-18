# Benchmarking zkVMs on the Ream consensus client

Benchmarking zero-knowledge proof generation and validation of state transition functions (STFs) for beacon chain and lean chain inside multiple zkVMs.

## Motivation

Ream is a lean consensus client. During the first months of its development, a beacon chain consensus implementation was also developed mainly for testing and learning. This project follows the same path. First, we worked on executing beacon chain STFs inside zkVMs, and then moving into lean consensus.
Lean consensus proposes chain snarkification—converting CL state updates into zk-provable computations. Under this model, block proposers execute and prove state transitions, while validators verify proofs. Because verification is dramatically cheaper than execution, this model enables validators to run on constrained hardware (smartphones, Raspberry Pi-class devices), improving decentralization.

This project provides two main benefits to the Ream team:
- It is the first systematic study of zkVM performance in the early stages of lean chain development.
- It helps in designing the Ream client to be zkVM friendly.

## Existing work

The project builds on top of the work done by the Ream team on proving beacon chain STFs with [SP1](https://github.com/ReamLabs/consenSP1us) and [RiscZero](https://github.com/ReamLabs/consenzero-bench) zkVMs:

- Only one state transition function can be benchmarked on each run (not the whole block body).
- The benchmarks include only cycle counts and execution time but not proof generation or verification time, nor profiling results.
- The benchmarks only pick a few data points to compare; however, the results from SP1 and RISC Zero have not been directly compared.

In particular,

#### SP1

The project structure is the following:
- `consensp1us/program/operations/src`: `main.rs` includes main logic of the program, reads the state transition function to be applied on the beacon block and commits `pre-state.<process-operation>` to the public output. Here, `process-operation` will be replaced by attestation, attester_slashing, block_header, etc. The zk proof will include this commit.
- `consensp1us/lib/src`: includes modules for reading files, providing beacon block fields as input, compression and serialization using snappy and SSZ respectively.
- `consensp1us/script`: contains scripts that run the program with or without proof and generates benchmarks.

#### RiscZero

The project structure is:
- `consenzero-bench/host/src/bin`: includes code for setting up execution environment and the prover, doing the proving and proof verification, creating logs and benchmarks (cycle counts).
- `consenzero-bench/methods/guest/src`: contains the beacon state transition functions that will be proven by the zkVM.
- `consenzero-bench/lib/src`: the same library for modules as in SP1.

The structure is similar in both cases: `lib` provides functionality, there is a script which automates building, running, proving and validating, there is a module that sets up the zkVM environment for proving and finally there is the guest code i.e. the STFs that will run in the zkVM.

## Work during the fellowship

### zkVM on the beacon chain

In the first part of the project, we expanded the benchmarking to introduce other zkVMs. Dimitrios worked on OpenVM and Utsav worked on Jolt, Pico and Zisk, as well as developing a [unifying framework](https://github.com/x-senpai-x/Ream-ZKVM-Benchmarks/tree/main/Beacon-Harness).

#### Introducing Ream-ZKVM-Benchmarks

Although there are differences among zkVMs in their proof system architectures (FRI-STARK-based systems like SP1, RISC Zero, OpenVM, Pico; lookup-based systems like Jolt; and Nova-based folding schemes), the main principle remained largely the same. We reused portions of the `lib` modules, guest code and some scripts, but we built the host code from scratch. We also implemented elapsed time measurement and execution cycles. Utsav's framework proved sufficient for our current purposes. We were advised to look at the [ere interface](https://github.com/eth-act/ere), but it was not a priority during the fellowship timeline. It will be interesting to investigate it in the future as an abstraction layer for both beacon chain and lean consensus proving.

The unified benchmarking framework structure is:

- **`guest/`**: Guest programs for each zkVM (`sp1/`, `risc0/`, `zisk/`, `app/`, `jolt/`). Each contains `main.rs` with circuit logic to deserialize inputs, process state transitions, and commit state roots.

- **`host/src/bin/`**: CLI orchestration in `main.rs`. Sets up provers, prepares inputs from test data, executes or proves based on MODE flag, and collects metrics.

- **`lib/src/`**: Shared utilities. `input.rs` (block + epoch operations), `file.rs` (I/O), `ssz.rs` (serialization), `snappy.rs` (compression).

- **`host/mainnet/tests/`**: Ethereum Foundation test data in snappy-compressed SSZ format, organized by `block/` and `epoch/` operations.

- **`host/summaries/`**: Benchmark outputs organized by zkVM backend and operation type with performance metrics.

- **`Makefile`**: Build interface for 5 zkVMs (SP1, RISCZERO, PICO, ZISK, JOLT) with MODE (execute/prove) and PROOF_TYPE options.

### zkVM on lean consensus
Consensus is achieved using 3-slot-finality, specifically a simplified version called 3SF-mini. Proving is optional and is initialized with a CLI flag. We created a new service in Ream called `LeanProverService` that is called by the blockproposer concurrently with `ValidatorService`. We again prove the execution of STFs. For now, we generate a proof every 200 slots (slot 2, 202 and so on) because proof generation extends beyond the slot that initialized it. It takes around 6 minutes (~90 slots) on a 16GB MacBook to generate the first proof at slot 2, while the slot lasts 4 seconds. As time passes, the state gets larger, leading to 12, 14, ... minutes of proving, which results in concurrent provings taking place that leads to out-of-memory crash. With this setup, we managed to reach up to slot 1000.

## Specification
### Beacon Chain Implementation

Implementation of the first part of the project was straightforward. The guest code was found in `consenzero-bench/methods/guest/src/main.rs` of RiscZero or `consensp1us/program/operations/src/main.rs` of SP1. The same was done with modules in `lib`. The host program was built according to the SDK instruction provided by each zkVM.

Each implementation followed the same pattern:
- **OpenVM** : The instructions are found in the SDK section of the [OpenVM book](https://book.openvm.dev/advanced-usage/sdk.html#using-stdin) specifying:
     -  the preambles that should be added to the guest program
    - the code to be added in host to build, transpile and prove the guest


- **Jolt** (`guest/jolt/src/main.rs`): Implemented using the `#[jolt::provable]` macro annotation system.

- **Pico** (`guest/app/src/main.rs`): Built using the Pico SDK with `pico_sdk::io` for input/output operations. Requires Rust nightly.

- **Zisk** (`guest/zisk/src/main.rs`): Implemented with `ziskos` runtime using the `entrypoint!` macro and bincode serialization.

All implementations share the `lib/` crate for consistent operation handling across backends. The host CLI in `host/src/bin/main.rs` uses Cargo feature gates (`sp1`, `risc0`, `zisk`, `pico`, `jolt`) to enable the selected zkVM backend at compile time.

#### Performance Constraints

We did not implement GPU optimization or other advanced optimizations for proof generation during the fellowship. Generating a proof for a single test of a single state transition function operation currently crashes laptops with 16GB RAM due to memory exhaustion.

On a desktop with a Ryzen 3400G and 64GB of RAM, it took approximately 10 hours to generate a proof with OpenVM for the test `attestation_invalid_correct_attestation_included_after_max_inclusion_slot`. On a MacBook Pro M1 (10 cores, 64GB RAM), it took 4.5 hours to generate the proof for `attestation_invalid_correct_attestation_included_after_max_inclusion_slot` and 6 hours for `basic_attestation` using SP1.

This created a major constraint in running multiple tests with extensive coverage since we would have needed more than 60 hours of proving for a single test across 6 zkVMs. Additionally, different zkVMs are optimized for different scenarios: GPU versus CPU proving, high versus low execution cycle counts, and varying proof system backends (STARK, SNARK, Groth16, PLONK).

Our project provides the following metrics for each zkVM backend: execution time, execution cycles, individual suboperation time (if implemented by the respective zkVM), and proof generation time for one or very few test cases.

### Lean Chain Implementation

The second part of the project is more complicated as it involves lean consensus. Lean Chain is under development with many questions still open. At the same time, as we have already experienced in the first part of the project, zkVMs are still slow for our use cases. These two factors impose some limitations on our project.

We generate a proof every 200 slots and we did not manage to test validation because setting up a network with proving initialized created issues both inside Docker and locally. Currently we use RISC Zero, but we plan to abstract it soon so that all 6 zkVMs become available. Our design introduces proving as an additional feature to the Ream client, leaving the core logic untouched. Further details are found in the associated [draft PR](https://github.com/ReamLabs/ream/pull/835).

#### LeanProverService Architecture

We introduced a new `LeanProverService` (`service.rs`) that generates proofs for blocks. The service runs independently and receives `GenerateBlockProof` messages with a slot number. When a message arrives, it spawns a background task to generate the proof without blocking other operations. Once the proof is ready, it's automatically gossiped to the network.

We had to modify networking by adding a `BlockProof` message, a proof gossip topic, and proof verification logic. We also created the usual guest-host pair, added proof triggers, and two structs in `ream/crates/common/consensus/lean/src/block.rs`:

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

We extended the P2P layer with proof-specific functionality:
- **New message type**: Added `LeanP2PRequest::GossipBlockProof(BlockProof)` to `p2p_request.rs`
- **New gossip topic**: Introduced `LeanGossipTopicKind::BlockProof` with topic name "block_proof" in `topics.rs`
- **Gossip message handling**: Added `LeanGossipsubMessage::BlockProof(BlockProof)` variant in `message.rs`
- **Encoding**: `BlockProofs` use bincode serialization (unlike blocks/votes which use SSZ) due to RISC Zero's `Receipt` type requirements
- **Proof verification**: Network service verifies received proofs using `receipt.verify(method_id)` in `mod.rs`

The zkVM implementation consists of:

**Guest code** (`methods/guest/src/main.rs`):
```rust
fn main() {
    let mut state: LeanState = env::read();
    let new_block: SignedBlock = env::read();
    eprintln!("{}:{}: {}", "read-signed-block", "end", env::cycle_count());
    state.state_transition(&new_block, true, false);
    env::commit(&state);
}
```

**Host code** (`service.rs`): Sets up the `ExecutorEnv`, injects the state and block, runs the RISC Zero prover with succinct options, and extracts the proven state from the receipt journal.

## Roadmap
<!-- Add what our draft PR has along with Unnawut's suggestions -->

1. Create a modular framework that runs STFs inside Jolt, Pico, ZisK, OpenVM, Risc0 and SP1 (OpenVM is temporarily excluded, we have to fix some dependency issues).
2. Add execution-time and cycle-count benchmarking across zkVMs.
3. Learn about 3-slot finality for Lean-Consensus.
4. Integrate LeanProverService inside the LeanChain to invoke zkVM for producing block and executing the STF.
5. Gossip the LeanBlockProof to the peers and validate the proof.

This concludes the fellowship.

## Possible challenges

- Every zkVM implements optimisations differently, including GPU optimisation. Taking into account fast and breaking changes even on minor updates it is hard for our current project to stay updated and ever harder to optimize. We should take a closer look at at [ere interface](https://github.com/eth-act/ere) and test if it handles automatically updates.
- Ream's codebase is also evolving which could create some issues. We don't think that this will be as serious though since our implementation operates as a separate service that only handles proving.
- Obtaining consistent metrics across zkVMs. Not all of them provide cycle counts for the execution of functions in the guest. Moreover, proof generation takes many hours which restricts us to testing very few cases.
- Memory proved to be the primary bottleneck rather than CPU performance, with 16GB being insufficient for single proof generation in most cases.

## Goal of the project
The goal was to evaluate zkVM feasibility for Ream’s beacon chain and lean chain STFs, and to build tooling to support multi-zkVM benchmarking.

During the fellowship, we achieved:
- Execution-time and cycle-count measurements for 6 zkVMs
- Proof-generation time measurements for selected operations
- A unified cross-zkVM benchmarking framework with support for multiple proof types
- Prototype integration of proving into Ream's lean consensus via `LeanProverService`
- Insights into which zkVMs are feasible for future Ream adoption based on performance characteristics and hardware requirements

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
