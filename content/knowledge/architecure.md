# 3. Flare Architecture

## 3.1 Avalanche Platform

Avalanche is an open-source platform for launching decentralized applications and enterprise blockchain deployments in
one interoperable, highly scalable ecosystem. Avalanche is the first decentralized smart contracts platform built for
the scale of global finance, with near-instant transaction finality. Ethereum's developers can quickly build on
Avalanche as Solidity works out-of-the-box.

A key difference between Avalanche and other decentralized networks is the consensus protocol. The Avalanche protocol
employs a novel approach to a consensus to achieve its strong safety guarantees, quick finality, and high-throughput
without compromising decentralization.

The purpose of Avalanche platform:

* creation of subnets and blockchains tied to the main Avalanche network for general and corporate purposes;
* creation and hosting of decentralized applications;
* creation of smart assets (for example, new financial derivatives and DeFi solutions).

### Subnets

Everything on Avalanche is a _subnet_, and every chain is part of a subnet. There is a special subnet called the _
Primary Network_, which validates Avalanche's built-in blockchains. Validators are required to be a member of the
Primary Network; all other subnets are optional. Each subnet can have multiple blockchains just like the primary
avalanche network. The Primary Network contains three build-in blockchains:

* Platform Chain or P-Chain (manages the subnets, coordinates all the validator nodes and the staking mechanism)
* Contract Chain or C-Chain (network for working with smart contracts)
* Exchange Chain or X-Chain (meant for creation, management and transaction of tokens on the network – based on DAG a
  unique kind of consensus model)

In total, the Avalanche ecosystem (as of the end of Decemner 2021) operates 13 subnets and 10 blockchains (three main
plus 7 minor ones: Dtehereum, Btehereum, C and X-chain PoW and others)

All the details regarding the operation of the platform, tutorials and tools are described in
the [official Avalanche website](https://docs.avax.network/build/tutorials/platform/create-a-subnet/.)

### Avalanche Consensus Protocol

Avalanche Platform reach a consensus by using Avalanche Consensus Protocol. It has three mechanisms that give structural
support to the network. These are the non-BFT protocol (Slush) that progressively built up to Snowflake and Snowball.
These are single-decree consensus protocols with increasing robustness and are all based on the common majority-based
metastable voting mechanism.

Avalanche is considered to be a sort of layered consensus protocol and can be broken down into:

* Slush
* Snowflake
* Snowball
* Avalanche

I'll keep this part brief and more detailed explanations can be found in the links provided at the end of this section.

To understand how the algorithm works, here is a simple example. We start with a _slush_. Let's say, we have 16 nodes
and the network has received a malicious transaction. Now these nodes should 'vote' whether this transaction is valid (
the sender has sufficient funds in this case). Another assumption is that several of these nodes are malicious (
I run them in the network to get my transaction through) or they are out-of-date (information they have regarding the
state of the network is a bit obsolete). To achieve an agreement regarding the transaction, each node picks several
other nodes randomly, and they have a vote. Each node can repeat this process several times to achieve a final
agreement. However, after each voting, the node can change its preferences depending on the decision made by the
majority. This can potentially make the process stuck at some point because the nodes can not achieve consensus. To
break this loop, _snowflake_ comes into play. Each node has a counter to keep track of the votes. Each time the node
votes with the same preference, counter increments. If the node changes its preference, the counter goes to zero and the
node starts the process from the beginning. It will do so until the number of consequence votes reaches some defined
threshold. This however can slow down the voting process, especially if the network has a big amount of malicious nodes.
To speed this up, _snowball_ upgrades snowflake's counters to 'confidence' counters. Two counters hold information about
the 'positive' and 'negative' votes. After some number of votes, the algorithm compares two counters and makes a
decision based on that. This speeds up the voting process and dramatically decreases the probability of the voting
process getting stuck. The final part is the Avalanche algorithm that generalizes the snowball and maintains the DAG (
direct acyclic graph). DAG is a critical part because it prevents double-spends. Because in DAG the transactions are
appended to one another, a vote for one transaction implicitly means a vote for all the transactions it is appended to.
When the transaction is being created, it holds the information about one or more parent transactions. Each node then
can check the transaction and all the ancestors and decide whether to accept or reject the transaction. In addition, the
node queries each transaction only once. A final decision is made if the transaction is present in the consequent
queries and has descendants (in case there are two conflicting transactions, one with more descendants will more likely
to be accepted, because it will appear in more queries than another).

A more detailed description regarding the Avalanche consensus algorithm can be found using the following links:

* [Official Avalanche Consensus page](https://docs.avax.network/learn/platform-overview/avalanche-consensus)
* [Step-by-step guide](https://devashishdxt.github.io/notebook/avalanche_consensus.html) of how consensus is reached.
* Some [examples](https://medium.com/@vardan.sevan/avalanche-examples-of-consensus-protocols-9ddc5b006264) how the
  consensus protocol is applied.

### Avalanche Subnet

A Subnet, or Subnetwork, is a dynamic set of validators working together to achieve consensus on the state of a set of
blockchains. Each blockchain is validated by exactly one Subnet. A Subnet can validate arbitrarily many blockchains. A
node may be a member of arbitrarily many Subnets and must be part of the primary network. Before any of the Subnets can
become a member of the network, it must stake some AVAX tokens. Clients interact with Avalanche through APIs calls to
nodes (each chain has its [own API](https://docs.avax.network/build/avalanchego-apis/README)).

AVAX tokens exist on the X-Chain, where they can be traded, on the P-Chain, where they can be provided as a stake when
validating the Primary Network, and on the C-Chain, where they can be used in smart contracts or to pay for gas. In this
tutorial, we’ll send AVAX tokens between the X-Chain and C-Chain.

#### P-Chain

The Platform Chain is the metadata blockchain on Avalanche and coordinates validators, keeps track of active subnets,
and enables the creation of new subnets. The Platform Chain implements the Snowman consensus protocol.

#### C-Chain

With the C-Chain API, clients can create smart contracts. It is an example of the Ethereum Virtual Machine that is
powered by the Avalanche Chain. The C-Chain also serves as the default smart contract blockchain of the Avalanche chain.
C-Chain is also very compatible with the Solidity smart contract as well as Ethereum tooling. Therefore, it makes it
easy for Ethereum developers to easily port applications into the Avalanche chain.

#### X-Chain

The X-Chain functions as a decentralized network for creating and trading digital assets. It is a representation of a
real-world resource such as bonds and equity with specific rules that govern their actions. Whenever you initiate a
transaction on the Avalanche blockchain, you get to pay a fee in AVAX. The Exchange Chain is an example of the Avalanche
Virtual Machine (AVM). Clients can create and trade assets on the X-Chain and other examples of the AVM with the help of
the X-Chain API.

## 3.2 Flare's FBA

TODO: add how Flare hijacks the codebase to run FBA on it.

## 3.3 State Connector

The State Connector – Flare’s system to observe the state of underlying chains. State Connector is one of the most
critical protocols on the Flare Network. The purpose of a state connector is to connect non-flare assets (like XRP,
Dode, LTC, etc.) with the Flare Network (represented by F-Assets) in a trustless manner, or other words, allow the smart
contracts on Flare to act on state changes from underlying blockchains. The State Connector adopts other blockchains’
consensus mechanisms and validator ecosystems.

### Advantages

A state connector system is a competitive approach for proving the state of an underlying chain to a smart contract, and
it has the following advantages:

* Transaction validity references back to an underlying chain's genesis block: Other approaches like the SPV (Simplified
  Payment Verification) proof does not check the validity of a transaction, because it is considered harmful for proving
  the state of an underlying chain to another network.
* Safety only depends on an underlying chain's validators: There is no trusted third-party service that has its own set
  of economic incentives and risks. Trust is minimized by leveraging the guarantee that safety can only be lost in the
  state connector if an underlying chain's validators encounter a Byzantine fault.
* No cooperation needed from an underlying chain's validators: Validators from an underlying chain do not need to modify
  their chain's codebase to permit Flare to interpret their network. An underlying chain's validators do not even need
  to be aware that Flare exists for the state connector system to operate.
* Can read the state of any blockchain: The state connector can operate on any possible Sybil-resistance technique of an
  underlying chain. For example proof-of-work, proof-of-stake, and even federated byzantine agreement where there is no
  global agreement on the set of validators in control of a network.
* No encoding of the current validators in control of an underlying chain to a smart contract on Flare: This requirement
  of other state-relay approaches such as the SPV proof leads to the hazardous scenario where the enforcement of bad
  behavior in relaying state needs to be conducted by the same set of operators that have performed the bad behavior.
* Constant-sized proofs: both the data availability proof and the payment proof are constant-sized, independent of the
  number of other payments in the data availability period being considered.
* Every Flare validator independently verifies an underlying chain's state: If your Flare validator observes the
  canonical state of an underlying chain, then you will not lose safety against that chain.

### Design

State Connector is built into the validators. These validators (or validation nodes) are tasked with ordering
transactions and confirming validity for consensus plus digesting state changes from integrated blockchains. Validators
on the Flare Network operate on the ecosystem as well as the integrated blockchains’ ecosystems.

The validation process consists of two stages:

* congestion control by agreement on data availability
* proving a transaction

#### 1. Congestion Control by Agreement on Data Availability

In this stage, validators agree on the state of the underlying chain. This process of tracking state change availability
occurs via a smart contract on the Ethereum Virtual Machine (EVM) level. The integrated blockchain validators can submit
a valid claim availability proof in a competition to be first but only allows them to do so again after the timeout
period has ended since the last valid claim availability proof. Additionally, a fee to submit the availability proofs is
required to prevent attacking and spamming. The timeout period, fee, and count of finalized blocks from the integrated
blockchains can all be changed via governance.

#### 2. Proving a Transaction

After enough validators agree on the state of the network (data availability proof), validators need to approve the
transaction. Validators will agree/disagree on the validity of the transaction using the Federated Byzantine Agreement
for the network to reach a consensus. Once these criteria are met and that data is available, then the stored
transactions in the storage contract are now available to be referenced by any other contract on the Flare Network.

## 3.4 Flare Time Series Oracle (FTSO)

The FTSO provides externally sourced data estimates to the Flare Network in a decentralized manner. It does so by
leveraging the distributed nature of the network and its participants. It generates rewards for active network
participants and holders of Spark (FLR) via Signal Providers and F-Asset holders.

The FTSO gets external data from Signal Providers (or Oracles) in the blockchain world. Signal Providers exist in the
off-chain to collect data, such as asset prices, and deliver it to the FTSO. Signal providers on the Flare Network are
responsible for delivering continuous and accurate data estimates to the FTSO to represent off-chain values on the Flare
Network. Signal providers can provide their F-Asset and FLR votes to do this, but they also can receive delegated votes
from other parties to perform this service on their behalf. Signal providers may take fees for this service.
Additionally, the entire process is noncustodial.

The FTSO ensures the data is fair and accurate and then makes the data available to network applications.

More details regarding how FTSO constructs the prices can be found
in [Constructing price distributions](https://flare.xyz/ftso-a-breakdown/) section.

## 3.5 F-Asset System

The F-Asset System is an integral application on the Flare Network. It facilitates the trustless issuance of asset value
onto the Flare Network. An F-Asset is a representation of an asset from another blockchain on the Flare Network via the
State Connector System, i.e. XRP = FXRP. Every F-Asset minted through the F-Asset System has a 2.5x collateral-backing
in FLR, which is denominated in USD. Agents are responsible for posting and maintaining the FLR collateral.
Additionally, they are responsible for answering redemptions, when users decide they would like to have their underlying
asset back. XRP, Litecoin, Dogecoin, and Stellar are the first four F-Assets and the addition of more is subject to FLR
holder governance.

F-Asset System allows the supported cryptos to have the capability to run smart contracts without any changes to the
underlying blockchain. The reason to do so is that nearly 75% of the value on public blockchains cannot be currently
used in a trust-less manner with smart contracts.

Two important parties in the F-Asset System are responsible for issuing F-Assets: Agents and Originators. An originator
is a person or an institution who wants to convert their cryptocurrency into an F-asset, and an agent (
more likely to be an institution) is a participant on the Flare network who will be issuing the F-asset desired by the
originator. Agents are also responsible for posting and managing the FLR collateral used to back the minting of
F-Assets. Additionally, as F-Asset holders have the right to claim the underlying asset, the agents will be responsible
for honoring redemptions promptly.

More detailed information about F-Asset System can be found on these resources:

* [The F-Asset System - Smart Contracts for Everyone](https://www.alphaoracle.io/flare-network/f-assets)
* [F-Asset System](https://www.thedefistandard.com/flare-network/f-assets)
