# Implementation of FOCIL in Nimbus CL

Bringing Fork-Choice Enforced Inclusion Lists to the Nimbus Consensus Layer .
(Execution work left as an optional stretch goal)

## Motivation

Fork-Choice Enforced Inclusion Lists (FOCIL) represent a protocol-level mechanism to guarantee timely and censorship-resistant transaction inclusion by embedding inclusion constraints directly into Ethereum’s fork-choice rule. 

Basically:
- **FOCIL (EIP-7805)** restores censorship-resistance by letting a 16-validator committee publish inclusion lists (ILs) each slot
- blocks that omit valid IL transactions receive no fork-choice weight.

The Nimbus CL does not have an implementation of this proposed change so we  pursue to persue to implement for the aforementioned this during EPF cohort.

## Project description

We will upstream complete support for FOCIL into Nimbus:

1. **Validator duties** – selection of IL committee, mempool sampling via existing Engine-API, IL construction, signature & gossip.
2. **P2P layer** – new global IL topic, gossip validation (slot freshness, equivocation limit, sig checks, ≤8 KiB lists).
3. **Fork-choice enforcement** – store & evaluate ILs, call `engine_newPayloadV5`, give *zero weight* to blocks flagged `INVALID_INCLUSION_LIST`.
4. **Honest-validator logic** – duties scheduling, view-freeze timing, cut-off rules.
5. **Interop test** – local dev-net with Prysm to show Nimbus correctly publishes ILs and rejects censoring blocks
6. **FOCIL Devnet deployment** - Following the interop testing phase,we move on to deploy Nimbus in a devnet to monitor its focil-relaed capabilities.
7. **Stretch goal (time-permitting)** – collaborate further to allow Nimbus EL to produce and consume FOCIL blocks

## Specification 
 
### Nimbus-CL implementation 

#### Validator

The FOCIL Spec involves changes to the honest Validator Spec,which not limited to 

- New get_inclusion_list method
- Changes to prepare_execution_payload
- New get_inclusion_list_signature

#### Networking

Changes to P2P 

- New Topic inclusion_list
- Changes to batch and gossip validation which includes signed_inclusion_list validation and propagation

#### Fork-Choice

- Adding `is_inclusion_list_satisfied` method
- modify `notify_forkchoice_updated` method
- Modify `PayloadAttributes`
- Modified `get_forkchoice_store`
- New `validate_inclusion_lists` method
- New `get_attester_head` method
- New `on_inclusion_list` method

changes to other function based on inclusion Lists

#### Inclusion List Logic

- New InclusionList Store: Implements a cache
- Adding new `get_inclusion_list_store, process_inclusion_list, get_inclusion_list_transactions`methods


#### EngineAPI Related changes

- Adding new `engine_newPayloadV5, engine_getInclusionListV1, engine_updatePayloadWithInclusionListV1` calls

## Roadmap

| Phase                           | Weeks | Deliverables                                                                                                                |
| ------------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------------- |
| **1 – Coding**               | 5-14   | Implementation of FOCIL spec based on Release Pikachu of consensus spec.                                                       |
| **2 – Local Devnet Testing** | 14-18  | dev-net (Nimbus + Prysm + Geth) showing IL propagation & block rejection via CL logic working.  |
| **3  devnet Deployment**         | 18-25 | Deploy,monitor and assist in FOCIL related problems           |
| **(Stretch)**                  | 17-25 | If ahead of schedule: integrate Nimbus EL to the fix by implementation                     |

## Possible challenges

- Lack of static testcases to validate upon
- FOCIL spec is still in active development and many a things are bound to change 

## Goal of the project

*Mainnet merge ready consensus layer FOCIL in Nimbus*

Success looks like:

1. Nimbus nodes create & gossip ILs when in committee.
2. Nimbus rejects censoring blocks on dev-net .
3. Code passes CI, spec test-vectors, and is merged to `status-im/Nimbus-eth2` master prior to Glamsterdam test-net.
4. Stretch: demo full Nimbus EL enforcement on a private dev-net two weeks before cohort end.

## Collaborators 

- Tamaghna (@razorclient)
    - Responsible for Inclusion list 
    - Responsible for Validator Changes
    - Responsible for Engine changes
- Akash (@h3lio5)
    - Responsible for P2P
    - Responsible for Fork-Choice Changes

We will be pursuing this work together and divide work as we see fit with blockers and work items

## Mentors

@Agnishghosh from the Nimbus team 

## Resources

* EIP-7805 – [https://eips.ethereum.org/EIPS/eip-7805](https://eips.ethereum.org/EIPS/eip-7805)
* Consensus release Pikachu – https://github.com/ethereum/consensus-specs/tree/a0f12bf7d8ba7390da67d4fe173241b5471769f8
* HackMD note – [https://hackmd.io/@jihoonsong/ryTt1Fv7le]
* Nimbus Branch - https://github.com/status-im/Nimbus-eth2/tree/focil
* Nimbus-web3 branch - TODO(not yet merged)