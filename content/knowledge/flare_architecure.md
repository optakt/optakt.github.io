# Flare Architecture

This section contains information about some parts of the Flare Network.
In addition, it also contains some information regarding the Avalanche platform, since Flare adopted some of its parts within its ecosystem.

## Avalanche Platform

Avalanche is an open-source platform for launching decentralized applications and enterprise blockchain deployments in one interoperable, highly scalable ecosystem.
Avalanche is the first decentralized smart contracts platform built for the scale of global finance, with near-instant transaction finality.
Ethereum's developers can quickly build on Avalanche as Solidity works out-of-the-box.

A key difference between Avalanche and other decentralized networks is the consensus protocol.
The Avalanche protocol employs a novel approach to a consensus to achieve its strong safety guarantees, quick finality, and high throughput without compromising decentralization.

The purpose of the Avalanche platform:

* creation of subnets and blockchains tied to the main Avalanche network for general and corporate purposes;
* creation and hosting of decentralized applications;
* creation of smart assets (for example, new financial derivatives and DeFi solutions).

### Subnets

Everything on Avalanche is a _subnet_, and every chain is part of a subnet.
A Subnet, or Subnetwork, is a dynamic set of validators working together to achieve consensus on the state of a set of blockchains.
Each blockchain is validated by exactly one Subnet.
A Subnet can validate arbitrarily many blockchains.
Before any of the Subnets can become a member of the network, it must stake some AVAX tokens.
Clients interact with Avalanche through API calls to nodes (each chain has its [own API](https://docs.avax.network/build/avalanchego-apis/README)).

There is a special subnet called the _Primary Network_, which validates Avalanche's built-in blockchains.
A node may be a member of arbitrarily many Subnets and must be part of the primary network.
The Primary Network contains three build-in blockchains: P-Chain, C-Chain, and X-Chain.

In total, the Avalanche ecosystem (as of the end of December 2021) operates 13 subnets and 10 blockchains (three main plus 7 minor ones: Dtehereum, Btehereum, C and X-chain PoW, and others).

All the details regarding the operation of the platform, tutorials, and tools are described in the [official Avalanche website](https://docs.avax.network/build/tutorials/platform/create-a-subnet/.)

#### P-Chain

The Platform Chain is the metadata blockchain on Avalanche and coordinates validators, keeps track of active subnets, and enables the creation of new subnets.
The P-Chain uses a UTXO (Unspent transaction output) model and implements the Snowman consensus protocol.
The reason P-Chain doesn't implement DAG is that linear chains give you a well-defined notion of time, but DAG does not.
This is necessary to add/remove a validator from the validator set at some given time.

#### C-Chain

With the C-Chain API, clients can create smart contracts.
It is an example of the Ethereum Virtual Machine that is powered by the Avalanche Chain.
The C-Chain also serves as the default smart contract blockchain of the Avalanche Chain.
C-Chain is also very compatible with the Solidity smart contract as well as Ethereum tooling.
Therefore, it makes it easy for Ethereum developers to easily port applications into the Avalanche chain.
C-Chain uses Snowman Consensus Protocol since an application requires a total ordering of transactions.
That is, if it needs to be able to compare any two transactions and determine which came first, it can't use a DAG because the ordering of some transactions is not defined.

#### X-Chain

The X-Chain functions as a decentralized network for creating and trading digital assets and uses the Avalanche consensus protocol.
It is a real-world representation of a resource such as bonds and equity with specific rules that govern their actions.
Whenever you initiate a transaction on the Avalanche blockchain, you get to pay a fee in AVAX.
The Exchange Chain is an example of the Avalanche Virtual Machine (AVM).
Clients can create and trade assets on the X-Chain and other examples of the AVM with the help of the X-Chain API.

## Flare's FBA

TODO: add how Flare hijacks the codebase to run FBA on it.

## State Connector

The State Connector – Flare’s system to observe the state of underlying chains.
State Connector is one of the most critical protocols on the Flare Network.
The purpose of a state connector is to connect non-flare assets (like XRP, Dode, LTC, etc.) with the Flare Network (represented by F-Assets) in a trustless manner, or in other words, allow the smart contracts on Flare to act on state changes from underlying blockchains.
The State Connector adopts other blockchains’ consensus mechanisms and validator ecosystems.

### Advantages

A [State Connector System](https://docs.flare.network/en/state-connector) is a competitive approach for proving the state of an underlying chain to a smart contract, and it has the following advantages:

* Transaction validity references back to an underlying chain's genesis block: Other approaches like the SPV (Simplified Payment Verification) proof do not check the validity of a transaction, because it is considered harmful for proving the state of an underlying chain to another network.
* Safety only depends on an underlying chain's validators: There is no trusted third-party service that has its own set of economic incentives and risks.
  Trust is minimized by leveraging the guarantee that safety can only be lost in the state connector if an underlying chain's validators encounter a Byzantine fault.
* No cooperation needed from an underlying chain's validators: Validators from an underlying chain do not need to modify their chain's codebase to permit Flare to interpret their network.
  An underlying chain's validators do not even need to be aware that Flare exists for the state connector system to operate.
* Can read the state of any blockchain: The state connector can operate on any possible Sybil-resistance technique of an underlying chain.
  For example proof-of-work, proof-of-stake, and even federated byzantine agreement where there is no global agreement on the set of validators in control of a network.
* No encoding of the current validators in control of an underlying chain to a smart contract on Flare: This requirement of other state-relay approaches such as the SPV proof leads to the hazardous scenario where the enforcement of bad behavior in relaying state needs to be conducted by the same set of operators that have performed the bad behavior.
* Constant-sized proofs: both the data availability proof and the payment proof are constant-sized, independent of the number of other payments in the data availability period being considered.
* Every Flare validator independently verifies an underlying chain's state: If your Flare validator observes the canonical state of an underlying chain, then you will not lose safety against that chain.

### Design

State Connector is built into the validators.
These validators (or validation nodes) are tasked with ordering transactions and confirming validity for consensus plus digesting state changes from integrated blockchains.
Validators on the Flare Network operate on the ecosystem as well as the integrated blockchains’ ecosystems.

### Voting

There are three phases of the State Connector voting protocol:
* Request
* Commit
* Reveal

#### Request Phase

At any point in time, any user can submit a request to the State Connector contract to have an event proven. 
The window in time that this request enters the network state is known as the request phase from its perspective.

#### Commit Phase

During the next window of time, attestation providers have the opportunity to commit a hidden vote regarding their belief in the outcome of the events requested in the previous phase. 
Anyone may operate as an attestation provider without any capital requirement, but a default incentivised set is used as the minimal requirement for passing a vote about the events in the previous set.

#### Reveal Phase

Finally, in the next window of time, attestation providers reveal their votes that they committed to in the previous round. 
Once this reveal phase concludes and the next phase begins, the revealed votes are automatically counted and all valid events become immediately available to all contracts on Flare.
Once these criteria are met and that data is available, then the stored transactions in the storage contract are now available to be referenced by any other contract on the Flare Network.

### Branching

The State Connector branching protocol protects Flare against incorrect interpretation of real-world events, proactively, such that there are never any rollbacks on the Flare blockchain state. 
Instead of having rollbacks, contention on state correctness is handled via automatic state branching into a correct and incorrect path. 
The security assumption is that if you as an independent node operator are following along with the correct real-world state, then you will always end up on the correct branch of the blockchain state.

## Flare Time Series Oracle (FTSO)

The FTSO provides externally sourced data estimates to the Flare Network in a decentralized manner.
It does so by leveraging the distributed nature of the network and its participants.
It generates rewards for active network participants and holders of Spark (FLR) via Signal Providers and F-Asset holders.

The FTSO gets external data from Signal Providers (or Oracles) in the blockchain world.
Signal Providers exist in the off-chain to collect data, such as asset prices, and deliver it to the FTSO.
Signal providers on the Flare Network are responsible for delivering continuous and accurate data estimates to the FTSO to represent off-chain values on the Flare Network.
Signal providers can provide their F-Asset and FLR votes to do this, but they also can receive delegated votes from other parties to perform this service on their behalf.
Signal providers may take fees for this service.
Additionally, the entire process is noncustodial.

The FTSO ensures the data is fair and accurate and then makes the data available to network applications.

More details regarding how FTSO constructs the prices can be found in [Constructing price distributions](https://flare.xyz/ftso-a-breakdown/) section.

## F-Asset System

The F-Asset System is an integral application on the Flare Network.
It facilitates the trustless issuance of asset value onto the Flare Network.
An F-Asset is a representation of an asset from another blockchain on the Flare Network via the State Connector System, i.e.XRP = FXRP.
Every F-Asset minted through the F-Asset System has a 2.5x collateral-backing in FLR, which is denominated in USD.
Agents are responsible for posting and maintaining the FLR collateral.
Additionally, they are responsible for answering redemptions, when users decide they want to have their underlying asset back.
XRP, Litecoin, Dogecoin, and Stellar are the first four F-Assets and the addition of more is subject to FLR holder governance.

F-Asset System gives the supported cryptos the capability to run smart contracts without any changes to the underlying blockchain.
The reason to do so is that nearly 75% of the value on public blockchains cannot be currently used in a trustless manner with smart contracts.

Two important parties in the F-Asset System are responsible for issuing F-Assets: Agents and Originators.
An originator is a person or an institution that wants to convert their cryptocurrency into an F-asset, and an agent (more likely to be an institution) is a participant on the Flare network that issues the F-asset desired by the originator.
Agents are also responsible for posting and managing the FLR collateral used to back the minting of F-Assets.
Additionally, as F-Asset holders have the right to claim the underlying asset, the agents are responsible for honoring redemptions promptly.

More detailed information about F-Asset System can be found on these external resources:

* [The F-Asset System - Smart Contracts for Everyone](https://www.alphaoracle.io/flare-network/f-assets)
* [F-Asset System](https://www.thedefistandard.com/flare-network/f-assets)
