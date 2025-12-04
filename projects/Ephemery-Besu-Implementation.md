# Ephemery Testnet Implementation on Besu

Enables an automatically reset public testnet embedded on Besu

## Motivation

By the implementation of this proposal, [the Besu execution client](https://github.com/hyperledger/besu) is going to be the only execution client with automated reset functionality within the node. This way, infrastructure providers don't need to run any external tooling.

This project is mainly going to implement the automatically restart feature for [the Ephemery](https://eips.ethereum.org/EIPS/eip-6916) public testnet natively inside the Besu client. [EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) (Ephemery) is introducing an ephemeral testnet that automatically resets and can provide an alternative environment for short-term testing of applications, validators, and also breaking changes in client implementations. Implementing EIP-6916 natively inside Ethereum clients such as Besu avoids issues of long-running testnets, which suffer from state bloat, lack of testnet funds, or consensus issues. Periodically resetting the network back to genesis cleans the validator set and returns funds back to faucets while keeping the network reasonably small for easy bootstrapping. This way a wider range of options would be available for users to examine their applications.


## Project description

This project aims to complete the remaining implementation of EIP-6916 in the [Besu client](https://github.com/hyperledger/besu), focusing specifically on the reset feature on the database, network, and peer discovery. At the end, if done with Besu sooner than planned, I will research the feasibility of the same features on [the Teku consensus client](https://github.com/Consensys/teku).


## Specification

The overall specification is defined in the EIP-6916. Besides that, the implementation on the client should be tailored to Besu's architecture. In the following, considering what's been done so far, the steps required to get to final development are pointed out. 

What is available for now:

When the network passed to the command is Ephemery, it reads from Ephemery genesis0 and changes the networkId based on the timestamp. [Link to the code](https://github.com/hyperledger/besu/blob/6f5f22372625cd2852dea6cabc144e9bb0b02cf7/app/src/main/java/org/hyperledger/besu/util/EphemeryGenesisUpdater.java#L32)


#### Restart network automatically

- When the 28-day cycle ends, the network should read from new genesis.

- Research on how to restart Besu. The `updateNetworkConfig(final NetworkName network)` should be invoked.


#### Reset database implementation

- DB reset detection: This is when the current timestamp passes the genesis timestamp plus the period. It can be invoked when the network is starting. In some cases while syncing, when an internet connection arises, it resumes the previously synced data. So for the sake of backward compatibility, it's better to be based on the network; in this case, Ephemery.

- DB cleaning: The database should be refreshed after each genesis update. Besu is using RocksDB; the implementation of cleaning the database needs adding functionality inside the `RocksDBKeyValueStorageFactory` class inside the rocksdb module. 

- Refresh genesis state: after each network iteration, the `GenesisState` class inside the ethereum/core module should be updated with the new genesis state for a dynamic reset network. 


#### Reset peers implementation

- Clear peer table data: remove old peer data by invoking the `dropPeer(final PeerId peer)` method in the `PeerDiscoveryController` class. This way it will clear the table by calling the `tryEvict(final PeerId peer)` method inside the `PeerTable` class.

- Rediscover new peers: Rediscover new peers by starting over the `start()` method inside the `PeerDiscoveryController` class via the `PeerDiscoveryAgent` class. 

- Add new peers: invoking the `addToPeerTable(final DiscoveryPeer peer)` function inside the `PeerDiscoveryController` class will add new peers.


#### Enable the Besu client itself to become a bootnode. 

- Research on making sure Besu can be run as a bootnode on Ephemery:

 From my mentor:

A bootnode can be a regular node that runs discv5/discv4 and helps new clients to connect and discover peers. Some clients, like Lighthouse, have a special mode for running bootnodes; check if Besu/Teku have that. But in terms of Ephemery, you need to make sure it behaves like a bootnode during the reset—it correctly switches to the new chain by itself, keeps the same enode ID, and serves the right peers.

- When a reset happens, the node needs to maintain its public key, drop old peers, and rediscover peers on the new network.


#### Testing, logs, and considerations

- Comprehensive unit tests for every change and added feature are needed.

- Integration tests will be added if it's necessary to prove that modules are running as requested.

- Acceptance tests, which are more specific to Besu, will be added if there is a need for it.

- All tests (newly added or old ones) should be run successfully; no breaking tests.

- The result on every development should be backward and forward compatible. 

- Check for concurrent access when it's possible to happen. 

- Emit logs and publish events when needed. 


## Roadmap

I hope the implementation of this proposal takes me less than 22 weeks, so I will have time to work on more features.


#### Week 4

- Check into the Besu codebase structure. &check;


#### Week 5

- Review previous Ephemery-related PRs and discussions by other fellows from previous cohorts: Glory from cohort5 and Teri and Holly from cohort4. &check;


#### Weeks 6 and 7 

- Deep dive into the Besu codebase to come up with a proper plan on implementation for the proposal. &check;

- Run the Ephemery testnet on Besu to examine what features are available and what needs to be developed in order to add new functionality like automatic DB reset after each network refresh. &check;

- Presentation on my project proposal at office-hour calls. &check;

- Review and rewrite the project proposal based on my mentor's feedback. &check;

- Write up about how to run Ephemery on Besu and Teku as an amateur user. &check;


#### Weeks 8 and 9

- Research on the 'restart network' task. &check;

- Implement the task. &check;


#### Weeks 10 and 11

- Research on the 'reset DB' task. &check;

- Implement the task. &check;


#### Weeks 12 and 13

- Write tests for 'restart network' and 'reset DB.' &check;

- Add any documentation that is needed for the tasks. &check;


#### Weeks 14 and 15

- Research on the 'reset peers' task. &check;

- Implement the task. &check;


#### Weeks 16 and 17

- Write test for 'reset peers' task. &check;

- Write related documents. &check;


#### Weeks 18 and 19

- Research on how to enable the Besu client itself to become a bootnode. &check;

- Write related documents. &check;


#### Weeks 20 and 21

- Implement the task. &check;

- Write related tests. &check;


#### Week 22

- Work on undone tasks that have been mentioned. &check;

- Research on Teku to see if the reset feature is possible to implement, if I have time.

- Prepare for the final presentation. &check;


## Possible challenges

The Reset feature will be active at the end of each 28-day cycle; therefore, for testing the reset functionality of the Ephemery network, which is a public testnet, you need to wait for almost a month. I try to figure out a way to build a private Ephemey sandbox to test it.

Based on a quick search of the code base and my intuition for the assignment, I included the names of classes and functions in the specification. More research on the codebase is essential, to be more precise.

The remaining tasks of this project have been tagged with good-first-issue. However, they may take longer than anticipated and come with some hurdles. I therefore allocated adequate time for the design and implementation step to tackle any issues that might come along. I expect to have the opportunity to work on another project if I finish the tasks before then. All in all, I think as a first-time contributor, I'm going to face unknown challenges.


## Goal of the project

- Implementing [EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) (Ephemery testnet) natively inside the Besu client.

- Restart the network automatically at the end of the Ephemery testnet cycle.

- Add database reset for the Ephemery network on restart.

- Restart peers discovery and discard old peers at each new Ephemery cycle.

- A Besu node, when running Ephemery, becomes a bootnode itself.

- Provide comprehensive tests on Besu for the implemented parts that I've been working on.

- Provide insightful documents to the features being implemented throughout the project.


## Collaborators

### Mentors

[Mário Havel](https://github.com/taxmeifyoucan)


## Resources

[Ethereum Java-based execution client: Besu](https://github.com/hyperledger/besu)

[Ephemery testnet: EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) 

