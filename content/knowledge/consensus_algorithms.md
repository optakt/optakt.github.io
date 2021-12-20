# Consensus Algorithms

This document explains, at a high level, what are the mechanisms behind the different consensus algorithms and how they evolved.
After reading it, the engineer should understand the ideas behind Flare's consensus mechanisms.

## FBA and Separating Sybil Attack Resistance from Consensus by Making it a Local Selection

### Consensus

A consensus algorithm is an agreement system that follows a predefined set of steps for a group of participants (nodes) to reach the same decision about a statement.
In a blockchain network nodes repeatedly decide what the complete history of a shared ledger is, from among multiple possible versions that occasionally conflict with each other.
This network-wide agreement allows the recipient of a crypto coin to have faith that the coin is both valid (not counterfeit) and not already spent elsewhere.

### Ripple

[Ripple consensus](https://ripple.com/files/ripple_consensus_whitepaper.pdf) is a voting consensus that works with nodes voting for each of its rounds.
The nodes that run the Ripple Server software and participate in the consensus protocol are called servers.
Each of them maintains a list of other servers - Unique Node List (UNL), that it queries for each validation round.
The UNL is not a list of all servers in the system, but rather only a snapshot that the node knows about.
Additionally, no participant knows what is the exact number of nodes in the system at any give time and does not need to.

Each server queries the nodes from its UNL and receives their votes and depending on what the majority (a defined percentage of them) of the list votes on a transaction, it is accepted or rejected.

The record of confirmed transaction history by the network is the ledger.
It is kept in each user account and represents the _ground truth_ of the network.

The _last-closed ledger_ is the most recent ledger that has been ratified by the consensus process and thus represents the current state of the network, as opposed to the "open ledger", which is the current operating ledger status of each node.
When transactions are sent from the users they become part of the open ledger for each node and are included in the last-closed ledger after they pass through the consensus process.

Each server includes every valid transaction it knows about in its broadcast messages to others when the consensus round starts, creating a list of transactions called a _candidate set_.
Each server queries the nodes from its UNL when executing consensus and only the votes from UNL members are considered.
The UNL is trustful only as a whole, meaning the set is expected not to collude in an attempt to defraud the network, but every single member of it is not implicitly considered trustful.

Transactions with more than a certain percentage of `no`-votes are either discarded or included in the next candidate set for the next consensus round.
If at least 80% of a UNL list agrees on a transaction, it is applied to the ledger.
After the servers exchange their candidate sets, each of them unites its own with the ones received from its UNL peers, and on the next step servers then vote on the accuracy of the transactions.

When servers vote, the process is a simple binary decision problem: deciding on a value 0 or 1 for the given transaction.

The assumptions that define the Ripple consensus are:

* Every well-behaved node makes its decision for a finite time
* All well-behaved nodes reach the same decision value
* 0 and 1 are both possible values for all well-behaved nodes

The goal is reaching a consensus amongst all nodes and preventing a situation, in which two disjoint sets of nodes each reach a consensus independently thus having two last-closed ledgers.

The consensus algorithm is applied every few seconds by all nodes and once the consensus is reached - the ledger is considered _closed_ and all well-behaved nodes will have identical ledger history.

Each of the servers can join or leave the network at some point, which means that in some voting rounds some servers will have different nodes in their UNL lists.
The notion of _majority rule_ is applied on the UNL set requirement for agreement, but a hard majority cannot be applied, because the total number of servers in the system is unknown.
As a result, there is no hard number of voters for any of the participants and existing groups of validating nodes overlap with each other.

This is a version of a consensus algorithm running in a federated manner, where each node knows about some number of others validating nodes, but not about all nodes in the network.

### Stellar and the Federated Byzantine Agreement

#### Introduction

Any agreement system in a distributed computing network needs to be fault-tolerant.
This means it must produce consistent results despite errors like slow communication links, unresponsive nodes, and misordered messages.

A Byzantine agreement system is additionally tolerant of [`Byzantine faults`](https://en.wikipedia.org/wiki/Byzantine_fault): nodes that give false information, whether due to error or in a deliberate attempt to subvert the system or gain some advantage.
Such a system prevents the double-spending problem by using a form of majority called _quorum_.
A node in such a network refuses to commit to a particular version of the ledger history until it sees that enough of its peers are also prepared to commit to it.
The number of peers that is enough to make a decision is the quorum and once each of the peers commits to a value after its quorum had committed, the system forms a large enough voting block to force the remaining nodes to agree with the decision.
In such a system, attempts from malicious nodes to create double-spending events, or in another way invalid transactions, are overwhelmed by the network votes from honest nodes.

For a hard majority to be formed, the number of all nodes in the system must be known and its quorum calculated at any time.
Opposed to this concept, in a Federated Byzantine Agreement (FBA) system, the participants in the network are loosely defined and can join and leave at any time without coordinating with any central authority.
Quorums are incomplete membership snapshots that can change at any time and consensus is reached in a decentralized form.
This makes the network consensus probabilistic with some uncertainty level (probability of not reaching a consensus not equal to zero) present at all times.

When making a decision, a node in an FBA system waits for its quorum nodes to agree on a value before considering a transaction settled.

Each node is potentially a member of another node's quorum and eventually when enough network quorums accept a transaction, it becomes infeasible to be rolled back.
On a higher level, the structure looks like multiple overlapping groups of participants, that function as a federation of groups, reaching consensus in a decentralized way, therefore the term _federated_.

This structure brings a high level of decentralization into decision-making process among network members and mimics the behavior of social groups spreading a piece of information.

FBA concept is defined as the [Stellar Consensus Protocol (SCP)](https://www.stellar.org/papers/stellar-consensus-protocol).

#### Quorum Slices

In an FBA consensus protocol implementation, nodes exchange messages asserting statements.
When a node hears a sufficient set of nodes assert one statement, it assumes no functioning node will ever contradict that statement.
This sufficient set of nodes is called a _quorum slice_ or just a _slice_.
A node in an FBA network refuses to commit to a particular value until it sees its peers are also committing to it.

Since in FBA the collections of participants are loosely defined, quorums are dynamically defined from an ever-changing and inevitably incomplete snapshot of membership.
Each node has multiple slices, any of which is sufficient to convince it of a value, so we can say that an FBA system consists of a loose federation of nodes each of which has chosen one or more slices to serve as a benchmark for making decisions.

To form a quorum slice, a node adds all nodes it knows about to its quorum slice, as well as all nodes' peers.
Continuing this process leads to encountering more and more nodes, and when no new nodes are found the quorum is defined.
This process of dynamically creating quorums uses the transitive closure property of the network.
With the help of this property, in an FBA system, every node is potentially connected by its indirect connections to other nodes.

Nodes know about other nodes' quorum slices, in the same manner, they know the rest of the information about their network peers — from the broadcast information that every node sends to the network whenever its' voting state changes.
Each broadcast message includes the details of the sending node's IP and its quorum slices nodes - a typical implementation used in FBA systems (and generally in other blockchains) is a [gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol).

#### FBA and Stellar Implementation

[Stellar Consensus Protocol (SCP)]((https://www.stellar.org/papers/stellar-consensus-protocol?locale=en)) is used to build [Stellar Payment Network](https://www.stellar.org/?locale=en) and implemented FBA as an essential building block of its system.

FBA systems create confidence in the outcome of a decentralized vote by running a process called "federated voting".
This is a procedure for discovering whether a network of participants can agree on a proposal.
In the cryptocurrency world a proposal is a ledger state with included transactions.
In a round of federated voting, each node chooses one of the potentially many possible values as an outcome of the round, but it first needs to confirm the nodes from its quorum will choose the same outcome.

The Stellar network's nodes conduct multiple federated votes on multiple values until some of them are accepted.
Since it is a payment network, it is logical to assume consensus needs to be reached on ledger values, but in fact, it is on "statements" about these values.

In an open system, someone can gain influence on the network by conducting a so-called [Sybil attack](https://www.sciencedirect.com/topics/computer-science/sybil-attack).
In short, a Sybil attack is an attempt to control a peer network by creating multiple fake identities that appear as unique peers to the outside world.

For the federated voting to be successful, Stellar's solution is to require a network property called _quorum intersection_.
This means that any two quorums that can be constructed in the network will always share at least one node after all Byzantine nodes are excluded.
For determining the prevailing sentiment of the network, this is as good as having a majority.
Intuitively, if any quorum agrees to statement `X`, no other quorum can ever agree to `not-X`, because it will necessarily include some node from the first quorum that has already voted for `X`.
This is the transitive closure property that these types of systems typically use to fight against Sybil attacks.

A living example of such a network is the Internet itself.
It consists of independent nodes that know about a few other nodes local to them, but the sets overlap with each other and every node is reachable through any other by routes and gateways.
In the same manner, an FBA network resembles inter-domain network routing with no central authority dictating or arbitraging agreements.
Quorum slices can be perceived as analogous to internet networks in terms of reachability.
Servers in the internet networks can be reached via requests from neighboring networks, using gateways and hubs, making each server accessible through any client.
In quorums, nodes are reachable through their neighboring nodes, which might be in another quorum, but because of the nodes' connections between each other, each node can be reached theoretically from any other.
The Internet's nearly complete transitive reachability suggests it can likewise ensure a worldwide consensus with FBA.

Voting in Stellar consists of two phases: nomination and balloting.

At the beginning of the nomination phase, a node randomly votes for a statement on a ledger version it receives from another node in the network.
The nodes cast out their nomination votes via messages to their peers, as well as the nominations of their peers, meaning separate federated votes are batched together.
Voting against a value is not defined and a node can nominate as many statements as it likes so after the FBA process is run on each node, a nominee is confirmed and it becomes a _candidate_.

The nomination may produce multiple confirmable candidates, so SCP requires the application layer to supply some method for combining them into a single composite, with the only requirement for this method to be deterministic.

SCP is an asynchronous protocol and nodes are not coordinated by time, but by the messages they exchange.
Thus, from a node’s perspective, it is not clear when the nomination process has ended.
During the nomination, it cannot be said which of the composites is the final one, but this is expected because the purpose is to produce several virtuous candidates for the balloting.

In the balloting phase nodes exchange their ballots on created composites and repeatedly conduct federated votes on statements about ballots.
From the point of view of a given node, a consensus is reached when it finds a ballot for which it can confirm (find a quorum accepting) a statement, and then it can be sure that every other well-behaved node has committed, or inevitably will commit to, the same value.

Nodes can be categorized as either well-behaved or ill-behaved.
A well-behaved node chooses sensible quorum slices and obeys the protocol, including eventually responding to all requests.
Ill-behaved nodes do not follow the protocol, can suffer Byzantine failure, and behave arbitrarily.
Since the blockchain network's most important goal is to reach consensus deterministically, nodes that generally act arbitrarily (non-deterministically) cannot be considered functioning parts of the system.
The expected behavior of well-behaved nodes is always _deterministic_ in the sense that they choose the same right value among others every time.

The goal of the Byzantine agreement is to ensure that well-behaved nodes agree on the same values despite the presence of the ill-behaved nodes.
The first part of this goal is to prevent nodes from diverging and committing different values for the same ledger.
The second part is to ensure the nodes will always go through the consensus process, as opposed to getting blocked in some dead-state.
The first property corresponds to the notion of safety, while the second one corresponds to the notion of liveness.

FBA networks implementations reach consensus in a decentralized, but the deterministic way and are the next step in building safe and live networks.
The loosely coupled quorums structures are the building blocks of the system and consensus rules need to manage only their behavior because the network is reaching its consensus in its decentralized manner itself.
In FBA networks the exact number of participants is never available, and it is not needed for anyone to know it to run through the consensus successfully.

## Avalanche and How It Uses Sampling to Achieve Deterministic Probabilistic Consensus on a Leaderless Manner that Can Scale

### Avalanche

[Avalanche](https://assets.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf) is a probabilistic consensus protocol based on Proof of Stake.
It was proposed by a pseudonymous group called [Team Rocket](https://medium.com/@mr_davis_12/avalanche-and-the-team-rocket-chronology-mystery-and-pseudonymity-30c17331f6a2).
The most important property that sets it apart from most of the existing protocols, is that like Nakamoto consensus and open permissionless blockchains, it can operate with no known upper bound of participants in the network (in the way FBA functions).
It aims to reach the decentralization level of an open network, resilience to 51% attacks, as well as high throughput and low latency.
It follows the idea of having some neglectable error when reaching of consensus as Nakamoto suggested, so it is probabilistic and there is no 100% certainty of reaching a consensus.

The number of messages each node has to handle per decision has O(log N) complexity and does not increase as the network scales up to a higher number of nodes.

Avalanche is a validator protocol and each node has a known number of validators (other nodes) called _validator list_.
Transactions that appear virtuous are accepted by validators, and validators are always expected to have deterministic behavior.
A transaction should be rejected if it conflicts with a previous one that the validator had voted for.
Each validator is a completely independent decision-maker and there is no leader election.

At a high level, each node randomly selects others from its validator list to get their current statements on some value.
This process is referred to as "repeated random subsampling" and it determines whether the rest of the network agrees on a transaction.
The procedure is repeated multiple times on new randomly selected nodes until the node builds up enough data to determine that the probability of being correct is high enough that it can be considered impossible to consider the opposite value.

To perform the consensus algorithm, Avalanche contains multiple decree Snowball instances instantiated as a multi-decree protocol that maintains a dynamic, append-only Directed Acyclic Graph (DAG) of all known transactions.
Maintaining a DAG improves efficiency in the sense that a single vote on a DAG vertex implicitly votes for all transactions on the path to the genesis vertex.

When a client creates a transaction, it names one or more parent transactions, which are included inseparably in the inputs and their connections from the edges of the DAG.
Transactions that are reachable via parent edges back in history are called _ancestor set_ and child transactions and their offspring are called _progeny_.

The central challenge in the maintenance of the DAG is to choose among conflicting transactions, for example, transactions that spend the same funds form a conflict set and only one of them can be accepted.
Avalanche instantiates a [Snowball](https://medium.com/@zaver.max/exploring-liveness-of-avalanche-d22f13b2db00) instance for each conflict set to run repeated queries and capture the level of confidence in each conflicting transaction.

In a given round each validator randomly selects some nodes from its list to query their preferred decision on a value.
The validators respond with their statements and if the majority of responses returned in a round differ from the one the query performer has, then it will update its decision and respond to other nodes with that answer.

When a transaction is queried, all transactions reachable from it by following the DAG edges towards its ancestor set, and its progeny are implicitly part of the query and the node will respond positively for it if the transaction and its entire ancestry are currently the preferred option in their respective conflict sets.
Each time a specific threshold of positive responders is reached, the transaction collects a _chit_ and nodes compute confidence in transactions as the total amount of chits they obtained.

Nodes perform the querying round independently and asynchronously.
So since one node does not query every other node like in classical consensus, each node performs its sample of randomly selected nodes, thus the total number of network participants is unknown and no one needs to know about it to reach consensus.

Avalanche follows the approach to building a probabilistic consensus algorithm in a decentralized way with having multiple validator lists that do not represent all validators in the network.
If the network is parameterized for a 33% attacker, and an attacker has a 34% stake, unlike with Classical consensus protocols where launching a double-spend attack is guaranteed to succeed, with Avalanche it just means they have a slightly higher chance to succeed rather than guaranteed.

## Turning Avalanche into FBA by Making the Algorithm with Locally Defined Validator Set

### Hash Graph as a Step before FCP

HashGraph is a consensus algorithm that offers a different approach to distributed ledger technology.
Compared to most blockchain networks, it is a privately held network with its only public version being the Hedera Hashgraph network.
Hashgraph is another distributed ledger technology, and it is described as a continuator or successor to the blockchain.

In Hedera, as well as in other blockchains, a user can create a transaction, that will eventually be put in a block and then spread out in the network as a part of this block.
But, instead of choosing one block to be chained to the previous one forming a chain of blocks, Hedera incorporates every block in the ledger without discarding any of the records.

The Hashgraph algorithm is an asynchronous Byzantine Fault Tolerant (aBFT) and the Hedera system is completely asynchronous.
The main assumption upon which the consensus relies is that 2/3 of the nodes are well-behaved nodes, which are expected to act deterministically.

So on a high level, the network operates on a DAG structure that records time-sequenced transactions at its vertices.
Each message contains one or more transactions, a timestamp, digital signature, hashes of two earlier events.

Hashgraph is also implemented using a DAG and gossip protocol for communication between network nodes.

### Flare System

Flare system is a multi-chain platform that manages multiple blockchains and handles value exchange between them.
It uses FBA and implements its own component (State Connector System) for connectivity and validation of the chains.
Currently, Flare network uses some of the Avalanche concepts.

Avalanche implementation consists of multiple subnets that have [multiple chains connected to them](./flare_architecture.md#Avalanche-Platform).
Subnets in Avalanche are dynamic sets of validators working to achieve consensus on one or multiple blockchain networks.
Generally, there are three chains - [Platform Chain](./flare_architecture.md#P-Chain), [Contract Chain](./flare_architecture.md#C-Chain) and [Exchange Chain](./flare_architecture.md#X-Chain).

Flare takes advantage mainly from the Contract Chain to run FBA voting locally on its nodes.

There are currently three sets of participants in the Flare system: validators, block producers and attestors.
Flare validators set is set and run locally by each node.
Attestors vote on state changes on underlying chains, including voting on the number of blocks produced by validators.
Block producers correspond to validators on the underlying chains.
They propose blocks to the network, according to their weight voted on by attestors.

The component that monitors the state of the different blockchains is the [State Connector](./flare_architecture.md#State-Connector).
This system provides the Flare Network with flexibility to integrate other blockchains to the Flare Network.

The model design of the State Connector System is crucial to maintaining the accurate digesting of integrated blockchain state changes.
This design requires attackers to alter the view of integrated blockchains through their own Flare Network validator and a quorum of other Flare Network validators.

Flare system runs its own integrated Ethereum Virtual Machine (EVM) that is locally maintained.
In order for the State Connector to digest the changes on the different blockchains connected to the network, it needs to know about the number of blocks in each of the chains.

In the Flare network, the problem of updating replicated states of blockchains is addressed using quorums of validators.
They are snapshots of the validator capacity of the network and function as a federated group of quorums.
This way of building the network is following the logic of the [Flare consensus algorithm](#The-Future-with-the-Flare-Consensus-Protocol), but not fully following it.
The full implementation is expected for the near future.

### The Future with the Flare Consensus Protocol

[Flare Consensus Protocol (FCP)](https://drive.google.com/file/d/15adsTes_lhi12PGMxlXfxCsoQsjXvPnS/view) uses FBA and virtual voting (like _Hashgraph_).
Flare follows the FBA requirements for reaching network quorum intersection after removing malicious nodes from the system, consequently, quorum intersection is again required between well-behaved nodes.

The protocol relies on two other assumptions.
The first is the impossibility of forging messages.
When communicating with each other, nodes are named by public keys and use cryptographically secure digital signatures when exchanging messages.
In the context of asynchronous communication, it is assumed also, that messages exchanged between well-behaved nodes can get delayed but will eventually be delivered so the order in which they were sent might not be the order they are received.

In Flare network nodes synchronize with each other using the exchanged messages information via a gossip protocol, consequently functioning asynchronously.
After each node finds its quorum slices, they start to send messages to a random number of peers in them.
The first messages are the _initial_ messages and they contain information about the transaction amount, the recipient, and a hash of the sender's quorum of nodes.
Recipients of messages in their turn create new messages that contain information about the received message and the messages they previously created themselves (and sent to some other node in a previous step).

In the second turn, messages contain more information including the hash of previous messages received and sent by the node that are used to build up the connections between them.
The creation and propagation of messages across the network are represented by a global DAG structure.
It tracks the history of all messages exchanged by the recipients over time.
In the DAG, messages are connected by edges and an existing edge between two messages means that it is possible to go from one vertex to the other, each message being a vertex of the structure.
Using the topology of the network that the DAG brings, ancestors of a message are all the messages that it is referencing as previous to it, in the order in time the node receives them.

To keep track of the information they have, nodes maintain also DAGs that represent the state of their local knowledge.
As time progresses, some messages, by being encoded in the history of the new message, will be more widely shared throughout the network.
Once all participants are aware of them, the spread messages and their content are in the scope of global knowledge of the network.
This is captured by the idea of reachability.

Following the logic of FBA, an important part of exchanging messages between nodes is sharing them not amongst any participant, but only with the important ones for each node.
This implementation of the FBA federated quorum overlapping structure aims to reach flexible trust between the groups.

Each participant in the network must end up with the same ordering of messages to reach a consensus on the ledger history.
After each participant has constructed its local DAG, the protocol proceeds to [Federated Virtual Voting process](https://drive.google.com/file/d/15adsTes_lhi12PGMxlXfxCsoQsjXvPnS/view).

An FBA construction does not rely on economic mechanisms for securing consensus because it enables individual nodes to independently make quorum slice decisions.
Market forces cause quorum slices to overlap and this gives rise to the network-wide rule for consensus running in a decentralized manner.

FCP achieves fairness by being both leaderless and ordered, making it excessively difficult for an attacker to influence which of two transactions will be ordered first in a transaction set.
These properties make FCP a compelling model for internet-level `Turing-complete` consensus.
