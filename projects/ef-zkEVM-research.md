# zkEVM Research [Ethereum Foundation]

## Motivation

With the goal of improving the scalability of Ethereum, one of the most promising paths right now is the use of zero knowledge virtual machines (zkVMs), applying zkVMs on the Execution layer and the Consensus layer.
In order to do this correctly, it is important for the ecosystem to have a team of researchers serving as scouts, exploring various paths and approaches, and collaborating with other teams to ensure that the zkEVM is developed in a way that is compatible with the rest of the Ethereum ecosystem.


## Project description
This project is a continuous effort to research and develop zkEVM related technology, tips and tricks, and best practices.
1. Research and PoC implementation for a hybrid EVM environment (native EVM and RISC-V)
2. Research Exploring the generality of the ZisK zkVM trace generation technique
3. Research exploring the introduction of Linux ABIs and syscalls to zkVMs natively, enabling guest programs writen in Go or C# proveable also.

More research task would be assigned to me as all these are concluded, also the item and number 1 and 2 are 90% completed.

## Specification
### Research and PoC implementation for a hybrid EVM environment (native EVM and RISC-V)
> This research item started before joining the EPF program and my mentor ask I finish it up as it is a research item here at the EF.

**Phase One: Custom RISC-V EVM Architecture**
[code](https://github.com/developeruche/riscv-evm-experiment/tree/main/crates/research-draft)
[paper](https://github.com/developeruche/riscv-evm-experiment/blob/main/Bridging%20Worlds_%20A%20Performance%20and%20Feasibility%20Analysis%20of%20RISC-V%20Integration%20with%20Ethereum%20Virtual%20Machine.pdf)

1. Design of the core components of this new RISC-V EVM VM, this would also involve blockchain related operations fashioned as `opcodes` [rust implementation](https://github.com/developeruche/riscv-evm-experiment/tree/main/crates/research-draft)
2. Implementation of a RISC-V IM32 assembler compatible with blockchain requirements. This would be task to assemble RISCV assemble written smart-contracts to RISCV machine code [code](https://github.com/developeruche/riscv-assembler).
3. Optimization of the implementation for performance, security specs constraints.
4. Benchmarking and preliminary analysis

**Phase Two: Integration with Existing Ethereum Runtime**
[code](https://github.com/developeruche/hybrid)
1. RISCV Smart Contract Development: Write contracts in Rust compiled to RISCV bytecode
2. Local Blockchain Node: Run a development blockchain with RISCV VM support
3. Deployment Tools: Deploy RISCV contracts with a simple command
4. Dual VM Integration: Support for both RISCV VM (r55) and EVM in a single node
5. Hybrid Execution Environment: Seamlessly switch between EVM and RISC-V execution
6. Rust Ecosystem Integration: Extends Cargo to support blockchain development workflow

**Phase Three: Data collection**
During this Phase, the result of this experiment is documented.

### Research Exploring the generality of the ZisK zkVM trace generation technique

#### **Phase 1: Architectural Deep Dive**

* **Objective:** Reverse-engineer the ZisK compilation and trace generation pipeline.
* **Key Actions:**
    * Analyze ZisK's custom `riscv64ima-zisk-zkvm-elf` Rust compiler target.
    * Map the Ahead-of-Time (AOT) process that translates custom bytecode into `x86_64` assembly with injected trace-generation instructions.
* **Deliverable:** A detailed architectural specification of the ZisK engine.


#### **Phase 2: Universal Translator Feasibility Study**

* **Objective:** Design a theoretical "ZisK-ifier" tool and assess its viability for other zkVMs.
* **Key Actions:**
    * Outline the design for a universal translator that takes standard zkVM binaries (e.g., from Risc Zero) as input.
    * Analyze the primary challenge: adapting the injected trace logic to the unique proof system requirements of different zkVMs.
* **Deliverable:** A technical paper on the design, feasibility, and inherent complexities of a universal tool.


#### **Phase 3: Conclusion and Dissemination**

* **Objective:** Synthesize and document the research findings.
* **Key Actions:**
    * Consolidate findings from Phases 1 and 2 into a final research paper.
* **Conclusion to be Validated:** ZisK's performance stems from a deeply integrated, holistic architecture, making its technique difficult to generalize as a simple plugin. The future of zkVM performance lies in similar "architecture-specific innovation."

### Research exploring the introduction of Linux ABIs and syscalls to zkVMs natively, enabling guest programs writen in Go or C# proveable also.
This research explores the introduction of a minimal, provable Linux ABI to a RISC-V zkVM. The primary goal is to enable the execution and proving of guest programs written in languages like Go (Geth) and C# (Nethermind) without requiring custom compiler toolchains.

1. During this phase I would be implementing a binary for a the execution of a block statelessly, just based off the block data and witness. Would be making implementations for RETH, GETH and Nethermind. [codebase](https://github.com/developeruche/stateless-block-exec)

2. Syscall Usage Documentation: Formally document the complete set and frequency of Linux syscalls used by the stateless Geth binary. This data will serve as a definitive reference for supporting Go applications in a zkVM.

3. Final Report and Publication: Produce a comprehensive research paper detailing the implemented architecture, the syscall analysis for Geth, the performance benchmarks, and the overall feasibility of the native Linux ABI approach.

4. Generalization for C# (Nethermind): Analyze the syscall requirements for a stateless Nethermind (C#) client. Based on the Geth findings, outline the specific, minimal additions required to the ABI layer to achieve provable execution for C# programs, solidifying the general-purpose nature of this approach.


## Roadmap
Research and PoC implementation for a hybrid EVM environment (native EVM and RISC-V): Research paper and PoC implemantion is done, leaving data extraction as the only remaining task, This is due for `15 Aug, 2025.`[report](https://hackmd.io/@0xdeveloperuche/Hk18BWxkxl)

Research Exploring the generality of the ZisK zkVM trace generation technique: Research on this has been conclude and report submitted to my mentor. [report](https://hackmd.io/@0xdeveloperuche/S1sZEi7Lxl)

Research exploring the introduction of Linux ABIs and syscalls to zkVMs natively, enabling guest programs written in Go or C# to be provable as well: This is currently in progress. Phase one has been completed, and I am currently working on phase two. Research completion is due by 30th August, 2025.

- Highlight the nature of the guest program the zkVM will need to run in order to snarkify the execution layer.
- Write that program in the Rust programming language (compiling to the target OS's native standard library).
- Write the same program in the Go programming language.
- Write the same program in the C# programming language.
- Evaluate and highlight the syscalls employed by these different implementations.

> Due to the nature of the team I am working with more research taskes would be allocated to me as I exhaust what I have right now, I would do well to update this note all along.

## Possible challenges

The main challenge I anticipate is the introduction of Linux ABIs and syscalls to zkVMs. Identifying the key syscalls and ABIs needed to support guest programs written in Rust is relatively straightforward, as the RISC-V64 binary is just a binary without an embedded runtime (unlike the Go runtime). Additionally, Rust is my primary language, so I’m already comfortable working in that ecosystem. On the other hand, supporting Go programs would require extra effort and learning, as I am less familiar with its runtime and internals.

## Goal of the project

1. PoC implemenation for a hybrid node and performance anaysis report
2. Report on the viablity of exploring the generality of the ZisK zkVM trace generation technique
3. A deatils report of the Linux ABI and syscalls zkVM need to support

## Collaborators
[developeruche](https://github.com/developeruche)

### Fellows 

### Mentors

[Kev](https://x.com/kevaundray)

## Resources
https://github.com/developeruche/stateless-block-exec
https://github.com/developeruche/hybrid
https://github.com/developeruche/riscv-evm-experiment
