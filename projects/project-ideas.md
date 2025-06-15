# Proposed projects

Project ideas proposed by client teams and mentors. These are projects that fit the EPF scope and would be the most valuable to the team. 

If you are interested in the project, take your time to learn about it, the team it's coming from and related area of the protocol. Reach out to client organizers if you have any questions or need connection to a mentor.

## Previous cohorts

In project ideas from previous cohorts, you might find some up to date ideas which haven't been solved yet.

- [Project ideas in the fifth cohort](https://github.com/eth-protocol-fellows/cohort-five/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-five/blob/master/projects/)
- [Project ideas in the fourth cohort](https://github.com/eth-protocol-fellows/cohort-four/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-four/blob/master/projects/)
- [Project ideas in the third cohort](https://github.com/eth-protocol-fellows/cohort-three/blob/master/projects/project-ideas.md), [Projects executed](https://github.com/eth-protocol-fellows/cohort-three/blob/master/projects/)
- [Project ideas in the second cohort](https://github.com/ethereum-cdap/cohort-zero/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22)
- [Project ideas in the first cohort](https://github.com/ethereum-cdap/cohort-one/issues?q=is%3Aissue+Project+idea)

## Ideas proposed by core devs 

## Protocol Security tooling wishlist

By Fredrik

Ongoing wishlist of ideas for tools that will help with securing the protocol.

https://security.ethereum.org/wishlist

### PandaOps tooling wishlist

By Pari

Ongoing wishlist of ideas for tools that help with development and testing of the protocol. 

https://github.com/ethpandaops/tooling-wishlist

### RIG Opened Problems

By Barnabé Monnot

Explore Robus Incentives Group Opened Problems. Most relevant for EPF are tagged https://efdn.notion.site/ROPs-RIG-Open-Problems-c11382c213f949a4b89927ef4e962adf

### Ephemery testnet

By Mario Havel

Contribute to integrations of public ephemeral testnet defined in eip-6916. Missing implementations in clients, deployments, devops tooling and client testing.
https://github.com/ephemery-testnet/ephemery-resources/blob/master/client-implementations.md
https://ephemery.dev/

### Lodestar: Backfill

By Lodestar

Lodestar's TypeScript-based Ethereum consensus client currently does not have a backfill mechanism to retain blocks/blobs required, conduct backfill syncing and eventually serve custody column groups when moving towards PeerDAS. This feature will expose a candidate to various parts of the Ethereum consensus protocol, including new chain features whilst producing a high priority need for the implementation. For more information, please check: https://github.com/ChainSafe/lodestar/issues/7753

### Lodestar: ERA file support

By Lodestar

Lodestar's TypeScript-based Ethereum consensus client is seeking a fellow interested in helping to produce and use Era files for historical state regeneration as an alternative to backfill syncing and minimizing unnecessary data stored on nodes. For more information, please check: https://github.com/ChainSafe/lodestar/issues/7048

### Besu: discv5

Besu - implementing discv5 support https://github.com/hyperledger/besu/issues/4089

By Besu

### Besu: eth/70

Besu - implementing eth/70 - https://github.com/hyperledger/besu/issues/8288

By Besu

### Besu: ephemery 

Besu - ephemery - basic support is implemented but there are still some remaining tasks - https://github.com/hyperledger/besu/issues?q=is%3Aissue%20state%3Aopen%20ephemery

By Besu

### Teku / Nimbus / Lodestar / Grandine: Implement EIP-7917 - Deterministic proposer lookahead  

[EIP-7917](https://eips.ethereum.org/EIPS/eip-7917) is a mechanism to pre-calculate and store a deterministic proposer lookahead in the beacon state at the start of every epoch.
Implementing this EIP in one or more of the above mentioned clients is a low hanging-fruit as well as [super valuable](https://hackmd.io/@linoscope/eip-7917-from-preconf-protocol), and may serve as an excellent starting point for a fellow to get involved with core protocol work.

By Justin Drake & Lin Oshitani



### Erigon: FOCIL & Alternatives 

By Mark Holt

[EIP-7805](https://eips.ethereum.org/EIPS/eip-7805) FOCIL implements a robust mechanism to preserve Ethereum’s censorship resistance properties by guaranteeing timely transaction inclusion.  We are starting to implement this for both Erigon's CL and EL components and would like to catch up with  other [client's progress](https://meetfocil.eth.limo/)  so we can become an active participant in the public devnet process once it commences.

We are also interested in exploring alternatives for censorship resistance such as [BRIAD](https://ethresear.ch/t/censorship-insurance-markets-for-braid/20288) so we envision this project as a combination of background research and implementation.

### Erigon: Parallel Execution

By Mark Holt

We have recently completed development of a parallel execution for Erigon's EL based on [Block STM](https://arxiv.org/abs/2203.06871).  This is an adaption of previous work by Polygon for the [POS Chain](https://polygonscan.com/).  Our adaption is intended to work for all chains supported by Erigon.  We expect to release this as an experimental feature on Erigon 3.1.

The POS chain contains header hints to help the parallel execution process by including dependency data in its block header. [EIP-7928](https://github.com/ethereum/EIPs/blob/954213305aa55eeda61a3fe3fe247011c038a9d4/EIPS/eip-7928.md) provides a similar but more extensive project.

From the current testing of our execution process we are aware that execution throughput in terms of gas is an optimization problem which trades off parallelism (number of active threads), disk/cache read time and failures due to cross dependencies.  When optimising this at the network level (which is the basis of 7928) there is an additional factor which is the size and associated storage and transmission costs of passing the hint data.  (Polygon uses a minimal dependency set).

As we have a working parallel implementation we intend this project to explore the options for optimizing this process in the context of Ethereum and to contribute to the practical development of EIP-7928 as a standards proposal and to parallelism across all network clients in general.   

### Erigon: Parallel Embedded Indexing

By Mark Holt

With erigon 3.1/3.2 we are developing the functionality of the data storage technology at the core of the product to facilitate the distribution of out of protocol data sets.  We are also developing a streaming mechanism which allows historical versions of that data to be generated by 3rd party code.

We believe that this will add significantly to the efficiency of the services that surround a typical blockchain client by allowing the data these services rely on to be generated in parallel using the spare CPU cycles available as blocks are processed in parallel.

In order to make this spare capacity available we intent to initiate a project to standardize the map/reduce framework we already use for the otter scan api and for custom tracing so that it can be used as a plug in framework for external developers.

It is likely this this framework will work an an analogous fashion to existing cloud function models such as [Google Cloud Functions](https://github.com/GoogleCloudPlatform/functions-framework).  The work on this project would involve reviewing the available frameworks and working with erigon team member to come up with a POC of such a framework.

Examples of indexing tasks which could form part of this POC include DEX market data, e.g. embedded EBBO as a price benchmark and the project below which is aimed at generating provable RISCV output inline with the current execution pipeline. 

### Erigon: RISCV Executable Proof Sourcing

By Mark Holt

As zk based proving has matured it has started to use indistry standard VM's for driving its proof set.  These rely on producing and analysing executable code in one of these instruction setc, RISC-V and MIPS being the current candidates used by various market participants.  RISCV seems to have an edge in adoption due as it is has simpler instructions and this has lead to an edge in delivery. 

At presnent most provers for EVM code work by compiling an EVM implementation into the instruction set architecture and then proving the operation of the resultant code.  Given the availibility of various the components necessary to run a stand alone EVM this has been relatively quick to delivery and provides an operating prover.  However this has a number of drawbaccks the most signifigant being that this leads to proving a far larger set of executable instructions than is needed to execute a simple set evm transations in a block, it also introduces a signifigant number of external dependencies which need to be proved.

An alternative is to create an ISA based execution module which is directly provable, this has implications for the tooling of current chains as it means a transition of a core component of the current Ethereum platform, a component which has a large surface area of dependencies.  Another alternative is to add a transpilation process to the existing execution module - so that it provides an executable equivalent which can be used by the proving system.

We are interested in using the erigon execution component with instrtuction level hooks to explore the practicality of the transpilation approach generating an ISA compatable executable object inline with the executions process.  We see this as a POC so it does not necessarily need t be a complete solution at this stage but it does noeed to deal with enough instructions to process a subset of viable smart contract based transactions.

Some simplifying assumtions could for example be:

    * All state is provided in the execution model as constants
    * Precomiles can be treated as idempotent functions which check the input values and return a constant outpout
    * Output value and side effects are pushed to a static constant output function or similar for runtime checking - this may require a target supplied library function

The intention is the produced output can be assembled and executed by an existing zkvm to prove the process. One option that we have considered is to use the Taiko prover framework for this, but part of the project would be to identify prover candidates to test and intergrate to.

### Erigon: L1 Clearing Bridge

By Mark Holt

This project is more R&D than development based.  We curently interact with a number of L2's each of wich has an associated bridging product.  These have a common pattern but largely incompatable operating models typically they rely on the use of logs and proofs from the source chain and rely on a trusted off chain component to initiate activity on the target chain.  They have a variety of trust and confirmation processes which may automated or semi-automated depending upon the implementation details and the cross chain economics.

WIth the pectra hard fork and the deployment of EIP-7702 one of the main points of friction in this process, namely who pays for gas transactions on the target chain is eliviated.  7702 allows payment to be transfered to an onchain paymester rather than the bridge provider.  This makes by directional bridging easier to achieve as payment transfer can become more seamless.  Whilst this still leaves several trust and finality basied inconsistencies it opens the way for a more seamless interactio0n.  It also provided the potential for bridging to be conducted by as a technical operation by node operators without them having to bear the cost of financial costs of bridge transations.

A further extension of the paymaster model could be for an L1 inchain party to act as a trustless clearer of cross chain transactions.  Exactly how this model would work and the implications and practicalities for the parties involved is unclear at this stage.  We think that post Pectra there is an interesting oportunity to explore how this process could work using an L1 CL/EL pair to expore the varios components that would be necessary to facilitate a trustless clearing based bridging model where the L1 acts as a default bridge between L2's.

We see this project as an R&D project which may provide an example contract design and at a streach, given the time scales involved, some form of working model which can be used as the basis for future phases of development.

### Ream Client - A Beam client in Rust: Build Beacon Validator Client

By Kolby ML and Kayden ML

We want to support transitioning Validators from the Beacon Chain to the Beam Chain. We also expect a lot of lessons and tooling required to build a Beacon Validator client will expedite our progress in building a Beam Validator client when the specifications are released later this year. This project would be good for one or multiple people to collaborate. https://github.com/ReamLabs/ream/issues/361

### Ream Client - A Beam client in Rust: Build out P2P testing

By Kolby ML and Kayden ML

We are building out the Beacon Chain’s P2P stack as there is likely to be a lot of overlap with the Beam Chain, this will also allow us to participate in Beacon to Beam transition testnets. We want to add tests that ensure our P2P stack is working as expected and will properly interop with other clients.

### Ream Client - A Beam client in Rust: Implement Beacon API Endpoints

By Kolby ML and Kayden ML

The Beacon API allows us to interact/test our Beacon Chain implementation to verify if everything is working. The Beacon API is also required to be able to run a validator client. It would be valuable for us to implement all the endpoints required to run a validator. Another important task would be to add tests that our implementation is compliant with the specification.

### Ream Client - A Beam client in Rust: Benchmark zkVM performance on Ream's Beacon state transition functions

By Unnawut

We've started benchmarking SP1 and RISC Zero against ream's consensus codebase, giving us early data on performance and how different zkVMs behave when running consensus logic. This lets us track performance regressions as we iterate on the codebase and pinpoint the specific roadblocks and bottlenecks that arise when running consensus specs inside zkVM environments.

We are looking to extend our benchmarking setup to cover other zkVMs like OpenVM, Zisk, Jolt, Brevis Pico, Valida, etc. You'll be implementing benchmarks for these systems, modifying the ream codebase as needed to support the snarkification, while keeping the benchmark framework modular enough to keep track of differences across code iterations, and to easily convert for beam chain specs down the road.

We hope this project provides essential data for making informed zkVM choices in Beam Chain development. For the fellow, it's a chance to get hands-on with both beacon chain consensus implementation details and the rapidly evolving zkVM ecosystem, and prepare yourself to snarkify the upcoming Beam Chain specs.


### Lighthouse: Checkpoint sync from a non-finalized checkpoint
By: Lighthouse team
Github issue: https://github.com/sigp/lighthouse/issues/7089

Allow Lighthouse to checkpoint sync from a non-finalized checkpoint. This feature is critical during long periods of non-finality and would have been super useful during the Holesky incident. You can read the Syncing section of this blog post for more context: https://blog.sigmaprime.io/pectra-holesky-incident.html

### Lighthouse: Better state cache heuristics
By: Lighthouse team
Github issue: https://github.com/sigp/lighthouse/issues/7449 and https://github.com/sigp/lighthouse/issues/7450

As an optimization, Lighthouse holds recently accessed states in memory. The current state cache size is 32. We would like to be able to dynamically grow/shrink the state cache size based on a set of heuristics (like for example current memory usage, individual state size etc.). During the Holesy non-finality incident, beacon states grew in size. Holding these larger states in memory caused Lighthouse to OOM. We would like the state cache to be more resilient in these scenarios while ensuring we are still optimizing for healthy network conditions. You can read the Out of Memory section of this blog post for more context: https://blog.sigmaprime.io/pectra-holesky-incident.html

### Lighthouse: Add PostgresDB as an optional DB implementation for the Beacon Node
By: Lighthouse team
Github Issue: TBA

The Lighthouse beacon node backend has been previously abstracted to handle different database implementations. Currently the two enabled database implementations are LevelDB and Redb. LevelDB is currently the most performant DB implementation, but we've seen issues where a LevelDB database hosted on a NFS (network file system) can get corrupted. This is because LevelDB (and probably Redb as well) is not designed to handle concurrent access and other inconsistencies/latencies introduced by NFS. Postgres, with the correct settings should be able to run on an NFS without these risks.

### Grandine: FOCIL (EIP-7805) Implementation and Testnet Deployment

By Saulius Grigaitis

EIP-7805 introduces FOCIL, a promising feature for Ethereum consensus clients. While a few teams already have early implementations, testnets are expected to launch soon. Grandine intends to develop a robust FOCIL implementation to ensure compatibility and actively participate in testnet experimentation. This work will help validate and refine the proposal through testnet deployment.

### Grandine: Optimizing PeerDAS for Production

By Saulius Grigaitis

Grandine includes an initial implementation of PeerDAS, Ethereum’s upcoming data availability sampling protocol. We aim to significantly optimize its performance, building on our custom cryptography library, rust-kzg, which already includes several enhancements. We will target further performance gains at both the cryptographic level and in broader system integration areas to ensure PeerDAS is production-ready.

### Grandine: Execution Layer (EL) Client Integrations

By Saulius Grigaitis

We are integrating Grandine into the Nethermind EL client with promising early results. We now propose to extend this work by integrating Grandine with other EL clients. In parallel, we plan to investigate inverse integrations—embedding EL functionality into Grandine. This may include low-level optimizations such as replacing HTTP-based Engine API calls with efficient native function interfaces, reducing latency.

### Grandine: zkVMs for Beacon Chain Snarkification

By Saulius Grigaitis

We are experimenting with zkVMs (zero-knowledge virtual machines) to enable SNARK-based proving of the Beacon Chain’s state transition function. Initial results using SP1 and RISC Zero show promise for networks with tens of thousands of validators. We now propose to scale this work to support larger validator sets and to explore new zkVMs — such as OpenVM and Zisk — that offer continuation supports.

### Grandine: Disk Usage Optimization for State Storage

By Saulius Grigaitis

Grandine currently stores full Ethereum states every 32 epochs. This approach leads to redundant on-disk data and slow state reconstruction. In-memory, we use structural sharing to eliminate overlapping data; we now propose to bring similar deduplication and delta encoding techniques to disk storage. This will reduce disk usage and accelerate state transitions, especially for historical state lookups.

### Grandine: Implementing Tokio Tracing for Debugging and Performance Analysis

By Saulius Grigaitis

Observability is critical for high-performance Ethereum clients. While we have long planned to integrate Tokio Tracing, it has remained a lower priority. We now propose to prioritize this integration to provide structured logging, asynchronous performance profiling, and rich diagnostics across the Grandine stack.

### Grandine: Exploring Distributed Validator Technology (DVT) Compatibility

By Saulius Grigaitis

Distributed Validator Technology (DVT) is gaining momentum within the Ethereum staking ecosystem. We propose to assess Grandine’s compatibility with major DVT solutions, evaluate integration challenges, and explore potential extensions to support secure, decentralized validator operations.

### Grandine: Standalone Validator Client

By Saulius Grigaitis

Grandine only has built-in Validator Client currently. Standalone Validator Client would help to attract some users that prefers a separate Validator Client.

### Grandine: Open Call for Collaboration
By Saulius Grigaitis

We are open to collaboration on additional ideas that align with Grandine’s mission and Ethereum’s long-term roadmap. If you have a compelling proposal not listed above, we’d love to explore it together.

### Nimbus EL: Support producing and/or consuming era1, era, e2ss, and e2hs files
By Nimbus Team

https://github.com/eth-clients/e2store-format-specs defines these. They allow an EL client to produce and consume various types of state and block archival data. The Nimbus EL already supports consuming some of these, such as era1 and era files. It has no current supprt for e2ss or e2hs files. The goal of this project is fill the remaining gaps, whether they be in the producing or consuming these formats.

### Nimbus CL: FOCIL (EIP-7805) Implementation
By Nimbus Team

FOCIL facilitates scaling L1 while retaining censorship-resistance. The goal of this project is to enable Nimbus to participate in upcoming FOCIL devnets and testnets, to help enable FOCIL adoption more broadly.

### Nimbus EL: Discovery V5 support
By Nimbus Team

The Nimbus EL discovers peers using the Discovery V4 protocol. The devp2p network is gradually shifting to using the Discovery V5 protocol, and it would internally align Nimbus EL's implementation with the Nimbus CL's V5-based implementation, to share a codebase. The goal of this project is to add V5 support to the Nimbus EL.

### Besu / Erigon / Geth / Nethermind / Nimbus EL / Reth: Pureth (EIP-7919)
By Etan Kissling (Nimbus)

Pureth aims at making Ethereum data easier to access and verify without relying on trusted RPC providers or third-party indexers. This improves UX of wallets (verifiable eth_getLogs response with proof of correctness and completeness), reduces the need for third-party indexers for dApps, and reduces gas cost when consuming partial data in smart contracts (e.g., proving presence of a single log within a receipt as part of an L2 bridge). We are looking for a prototype EL implementation (any EL) that serves JSON-RPC with proofs. The implementation will be used as part of a devnet to collect feedback from light client developers (e.g., wallets, dApps).

### Prysm: Migrate e2e to Kurtosis

By Prysm

For the longest time Prysm has had a suite of end-to-end (e2e) tests: https://github.com/OffchainLabs/prysm/tree/develop/testing/endtoend. The core part of it are evaluators that assert the state of the e2e run: https://github.com/OffchainLabs/prysm/tree/develop/testing/endtoend/evaluators. Any evaluator failure causes the whole run to fail. During recent forks it's been increasingly hard for us to add new evaluators, and the maintanance associated with e2e keeps growing with each fork. Some time ago the EF EthPandaOps team began using Kurtosis as the primary tool for running devnets: https://github.com/kurtosis-tech/kurtosis. (specifically the Ethereum Package: https://github.com/ethpandaops/ethereum-package). They also developed Assertoor whose purpose is roughly equivalent to our e2e - assessing the state of the chain: https://github.com/ethpandaops/assertoor. We think it's about time Prysm switched from a built-from-scratch e2e framework to using Kurtosis with Assertoor as our e2e component. This means moving all our e2e setup into Kurtosis and recreating our evaluators using Assertoor. What is most likely outside of scope for this project is integrating this new e2e setup into our CI/CD pipeline.

### Prysm: Lazy Slasher

By Prysm

Implement https://ethresear.ch/t/a-lazy-approach-to-slashers/22041 which is a "lazy" slasher implementation that can potentially strengthen the slashing protection mechanism. There is already an initial tracking issue on Github that lists a possibly non-exhaustive list of tasks that are required for the lazy slasher to work: https://github.com/OffchainLabs/prysm/issues/15066.

### Prysm: SSZ Query Language

By Prysm

Design and implement a query language for querying arbitrary SSZ trees. Possible features include multiple fields, filtering, anchored proofs, etc.

example of such query:

```
{
 "anchor": "<stateRoot>",
 "querySpec": {
    "validators": {
        "range": [0, 100],
        "filter": {
            "slashed": true,
            "effective_balance": {
                "gte": 32
             }
         },
    "fields": ["effective_balance", "some_other_field"]
    }
 }
 "includeProof": true
}
```

More info can be found here: https://hackmd.io/@etan-status/electra-lc#SSZ-query-language

### Prysm: Merkle Proofs of Everything

By Prysm

In the Prysm codebase there is a https://github.com/OffchainLabs/prysm/blob/develop/beacon-chain/state/state-native/proofs.go file that includes functionality to calculate a Merkle proof of the current sync committee, next sync committee and the finalized checkpoint's root. In case we'd like a proof for anything else, a new function must be defined just for that one proof. Ideally we'd like a generic way of producing arbitrary proofs for states and blocks, although it should be possible to support arbitrary SSZ objects.

### Implement the Fast Confirmation Rule

by the EF Protocol Consensus Team

The Fast Confirmation Rule is an algorithm that, assuming low latency, allows determining in 1-2 slots only that a block will always be part of the canonical chain without having to wait for finalization.

The objective of this project is to implement the Fast Confirmation Rule in one of the CL clients. Ideally, not Teku or Prysm as these two teams already have plans to work on this feature.

To know more about this:
- [Devcon Presentation](https://www.youtube.com/watch?v=p7JPRTELnJc&embeds_referring_euri=https%3A%2F%2Fapp.devcon.org%2F&source_ve_path=OTY3MTQ)
- [PEEPanEIP Presentation](https://www.youtube.com/watch?v=dZU-Ch22MKY)
- [ethresear.ch blo post](https://ethresear.ch/t/confirmation-rule-for-ethereum-pos/15454)
- [Detailed technical report](https://arxiv.org/abs/2405.00549)
- Current alternative specifications: [https://github.com/ethereum/consensus-specs/pull/3339](https://github.com/ethereum/consensus-specs/pull/3339), [https://github.com/mkalinin/confirmation-rule/blob/master/confirmation_rule.py](https://github.com/mkalinin/confirmation-rule/blob/master/confirmation_rule.py)

