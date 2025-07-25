# Ephemery Testnet Implementation on Besu

Enables an automatically reset public testnet embedded on Besu



## Motivation

[EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) (Ephemery) is introducing an ephemeral testnet that automatically resets and can provide an alternative environment for short-term testing of applications, validators, and also breaking changes in client implementations. Implementing EIP-6916 natively inside Ethereum clients such as Besu avoids issues of long-running testnets, which suffer from state bloat, lack of testnet funds, or consensus issues. Periodically resetting the network back to genesis cleans the validator set and returns funds back to faucets while keeping the network reasonably small for easy bootstrapping. This way a wider range of options would be available for users to examine their applications.


## Project description



This project aims to complete the remaining implementation of EIP-6916 in the [Besu client](https://github.com/hyperledger/besu), focusing specifically on the database reset feature and Geth-style genesis parsing compatibility, along with enabling the bootnode command to read from a dynamic source.



## Specification


The overall specification is defined in the EIP-6916. Besides that, the implementation on the client should be tailored to Besu's architecture. In the following, the steps required to get to final development are pointed out. 


#### Reset database implementation

- DB reset detection: This is when the current timestamp passes the genesis timestamp plus the period. The `EphemeryGenesisUpdater` class inside the app module can be an option for it.


- DB cleaning: The database should be refreshed after each genesis update. Besu is using RocksDB; the implementation of cleaning the database needs adding functionality inside the `RocksDBKeyValueStorageFactory` class inside the rocksdb module. 



- Refresh genesis state: after each network iteration, the `GenesisState` class inside the ethereum/core module should be updated with the new genesis state for a dynamic reset network. 




#### Enable --bootnodes to read enode URLs from a file or remote URL


- Changing the logic of the --bootnode command to not just read from the command parameter but also read from a file. This will enable the Ephemery testnet implementation to dynamically download a file, such as [boot nodes,](https://ephemery.dev/latest/metadata/enodes.txt) from a fixed URL. The BesuCommand class should be extended for enabling this functionality.


- This way the bootnodes are going to be set dynamically, so peer-to-peer network logic might need to be changed throughout the codebase. 


#### Geth-style genesis parsing


- The genesis file in the Geth client, or in general in other genesis files, such as Ehemery-genesis, has some more or less parameters that need another parsing method to be valid.


- This feature can be developed via adding a layer to the map and distinguishing between a Besu config style and others by adding the GenesisAdapter class. 


- It's better to be written as generally as possible to be compatible with other clients (not just Geth) and also forward and backward compatible.



#### Testing 


- Comprehensive unit tests for every change and added feature are needed.


- Integration tests will be added if it's necessary to prove that modules are running as requested.


- Acceptance tests, which are more specific to Besu, will be added if there is a need for it.



#### Consideration with each implementation on the client side 


- All tests (newly added or old ones) should be run successfully; no breaking tests.


- The result on every development should be backward and forward compatible. 


- Check for concurrent access when it's possible to happen. 


- Emit logs and publish events when needed. 



## Roadmap

It is probably going to take me the whole 11 weeks that it is supposed to be based on the EPF program schedule. I think I'm going to start working on the project right away (week 4), which gives me the opportunity to deal with exploring the Besu project more than just getting done with the tasks. That being said, the project will be 12 weeks long.



#### Week 4

- Check into Besu codebase structure


#### Week 5

- Review previous Ephemery-related PRs and discussions by other fellows from previous cohorts: Glory from cohort5 and Teri and Holly from cohort4.



#### Week 6 

- Deep dive into the Besu codebase to come up with a proper design for implementing database restart. 


- Run the Ephemery testnet on Besu to examine what features are available and what needs to be developed in order to add new functionality like automatic DB reset after each network refresh.


- presentation on my project proposal at office-hour calls.



#### Weeks 7 and 8

- Start implementing the reset DB feature after finalizing the design for it.


- writing tests for resetDB.




#### Weeks 9 and 10

- Plan and implement to enable the bootnode command to read from a file or URL.

- Writing related tests.




#### Weeks 11 and 12 

- Start working on an idea to parse the genesis file in a more general way so that it is in compliance with other client genesis.

- Writing related tests.



#### Week 13 

- writing technical documentation and also docs that are tailored to end users’s concerns; helping users easily interact with the Ephemery testnet.




## Possible challenges



The remaining tasks of this project have been tagged with good-first-issue. However, they may take longer than anticipated and come with some hurdles. I therefore allocated adequate time for the design and implementation step to tackle any issues that might come along. I expect to have the opportunity to work on another project if I finish the tasks before then. All in all, I think as a first-time contributor, I'm going to face unknown challenges.





## Goal of the project

- Implementing [EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) (Ephemery testnet) natively inside the Besu client.

- Add database reset for the Ephemery network on restart.

- Allowing Besu to parse a Geth-style genesis like other clients.

- Allow --bootnodes to read from a file on the Besu client.

- Provide comprehensive tests on Besu for the implemented parts that I've been working on.

- Provide insightful documents to the features being implemented throughout the project.




## Collaborators

### Fellows

### Mentors

[Mário Havel](https://github.com/taxmeifyoucan)

## Resources

[Ethereum Java-based execution client: Besu](https://github.com/hyperledger/besu)




[Ephemery testnet: EIP-6916](https://eips.ethereum.org/EIPS/eip-6916) 




