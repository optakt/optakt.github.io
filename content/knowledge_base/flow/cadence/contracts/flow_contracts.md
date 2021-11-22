# Flow Contracts

There are core Cadence contracts that are deployed on the Flow network as soon as it is bootstrapped.
These contracts are essential to the normal operation of the Flow network.

## FungibleToken

The `Flow Fungible Token` standard is defined in [0xf233dcee88fe0abe](https://flowscan.org/contract/A.f233dcee88fe0abe.FungibleToken) for the mainnet.
It defines the `FungibleToken` contract interaface, to which all fungible token contracts need to conform to.

The `Vault` resource is also defined in the same contract, which is the resource that each account needs to have in storage in order to own tokens.

Resources and interfaces defined here allow sending and receiving of tokens peer-to-peer, by withdrawing tokens from one users `Vault` and depositing them to another user's `Vault.`

## FlowToken

Implementation of the `FungibleToken` interface for the FLOW token.

## FlowServiceAccount

The Flow Service Account is an account like any other on Flow except it is responsible for managing core network operations. The Service Account can be referred to as the keeper of core network parameters which will be managed algorithmically over time but are currently set by a group of core contributors to ensure ease of updates to the network in this early stage of its development.

## FlowEpoch

Top-level smart contract that manages the lifecycle of [epochs](https://docs.onflow.org/staking/epoch-preparation/#epochs-overview).
Epochs are the smallest unit of time during which the set of network operators is either static or can decrease due to nodes election or malicious nodes presence.
Nodes may only leave or join the network at epoch boundaries.
The list of participating nodes is kept in the **Identity Table**.

Epochs have three phases &#0151 [staking](#staking-phase), [setup](#setup-phase) and [committed](#committed-phase).

### Staking Phase

During the [staking phase](https://docs.onflow.org/staking/epoch-preparation/#phase-0-staking-auction) (or staking Auction), nodes submit staking requests for the **next** epoch.
At the end of this phase, the identity table for the next epoch is determined.

### Setup Phase

During the [setup phase](https://docs.onflow.org/staking/epoch-preparation/#phase-1-epoch-setup), the participants are preparing for the next epoch.
Collection nodes submit votes for their cluster's root **quorum certificate** and consensus nodes run the [**distributed key generation** protocol](https://docs.onflow.org/staking/qc-dkg/#consensus-committee-distributed-key-generation-protocol-dkg) (DKG) to set up the random beacon.

### Committed Phase

When the [committed phase](https://docs.onflow.org/staking/epoch-preparation/#phase-2-epoch-committed) begins, the network is fully prepared to move to the next epoch.
Failure to enter this phase before transitioning to the next epoch would be a critical failure and would cause the chain to halt.

## FlowFees

Contract that manages the fee `Vault` and deposits and withdrawal of fees to and from the fee vault.

## FlowStorageFees

Storage capacity of an account determines how much storage on chain it can use.
There is a set of minimum amount of tokens reserved for storage capacity that is paid during account creation, by the creator.
If any transaction that results in account's storage use greater that the storage capacity, the transaction will fail.

## FlowClusterQC

This contract manages the process of collecting votes for the root [Quorum Certificate (QC)](./../../../glossary.md#quorum_certificate) of the upcoming epoch for all Collection node clusters assigned for the next epoch.
During the setup phase of the epoch, collection nodes in a specific cluster generate a root block for the cluster.
The nodes then need to submit a vote for the root block.
Once enough cluster nodes have voted with the same unique vote, the cluster is considered complete.
Once all clusters are complete, the QC is complete.

## FlowDKG

This contract manages the process of generating a group key with the participation of all consensus nodes for the upcoming epoch (DKG).

## FlowIDTableStaking

The Flow ID Table and Staking contract manages the node operators' and delegators' information and Flow tokens that are staked as part of the protocol.
Nodes submit their stakes during the [staking phase](#staking-phase) of the epoch.
When staking, nodes receive an object they can use to stake, unstake and withdraw rewards.
Tokens are held in multiple token buckets depending on their status — staked, unstaking, unstaked or rewarded.

This is also where the `Delegator` functionality, which manages delegation of FLOW to node operators, is specified.

## FlowStakingCollection

This contract defines a collection for staking and delegating objects which allows users to stake and delegate to as many nodes as they want.

## LockedTokens

This contract implements the functionality required to manage FLOW buyers locked tokens from the token sale.

Each token holder gets two accounts.
The first account is the locked token account, jointly controlled by the user and the token administrator.
Token administrator cannot interact with the account without approval from the token holder except for depositing additional tokens, or to unlock existing tokens at the appropriate milestones.

Second account is the unlocked user account, which is in full possession of the user.
This account stores a capability which allows the account owner to withdraw tokens when they become unlocked, and also to perform staking operations with the locked tokens.

## StakingProxy

This contract defines an interface for node stakers to use to be able to perform common staking actions.

## Contract Addresses

* FungibleToken
    * Mainnet: [0xf233dcee88fe0abe](https://flowscan.org/contract/A.f233dcee88fe0abe.FungibleToken)
* NonFungibleToken
    * Mainnet: [0x1d7e57aa55817448](https://flowscan.org/contract/A.1d7e57aa55817448.NonFungibleToken)
* DapperUtilityCoin
    * Mainnet: [0xead892083b3e2c6c](https://flowscan.org/contract/A.ead892083b3e2c6c.DapperUtilityCoin)
