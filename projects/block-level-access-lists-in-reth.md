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

This project will allow creation of Block-Level Access List that will mark an integral step towards stateless Ethereum. We are implementing changes according to the EIP, execution specs and tests in Reth EL client.

## Specification
Eip-7928 proposes adding two new fields to the block structure. One is `BlockAccessList` to the block body and other is `bal_hash` to the block header, which contains the hash of the `BlockAccessList` so that it can be easily verified.
```rust
pub struct Header {
...// Existing Fields
    
    /// [EIP-7928]: https://eips.ethereum.org/EIPS/eip-7928
    pub bal_hash: Option<B256>,
}

pub struct BlockBody<T, H = Header> {
  .....// Existing Fields
    
    ///[EIP-7928]: https://eips.ethereum.org/EIPS/eip-7928 : Block-level access list
    pub block_access_list: Option<BlockAccessList>,
}


```

#### The `BlockAccessList` structure will look like:
```rust
pub struct BlockAccessList {
    /// List of account changes in the block.
    pub account_changes: Vec<AccountChanges>,
}

pub struct AccountChanges {
    /// The address of the account whoose changes are stored.
    pub address: Address,
    /// List of storage changes for this account.
    pub storage_changes: Vec<SlotChanges>,
    /// List of storage reads for this account.
    pub storage_reads: Vec<StorageKey>,
    /// List of balance changes for this account.
    pub balance_changes: Vec<BalanceChange>,
    /// List of nonce changes for this account.
    pub nonce_changes: Vec<NonceChange>,
    /// List of code changes for this account.
    pub code_changes: Vec<CodeChange>,
}

pub struct SlotChanges {
    /// The storage slot key being modified.
    pub slot: StorageKey,
    /// A list of write operations to this slot, ordered by transaction index.
    pub changes: Vec<StorageChange>,
}

pub struct StorageChange {
    /// Index of the transaction that performed the write.
    pub tx_index: TxIndex,
    /// The new value written to the storage slot.
    pub new_value: StorageValue,
}

pub struct BalanceChange {
    /// The index of the transaction that caused this balance change.
    pub tx_index: TxIndex,
    /// The post-transaction balance of the account.
    pub post_balance: u128,
}

pub struct NonceChange {
    /// The index of the transaction that caused this nonce change.
    pub tx_index: TxIndex,
    /// The new code of the account.
    pub new_nonce: u64,
}

pub struct CodeChange {
    /// The index of the transaction that caused this code change.
    pub tx_index: TxIndex,
    /// The new code of the account.
    pub new_code: Vec<Bytes>,
}
```
As per the discussion, we will use rlp serialization instead of the ssz. 
All of these changes are to be made in the [alloy-rs/alloy](https://github.com/alloy-rs/alloy/).  

#### Creating the `BlockAccessList`:

We will modify the [revm](https://github.com/bluealloy/revm) to trace the changes directly at the opcode level. This will be more efficient than using `Inspector` for the same.

#### State Transition Function:

In the `state_transition(block)` we will:
1. Iterate over all the transactions in block and execute them. 
2. Store the changes in BAL proposed core structures.
3. Build `BlockAccessList` using  the proposed `build_block_access_list` function in the eip.
4. Validate the `BlockAccessList` with the `bal_hash`.
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
* Consideration of the [block size](https://eips.ethereum.org/EIPS/eip-7928#block-size-considerations): The average size of bal is ~40 KiB . This will cause some extra overhead but will ultimately be beneficial for parallelization.
* Considering the [edge cases](https://eips.ethereum.org/EIPS/eip-7928#edge-cases) and properly implementing them.
* Being up to date with the evolving eip and specification.
* Sacrificing performance for the sake of correctness as modifying the revm might lead to regression.

## Goal Of The Project

The end project will successfully include a working prototype that adds complete and accurate Block-Level Access Lists in a block. This should support basic testing, benchmarking and help with future improvements.This will allow testing and understanding how true parallelization work.

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
