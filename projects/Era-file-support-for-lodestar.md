# Adding Era File Support in Lodestar

Implement support for `.era` files in Lodestar to enable efficient long-term storage and distribution of Ethereum consensus data.

## Motivation

Lodestar currently supports consensus layer operations but does not yet natively integrate with the `e2store` / `era` archival format. Era files are specialized `.e2s` files that bundle states and blocks in fixed-size groups (one per `SLOTS_PER_HISTORICAL_ROOT`). They provide a canonical cold-storage representation of Ethereum history, optimized for reading, validation, and checkpoint sync.

Native Lodestar support for `.era` files unlocks several benefits:
- **Historical sync efficiency**: Ability to bootstrap from finalized checkpoints using pre-verified era files instead of downloading full block/state histories.
- **Validator friendliness**: Era files allow lightweight devices to access verified history without recomputation.
- **Interoperability**: Ensures Lodestar can consume and produce files compatible with other clients like Nimbus, enabling ecosystem-wide standardization.
- **Auditability and distribution**: Researchers, light clients, and custodians can rely on `.era` files for fast consistency checks.

## Project description

The project involves implementing reading, verification, and  writing of `.era` files in Lodestar.

### Scope

- **Parsing `.e2s` records** (`packages/era/src/`):
  - Implement `readEntry` for 8-byte headers (2-byte type, 4-byte length LE, 2-byte reserved=0).
  - Implement `readSlotIndex` and `getEraIndexes` to parse trailing slot indices without full scans.
  - Add `EraReader` and `EraWriter` classes for sequential reading and writing of groups.

- **Era grouping**:
  - Groups contain `CompressedSignedBeaconBlock*`, one `CompressedBeaconState`, optional extension entries, and trailing `SlotIndex` records.
  - Genesis era has only the genesis state and a state index.

- **Verification** (`packages/era/src/verify.ts`):
  - Ensure `hash_tree_root(block) == state.block_roots[slot % SLOTS_PER_HISTORICAL_ROOT]` for each non-empty slot.
  - Apply continuity rule for empty slots (`root(slot) == root(slot-1)`).
  - Validate that all slot index offsets are in-bounds and point to the correct type.
  - Confirm era boundary alignment with `era * SLOTS_PER_HISTORICAL_ROOT`.
  - Anchor the era state against a finalized checkpoint root.

- **Integration with Lodestar sync** (`packages/beacon-node/src/sync/eraSource.ts`):
  - Add an `EraSource` implementing the existing block/state fetcher abstractions.
  - On `--checkpoint-era <file>` (new CLI option in `packages/cli/src/cmds/beacon/options.ts`), sync uses `EraSource` instead of network fetchers.
  - Provide programmatic APIs (e.g., get blocks by range, get state at slot) backed by `EraReader`.

- **CLI** (`packages/cli/src/cmds/era/`):
  - `lodestar era inspect <file>` → list groups, slot ranges, and indices.
  - `lodestar era verify <file>` → run verification checks, with `--checkpoint-root` option.
  - `lodestar era extract <file> --slot N (--block|--state)` → dump SSZ data at a given slot.
  - `lodestar era create --in … --out file.era` → build an era file and append indices.

## Work during the fellowship

Planned steps:
1. **Era IO** (`packages/era/src/`): Implement `EraReader`/`EraWriter` to parse headers, group blocks + states, and write trailing indices.
2. **Verification** (`packages/era/src/verify.ts`): Add utilities to validate internal consistency and run checkpoint anchoring.
3. **CLI integration** (`packages/cli/src/cmds/era/`): Add `lodestar era inspect|verify|extract` commands with JSON output.
4. **Sync integration** (`packages/beacon-node/src/sync/eraSource.ts`): Implement `EraSource` that feeds blocks/states to Lodestar’s sync pipeline.

### Zig-Side Contributions

In parallel to the ERA work, three EIP implementations were completed (EIP-7916, EIP-7495, EIP-8016) along with Fulu specifications for the State Transition zig library. Progressive types development passed specification tests across multiple categories including Progressive List, Progressive Bitlist, and Progressive Containers.

## Roadmap

1. **Week 8–10** – Implement reading and writing of `.era` files.
2. **Week 11–14** – Implement group parsing and slot index support; integrate sync support via `EraSource`.
3. **Week 15** – Build CLI tools for creating, inspecting, and validating era files.
4. **Week 16+** – Performance tuning and testing with Nimbus `.era` files.

### Deliverables

- A Lodestar module (`packages/era`) for `.e2s`/`.era` parsing and verification.
- Utilities for inspecting and validating era files from CLI (`packages/cli/src/cmds/era/`).
- Hooks in Lodestar sync (`beacon-node/src/sync/`) to use `.era` files as data sources.

## Possible challenges 

- **Performance tuning**: `.era` files can be hundreds of megabytes, so Node.js memory handling is a bottleneck. Mitigations:  
  - Lazy load only headers and indices up front.  
  - Use `SnappyFramesUncompress` (already in `reqresp/`) to stream decompression one entry at a time.   
  - **Future improvement**: there is an ongoing Zig SSZ library effort, which could be leveraged to offload SSZ encode/decode to a high-performance native backend, significantly reducing Lodestar’s CPU and memory overhead.
- Ensuring interoperability with Nimbus and other client outputs.

## Goal of the project

The main goal is to enable Lodestar to **consume and verify `.era` files **. Success would mean:
- Lodestar can parse, validate, and sync from `.era` files.
- Era files become a practical medium for checkpoint sync and historical lookups.
- Lodestar contributes to cross-client adoption of the era archival format.

This work will prepare Lodestar for Ethereum’s history expiry roadmap.

## Collaborators

### Fellows
- [Rahul Guha](https://github.com/guha-rahul)

### Mentors
- [Caymen](https://github.com/wemeetagain)

## Resources
- [ Lodestar Era Issue tracker](https://github.com/ChainSafe/lodestar/issues/7048)  
- [EthResearch: Era archival files for block and consensus data](https://ethresear.ch/t/era-archival-files-for-block-and-consensus-data/13526)
- [Geth docs: Downloader](https://geth.ethereum.org/docs/fundamentals/downloadera)
- [Nimbus Guide: Era store](https://nimbus.guide/era-store.html)
- [HackMD: Era archival notes](https://hackmd.io/@advaita/BkMvD9Qllg)
- [EtherWorld: History expiry moves forward in Ethereum’s Fusaka upgrade](https://etherworld.co/2025/06/16/history-expiry-moves-forward-in-ethereums-fusaka-upgrade/)
