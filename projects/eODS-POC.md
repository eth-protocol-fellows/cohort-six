# eODS proof of concept in client implementation
This projects aims to achieve a proof of concept implementation of eODS in Lodestar.

## Motivation
Delegating is a discussed topic, especially in a BeamChain future - see Rainbow Staking. 
A hypothetical model has been put forth by Dan Goron a year ago, and since it evolved into something more practical.
I was part of those specs writing and [design](https://github.com/gorondan/consensus-specs/commits/eods/?author=vlladin) so I want to take it further. I believe delegations, in one form or another, are an integral part of the future.
Having reached to various people and listening to their feedback, I think it's important to have a POC client implementation. Benefits are many, first of all it will allow us to weed out all design flaws and polish the specs even more. Then, it will allow us to understand the effort required to actually implement something like this in the existing clients. Then it will help us understand how big an impact it will have on various metrics of the system.

## Project description
This project is about writing spec tests for eODS and writing a proof of concept implementation in Lodestar.

### Tests
Tests will help us identify and weed out all the small things we missed in the specs.

### Proof of concept implementation
The proof of concept client implementation will allow us to understand the effort required to actually implement something like this in the existing clients. It will also help us understand how big an impact eODS will have on various metrics of the system.

## Specification

Tests will be handled in the Consensus Specs repo, and will follow the existing formats/rules. Not much to add on that topic, other than the fact that I will try to cover as many edge cases as possible - and focus my efforts in that area especially.

eODS can be divided up in several chunks. We call them actions.

A delegator can:
- **deposit** Gwei from EL to CL and store it in an undelegated balance
- **delegate** some (or all) of the funds to a Validator (that accepts delegations)
- **undelegate** some delegated amount from one or many Validators (provided it delegated to them prior to this action)
- **redelegate** some amount from Validator A to Validator B
- **withdraw** Gwei from CL to EL, from its undelegated balance to its wallet

A node operator that has a Validator can:
- set up his validator as an Operator, and thus start accepting delegations 

The execution layer will get a new contract, called a Delegations Operations Contract. We base this contract on the Deposit Contract, so it should not be anything too alien. What is new is the fact that this contract supports all of the actions above. An Execution Address can call this contract with a set of predefined parameters, choose an action, and have it triggered on the Consensus Layer.

An example of those parameters:

```python
class DelegationOperationRequest(Container):
    type: Bytes1
    source_pubkey: BLSPubkey
    target_pubkey: BLSPubkey
    amount: Gwei
    execution_address: ExecutionAddress
```

This requests will travel from EL to CL, on the same bus. This means that inside the CL we have an unbundling method that reads the `type` for each `DelegationOperationRequest` and sends the request down its own pipeline based on the `type`.

Inside the CL, each of these requests are accumulated in a queue inside the state, and they get consumed from there.

The BeaconState mutates like this:

```python
class BeaconState(Container):
    # ... the Electra BeaconState
    
    # Delegators, their balances, and the Delegated Validators list
    delegators: List[Delegator, DELEGATOR_REGISTRY_LIMIT] 
    delegators_balances: List[Gwei, DELEGATOR_REGISTRY_LIMIT] 
    delegated_validators: List[DelegatedValidator, VALIDATOR_REGISTRY_LIMIT] 

    # Accumulators for delegation operations. Requests have been transformed from DelegationOperationRequest and queued here.
    pending_operator_activations: List[PendingActivateOperator, PENDING_DELEGATION_OPERATIONS_LIMIT]  
    pending_deposits_to_delegate: List[PendingDepositToDelegate, PENDING_DELEGATION_OPERATIONS_LIMIT] 
    pending_delegations: List[PendingDelegateRequest, PENDING_DELEGATION_OPERATIONS_LIMIT] 
    pending_undelegations: List[PendingUndelegateRequest, PENDING_DELEGATION_OPERATIONS_LIMIT] 
    pending_redelegations: List[PendingRedelegateRequest, PENDING_DELEGATION_OPERATIONS_LIMIT] 
    pending_withdrawals_from_delegators: List[PendingWithdrawFromDelegatorRequest, PENDING_DELEGATION_OPERATIONS_LIMIT]  

```

A `DelegatedValidator` is a wrapper around a `Validator`. eODS does not modify the internal logic of existing Validators, only wraps around them (if needed) the extra logic of delegations.

It looks something like this:
```python
class DelegatedValidator(Container):
    validator: Validator
    delegated_validator_quota: uint64
    delegators_quotas: List[Quota, DELEGATOR_REGISTRY_LIMIT]
    delegated_balances: List[Gwei, DELEGATOR_REGISTRY_LIMIT]
    total_delegated_balance: Gwei
    fee_quotient: uint64
```

The idea here (very nutshell'ish) is to allow the Operator (Validator that accepts delegations) to have its own internal balance - like normal Validators do now, and add on top of that the delegated balances from the delegators that picked this Operator as a delegation target. We introduce a quota system that will be used to calculate the rewards/penalties for the Operator and for each delegated amount, and to calculate the slashing amounts for each of the above.

We also add a widget we called Beacon Chain Accounting, which is basically a set of methods that will be used to keep track of the delegated amounts and will handle the applying of penalties/rewards/slashings. Some of the methods to:
- reward/penalize/slash the exit queue
- apply the delegations
- apply the undelegations
- recalculate the quotas inside the Delegated Validators
- increase/decrease balances

#### Delegation actions
Each delegation action (the ones listed above) has its own execution timing and pipeline inside the Consensus Layer. Some of them need to respect certain churns and time limitations - eg.: delegating and undelegating affects the financial security of the protocol so we must think about churning them, we must take WS into considerations. Others like depositing are more relaxed.

The most complex execution pipeline is handling the undelegations. For that we created a delegation exit queue where we queue:
```python
class UndelegationExit(Container):
    amount: Gwei
    total_amount_at_exit: Gwei
    execution_address: ExecutionAddress
    validator_pubkey: BLSPubkey
    exit_queue_epoch: Epoch
    withdrawable_epoch: Epoch
    is_redelegation: boolean
    redelegate_to_validator_pubkey: BLSPubkey
```

This is necessary because we have a period of M epochs in which the amount undelegated must remain slashable - between the delegation exit epoch and the delegation withdrawability epoch. So we keep that amount in a queue, and only return the funds to e delegator once WS has been cleared.

So I must take each one of these execution pipelines and implement them in Lodestar. The Execution Layer component must be handled also to achieve a functional dev net with eODS, at this state I have no plans for that.

## Roadmap

There are about 4 months available on the writing of this md. In the first 4-5 weeks I have pushed to have the eODS specs finalised so I can focus on implementation moving forward. I also explored a bit Lodestar's codebase, as much as was possible in the given time and priorities.

I will take 12 weeks for the roadmap, with a few left for buffers: 

### Testing
`weeks 1-2`

I will cover eODS specs with tests, trying to cover all the edge cases and weed out most of the (big) issues. I don't expect eODS specs to be in a perfect form right now, but these tests and the required patches will get us close to that. 

*Deliverables: Written tests and patches for eODS.*

### Lodestar 
`weeks 3-4`

I need at least a couple of weeks to **really** get into Lodestar. Fixing bugs and interacting with the community will help me do that.

*Deliverables: I expect to fix some bugs and PR those fixes into Lodestar's repo.*

### Proof of concept
`weeks 5-6-7-8-9-10`

This is where the proof of concept implementation takes place.
Since it's a big timebox, let's divide it further:
- `weeks 5-6` BeaconState changes and unbundling of the Delegation Operations Contract data bus
- `week 7 `DepositToDelegate, ActivateOperator, Delegate 
- `week 8` Undelegate / Redelegate
- `week 9` WithdrawFromDelegator
- `week 10 `Handling the intersections between eODS and existing features: Rewards / Penalties / Slashing 

*Deliverables: A proof of concept consensus client implementation in Lodestar*

### Execution Layer
I've given myself a few weeks of buffer, because things happen.
Best case scenario I will use those weeks to write the EL contracts needed for eODS.

I will update this section later, as work progresses and things become more clear to me.

## Possible challenges

I have no prior experience with Lodestar. I am very fluent in TypeScript, I have a very strong background as a developer. Given all this, I don't yet know how much I don't know about the client and about the rest of the moving parts. I have yet not touched the p2p stuff, all the engine api's and other forms of communication that *might* be needed for a functioning implementation of eODS.

The learning curve getting into the ecosystem is quite steep, and I have to basically tackle all existing curves to get to the end of the project with a functional dev net.

As of today, and from my prior experience with Ethereum, I can say I do not expect to have a FULL implementation of eODS in the given timeframe, but I will definitely try, and get there at some point.

## Goal of the project

I think this project has a few different definitions for success. 

The end goal of the project (in the current timeframe) is to have the spec tested, all the execution pipelines and changes implemented in the Consensus Layer in Lodestar.

The end-end goal is to bring in the EL and all the missing pieces so that there can be a dev net spawned with eODS functional.

Because we want to see what would it take for something like this to be implemented in the existing clients, another goal of the project is to have a post that details exactly what went good/bad during implementation, and describe the pain points if any.

## Collaborators

### Fellows 

Nobody offered help yet

### Mentors

No mentors yet

## Resources
So far:

[eODS consensus specs](https://github.com/gorondan/consensus-specs/tree/eods)

[POC Lodestar repo](https://github.com/vlladin/lodestar)

[eODS (Enshrined Operator Delegator Separation): a Delegation model proposal](https://ethresear.ch/t/eods-enshrined-operator-delegator-separation-a-delegation-model-proposal/22826)

[eODS from a developer's point of view](https://hackmd.io/Sw48H9qMQ0ukWZ08egkpkA)

