# A Brief History of Decentralised Scaling

This article covers how the scaling solutions for decentralized systems in the crypto world have evolved with regards to the consensus algorithms that have been used, what their problems are and how Flare tries to address them.
It also addresses how various blockchains fare in the realm of the [blockchain trilemma](https://vitalik.ca/general/2021/04/07/sharding.html) which roughly states that it is hard to achieve more than 2 out of 3 from security, scalability and decentralization.

The concepts at the foundation of blockchain technology, such as cryptography (for data encryption) and peer-to-peer networks (that handle information exchange in a distributed way) were around before the 2000s.
However, there was no concept of a truly digital currency due to the [double spending problem](https://en.wikipedia.org/wiki/Double-spending).

## Proof of Work

Then came Satoshi Nakamoto who introduced the pre-existing concept of Proof of Work (PoW) on top of a Blockchain which acts as a distributed ledger of all the transactions of currency.
So now imagine Brian has 100 bitcoins.
There are many machines/nodes which have this information that Brian has 100 bitcoins.
If Brian sends 50 of them to Alice then certain things happen before the transfer gets finalized.
Nodes collect pending transactions (sent out by wallet applications) in a memory pool.
When a node solves the computational problem, it gets to pick the transactions from its memory pool that get included in the next block and thus become part of the consensus state.
However, only under the condition that the block follows the rules on transaction validity, i.e. does not include any transactions spending the same unit of account twice.
This is termed as "Nakamoto consensus".
Check this [visualization](https://youtu.be/_160oMzblY8) of how PoW consensus might work and create a chain of blocks.

PoW by itself is not very new.
A similar implementation was thought of in 1992 to [combat email spam](https://en.wikipedia.org/wiki/Hashcash).
Using the _Hashcash_ system, every email has some form of simple proof of work, which makes it is easy for someone to send an email to a friend but makes it difficult for a spammer to send millions of emails at the same time.

In theory, PoW can scale infinitely.
The more successful the network is, the higher the demand for the token.
The higher the demand for the token, the more its value increases.
As the price increases, more miners can profitably run miners, and more power is wasted.
Also, some PoW based blockchains like Ethereum suffer from huge transaction fees as well as low throughput of 10-15 transactions per second which make them impractical for true global usage.
Bitcoin's PoW has the potential to scale a lot in terms of a payment system, but the wastage of energy is its biggest problem.

## Federated Byzantine Agreement

[Byzantine Generals Problem](https://bravenewgeek.com/tag/byzantine-generals-problem/) is the problem when there are various nodes communicating and a few of them are “Byzantine” — either faulty or malicious.
Byzantine nodes might not reply to messages, or deliberately send wrong information like false signatures, double spent transactions, etc.
Therefore, any blockchain should have mechanisms to solve the Byzantine Generals Problem.
In case of Bitcoin, the mechanism is PoW.
Federated Byzantine Agreement (FBA) consensus in a family of consensus protocols, which eliminates Byzantine faults, and provides deterministic finality (unlike PoW which has only probabilistic finality), without having the selection of validators as part of the protocol itself.
This was introduced by [Stellar](https://www.stellar.org/papers/fast-and-secure-global-payments-with-stellar?locale=en).
In FBA, every node has its “quorum set” which is a list of other nodes which this particular node trusts.
Therefore, to achieve consensus for a transaction, the node relies on the members in its quorum set.
A node's "quorum slices" are the list of nodes from quorum set such that if all members of the quorum slice agree about the state of the system, then the node considers them right.
Each node unilaterally declares its quorum slices.
As long as the majority of nodes are not malicious, the security of the system is ensured.
Various overlapping quorum slices across the network make it almost impossible for the majority of nodes to collude to control consensus.
Safety and fault tolerance are [preferred over liveness](https://www.youtube.com/watch?v=aU08km2xrz0&ab_channel=Lumenauts).
In case of an accidental fork, the system is halted until consensus is reached.
This is important in banking applications.
A consensus that requires only message exchange followed by a voting process leads to high messages volume per second and less expenditure of electricity than PoW.
However, a criticism of pure FBA is that it leads to fragile structures of constituent nodes, permitting topology scenarios where a single node failure can cause a network-wide failure.
For example, [in November 2021](https://u.today/xrp-ledger-is-back-on-track-after-temporary-halt), four validators in Ripple went down which halted the whole network for 15 minutes because consensus could no longer be reached.

## Proof of Stake

Then came the [Proof of Stake](https://www.investopedia.com/terms/p/proof-stake-pos.asp) (PoS) consensus algorithms (e.g. Casper).
This type of consensus algorithms no longer needs the computational power used for block validation but rather an amount staked by network participants (being single or in clusters).
Staking is putting some native cryptocurrency as collateral to become a validating entity in the network.
The higher the stake, the higher the chances of getting to write a block on behalf of the whole ecosystem and in return receive some cryptocurrency.
Two keywords are different here compared to PoW: computers that participate in PoS consensus are called _validators_, and once a block has been created and accepted by the network, it is said to be _forged_ (not mined).
Then the validator gets some reward.
If a user does not want to run his own validator, then he can generally simply "delegate" his tokens to the validators, which is like staking without running his own validator.
The weight of the delegated tokens is added to the staked tokens of the chosen validator, who gets to keep part of the reward.
Malicious validators get penalized by losing some of their staked currency.
This is often referred to as _slashing_.

One key difference between PoS and PoW is that a block on a PoS chain has deterministic finality, meaning that once it has been accepted fully, there is no way to ever undo it.
In PoW, it can theoretically always be undone by creating a longer valid chain of blocks.
PoS is relatively faster than PoW as there is no need to solve complex cryptographic puzzles to get to be the block writer and the process is much more streamlined.
Also no wastage of energy means PoS is more environment friendly.
PoS algorithms based on BFT need 2/3+1 of the nodes to be honest, so the required threshold is higher than PoW where an attacker needs 51% of all computing power.
Sustainability and security are achieved in a PoS model like the one in Cardano.
In PoS, an explicit agreement is required before a block becomes valid.
If a sufficient number of validators do not sign the block, it gets rejected, even if all consensus rule is followed by the block.
A block becomes valid only after the necessary number of participants to the consensus algorithm explicitly vote for it by signing it.
This means that a lot of communication is needed.
In general, voting on the next correct block requires multiple rounds of communication (preparation round, confirmation round and commit round), and the more validators there are, the more overhead there is for everyone to get their messages to the leader of a round.
Scalability is negatively impacted by the high amount of communication.

Also the higher the adoption, the higher the total value represented by the native cryptocurrency and therefore higher the value that needs to be locked up as staking by the nodes.
This means the native currency's full potential is not utilised as, once it is locked, it can no longer be used for any other purposes.
The network’s security is proportional to the value of stake committed.

## Sharding and Layer 2 Solutions

For solving this scaling issue, 'sharding' is introduced (e.g. in the case of [Ethereum 2.0](https://www.youtube.com/watch?v=ctzGr58_jeI&t=657s&ab_channel=Finematics)).
Sharding is a way of spreading the computing and storage workload from a blockchain network, so that each node no longer has to process the entire network's transactional load.
Each node only needs to maintain info corresponding to its specific shard or partition.
For example, addresses starting with “0x00” are part of shard 1 and addresses starting with “0x01” are part of shard 2 etc.
A mechanism for inter-shard communication exists so that the whole blockchain is still viewable from any node.
All the shards are connected to the “Beacon chain” which has access to all the chain's history and acts as an orchestrator of the whole network.
The consensus between all the shards is maintained by a "beacon chain".
However, sharding has its challenges.
Corrupting nodes in a given shard by an attacker may lead to permanent loss of data.
One way to tackle this issue is by [randomly assigning](https://www.investopedia.com/terms/s/sharding.asp) nodes to certain shards and constantly reassigning them at random intervals.
This random sampling would make it difficult for hackers to know when and where to corrupt a shard.
Sharding has another problem: smart contract compatible blockchains are no longer composable.
For example, if a Decentralized Finance (DeFi) platform built on top of Ethereum relies on or combines few other projects, then in a sharded system of Ethereum 2.0, ideally they should be on the same shard which may not always be the case.
If they are not on the same shard then they all need to support asynchronous updates which is [not ideal](https://www.coindesk.com/tech/2020/10/13/will-a-sharded-ethereum-be-flexible-enough-for-decentralized-finance/).

## Avalanche

The next level solution for scalability in terms of consensus algorithms is by Avalanche consensus which uses Snow algorithm with PoS as the underlying [sybil control mechanism](https://en.wikipedia.org/wiki/Sybil_attack).
In Snow consensus, in short, a particular node randomly sub-samples from other nodes multiple times across multiple cycles and eventually arrives at a consensus.
This is then recorded in a Directed Acyclic Graph (DAG) as opposed to networks that work with only a chain of Blocks and no structure for the transaction history.
Every node has its own view of the current consensus state; they do not progress in lock-step as they do for BFT algorithms.
However, the local view of each node is eventually consistent with the view of all other nodes on the network.

Snow consensus can easily scale to a higher level compared to PoW or PoS.
Because the nodes only need to communicate with random sub-samples of the network for each round, the communicational complexity is much lower than BFT.
If these are logarithmically smaller than the entire set of validators on the network, the performance does not decrease exponentially with the size of the validator set.
However, one common issue for PoS blockchains remains: the security of the network depends on the ratio of capital locked up for validation.
Usually many PoS based networks impose a penalty for malicious validators or validators that are not online continuously by slashing which is taking away some of the staked tokens as a punishment.
In case of Snow consensus, nodes monitor uptime and nodes do not receive rewards when they are not up long enough.

Avalanche is a leaderless protocol, which is the first one of its kind.
In BFT PoS, there is a leader for every round that has to propose the block; in Avalanche, everyone can propose state changes concurrently.

## Flare

Flare network combines the best of both worlds by introducing FBA into the Snow consensus of Avalanche.
Flare’s FBA achieves safety without relying too much on economic incentives that can interfere with high-value use cases.
In Flare, each node has its own local set of validators or quorum set also known as Unique Node List (UNL).
An added requirement compared to the usual FBA is that there should be a minimum level of overlap between UNLs.
This makes the network more resistant to single node failure as well as Sybil attack resistant.
This is because a malicious actor is unlikely to be able to spin up many node instances that manage to integrate the UNL list of other existing quorum sets.
Flare Consensus Protocol (FCP) is leaderless, asyncronous Byzantine fault tolerant and highly scalable due to its usage of a novel networking complexity-reduction technique called federated virtual voting.
Details on this can be found in [this article on consensus](https://optakt.github.io/knowledge/consensus/).

The native token, Spark, is used to delegate to the price providers for [Flare Time Series Oracle (FTSO)](https://flare.xyz/ftso-a-breakdown/) without having any locking thereby enabling other usage of Spark at the same time.
There is no hard link between the safety of the network and the value of native token Spark which allows greater flexibility for how Spark can be used.

The incentive to become a validator in Flare is as follows.
In Flare, there are two potentially overlapping sets of validators:

* the local set of validators that we rely on for consensus
* the set of underlying chain validators/miners, that can propose blocks

When someone proposes a block, they receive a reward.
However, someone running a price provider or a proof attestor also needs to run a node, which results in many more nodes than just the underlying chain validators/miners.

Below is a summary of characteristics of various distributed networks:

| Crypto Project       | Decentralisation | Security | Scalability | Energy Efficient | Composability | Throughput |
|----------------------|------------------|----------|-------------|------------------|---------------|------------|
| Bitcoin(PoW)         | ✅                | ✅        | ❌           | ❌                | ❌             | ❌          |
| Ethereum(PoW)        | ✅                | ✅        | ❌           | ❌                | ✅             | ❌          |
| Eth2.0(PoS+Sharding) | ✅                | ✅        | ✅           | ✅                | ❌             | ✅          |
| Avalanche(PoS)       | ✅                | ✅        | ❌           | ✅                | ✅             | ✅          |
| Flare(Ava+FBA+UNL)   | ✅                | ✅        | ✅           | ✅                | ✅             | ✅          |
