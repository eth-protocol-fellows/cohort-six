# Implementation of Discovery Version 5 (DiscV5) in Besu

Upgrading Besu’s Peer Discovery: From DiscV4 to DiscV5 for Faster, Smarter, and Future-Proof Networking.

## Motivation

Besu currently uses Discovery v4 (DiscV4) for peer discovery, which lacks support for modern Ethereum networking features like Ethereum Node Records (ENR), topic-based discovery, and protocol advertisement.

Many Ethereum clients, such as Teku and Lighthouse, have already implemented DiscV5

By implementing DiscV5 in Besu, we can:

- Align Besu with Ethereum’s networking roadmap.

- Improve peer discovery security and efficiency.

- Support advanced features like topic-based searches and protocol-aware peer selection.

## Project description

This project focuses on transitioning Besu’s node discovery protocol from DiscV4 to DiscV5, improving performance, scalability, and interoperability with consensus-layer Ethereum clients. DiscV5 supports encrypted peer discovery and advanced topic registration, aligning Besu with modern Ethereum network requirements.

This will involve:
- Adding ENR based peer discovery
- Adding diagnostic JSON-RPC endpoints for testing and monitoring.
- Validating implementation against Hive test suites.

## Specification

1) Implementation Plan

    - Review Existing Imports: Besu already imports the Consensys/discovery Java libraries that implement DiscV5. Validate that the dependency is up-to-date and properly scoped in Besu’s build files.
    - Socket Layer: Update network bootstrap logic to bind Discovery UDP sockets to the DiscV5 protocol handler. Ensure the socket listens and transmits using DiscV5 packet types.

2) Endpoint to implementation 

    - discv5_recursiveFindnodes —  it keeps querying closer-and-closer nodes until it finds the closest peers to a target node ID (i.e., a recursive/iterative “find nodes” search). 

    - discv5_ping — sends a DiscV5 PING to a node and waits for PONG. It’s a health check that also returns basic info like the peer’s ENR sequence and the IP/port seen at the packet level. 

    - discv5_lookupEnr — asks the network to lookup an ENR (Ethereum Node Record) for a given node ID or key, so you can fetch that node’s advertised endpoints and metadata. (Often exposed by DiscV5/Portal tooling as an ENR resolver.) 

    - portal_historyRoutingTableInfo — returns routing-table details from the Portal Network’s History subnetwork (size, buckets, local radius, etc.), useful for debugging how your node sees and organizes peers in that DHT. 

3) Testing & Validation

    - Unit & Integration Tests: Write tests covering PING-PONG exchange, FIND_NEIGHBOURS requests, ENR parsing, and routing table correctness.

    - Network Simulation: Use testnets or local networks to verify peer discovery works under realistic conditions and with a variety of bootnode configurations.

## Roadmap

### 1) Research & Design (Weeks 1–2)
- Analyze Besu’s current DiscV4 architecture.
- Review ConsenSys Discovery repo for DiscV5 Java implementation.
- Define design: optional DiscV5 module, togglable via config; support dual discovery during transition.

### 2) Integration (Weeks 3–6)
- Add DiscV5 discovery module, leveraging ConsenSys Discovery Java.
- Integrate ENR encoding/decoding.
- Implement route fallback: DiscV4 → DiscV5 smoothly.

### 3) Testing & Validation (Weeks 7–8)
- Unit tests (protocol handshake, ENR parsing).
- Interoperability testing with Teku, other DiscV5 nodes.
- Performance benchmarks versus DiscV4.

### 4) Documentation & Community Feedback (Weeks 9–10)
- Add config docs, migration guide, examples.
- Collect feedback from core teams and community.

## Possible challenges

Migrating Besu from DiscV4 to DiscV5 may face challenges like ensuring backward compatibility with existing networks, achieving interoperability with other clients, and smoothly integrating the new library into Besu’s networking stack. Testing real-world peer discovery behavior and maintaining performance while adding encryption can be complex. Clear documentation and careful rollout will be essential to avoid misconfiguration and adoption issues.

## Goal of the project

The goal is to make Besu work with Discovery v5 so it can easily find and connect to other Ethereum nodes that use the new system, while still working with older nodes. The project will be successful when Besu connects reliably, performs well in tests, and is easy for node operators to set up. This upgrade will make peer discovery faster, more secure, and ready for future Ethereum needs.

## Mentors

[Sally MacFarlane](https://github.com/macfarla)

## Resources

- Issue Detail: https://github.com/hyperledger/besu/issues/4089

- DiscV5 Specification: https://github.com/ethereum/devp2p/blob/master/discv5/discv5.md

- ENR Specification: EIP-778 https://eips.ethereum.org/EIPS/eip-778

- Consensys Discovery Library: https://github.com/Consensys/discovery

- Samba Implementation: https://github.com/meldsun0/samba

- Hive Test Guide: https://hackmd.io/@3juAdBVCRtaXnRB_valWsA/HkB2wU5ckx

- Teku Networking: https://github.com/ConsenSys/teku