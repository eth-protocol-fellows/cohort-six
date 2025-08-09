# Ream Validator Client

Implementation of the Validator Client for Ream

## Motivation

The Lean Ethereum proposal brings about major improvements in the Ethereum's Consensus Layer for hardening and scaling Ethereum. It aims to bring the following improvements to the consensus layer:

1. Quantum Resistant Cryptography
2. SNARKs used accross the chain
3. Smaller Validators (Min Stake: 32 ETH > 1 ETH)
4. Faster Slots (4 secs)
5. Faster Finality (3 Slot Finality)

These are only some of the changes proposed but are the ones I am most excited about. Ream is one of the most active teams working on developing a client for the Lean Chain.

Our motivation is to contribute effectively to Ream's efforts and accelerate development towards Lean Consensus.

We aim to contribute primarily to the development of Ream's Validator Client. Some of our work on this was presented [here.](https://docs.google.com/presentation/d/1nnDkp74VPYhdadV8LsrAvkd0Lf4u3DLwrIwW3DkeBU0/edit?slide=id.g2558abb5fb3_0_0#slide=id.g2558abb5fb3_0_0)

## Project description

The project is to build a Validator client for Ream (capable for running on all Post Quantum Devnets of the Lean Chain).

## Specification

The project can be broken down into the following:

### Core Validator Functionality

1. Block Signing
2. Block Proposals
3. Attestation Signing
4. Make Attestation
5. Attestation Aggregation
6. Voluntary Exits
7. Reorg handling (if the Lean Chain has reorgs)
8. Validator CLI
9. Grafana + Prometheus metric collection
10. Integration Tests (EthPandaOps: Kurtosis)
11. Devnet Deployment

We followed a REST API based approach for interfacing with a Beacon Node through the [Beacon API](https://ethereum.github.io/beacon-APIs/#/) to facilitate the interopability between our Validator Client and Beacon Client.

Beyond the core validator features, as Ream aspires to be the consensus client with a heavy focus on UX, we want to build features that improve the user experience of the Validator Client. We want to ensure ease of use, as users have stated they dislike how tedious it is to set up different tools and programs required for staking.

### Initial Beacon Chain Implementation

We began our development by building a Beacon Chain validator client to establish a solid foundation and understand the core validator architecture. The experience gained from this implementation has been invaluable in understanding the validator architecture and lifecycle.

Relevant work can be found in the following portions of the Ream repository:

1. [Beacon API Client](https://github.com/ReamLabs/ream/blob/master/crates/common/validator/beacon/src/beacon_api_client/mod.rs) includes all the required Validator API to interface with a Beacon Node following the [Beacon API Spec.](https://ethereum.github.io/beacon-APIs/#/) The `reqwest` crate was used to implement all client side requests.
2. [Beacon API Types Crate](https://github.com/ReamLabs/ream/tree/master/crates/common/beacon_api_types/src) to consolidate types across different crates that use the same structs.
3. Signing immplemented with [Zkcrypto](https://github.com/ReamLabs/ream/blob/master/crates/crypto/bls/src/zkcrypto/signature.rs) and [Supranational](https://github.com/ReamLabs/ream/blob/master/crates/crypto/bls/src/supranational/signature.rs) backends.
4. [Main Validator Functionality and Flow](https://github.com/ReamLabs/ream/blob/master/crates/common/validator/beacon/src/validator.rs)
5. [Voluntary Exits](https://github.com/ReamLabs/ream/blob/master/crates/common/validator/beacon/src/voluntary_exit.rs)
6. [Validator CLI config](https://github.com/ReamLabs/ream/blob/master/bin/ream/src/cli/validator_node.rs) and [startup](https://github.com/ReamLabs/ream/blob/master/bin/ream/src/main.rs)

This has positioned us to make informed architectural decisions as we transition to the requirements of the Lean Chain.

### Transitioning to the Lean Chain

![image](https://hackmd.io/_uploads/r1NhwONdlx.png)

> Source: [Post on Ream's Lean Architecture](https://hackmd.io/@reamlabs/B1V8VRxdxl?stext=7175%3A1172%3A0%3A1754724292%3AAo1a4V)

We will implement the majority of the Lean Validator Service, the timer and the keystores for the Ream Validator Client.

**Post-Quantum Cryptography Integration:** Moving from BLS signatures to quantum-resistant hash-based signatures using the hash sig crate.

**3 Slot Finalty:** The 3-slot finality (3SF) model presents a different duty model with 'Stakers' and 'Voters' which will be implemented as per the 3SF mini implementation and will be improved as new changes come forth.

**Faster Slot Timing:** With 4-second slots instead of 12 seconds, our validator timing mechanisms must be changed as per the new timing windows for proposals, attestations and so on.

The Lean Chain Validator Node for the PQ-Devnets also does not operate with a CL Node but would rather be a standalone node that performs its necessary operations over the P2P layer directly.

## Roadmap

<!--
What is your proposed timeline? Outline parts of the project and insight on how much time it will take to execute them.

Shariq -->

### Week 0 - Week 8:

1. Study Validator Architecture from epf.wiki video
2. Implement Validator functions from defined in Honest Validator specs
3. Implement BLS Signing in the Ream crypto crate
4. Implement duty fetching beacon api endpoints to better understand proposer and attester duty structs
5. Create Beacon API Types Crate to consolidate the structs used across the codebase for beacon api server-side in the node and client-side in the validator
6. Implement Validator Client functions to make requests to a Beacon Node as per the Beacon API spec
7. Implement function to fetch duties in the Validator
8. Implement propose block logic
9. Implement attestation logic
10. Implement ability to perform voluntary exits and CLI to initiate it

The following items were implemented in collaboration with Patchy, who is not continuing in the EPF:

1. Implemented EIP-2335 keystore validation, serialization and deserialization, and decryption.
2. Implemented validator sync committee processing
3. Implemented BLS Signature aggregation
4. Connected the events api to the validator
5. Implemented a validator slot clock
6. Implemented the validator service CLI for starting up the validator
7. Adding fetching for validator indicies for the validator
8. Implemented genesis time fetching for validator

### Week 9 - 12:

- Study Vitalik's [3SF Mini Implementation](https://github.com/ethereum/research/blob/master/3sf-mini/consensus.py)
- Study skeletal Ream PQ Devnet Code.
- Implement duty fetching in the Lean Chain
- Integrate signing using [hash based signatures](https://github.com/b-wagn/hash-sig)
- Implement clock ticker for timing validator operations

### Week 13 - 21:

- Integrate Post Quantum signature aggregation
- Integrate Prometheus & Grafana for metrics
- Implement fetching, signing and proposing blocks in the Lean Chain
- Implement attesting (voting) in the Lean Chain
- Participate in PQ-Devnet-0
- Participate in PQ-Devnet-1 (PQ Signatures)
- Maybe Participate in PQ-Devnet-2 (Signature Aggregation)
- Contribute to the creation of the Validator parts of the Lean Consensus specs
- Attend all Post Quantum Interop meetings
- Create technical documentation on discussed topics

### Week 21 - 22:

- Work on the Final report for EPF
- Prepare for talk at DevConnect

## Possible challenges

The specs for the Lean Consensus are not yet finalized and there is little documentation on the effort as of now. This means that we must attend every meeting as well as participate in every discussion so that we are not missing out on important details. This also gives us the opportunity for a lot of technical writing as we will aim to create documenation on the things we learn as we go forward.

The Lean Chain's cryptographic libraries are still maturing compared to established BLS libraries. This challenge positions us to collaborate with researchers by providing feedback that will help shape these libraries, making them developer-friendly and easy to integrate.

## Goal of the project

The goal of this project is to build the Validator Client for Ream. We were initially building a Beacon Chain validator as the work on the Lean Chain was still under research and we wanted to learn the foundation required to build a validator client. As Ream moves further into establishing itself as a Lean Chain client and work on the spec and devnet starts, we are now shifting into building the Validator Client for the Lean Chain.

This project aims to build the Lean Chain Validator Client that will participate in pq-devnet-0, pq-devnet-1 and pq-devnet-2 and will later go on to serve as the long term Validator Client for Ream.

Here are the [Proposed Devnets](https://github.com/leanEthereum/pm/blob/collab-pq-devnet-0/pq-interop/pq-devnet-0.md) as of now.

To accomplish this the validator client will implement the following:

### Validator Operations:

- **MUST** determine its duty for current slot
- **MUST** be able to maintain secure keystores
- **MUST** be able to sign blocks
- **MUST** be able to propose blocks
- **MUST** be able to attest (vote)

### Post Quantum:

- **MUST** use Post Quantum Signatures for signing
- **MUST** aggregate Post Quantum Signatures

## Collaborators

### Fellows

Shariq Naiyer - [Github](https://github.com/shariqnaiyer)

### Mentors

[Kayden ML](https://github.com/Kayden-ML) - Ream

## Resources

### Meta Issues:

[Ream Validator Client Issue](https://github.com/ReamLabs/ream/issues/361)
[PQ Devnet-0 Meta Tracking Issue](https://github.com/ReamLabs/ream/issues/654)

### Specifications:

1. [Validator Flow Documentation](https://ethereum.github.io/beacon-APIs/validator-flow.md)
2. [Consensus Specs](https://github.com/ethereum/consensus-specs/tree/dev/specs/electra)
3. [Beacon API Spec](https://ethereum.github.io/beacon-APIs/#/)
4. [3 Slot Finality](https://ethresear.ch/t/3-slot-finality-ssf-is-not-about-single-slot/20927)
5. [3SF Mini Implementation](https://github.com/ethereum/research/blob/master/3sf-mini/consensus.py)

### Videos:

1. [Ethereum Proof-of-Stake Mechanism](https://www.youtube.com/watch?v=5gfNUVmX3Es&t=165s&ab_channel=AltExplainer)
2. [EPSg - Validator Architecture](https://www.youtube.com/watch?v=rgrNMbYrOmM&t=4215s&ab_channel=EthereumProtocolFellowship)
3. [EPSg - Consensus Layer](https://epf.wiki/#/eps/week3?id=pre-reading)
4. [Lean Chain Videos](https://leanroadmap.org/)

### Blogs:

1. [Validator Lifecycle](https://mixbytes.io/blog/ethereum-validator-lifecycle-a-deep-dive)
2. [Simplifying the L1 - Vitalik](https://vitalik.eth.limo/general/2025/05/03/simplel1.html)
