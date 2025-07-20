# Grandine: zkVMs for Beacon Chain Snarkification

## Motivation

[Description from Grandine team](https://github.com/eth-protocol-fellows/cohort-six/blob/master/projects/project-ideas.md#grandine-zkvms-for-beacon-chain-snarkification).

Grandine team are experimenting with zkVM to enable SNARK-based proving of Beacon's Chain state transition functions.

This project will help advancing on chain snarkification, and eventually zk-proving block in real-time.

## Project description

Grandine team has some success on running SP1 and r0vm with Grandine during block production. Now, we want to run benchmark against other zkVMs, e.g. openVM, Zisk, Pico.

We also want to test out the scalability when running with numerous (100+) validators.

## Roadmap

- Study on Ream implementation of zkVM integration (Jul 27)
- Study [Ere – Unified zkVM Interface & Toolkit](https://github.com/eth-act/ere) (Jul 27)
- Check [grandine-zk repo](https://github.com/grandinetech/grandine-zk) (Jul 27)
- Integrate grandine-zk with [**OpenVM**](https://github.com/openvm-org/openvm), running benchmark for 2-3 validator nodes (Aug 10th)
- Scale up running zk-benchmarking with more validators (Aug 17th)
  - how to setup p2p network simulating this to 10 vals, 100 vals?
- Explore zkVM integeration with the following, focus on **two more zkVMs** for the rest of the cohort period:
  - [Polygon Zisk](https://github.com/0xPolygonHermez/zisk)
  - [Brevis Pico](https://github.com/brevis-network/pico)
  - [a16z Jolt](https://github.com/a16z/jolt)
  - [valida VM](https://github.com/lita-xyz/valida-vm)

## Possible challenges

- Understanding the API interactions between consensus engine and zkVM.
- Coding of the above in Grandine codebase.
- Getting the benchmark result of 100+ validators.
- Integrating multiple zkVMs into Grandine. I'm sure each zkVM has their own quirkiness.

## Goal of the project

- Completed zk-benckmark code of hooking OpenVM, and 2 more VMs with Grandine.
- Completed benchmarking of running Grandine with zk snarkification on block production with 100+ validators.

## Collaborators

- Cohort 6 fellows who are working on zkVM benchmarking subject, on both Ream and Grandine consensus engine.
- Both Ream and Grandine client teams

### Mentors

- [Saulius Grigaitis](https://discord.com/channels/945714351841607690/1253330175261806634), Grandine team (actively interact with him)
- [Unnawut](https://t.me/Kami_official0531), Ream team (learn from his work)
- [Jun Song](https://github.com/syjn99), cohort 6 fellow (learn from his work)

## Resources

Ream-related
- https://github.com/ReamLabs/consensp1us
- https://github.com/ReamLabs/consenzero-bench

zkVM interface & toolkit
- https://github.com/eth-act/ere
- https://github.com/syjn99/ere-ream

Grandine-related
- https://github.com/grandinetech/grandine-zk

zkVM
- [OpenVM](https://github.com/openvm-org/openvm)
- [Polygon Zisk](https://github.com/0xPolygonHermez/zisk)
- [Brevis Pico](https://github.com/brevis-network/pico)
- [a16z Jolt](https://github.com/a16z/jolt)
- [valida VM](https://github.com/lita-xyz/valida-vm)
