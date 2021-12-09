
# A Brief History of Decentralised Scaling

This article will cover how the scaling solutions for decentralised systems in the Crypto world have evolved with regards to the consensus algorithms that have been used, what their problems are and how Flare tries to address them.
We will cover how various blockchains fare in the realm of the [blockchain trilemma](https://vitalik.ca/general/2021/04/07/sharding.html) which roughly states that it is hard to achieve more than 2 ouf 3 from Security, Scalability and Decentralisation. 

The concept of Blockchain or Cryptography are not new and have been around before the 2000's. Still there was no concept of a truly digital currency/value due to the double spending problem. For example, say I have $100 worth of digital value and I send $100 to both Alice and Bob at the same time. There used to be no fully robust way to stop this from happening.

## 1. Proof of Work (PoW)

   Then came Satoshi Nakamoto who introduced the concept of Proof of Work (PoW) on top of a Blockchain which acts as a distributed ledger of all the transactions of value/currency. 
   So now imagine I have 100 Bitcoins. 
   There are many machines/nodes which have this information that I have 100 Bitcoins. 
   If I send 50 of them to Alice then certain things happen before it gets "finalised". 
   These things include me sending out this transaction info, some number of machines/nodes accepting it (along with transactions from others), verifying that I am truly the one sending it out and then they solve a complicated problem which takes a lot of computing power. 
   The first one to solve it actually gets to "write" these transactions in the form of a block and that's how it is finalised. 
   A good interactive visualisation of how PoW consensus might work and create a chain of blocks can be found [here](https://youtu.be/_160oMzblY8).  
   
PoW by itself isn’t very new. 
Original similar implementation was thought of in 1992 to [combat email spam](https://en.wikipedia.org/wiki/Hashcash). 
So every email will have some form of simple proof of work so that it is easy for me to send email to a friend but gets very hard for a spammer to send millions of emails at the same time.
   
In theory, PoW can scale infinitely. 
But with it comes the extra overhead of computational complexity which may not be practical in a truly global sense. 
Also some PoW based blockchains like Ethereum1.0 suffer from huge transaction fees as well as low throughput of 10-15 transactions per second which make them impractical for true global usage.
   Therefore the PoW consensus based blockchains can achieve decentralisation and security but not scalability.

## 2. Federated Byzantine Agreement (FBA)

   After Bitcoin and PoW came into picture in 2008, another class of consensus protocols surfaced around 2014 which tried to tackle a different problem known as “Byzantine Generals Problem”. 
   This is the problem when there are various nodes communicating with each other and a few of them are “Byzantine” which means either faulty or malicious. 
   A good consensus protocol should be able to tackle this issue in a decentralised manner for better security of the network. 
   One of the consensus algorithms that manage to do it is called “Federated Byzantine Agreement”. 
   This was introduced by Stellar.
   In FBA, every node has its “quorum set” which is a list of other nodes which this particular node trusts. 
   Therefore to achieve consensus for a transaction, the node relies on the members in its quorum set. 
   As long as the majority of nodes are not malicious, this will ensure security of the system. 
   Complex web of overlapping quorum slices makes it almost impossible for the majority of nodes to collude to control consensus. 
   FBA can be used on top of PoW or PoS (to be discussed later). 
   Safety and fault tolerance are [preferred over liveness](https://www.youtube.com/watch?v=aU08km2xrz0&ab_channel=Lumenauts). 
   In case of an accidental fork, the system is halted until consensus is reached. 
   This is important in case of banking applications. 
   Consensus that requires only message passing followed by voting process leads to high transaction volume per second and less expenditure of electricity than PoW.
   However, a criticism of pure FBA is that it leads to fragile structures of constituent nodes, permitting topology scenarios where a single node failure can cause a network-wide failure
   (Resistant to Forks?)

   TODO: Drawback: Single point failure if quorum sets share a single node?
   (Not mentioning about BFT which require a pre recommended validator list as it is against decentralisation)

## 3. Proof of Stake (PoS)

   Then came the Proof of Stake(PoS) consensus algorithms (e.g. Casper). 
   This no longer needs the computational complexity for validation of blocks but rather how much a node or node pool has staked and how long it has been active. 
   Staking is like putting some native cryptocurrency as a collateral. 
   The higher the stake and time in the game, the higher the chances of getting to write a block on behalf of the whole ecosystem and in return receive some cryptocurrency. 
   This can be distributed among the participating nodes in case of a node pool where there are multiple parties participating by delegating their tokens. 
   Two keywords are different here compared to PoW: (1) Computers that participate in PoS consensus are called Validators, (2) Once a block has been created and accepted by the network, it is said to be forged (not mined). 
   Then the validator gets some reward. 
   If a validator tries to be malicious then it gets penalised losing some of its staked cryptocurrency.
   PoS is much faster than PoW as there is no need to solve complex cryptographic puzzles to get to be the block writer and the process is much more streamlined. 
   Also no unnecessary wastage of energy means PoS is more environment friendly.
   The security of the network is ensured by the fact that if an attacker were to attack the system, it needs to have 51% of all the value staked. 
   As opposed to 51% of all computing power in case of a PoW system. 
   Sustainability, security are achieved in a PoS model like the one in ethereum2.0 which is on its way to be released.
   PoS ensures more throughput compared to PoW model. 
   However there is a communication overhead which hinders scalability (TODO: Explain more). 
   Also the higher the adaptation, the higher the overall value represented by the overall native cryptocurrency and therefore higher the value that needs to be locked up as staking by the nodes. 
   This means the native currency's full potential isn't utilised as once they are locked, they can no longer be used for any other purposes unless they are unlocked after the locking period expires. 
   Network’s safety is proportional to the value of stake committed.

## 3.1 Sharding and layer 2 solutions

   For solving this scaling issue, 'sharding' is introduced(e.g. In case of [Ethereum2.0](https://www.youtube.com/watch?v=ctzGr58_jeI&t=657s&ab_channel=Finematics). 
   Sharding is a way of spreading the computing and storage workload from a blockchain network, so that each node no longer has to process the entire network's transactional load. 
   Each node only needs to maintain info corresponding to its specific shard or partition. 
   For example, addresses starting with “0x00” will be part of Shard1 and addresses starting with “0x01” will be part of Shard2 etc. 
   There is a mechanism in place for inter-shard communication so that the whole blockchain is still viewable from any node. 
   All the shards will be connected to the “Beacon chain” which will have all the history with it and acts as an orchestrator of the whole network. 
   The consensus between all the shards will be maintained by the Beacon chain.
   However sharding has its own challenges. 
   Corrupting nodes in a given shard may lead to permanent loss of data. 
   One way to tackle this issue is by randomly assigning a node to a shard and randomly reassigning the node to another shard.
   Sharding has another problem that the composability aspect of Smart contract compatible blockchain will no longer exist. 
   For example, if a Defi built on top of Ethereum1.0 ecosystem relies on or combines few other projects, then in a sharded system of Ethereum2.0, ideally they should be on the same shard which may not always be the case. 
   If they are not on the same shard then they all needs to be ok with asynchronous updates which is [not ideal](https://www.coindesk.com/tech/2020/10/13/will-a-sharded-ethereum-be-flexible-enough-for-decentralized-finance/ ).

## 4. Avalanche

   The next level solution for scalability in terms of consensus algorithms is by Avalanche network. 
   In Avalanche, the underlying [sybil control mechanism](https://en.wikipedia.org/wiki/Sybil_attack) is still PoS but there is a different mechanism in place to finalise validating transactions. 
   This is achieved by a particular node randomly subsampling from other nodes multiple times across multiple cycles and eventually arriving at a consensus. 
   This is then recorded in a Directed Acyclic Graph as opposed to a chain of Blocks. 
   This consensus can easily scale at a much higher level compared to Bitcoin/PoW or Ethereum2.0/PoS because of faster agreement of transactions.
   The issue with Avalanche is similar to what other PoS based networks also have i.e. the economic limit of having a lot of value locked as stake in order to have the PoS running which may not be practical when the network overall value is in trillions of dollars. 
   The minimum staking that is required to become a validator may not be easy for an average person to be a validator without delegating to another validator thereby contributing to centralisation of the network. 
   Thus Avalanche has validators only in the number of thousands and not tens or hundreds of thousands so far.
   Usually many PoS based networks impose a penalty for malicious validators or validators that aren’t online continuously by slashing which is taking away some of the staked tokens as a punishment. 
   Avalanche doesn’t have this mechanism to encourage more participation as a step towards more decentralisation thereby sacrificing a bit more of security.

   (Doubt: How the rich get richer compounding effect is absent in Avalanche?)
   (Doubt: How exactly the transactions get written after consensus and how reward distribution happens in Avalanche?)

## 5. Flare

   Flare network combines best of both worlds by making Avalanche more resistant to Sybil attacks by introducing FBA into it. 
   Flare’s FBA achieves safety without relying too much on economic incentives that can interfere with high-value use cases.
   In Flare, each node has its own local set of validators or quorum set also known as Unique Node List (UNL). 
   This varies from the usual FBA because of another requirement that there should be a minimum level of overlap between UNLs. 
   This makes the network more single node failure resistant as well as Sybil attack resistant. 
   This is because a malicious actor is unlikely to be able to spin up many node instances that will also make it to the UNL list of other existing quorum sets.
   The native token, Spark, can be used to delegate to the price providers for FTSO (link to FTSO article) without having any locking thereby enabling other usage of Spark at the same time. 
   There is no hard link between the safety of the network and the value of native token Spark which allows greater flexibility for how Spark can be used.
   (TODO: What is the incentive to become a [validator in Flare](https://www.thedefistandard.com/flare-network/state-connector-system/)?
   
Flare’s overall vision is to achieve a truly global network that is:
   * Decentralised (Not present in Fiat)
   * Scalable (Not present in Bitcoin/Ethereum)
   * Secure
   * Resistant to Sybil Attack (Only present in current PoS systems where there is reliance on economic incentives for safety)
   * Trustless (Not present in Fiat)
   * Interoperable
   * Composable

Below is a summary of characteristics of various distributed networks:

| Crypto Project  | Decentralisation  |Security   |  Scalability |Energy Efficient   |Composability   | Throughput  |
|---|---|---|---|---|---|---|
|Bitcoin(PoW)   | ✅  | ✅  | ❌  |❌   | ❌  | ❌  |
|Ethereum(PoW)   | ✅  | ✅  | ❌  | ❌  | ✅  | ❌  |
|Eth2.0(PoS+Sharding)   |✅   |✅   |✅   |✅   | ❌  | ✅  |
|Avalanche(PoS)   | ✅  | ✅  | ❌  | ✅  |✅   |✅   |
|Flare(Ava+FBA+UNL)   | ✅  |✅   | ✅  | ✅  |✅   | ✅  |
