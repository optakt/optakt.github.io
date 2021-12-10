
# A Brief History of Decentralised Scaling

This article will cover how the scaling solutions for decentralised systems in the crypto world have evolved with regards to the consensus algorithms that have been used, what their problems are and how Flare tries to address them.
We will cover how various blockchains fare in the realm of the [blockchain trilemma](https://vitalik.ca/general/2021/04/07/sharding.html) which roughly states that it is hard to achieve more than 2 ouf 3 from security, scalability and decentralization.

The concepts at the foundation of blockchain technology, such as cryptography and peer-to-peer networks, were around before the 2000s.
Still there was no concept of a truly digital currency due to the double spending problem.
For example, say I have $100 worth of digital value and I send $100 to both Alice and Bob at the same time.
There used to be no fully robust way to determine which one of the two transactions is the valid one.

## 1. Proof of Work

   Then came Satoshi Nakamoto who introduced the pre-existing concept of Proof of Work (PoW) on top of a Blockchain which acts as a distributed ledger of all the transactions of currency.
   So now imagine I have 100 bitcoins.
   There are many machines/nodes which have this information that I have 100 bitcoins.
   If I send 50 of them to Alice then certain things happen before the transfer gets finalized.
Nodes collect pending transactions in a memory pool.
When a node solves the computational problem, it gets to pick the transactions from its memory pool that get included in the next block and thus become part of the consensus state. 
However, only under the condition that the block follows the rules on transaction validity, i.e. doesn't include any transactions spending the same unit of account twice.   
This is termed as "Nakamoto consensus".
   A good interactive visualisation of how PoW consensus might work and create a chain of blocks can be found [here](https://youtu.be/_160oMzblY8).

PoW by itself isn’t very new.
Original similar implementation was thought of in 1992 to [combat email spam](https://en.wikipedia.org/wiki/Hashcash).
In that use case, every email will have some form of simple proof of work so that it is easy for me to send email to a friend but gets very hard for a spammer to send millions of emails at the same time.

In theory, PoW can scale infinitely.
The more successful the network is, the higher the demand for the token. 
The higher the demand for the token, the more the price will increase.
As the price increases, more miners are able to profitably run miners, and more power is wasted.
Also, some PoW based blockchains like Ethereum suffer from huge transaction fees as well as low throughput of 10-15 transactions per second which make them impractical for true global usage.
Bitcoin's PoW has the potential to scale a lot in terms of payment system; but the wastage of energy is its biggest problem.

## 2. Federated Byzantine Agreement
“Byzantine Generals Problem” is the problem when there are various nodes communicating with each other and a few of them are “Byzantine” which means either faulty or malicious.
Any blockchain should have mechanisms to solve the Byzantine Generals Problem. 
In case of Bitcoin, the mechanism is PoW.
Federated Byzantine Agreement (FBA) consensus in a family of consensus protocols, which eliminates Byzantine faults, and provides deterministic finality (unlike PoW which has only probabilistic finality), without having the selection of validators as part of the protocol itself.   
This was introduced by Stellar.
In FBA, every node has its “quorum set” which is a list of other nodes which this particular node trusts.
Therefore, to achieve consensus for a transaction, the node relies on the members in its quorum set.
As long as the majority of nodes are not malicious, this will ensure security of the system.
Various overlapping quorum slices across the network makes it almost impossible for the majority of nodes to collude to control consensus.
Safety and fault tolerance are [preferred over liveness](https://www.youtube.com/watch?v=aU08km2xrz0&ab_channel=Lumenauts).
In case of an accidental fork, the system is halted until consensus is reached.
This is important in case of banking applications.
Consensus that requires only message passing followed by voting process leads to high transaction volume per second and less expenditure of electricity than PoW.
However, a criticism of pure FBA is that it leads to fragile structures of constituent nodes, permitting topology scenarios where a single node failure can cause a network-wide failure.
For example, [in November 2021](https://u.today/xrp-ledger-is-back-on-track-after-temporary-halt), four validators in Ripple went down which halted the whole network for 15 minutes because consensus could not longer be reached.

## 3. Proof of Stake

   Then came the Proof of Stake(PoS) consensus algorithms (e.g. Casper).
   This no longer needs the computational resources for validation of blocks but rather how much a node or node pool has staked and how long it has been active.
   Staking is like putting some native cryptocurrency as a collateral.
   The higher the stake, the higher the chances of getting to write a block on behalf of the whole ecosystem and in return receive some cryptocurrency.
   Two keywords are different here compared to PoW: (1) Computers that participate in PoS consensus are called Validators, (2) Once a block has been created and accepted by the network, it is said to be forged (not mined).
   Then the validator gets some reward.
If a user doesn't want to run his own validator, then he can generally simply "delegate" his tokens to the validators, which is like staking without running his own validator, and add the weight of the delegated tokens to the staked tokens of the chosen validator, in return for giving part of the reward to the validator.
   If a validator tries to be malicious then it gets penalised losing some of its staked cryptocurrency.

One key difference between PoS and PoW is that a block on a PoS chain has deterministic finality, meaning that once it has been accepted fully, there is no way to ever undo it. 
In Pow, it can (in theory) always be undone by creating a longer valid chain of blocks.
PoS is relatively faster than PoW as there is no need to solve complex cryptographic puzzles to get to be the block writer and the process is much more streamlined.
Also no unnecessary wastage of energy means PoS is more environment friendly.
PoS algorithms based on BFT need 2/3+1 of the nodes to be honest, so the required threshold is higher than PoW where an attacker needs 51% of all computing power.
Sustainability, security are achieved in a PoS model like the one in Cardano.
In PoS, explicit agreement is required before a block becomes valid.
If a sufficient number of validators don't sign the block then it will be rejected even if all consensus rule is followed by the block.
Only after the necessary number of participants to the consensus algorithm explicitly vote for the block by signing it, will it become valid.
This means that a lot of communication is needed. 
In general, voting on the next correct block requires multiple rounds of communication (preparation round, confirmation round and commit round), and the more validators there are, the more overhead there is for everyone to get their messages to the leader of a round.
Due to this high number of communication scalability is hindered.

   Also the higher the adaptation, the higher the overall value represented by the overall native cryptocurrency and therefore higher the value that needs to be locked up as staking by the nodes.
   This means the native currency's full potential isn't utilised as once they are locked, they can no longer be used for any other purposes unless they are unlocked after the locking period expires.
   Network’s security is proportional to the value of stake committed.

## 3.1 Sharding and layer 2 solutions

   For solving this scaling issue, 'sharding' is introduced (e.g. in the case of [Ethereum 2.0](https://www.youtube.com/watch?v=ctzGr58_jeI&t=657s&ab_channel=Finematics)).
   Sharding is a way of spreading the computing and storage workload from a blockchain network, so that each node no longer has to process the entire network's transactional load.
   Each node only needs to maintain info corresponding to its specific shard or partition.
   For example, addresses starting with “0x00” will be part of shard 1 and addresses starting with “0x01” will be part of shard 2 etc.
   There is a mechanism in place for inter-shard communication so that the whole blockchain is still viewable from any node.
   All the shards will be connected to the “Beacon chain” which will have all the history with it and acts as an orchestrator of the whole network.
   The consensus between all the shards will be maintained by a "beacon chain".
   However, sharding has its own challenges.
   Corrupting nodes in a given shard may lead to permanent loss of data.
   One way to tackle this issue is by randomly assigning a node to a shard and randomly reassigning the node to another shard.
   Sharding has another problem that the composability aspect of smart contract compatible blockchain will no longer exist.
   For example, if a Decentralized Finance (DeFi) platform built on top of Ethereum relies on relies on or combines few other projects, then in a sharded system of Ethereum2.0, ideally they should be on the same shard which may not always be the case.
   If they are not on the same shard then they all needs to be ok with asynchronous updates which is [not ideal](https://www.coindesk.com/tech/2020/10/13/will-a-sharded-ethereum-be-flexible-enough-for-decentralized-finance/).

## 4. Avalanche

The next level solution for scalability in terms of consensus algorithms is by Avalanche network which uses Snow Consensus with PoS as the underlying [sybil control mechanism](https://en.wikipedia.org/wiki/Sybil_attack).
In Snow consensus, in short, a particular node randomly sub-samples from other nodes multiple times across multiple cycles and eventually arrives at a consensus.
This is then recorded in a Directed Acyclic Graph (DAG) as opposed to a chain of blocks.
Every node has its own view of the current consensus state; they don't progress in lock-step like they do for BFT algorithms.
However, the local view of each node is eventually consistent with the view of all other nodes on the network.

Snow consensus can easily scale to a higher level compared to PoW or PoS.
Because the communicational complexity is much lower than BFT; we only need to communicate with random sub-samples of the network for each round. 
If these are logarithmically smaller than the entire set of validators on the network, the performance doesn't decrease exponentially with the size of the validator set.
However, one common issue for PoS blockchains remains: the security of the network depends on the ratio of capital locked up for validation.
Usually many PoS based networks impose a penalty for malicious validators or validators that aren’t online continuously by slashing which is taking away some of the staked tokens as a punishment.
In case of Snow consensus, nodes monitor uptime and nodes don't receive rewards when they are not up long enough.

Avalanche is a leaderless protocol, which is the first one of its kind. 
In BFT PoS, there is a leader for every round that has to propose the block; in Avalanche, everyone can propose state changes concurrently.

## 5. Flare

Flare network combines best of both worlds by introducing FBA into the Snow consensus of Avalanche.
Flare’s FBA achieves safety without relying too much on economic incentives that can interfere with high-value use cases.
In Flare, each node has its own local set of validators or quorum set also known as Unique Node List (UNL).
This varies from the usual FBA because of another requirement that there should be a minimum level of overlap between UNLs.
This makes the network more single node failure resistant as well as Sybil attack resistant.
This is because a malicious actor is unlikely to be able to spin up many node instances that will also make it to the UNL list of other existing quorum sets.
The native token, Spark, can be used to delegate to the price providers for FTSO (link to FTSO article) without having any locking thereby enabling other usage of Spark at the same time.
There is no hard link between the safety of the network and the value of native token Spark which allows greater flexibility for how Spark can be used.

In Flare, there are two sets of validators:
* the local set of validators that we rely on for consensus;
* the set of underlying chain validators/miners, that can propose blocks.

When someone proposes a block, they do receive a reward. 
These sets can overlap. 
However, someone running a price provider or a proof attestor also needs to run a node, so there will be many more nodes than just the underlying chain validators/miners.

Below is a summary of characteristics of various distributed networks:

| Crypto Project  | Decentralisation  |Security   |  Scalability |Energy Efficient   |Composability   | Throughput  |
|---|---|---|---|---|---|---|
|Bitcoin(PoW)   | ✅  | ✅  | ❌  |❌   | ❌  | ❌  |
|Ethereum(PoW)   | ✅  | ✅  | ❌  | ❌  | ✅  | ❌  |
|Eth2.0(PoS+Sharding)   |✅   |✅   |✅   |✅   | ❌  | ✅  |
|Avalanche(PoS)   | ✅  | ✅  | ❌  | ✅  |✅   |✅   |
|Flare(Ava+FBA+UNL)   | ✅  |✅   | ✅  | ✅  |✅   | ✅  |
