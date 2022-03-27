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
Each of them maintains a list of other servers — Unique Node List (UNL), that it queries for each validation round.
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

The consensus algorithm is applied every few seconds by all nodes and once the consensus is reached — the ledger is considered _closed_ and all well-behaved nodes will have identical ledger history.

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
Each broadcast message includes the details of the sending node's IP and its quorum slices nodes — a typical implementation used in FBA systems (and generally in other blockchains) is a [gossip protocol](https://en.wikipedia.org/wiki/Gossip_protocol).

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
Voting against a value is not defined and a node can nominate as many statements as it likes so after the FBA process is run on each node, a nominee is confirmed, and it becomes a _candidate_.

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
Currently, Flare network borrows concepts from Avalanche.

Avalanche implementation consists of multiple subnets that have [multiple chains connected to them](./architecture.md#Avalanche-Platform).
Subnets in Avalanche are dynamic sets of validators working to achieve consensus on one or multiple blockchain networks.
Generally, there are three chains: the [Platform Chain](./architecture.md#p-chain), [Contract Chain](./architecture.md#c-chain) and [Exchange Chain](./architecture.md#x-chain).

Flare takes advantage mainly from the Contract Chain to run FBA voting locally on its nodes.

There are currently three sets of participants in the Flare system: validators, block producers and attestors.
Flare validators set is set and run locally by each node.
Attestors vote on state changes on underlying chains, including voting on the number of blocks produced by validators.
Block producers correspond to validators on the underlying chains.
They propose blocks to the network, according to their weight voted on by attestors.

The component that monitors the state of the different blockchains is the [State Connector](./architecture.md#State-Connector).
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
The first messages are the _initial_ messages, and they contain information about the transaction amount, the recipient, and a hash of the sender's quorum of nodes.
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

## Byzantine-Resistant Total Ordering Algorithms

Multicast group communication protocols are used extensively in fault-tolerant distributed systems.
For many such systems, the acknowledgements for individual messages define a casual (partial) order of messages.
Maintaining the consistency of information, that is being replicated on several node instances to protect it against faults, is greatly simplified by reaching a total ordering on messages.
Algorithms that process the casual (partial) order of messages into a total order exist under the group name "Total Ordering" algorithms.

### Characteristics

Modern fault-tolerant distributed systems are often built on multicast group communication protocols, which impose a partial order of messages and on many of them information is replicated on several (or all) processors in order to achieve faster access and protection against faults.
Maintaining the consistency of replicated information is an important and problematic aspect of the implementations, and it can be simplified by imposing a total order, rather than a partial order, on messages.
The total order on messages ensures that exactly the same updates are applied to the replicated data in the same order at all nodes.

The Total Ordering algorithms are executed independently and concurrently by each process and each of them incrementally creates a total order, without requiring the transmission of any additional messages in the general case.
They operate in the presence of asynchrony and communication faults, which may cause different processes to receive messages in different orders, and they tolerate both crash and Byzantine process fault.
The system is asynchronous in the sense that no bound can be placed on the time required for a computation or for a communication of a message.
A crash fault causes a process to stop processing and generating messages.
A Byzantine fault is some arbitrary behavior by the faulty process, including generating of messages intended to mislead the other processes.

The total order ensures consistency by having the same updates applied to the replicated pieces of data in the same order at all sites.

Each broadcast message has a header consisting of a process identifier, a message sequence number and payload content.
A typical implementation might use a digital signature to substantiate the source of a message (process identifier) for each node.
Systems capable of tolerating Byzantine faults must be able to confirm the source of a message and to detect messages that have the same header but different contents and/or acknowledgements within the message.
Another option is to also embed a digest of the message content in its identifier, so that acknowledgements of messages with different contents but same sequence number will be regarded as acknowledgements of distinct messages.
The acknowledgement and re-transmission mechanisms can ensure that all versions of a message are delivered to all destinations.
Digital signatures and digests are computationally expensive, but do not require the additional rounds of message exchange of alternative strategies.

The output of the algorithms is a total order of messages that is identical at all non-faulty processes in the system.
Messages from non-Byzantine processes follow all previously broadcast messages.
The relation between messages is transitive, and since each message also follows itself, it is reflexive as well.
Messages need to acknowledge previously broadcast messages to be considered non-faulty.

These requirements cannot be assured for messages from Byzantine processes, because such process might have transmitted multiple messages that acknowledge the same or different prior messages or even transmitted messages of which other processes are unaware.
A Byzantine process can also originate a message that occurs within a cycle in the partial order.
A non-Byzantine process cannot originate a message that occurs in a cycle, because a message from a non-Byzantine process cannot acknowledge another message that itself acknowledges the first one.
A non-faulty process executing a Total ordering algorithm does not advance a message to the total order unless it has already advanced to the total order all the messages that precede it in the partial order.
Thus, a non-faulty process does not advance to the total order any message in a cycle, or any message that follows a message in a cycle.
If a Byzantine process ever participates in a cycle, then none of its subsequent messages will be ordered.

A Byzantine process can originate two or more concurrent messages with the same header, but with different contents and/or acknowledgements, neither of which follows the other and possibly each of which purports to follow different messages.
These messages are called _mutants_.

The input to the Total Ordering algorithms is a Byzantine partial order of messages and the output is a total order of messages.
Only information derived from the Byzantine casual order it used to construct the total order, meaning that no additional communication between processes is required.

A "candidate message" is a message from the process's Byzantine casual order that is not yet in the total order but only follows a message from it.
A set of candidate messages is called a "candidate set".
Thus, messages that occur in a cycle cannot be candidate messages, however mutant messages that are not in a cycle can be candidate messages and even members of the same candidate set.
Except for mutants, at most one candidate message from a given process is considered for the next advancement to the total order.

Prefixes in the Byzantine casual order are subsets of messages that are related to each other by their follows relations and can contain multiple messages followed by a single one.
The size of a Byzantine casual order is the number of messages in that subset at any point.
A prefix in the total order of messages is a set of the messages that are totally ordered and can be represented as a finite sequence.
The _internal state_ of a process consists of a prefix of the Byzantine casual order and a prefix of the total order, and the _initial state_ consists of an empty prefix of the Byzantine casual order and an empty prefix of the total order.

Messages in the Byzantine casual order prefix (except those in a cycle) provide votes on the candidate sets.
These votes are not contained explicitly in the messages, but are deduced from the partial relationship between messages.
Messages relation in most of the cases is represented by a DAG structure, so votes are implicitly counted using edges between vertices from each position towards its ancestry and progeny.
A non-faulty process decides to advance a candidate set to the total order based on the votes of the messages in its partial order prefix and each process makes its own decisions independently.
Even though the messages can be delayed or lost, every non-faulty process must decide on the same message by which to extend to the total order.

An _execution step_ consists of adding one message to the Byzantine casual (partial) order prefix and executing the ordering algorithms.
In a step, all candidate sets that can be constructed from the candidate messages in the casual order prefix are considered.

Examples and further specifications of [Total Ordering algorithms](https://core.ac.uk/display/82213948) specify conditions for each one of these algorithms and include calculations on their communication complexity and performance.

The algorithms employ a multistage voting strategy to achieve agreements on the total order and exploits the random structure of the casual order to achieve probabilistic termination.
They ensure that non-faulty processes construct identical total orders on messages and that non-Byzantine processes construct consistent total orders, provided that the resilience requirements are satisfied.

A process has no control over which message extends its casual order prefix in a step, and it cannot determine whether any future extension of its casual order prefix will involve a message that follows a particular message.
The asynchronous nature and faulty behavior of the process and the communication medium are reflected in the Byzantine partial order of messages that is input to the algorithms.

### Aleph Algorithm

One of the examples of Total Order algorithms with total ordering execution is [Aleph](https://arxiv.org/abs/1908.05156).
Aleph is a leaderless, fully asynchronous, Byzantine Fault Tolerant consensus protocol for total ordering of messages that are exchanged between entities in a network.
Consequently, it is based on a distributed construction of a partially ordered set and the execution step - the algorithm for reaching a consensus on its extension to a total order.
To achieve a consensus, the processes perform computations based on only a local copy of the data structure, however, they are bound to end up with the same result as required by the total ordering process.

The nodes in Aleph protocol network send arbitrary messages to each other using a combination of multicast and random gossip for communication.
Each node knows the identities of all other nodes and identifies itself through its public key, for which it holds a private key.

From the high level perspective Aleph consensus algorithm functions on a set of nodes that listen for transactions.
The nodes need to agree on a total ordering of the received transactions arriving in some [unreliable streams of data that the nodes observe locally](#Characteristics).
The set of transactions that each one sees is a growing blockchain that does not have a built-in notion of finality and the streams of data are an advancing sequence of blockchain "tips" that each of the participants sees.

The problem of ordering transactions in asynchronous types of protocols, where they are received with some time delay and in mixed order, is also known as _Atomic Broadcast_.
It describes the situation where a set of nodes commonly constructs a total ordering of a set of transactions, where these transactions might not arrive all at the same time.
Every time a node receives a transactions, it processes it and places it at its position.
The sequence of transactions must be output in an incremental manner, meaning that before putting a transaction at some position, previous positions must have their respective transactions set.
It is assumed that whenever an honest node outputs a transaction at some position, every other honest node will also eventually output the same transaction for this position.
Also, every transaction that is input at some honest node is eventually output by all honest nodes.
This ensures that not only all honest nodes will produce the same ordering, but also that no transaction is lost due to censorship because all honest nodes will output it.

So Aleph produces a single stream of data that "combines" all the individual streams of the nodes and is consistent among them, meaning the nodes produce the same unique sequence of finalized blocks.
As a Byzantine Tolerant type of protocol, Aleph implements Atomic Broadcast over a set of `3f+1` number of nodes, where `f` denotes the number of dishonest nodes.
The protocol deals with an expected latency for each transaction, which is the time it is output by all honest nodes after it has been input to at least some preset number of nodes.

The consensus separates the network communication from the protocol logic.
The only role for the network layer is to maintain a common data structure in the form of a DAG tree among all nodes.
The purpose of using it is to divide the algorithm execution into separate virtual rounds.
In each round the nodes emits exactly one "unit" that should be thought of as a message broadcast to all the other nodes, and it is a vertex position on its local DAG version.
Every unit contains information about its creator, its parents and additional data that can be included in it.
All the nodes are expected to generate such units and maintain their local copies of the common DAG, to which new units are continuously added.
Aleph uses transitive properties of the underlying DAG.
Each unit has a "DAG-round" that is defined by the maximum length of a downward chain starting from the current transaction, meaning getting as long as possible path of its ancestors.
A unit with no parents has a DAG-round of `0`, otherwise it has a DAG-round equal to the maximum of DAG-rounds of its parents plus `1`.
Every node should create one unit in every round, and it should do so after learning a large portion (at least `2f+1`) of units created in the previous round.

Then the protocol is stated entirely through [combinatorial properties of this structure](https://cardinal-cryptography.github.io/AlephBFT/how_alephbft_does_it.html)
Each of the nodes votes on the units it sees and decisions on accepting it or not are binary.

In a network running Aleph protocol, where a steady inflow of transactions is assumed, the communication complexity achieved is `O(N² logN)`.
There is no requirement for a trusted dealer to distribute certain cryptographic keys among nodes, but Aleph implements an ABFT Randomness Beacon with a trustless setup.

Aleph protocol is an example of a Total Ordering algorithm that creates a total order of blocks, thus creating a finalized chain at run, and keep its DAG history records only locally.
This gives to the nodes a higher level of independence when executing the algorithm, and the weight of synchronizing towards reaching the same total order of blocks is transferred to the combinatoric selection of messages from the Byzantine flow.

### Blockmania

Blockmania is also a leaderless Byzantine Fault Tolerant consensus protocol, detailed in the [Blockmania academic paper](https://arxiv.org/abs/1809.01620).
It is a part of the [Chainspace platform](https://chainspace.io/docs/#introduction), but can be used on its own.
Blockmania is another example of a Total Ordering algorithm.

Typically, clients are external to the network nodes and emit transactions that need to be agreed upon.

Nodes in the network maintain an asymmetric signature scheme and all others can authentically verify their messages via public keys.
Blockmania keeps consistent blocks history using a DAG structure, that is subsequently interpreted by each node separately.

The basic inter-node network operation in Blockmania consists of each node periodically broadcasting a block to all other nodes.
The nodes are loosely synchronized, using a [byzantine clock synchronization](https://lamport.azurewebsites.net/pubs/clocks2.pdf) and emit blocks at regular intervals.
Blocks from each node form a sequence and each refers to the hash of the previous one, forming a hash chain.
A block contains information about its creator node, its sequence number in the chain, a sequence of entries representing the content of the block.
Each block emitted by a node is first a _candidate_ block and contains a position defined by its creator node, its sequence number and its content.
Each entry in the block content is either a transaction received from a client or a reference to the hash of a valid block received from another node.
The full block is signed using the private key of its creator and broadcast to all other nodes.

Besides broadcasting blocks, nodes listen to requests from other nodes for specific blocks, and respond by sending them the full contents of the blocks requested.

An honest node includes all transactions received from clients as entries into a block it emits.
Also, all honest nodes receive and checks for validity all blocks they directly or indirectly reference in their created blocks.
This means that an honest node receives and stores locally a copy of the full DAG record, starting from the genesis vertex until the last block it emits.
And if an honest node emits a block, all honest nodes eventually receive and consider it valid (after validating) as well as all blocks that it references directly or indirectly in its content entries.

A dishonest node may contradict itself and send two distinct blocks for the same position in the chain or not send a block for a position for some time, or ever.
All honest nodes reference all valid blocks in their own blocks, including contradictory ones, and proceed with the next phase of the protocol called "interpretation".

The aim of the interpretation phase is for all honest nodes, despite holding a different subset of the DAG record at any point, to eventually agree on a specific block for any position, or alternatively, to agree that there is no such block, and to produce the next position in the total ordering of blocks.
Blockmania is created as a simplified variant of PBFT algorithm and all parties need to agree on a single position rather than the sequence and all nodes perform their interpretation process independently.

For reaching a decision for a single position, Blockmania runs an abstract terminating reliable broadcast protocol that is never materialized in real exchanged messages but rather the structure of the block DAG is interpreted by each node.
At any time, a node can emit a message to propose a block in a _pre-prepare_ message to all nodes.
Honest nodes only propose one block for any given position.
This message contains a _view_ value that represents the change in the blockchain state and has a sequence number starting from zero.
Upon receiving the _pre-prepare_ message, nodes that have the same view broadcast a _prepare_ message that contain their identity, view sequence number, block hash and content.
If the view sequence number is zero, meaning this is the first proposed view change, the proposer signature is checked before accepting message.
If the node does not have the same view interpreted, it does not send out _prepare_ message, but sets the received _pre-prepare_ and _prepare_ messages for that view in its input records buffer.
Upon receiving `2t` _prepare_ messages for a view and a corresponding _pre-prepare_ message in its input buffer, nodes broadcast _commit_ messages for that view.
After receiving `2t+1` number of _commit_ messages (also saved inside the input buffer), nodes consider view with the next position "decided" and update their local DAG versions.

The consensus assumes that if two honest nodes reach a decision about a position in the chain, defined by block creator and a block sequence number, they always reach the same decision, assuming at most `f` byzantine nodes.
A decision is eventually reached for any position in the total order of blocks by all honest nodes.

The Blockmania considers a reliable transmission between honest nodes to be a Byzantine network that may arbitrarily delay blocks, but where they eventually get delivered.
In practice, this is achieved using re-transmissions.
Each node monitors chains of other nodes and in case their own blocks are not included into those within an estimated round trip time, they may re-transmit them.
Also, an honest node may request missing blocks from other nodes, when those have been included in their chains as valid.
Nodes use the gossip protocol to communicate with each other and do not need to know all participants in the network to reach to a decision.

The goal of Blockmania protocol is to ensure that all honest nodes arrive at the same ordering of blocks, despite the presence of Byzantine nodes.
Blockmania may be used with a Proof of Stake system by dynamically defining the quorum of nodes that reach agreement on a block before committing (`t`), thus two honest parties with a partial view of the Block DAG, can reach an agreement on which blocks will be accepted by all.

In the Blockmania consensus, nodes update their block history using a DAG tree available to all, parts of which they just keep as copies in their local memory.
The consensus is built in a modular manner and combines the quorum-based reaching of agreement with the total ordering creation, meaning it is one step further from PBFT types of algorithms.

### SafetyScore

[SafetyScore consensus](https://safetyscore.app/whitepaper#kairos) is an example of an algorithm that builds a total ordered ledger, inspired by the idea of an epidemic spread of a virus that is hard to detect.
It is a decentralized protocol that disseminates risk measures among the network participants based on their interactions with a disease.
It is basically a digital tracing algorithm that can be used for all kinds of applications, while preventing leaks of personal data of network participants.

It is Byzantine Fault tolerant, permissionless consensus that aims to produce a global consistent totally-ordered log of records, which is the finalized blockchain state at any time.
Network can tolerate up to `f` faulty nodes for total number of network participants `2f+1`.

The database is a distributed ledger.
Each node generates its own sequence of blocks and all nodes collectively write history to a DAG structure that acts as an underlying representation of a distributed ledger current state.

Nodes that can validate blocks published on the distributed ledger are known as validator nodes.
The right to become a validator is earned by nodes that are operating reliably and are as far away as possible from each other in order to increase the overall resilience of the network.

Nodes use asymmetric model of digital signatures that are used for authentication of messages they transmit and use shared secrets derived from ECDG X25519.

In SafetyScore certain organizations are able to run specialized nodes that act as a distributed notary.
They perform roles similar to notaries in the real world and act as witnesses ensuring the documents and transactions are properly executed.
The right to operate a Distributed Notary is limited to organizations that have had a track record of working to preserve digital rights and freedoms, e.g. Electronic Frontier Foundation, Mozilla Foundation, Open Rights Group, etc.
The distributed notary nodes have access to some confidential information like link between two activity keys and the operators should push back against parties looking to de-anonymize users.
Participants in the distributed notary entity also enforce punishments to network nodes that violate the protocol, after a valid claim for the faulty act is presented.

Each node validation of blocks is based on [Kairos algorithm](https://kairos.com) for face recognition.
The IDs and the public keys of initial validators set are used to bootstrap the network.

The Kairos algorithm uses each of the participants IDs to calculate their risk rate of infection based on interactions with other participants that also have some non-zero risk records in the network.
Among other specifics, risk is calculated based on time after last contact and current level of recent interactions as well as on risk tokens calculated for each calendar day.
The maximum proximity distance allowed, that devices take into consideration when calculating interactions, is 3 meters of each other.

Nodes act as servers to clients and receive requests from them and can read from and write to distributed ledger
Nodes public keys are broadcast over the network and participants do not need to know all the others in the network.

In the case of SafetyScore, transactions and data coming from clients are records that are preset to be sent by software on different types of devices.
The aim of the algorithm is to build a total order of records with metadata from the incoming Byzantine casual order of messages.

SafetyScore is not a network for records of any value transfer, but rather a network for keeping records of history activity that is available to a certain number of participants in the network.
It is built as a Total Ordering algorithm in its consensus process in the sense that it outputs a total ordered blocks with data that contain references to records that all network participants agreed on.
After the ledger is updated with the latest record, all nodes update their local copies.

SafetyScore is an example of a blockchain network with multiple logical layers, that implements the total ordering of blocks for its network layer and validator committee on its communication layer.
It shows that the total ordering in an asynchronous environment, not only can be successfully reached, but also included as a building block in a much more complicated network of records.
