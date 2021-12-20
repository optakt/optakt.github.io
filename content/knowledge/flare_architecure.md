# Flare Architecture

This section contains information about some parts of the Flare Network.
In addition, it contains some information regarding the Avalanche platform since Flare adopted some of its parts within its ecosystem.

## Avalanche Platform

Avalanche is an open-source platform for launching decentralized applications and enterprise blockchain deployments in one interoperable, highly scalable ecosystem.
It is a platform with a new approach to scaling that allows for fast finality of transactions and the ability to run Solidity out-of-the-box.

The most significant difference between Avalanche and other decentralized networks is the [consensus protocol](./flare.md).
The Avalanche protocol employs a novel approach to a consensus to achieve strong safety guarantees, quick finality, and high throughput without compromising decentralization.

The purpose of the Avalanche platform:

* creation of subnets and blockchains tied to the main Avalanche network for general and corporate purposes;
* creation and hosting of decentralized applications;
* creation of smart assets (for example, new financial derivatives and DeFi solutions).

### Subnets

Avalanche is composed of logical entities called "subnets," and every chain from Avalanche blockchain is a part of a _subnet_.
A subnet, or subnetwork, is a dynamic set of validators working together to achieve consensus on the state of a set of blockchains.
Each blockchain is validated by one subnet, but a subnet can validate many blockchains.
The validators of each subnet must stake AVAX tokens to become a member of the network.
Clients interact with Avalanche through API calls to nodes (each chain has its [own API](https://docs.avax.network/build/avalanchego-apis/README)).

There is a particular subnet called the _primary network_, which validates Avalanche's built-in blockchains.
A node may be a member of arbitrarily many subnets and must be part of the primary network.
The primary network contains three build-in blockchains: P-Chain (Platform Chain), C-Chain (Contract Chain), and X-Chain (Exchange Chain).

In total, the Avalanche ecosystem (as of the end of December 2021) operates [13 subnets and 10 blockchains](https://explorer.avax.network/subnets?tab=validators) (3 main plus 7 minor ones: Dtehereum, Btehereum, C and X-chain PoW, and others).

All the details regarding the platform operation, tutorials, and tools are described in the [official Avalanche website](https://docs.avax.network/build/tutorials/platform/create-a-subnet/.)

#### P-Chain

The Platform Chain is the metadata blockchain on Avalanche that coordinates validators, keeps track of active subnets, and enables the creation of new subnets.
The P-Chain uses a [UTXO](https://www.investopedia.com/terms/u/utxo.asp) (Unspent Transaction Output) model and implements the Snowman consensus protocol.
Adding or removing a validator from the validator set at some given time by the P-Chain needs absolute ordering, so parallel lines of computation (DAG approach) are not allowed.

#### C-Chain

With the C-Chain API, clients can create smart contracts.
It is an example of the Ethereum Virtual Machine that the Avalanche Chain powers.
The C-Chain also serves as the default smart contract blockchain of the Avalanche Chain.
C-Chain is also very compatible with the Solidity smart contract and Ethereum tooling.
Therefore, it makes it easy for Ethereum developers to port applications into the Avalanche chain easily.
C-Chain uses an account-based Snowman Consensus Protocol since an application requires a total ordering of transactions and runs consensus on blocks, not transactions.
That is, if it needs to be able to compare any two transactions and determine which came first, it can't use a DAG because the ordering of some transactions is not defined.

#### X-Chain

The X-Chain functions as a decentralized network for creating and trading digital assets and uses the Avalanche consensus protocol.
The X-Chain uses a DAG with the full Avalanche consensus protocol, so it is UTXO (Unspent Transaction Output) based.
UTXO-based systems are inherently more scalable than account-based systems.
They are naturally parallelizable, whereas account-based systems must lock around the account to process transactions.
It is a real-world representation of assets such as bonds and equity with specific rules that govern their actions.
Whenever you initiate a transaction on the Avalanche blockchain, you get to pay a fee in AVAX.
The Exchange Chain is an Avalanche Virtual Machine (AVM) example.
Clients can create and trade assets on the X-Chain and other examples of the AVM with the help of the X-Chain API.

## Flare's FBA

_TODO_ add more details to each of the items

Flare uses some parts of the Avalanche codebase with some changes:

* deactivate most of the P-Chain and X-Chain functionality;
* run the C-Chain with a locally defined set of validators instead of using the P-Chain for the list of validators;
* hijack the EVM state transition to implement Flare state connector functionality;
* implement the rest of the Flare systems on top of the C-Chain as Solidity smart contracts.

## State Connector

The state connector in Flare's system is designed to observe the state of underlying chains and is one of the most critical protocols on the Flare Network.
The state connector enables Flare smart contracts to act on state changes from underlying blockchains, for example, payments.
The state connector adopts other blockchains' consensus mechanisms and validator ecosystems.

### Advantages

The [state connector system](https://docs.flare.network/en/state-connector) is a competitive approach for proving the state of an underlying chain to a smart contract, and it has the following advantages:

* Transaction validity references back to an underlying chain's genesis block: Other approaches like the SPV (Simplified Payment Verification) proof do not check the validity of a transaction because it is considered harmful for proving the state of an underlying chain to another network.
* Safety only depends on an underlying chain's validators: no trusted third-party service has its own set of economic incentives and risks.
  The need for trust is minimized by leveraging the guarantee that safety can only be lost in the state connector if an underlying chain's validators encounter a Byzantine fault.
* No cooperation is needed from an underlying chain's validators: validators from an underlying chain do not need to modify their chain's codebase to permit Flare to interpret their network.
  An underlying chain's validators do not even need to know that Flare exists for the state connector system to operate.
* Can read the state of any blockchain: the state connector can operate on any possible Sybil-resistance technique of an underlying chain.
* Constant-sized proofs: both the data availability proof and the payment proof are constant-sized, independent of the number of other payments in the data availability period being considered.

### Voting

There are three phases of the state connector voting protocol:

* Request
* Commit
* Reveal

#### Request Phase

Any user can submit a request to the state connector contract to have an event proven at any point in time.
From its perspective, the time window that this request enters the network state is known as the request phase.

#### Commit Phase

During the next window, attestation providers have the opportunity to commit a hidden vote regarding their belief in the outcome of the events requested in the previous phase.
Anyone may operate as an attestation provider without any capital requirement, but a default incentivized set is used as the minimal requirement for passing a vote about the events in the previous set.

#### Reveal Phase

Finally, in the next window of time, attestation providers reveal the votes they committed to in the previous round.
Once this reveal phase concludes and the next phase begins, the revealed votes are automatically counted, and all valid events become immediately available to all contracts on Flare.
Once these criteria are met, and that data is available, the stored transactions in the storage contract are now available to be referenced by any other contract on the Flare Network.

### Branching

The state connector branching protocol protects Flare against the incorrect interpretation of real-world events proactively, such that there are never any rollbacks on the Flare blockchain state.
Instead of having rollbacks, contention on state correctness is handled via automatic state branching into a correct and incorrect path.
The security assumption is that if you, as an independent node operator, follow along with the correct real-world state, you will always end up on the correct branch of the blockchain state.

## Flare Time Series Oracle (FTSO)

The [FTSO](https://flare.xyz/ftso-a-breakdown/) provides externally sourced data estimates (e.g., the current price of XRP) to the Flare Network in a decentralized manner.
It does so by leveraging the distributed nature of the network and its participants.
At the moment, only Spark holders get to vote; in the future, each asset's voting power will be split 50-50 between Spark holders and the holders of the particular asset.

Signal providers on the Flare Network are responsible for delivering continuous and accurate data estimates to the FTSO to represent off-chain values on the Flare Network.
Signal providers exist in the off-chain to collect data, such as asset prices, and deliver it to the FTSO.
For example, a provider may collect the price of XRP/$ from three different exchanges, average the price and have this price estimate ready for the FTSO every voting round to submit.
Signal providers can provide their F-Asset and FLR votes to do this, but they also can receive delegated votes from other parties to perform this service on their behalf.
Signal providers may take fees for this service.
Additionally, the entire process is noncustodial (i.e., participants retain complete control of their data).
The FTSO ensures the data is fair, accurate, and available to network applications.

In a nutshell, signal providers submit their votes, which results in a weighted distribution regarding prices at a given time, where the weight is related to the number of tokens held by a given address.
Next, the top and bottom 25% of these votes are deleted, resulting in a truncated distribution.
The oracle estimate is computed as the median of this truncated distribution.
Signal providers get rewarded with FLR tokens if their vote remained in the distribution after truncation.
The signal provider is then responsible for distributing this award between itself and its delegators.

Additional info and implementation details are available in [this example of price provider](https://github.com/flare-foundation/FTSO-price-provider).

### Delegation

Anyone can submit price data to the FTSO and vote for their price estimate.
Price submission may require effort and technical skills to build such a provider.
Most Spark and F-Asset holders can delegate their tokens to an FTSO provider whom they trust will provide accurate price estimates.
The delegation will be as simple as logging into a Flare wallet, choosing an FTSO provider, and finally selecting the amount tokens they wish to delegate.

### Rewards

Signal providers receive a reward for accurately voting on prices, as determined by the algorithm.

The top and bottom 25% of prices are discarded, and the remaining average is used.
The 50% of price providers who are closest to the price then receive rewards for their tokens plus the percentage of fees, as defined on the smart contract.
The default is 20%, and they can change it in 5% increments/decrements.

The exact amount of compensation is based on the number of delegated tokens minus a fee set by the FTSO provider.

For example, Alice, Bob, and Charlie choose to delegate their Spark tokens, 1000, 500 & 800 respectively, for a total of 2,300 Spark delegated to FTSO Provider _X_.
FTSO Provider _X_ submits their price estimate for the round and wins using their proprietary price estimate.
Say 100 Spark is awarded as compensation to FTSO Provider _X_.
The provider will take 10 Spark as a fee (10%) and divide the rest of the 90 tokens to the delegators based on their stake ratio (delegators stake / total Spark delegated to FTSO Provider _X_).
Thus, Alice gets 43.5% (39.15 Spark), Bob gets 21.7% (19.53 Spark) and Charlie gets 34.8% (31.32 Spark).
Note: only Spark delegators are compensated, F-Asset votes are not compensated by the FTSO.

## F-Asset System

The F-Asset System is a critical application on the Flare Network.
It facilitates the trustless issuance of asset values onto the Flare Network.
An F-Asset represents an asset from another blockchain on the Flare Network via the state connector system, i.e., XRP = FXRP.
Agents are responsible for posting and maintaining the FLR collateral.
Additionally, they are responsible for answering redemptions when users decide to have their underlying asset back.
XRP, Litecoin, Dogecoin, and Stellar are the first four F-Assets, and the addition of more is subject to FLR holder governance.

The F-Asset system gives the supported cryptos the capability to run smart contracts without changing the underlying blockchain.
The reason is that nearly 65% of the value on public blockchains cannot be used in a trustless manner with smart contracts.

Two important parties in the F-Asset System are responsible for issuing F-Assets: _agents_ and _originators_.
An originator is a person or an institution that wants to convert its cryptocurrency into an F-asset.
An agent (more likely to be an institution since they usually hold more assets) is a participant on the Flare network that issues the F-asset desired by the originator.
Agents are also responsible for posting and managing the FLR collateral used to back the minting of F-Assets.
Additionally, as F-Asset holders have the right to claim the underlying asset, the agents are responsible for honoring redemptions promptly.

In the chance of the F-Asset position becoming undercollateralized, the F-Asset System encourages other agents to re-collateralize a failing agent's position via incentives.
If a default were to occur on an F-Asset position, the originator of the F-Assets would be paid out the value of their position in FLR plus an additional amount to cover trading fees to exchange back into their underlying asset.

These external resources contain more detailed information about F-Asset System

* [The F-Asset System - Smart Contracts for Everyone](https://www.alphaoracle.io/flare-network/f-assets)
* [F-Asset System](https://www.thedefistandard.com/flare-network/f-assets)
