# Block-Level Access Lists Implementation In Reth

The project will add [**Eip-7928**](https://eips.ethereum.org/EIPS/eip-7928) aka `Block-Level Access Lists` support in Reth.

## Motivation

EIP-7928 introduces **Block-Level Access Lists** (BALs), which record all accounts and storage locations accessed during block execution, along with their corresponding post-execution values. 

**Benefits Of Including Eip-7928 in Reth:**

* BALs provide a complete view of which storage slots are accessed and by which transactions. This enables safe and efficient parallel execution and disk operations—more reliable than optimistic parallelization techniques.

* Since BALs include full post-execution state diffs, the post-state can be reconstructed directly from the pre-state and BALs without re-executing transactions.

* BALs contribute to a partially stateless Ethereum by preserving only the required subset of state data.

* In zkEVM environments, BALs allow nodes to verify execution proofs and skip actual transaction execution while still maintaining full state integrity.

* BALs allow account balance tracking without the need for extensive RPC calls.

## Project Description

This project will allow creation of Block-Level Access List that which will mark an integral step towards stateless ethereum.We will try to implement it as per the changes listed in eip-7928 and its execution specs.

## Specification
The specification will include adding `bal_hash` to the block header and `BlockAccessList` to the block body. This involves creation of `StorageChange`,`BalanceChange`,`NonceChange`,`CodeChange`,`SlotChanges`,`SlotRead`,`AccountChanges` for BAL. Modification to `state_transition` function to validate the block-level access lists and adding `build_block_access_list`.

## Roadmap
Since the eip is relatively new, we will like to spend some more time in researching and understanding it. A rough time line will look something like:

**3-4 Weeks:** Modification of the block:
* Adding the `BlockAccessList` and `bal_hash`throughout the code base. 
* Fixing  tests.

**5-10 Weeks:** Implementation of the functionality as per EIP:

- Tracking  of the storage changes.
- Tracking of the balance changes.
- Tracking of the nonce changes.
* Tracking of the code changes.

**11 - 15 Weeks:** Assembling the tracked changes for *BAL*:
* Modifying the `state_transition` function.
* Implementing `build_block_access_list` as per the requirement of the EIP.

**16 - 20 Weeks:** Refactoring and optimizing the work:
* Improve the existing work based on review.
* Adding small features to the implementation.
* Adding comprehensive tests.

**21 - 25 Weeks:** Overall improvement: 
* Benchmarking the performance and improve implementation based on it.
* Possibly try it out in parallel execution.

## Possible Challenges
Amongst the challenges,some which typically standouts to us:
* Consideration of the [block size](https://eips.ethereum.org/EIPS/eip-7928#block-size-considerations).
* Considering the [edge cases](https://eips.ethereum.org/EIPS/eip-7928#edge-cases).
* Being up to date with the evolving eip and specfication.
* Sacrificing performance for the sake of correctness.

## Goal Of The Project

The end project will successfully be able to include complete and accurate Block-Level Access List in block and allow true perfect parallelization. This will improve the overall performance by a significant margin.

## Collaborators

### Fellows
* [Soubhik-10](https://github.com/Soubhik-10)
* [Rimeeeeee](https://github.com/Rimeeeeee)

### Mentor
* [Mattsse](https://github.com/mattsse)

## Resources
* [BAL Eip](https://eips.ethereum.org/EIPS/eip-7928)
* [Execution Specification For BAL](https://github.com/ethereum/execution-specs/compare/master...nerolation:execution-specs:BALs)
* [Implementation Overview Of BAL](https://hackmd.io/@Rimeeeeee/ByPNpeerll)
* [BAL Discussion](https://ethereum-magicians.org/t/eip-7928-block-level-access-lists/23337)
* [RLP vs SSZ Comparison Report](https://github.com/nerolation/eth-bal-analysis/blob/main/reports%2FRLP_vs_SSZ_BAL_Comparison_Report.md)
* [Analysis On Block-Level Access Lists](https://hackmd.io/@Nerolation/ByALtmDMee)

