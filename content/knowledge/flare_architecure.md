# Flare Architecture

This section contains information about some parts of the Flare Network.
In addition, it also contains some information regarding the Avalanche platform, since Flare adopted some of its parts within its ecosystem.

## Avalanche Platform

Avalanche is an open-source platform for launching decentralized applications and enterprise blockchain deployments in one interoperable, highly scalable ecosystem.
It's a platform with a new approach to scaling that allows for fast finality of transactions and adopted to run Solidity out-of-the-box.

A key difference between Avalanche and other decentralized networks is the consensus protocol.
The Avalanche protocol employs a novel approach to a consensus to achieve its strong safety guarantees, quick finality, and high throughput without compromising decentralization.

The purpose of the Avalanche platform:

* creation of subnets and blockchains tied to the main Avalanche network for general and corporate purposes;
* creation and hosting of decentralized applications;
* creation of smart assets (for example, new financial derivatives and DeFi solutions).

### Subnets

Everything on Avalanche is a _subnet_, and every chain is part of a subnet.
A subnet, or subnetwork, is a dynamic set of validators working together to achieve consensus on the state of a set of blockchains.
Each blockchain is validated by exactly one subnet.
A subnet can validate many blockchains.
The validators of the subnet must stake AVAX tokens to become a member of the network.
Clients interact with Avalanche through API calls to nodes (each chain has its [own API](https://docs.avax.network/build/avalanchego-apis/README)).

There is a special subnet called the _primary network_, which validates Avalanche's built-in blockchains.
A node may be a member of arbitrarily many subnets and must be part of the primary network.
The primary network contains three build-in blockchains: P-Chain, C-Chain, and X-Chain.

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
C-Chain uses an account-based Snowman Consensus Protocol since an application requires a total ordering of transactions and runs consensus on blocks, not transactions.
That is, if it needs to be able to compare any two transactions and determine which came first, it can't use a DAG because the ordering of some transactions is not defined.

#### X-Chain

The X-Chain functions as a decentralized network for creating and trading digital assets and uses the Avalanche consensus protocol.
The X-Chain uses a DAG with the full Avalanche consensus protoco, so it is UTXO (Unspent Transaction Output) based.
UTXO-based systems are inherently more scalable than account-based systems.
They are naturally parallelizable whereas account-based systems must lock around the account to process transactions.
It is a real-world representation of a resource such as bonds and equity with specific rules that govern their actions.
Whenever you initiate a transaction on the Avalanche blockchain, you get to pay a fee in AVAX.
The Exchange Chain is an example of the Avalanche Virtual Machine (AVM).
Clients can create and trade assets on the X-Chain and other examples of the AVM with the help of the X-Chain API.

## Flare's FBA

TODO add more details to each if the items

Flare uses some parts of the Avalanche codebase with some changes:

* deactivate most of the P-Chain and X-Chain functionality;
* run the C-Chain with a locally defined set of validators instead of using the P-Chain for the list of validators;
* hijack the EVM state transition to implement Flare state connector functionality; and
* implement the rest of the Flare systems on top of the C-Chain as Solidity smart contracts.

## State Connector

The state connector – Flare’s system to observe the state of underlying chains.
State connector is one of the most critical protocols on the Flare Network.
The state connector enables smart contracts on Flare to act on state changes from underlying blockchains, for example payments.
The state connector adopts other blockchains’ consensus mechanisms and validator ecosystems.

### Advantages

A [state connector System](https://docs.flare.network/en/state-connector) is a competitive approach for proving the state of an underlying chain to a smart contract, and it has the following advantages:

* Transaction validity references back to an underlying chain's genesis block: Other approaches like the SPV (Simplified Payment Verification) proof do not check the validity of a transaction, because it is considered harmful for proving the state of an underlying chain to another network.
* Safety only depends on an underlying chain's validators: There is no trusted third-party service that has its own set of economic incentives and risks.
  Need for trust is minimized by leveraging the guarantee that safety can only be lost in the state connector if an underlying chain's validators encounter a Byzantine fault.
* No cooperation needed from an underlying chain's validators: Validators from an underlying chain do not need to modify their chain's codebase to permit Flare to interpret their network.
  An underlying chain's validators do not even need to be aware that Flare exists for the state connector system to operate.
* Can read the state of any blockchain: The state connector can operate on any possible Sybil-resistance technique of an underlying chain.
  For example proof-of-work, proof-of-stake, and even federated byzantine agreement where there is no global agreement on the set of validators in control of a network.
* Constant-sized proofs: both the data availability proof and the payment proof are constant-sized, independent of the number of other payments in the data availability period being considered.

### Design

The state connector is built into the validators.
These validators (or validation nodes) are tasked with ordering transactions and confirming validity for consensus plus digesting state changes from integrated blockchains.
Validators on the Flare Network operate on the ecosystem as well as the integrated blockchains’ ecosystems.

### Voting

There are three phases of the state connector voting protocol:

* Request
* Commit
* Reveal

#### Request Phase

At any point in time, any user can submit a request to the state connector contract to have an event proven.
The window in time that this request enters the network state is known as the request phase from its perspective.

#### Commit Phase

During the next window of time, attestation providers have the opportunity to commit a hidden vote regarding their belief in the outcome of the events requested in the previous phase.
Anyone may operate as an attestation provider without any capital requirement, but a default incentivised set is used as the minimal requirement for passing a vote about the events in the previous set.

#### Reveal Phase

Finally, in the next window of time, attestation providers reveal their votes that they committed to in the previous round.
Once this reveal phase concludes and the next phase begins, the revealed votes are automatically counted and all valid events become immediately available to all contracts on Flare.
Once these criteria are met and that data is available, the stored transactions in the storage contract are now available to be referenced by any other contract on the Flare Network.

### Branching

The state connector branching protocol protects Flare against incorrect interpretation of real-world events, proactively, such that there are never any rollbacks on the Flare blockchain state.
Instead of having rollbacks, contention on state correctness is handled via automatic state branching into a correct and incorrect path.
The security assumption is that if you as an independent node operator are following along with the correct real-world state, then you will always end up on the correct branch of the blockchain state.

## Flare Time Series Oracle (FTSO)

The [FTSO](https://flare.xyz/ftso-a-breakdown/) provides externally sourced data estimates (e.g. the current price of XRP) to the Flare Network in a decentralized manner.
It does so by leveraging the distributed nature of the network and its participants.
At the moment, only Spark holders get to vote; in the future, each asset's voting power will be split 50-50 between Spark holders and the holders of the particular asset.

Signal providers exist in the off-chain to collect data, such as asset prices, and deliver it to the FTSO.
As an example, a provider may collect the price of XRP/$ from three different exchanges, average the price and have this price estimate ready for the FTSO every voting round to submit.
Signal providers on the Flare Network are responsible for delivering continuous and accurate data estimates to the FTSO to represent off-chain values on the Flare Network.
Signal providers can provide their F-Asset and FLR votes to do this, but they also can receive delegated votes from other parties to perform this service on their behalf.
Signal providers may take fees for this service.
Additionally, the entire process is noncustodial (i.e. participants retain full control of their own data).
The FTSO ensures the data is fair and accurate and makes it available to network applications.

Additional info and some implementation details are available in [this example of price provider](https://github.com/flare-foundation/FTSO-price-provider).

### Delegation

Anyone can submit price data to the FTSO and vote for their price estimate.
This may require some level to effort and technical skills to build such a provider.
Most Spark and F-Asset holders can delegate their tokens to an FTSO provider who they trust will provide accurate price estimates.
The delegation will be as simple as logging into a Flare wallet, choosing an FTSO provider, and finally selecting the amount tokens they wish to delegate.

### Rewards

Signal providers receive a reward for accurately voting on prices, as determined by the algorithm.

The top and bottom 25% of prices are discarded, and the average of the remaining is used.
The 50% of price providers who are closest to the price than receive rewards for their own tokens plus the percentage of fees, as defined by them on the smart contract.
The default is 20%, and they can change it in 5% increments/decrements.

The exact amount of compensation is  based on the number of delegated tokens minus a fee set by the FTSO provider.

For example, Alice, Bob and Charlie choose to delegate their Spark tokens, 1000, 500 & 800 respectively for a total of 2,300 Spark delegated to FTSO Provider _X_.
FTSO Provider _X_ submits their price estimate for the round and wins using their proprietary price estimate.
Say, 100 Spark is awarded as compensation to FTSO Provider _X_.
The provider will take 10 Spark as a fee (10%) and divide the rest of the 90 tokens to the delegators based on their stake ratio (delegators stake / total Spark delegated to FTSO Provider _X_).
Thus, Alice gets 43.5% (39.15 Spark), Bob gets 21.7% (19.53 Spark) and Charlie gets 34.8% (31.32 Spark).
Note: Only Spark delegators are compensated, F-Asset votes are not compensated by the FTSO.

## F-Asset System

The F-Asset System is an integral application on the Flare Network.
It facilitates the trustless issuance of asset value onto the Flare Network.
An F-Asset is a representation of an asset from another blockchain on the Flare Network via the state connector system, i.e.XRP = FXRP.
Agents are responsible for posting and maintaining the FLR collateral.
Additionally, they are responsible for answering redemptions, when users decide they want to have their underlying asset back.
XRP, Litecoin, Dogecoin, and Stellar are the first four F-Assets and the addition of more is subject to FLR holder governance.

The F-Asset system gives the supported cryptos the capability to run smart contracts without any changes to the underlying blockchain.
The reason to do so is that nearly 65% of the value on public blockchains cannot be currently used in a trustless manner with smart contracts.

Two important parties in the F-Asset System are responsible for issuing F-Assets: Agents and Originators.
An originator is a person or an institution that wants to convert their cryptocurrency into an F-asset, and an agent (more likely to be an institution) is a participant on the Flare network that issues the F-asset desired by the originator.
Agents are also responsible for posting and managing the FLR collateral used to back the minting of F-Assets.
Additionally, as F-Asset holders have the right to claim the underlying asset, the agents are responsible for honoring redemptions promptly.

In the chance of the F-Asset position becoming undercollateralized, the F-Asset System encourages other agents to recollateralize a failing agent’s position via incentives.
If a default were to occur on a F-Asset position, the originator of the F-Assets would be paid out the value of their position in FLR plus an additional amount to cover trading fees to exchange back into their underlying asset.

More detailed information about F-Asset System can be found on these external resources:

* [The F-Asset System - Smart Contracts for Everyone](https://www.alphaoracle.io/flare-network/f-assets)
* [F-Asset System](https://www.thedefistandard.com/flare-network/f-assets)
