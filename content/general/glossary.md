# Glossary

## General Blockchain Concepts

### Blockchain

Blockchain is a technology for implementing a stateful immutable distributed and decentralized ledger of records.
Participants that update the ledger records represent the peers that run the blockchain network.

Peers participate in the blockchain network collaborating with their resources in the same manner as in other types of distributed network architectures.

Records in the ledger, called transactions, update the ledger state and represent any type of social exchangeable value to the participants of the network — token representing ownership, money, access to some resource or event, etc.
[Transactions](#Transaction) in the blockchain represent transferring of value from one party to the other.

The fundamental difference the blockchain provide to the participants is that their values are exchanged `without` the participation of any third parties.
For cryptocurrencies this idea drastically changes the concept of money transfers.

Ledger records - transactions, are created and recorded in the ledger using specific steps and types of encryption on different parts of them - hashing, asymmetric and symmetric encryption.

Transactions can be grouped in blocks.
Blocks are created by the peers based on transaction updates they know about and then distributed in the network.
Depending on the consensus protocol that is run in the blockchain network, blocks are eventually accepted and inserted in to the ledger.

Each block of transactions follows the previous block inserted and reference it by its ID.
This creates an immutable chain of blocks of transactions that represent the state of the ledger.

Depending on the peers' ability to participate in the blockchain network, blockchains are [public and private](https://101blockchains.com/public-vs-private-blockchain/).

In public blockchains everyone is allowed to participate in the network.
Peers can be of different types depending on what amount of data from the ledger they keep and how they update it, but in terms of functionality and resources they provide to the network - they are equally placed.
This means that none of the peers, or a group of them, provide any specific functionality to the network that others do not.
Consequently, when peers leave or arrive in the blockchain network — its running is not affected.
Public blockchains do not require trust in other participants in the network.

This is one of the core concepts behind the idea of distribution in the blockchain world.

Private blockchains are a specific type of blockchains that have some type of authorization scheme used for identities that can enter the network and access its records.
Private blockchains require trust in other participants of the network and some peers inside them have different levels of access compared to others.
In this kind of blockchain networks, a random peer or a group of peers leaving (malfunctioning) or arriving in the network affects the stability of the whole network.

Depending on the access to participate in the network — being granted or not, blockchains are [permissioned and permissionless](https://101blockchains.com/permissioned-vs-permissionless-blockchains/)
Generally this difference lie in `providing` access from some authority running the network, which concerns again the trust between participants.

Permissioned blockchains are networks that require permission to enter it, usually used by organizations for managing their internal processes and data.
Permissioned blockchains shifts from the core feature of decentralization in the blockchain world and from the initial idea of blockchains in general.
A permissioned blockchain can also be a public network that only allows participation based on different access levels.
The participants are usually known by a permissioned blockchain network operator, while the transaction history is not publicly accessible.

A permissionless blockchain network does not require permissions to arrive as a peer in it, but can be public or private depending on the level or data access for the participants (and in the consensus trust levels between participants).
In both permissionless and permissioned blockchains peers and groups of peers can have different roles and their presence or absence can affect the network functioning.

Initial idea of blockchain networks, as described in [Nakamoto consensus description](https://bitcoin.org/bitcoin.pdf), aims to reach the highest possible levels of transparency, decentralization, anonymity and immutability at the same time.
During time, different types of blockchains emerged and today, they advance in some characteristics but lack in others (for example anonymity).

Core concepts of blockchain according to the initial Nakamoto Consensus descriptions are:

* Transparency. Every full node in the network has a copy of the whole chain of blocks.
This means that every transaction is available to each member, making the transactions traceable (not available in private and/or permissioned blockchains today).
* Immutability. Transactions included in the ledger become immutable records (in the general case).
Immutability is present for all types of blockchains today.
* Decentralization. This is one of the biggest advantages of the blockchain, being a decentralized system allows for the lack of a central authority to control the transactions.
Every full node has a copy of the chain, which they can update with new information, and every SPV node can update their records requesting them from a full node.
Decentralization at its fullest is not present in permissioned and private blockchains today as described above.
Once a new block is created and inserted on the chain, the new block will have a link to the previous block, creating a chain.
They all include the hash of the previous block except for the first one, which is called the genesis block and has a zero hash value.

### Staking

Staking is putting some native cryptocurrency as collateral to become a validating entity in the network.

### Consensus

TODO: add

### Gossip Algorithms

TODO: add

### Liveness Threshold

The number of malicious participants that can be tolerated before the consensus protocol stops functioning.

### Nakamoto Type of Consensus

TODO: introduce (for BTC)
A consensus algorithm that provides a probabilistic safety guarantee; its decisions may revert with some probability `ε`.
A permissionless protocol allowing each node to join or leave the system at any time.
TODO: finish

### Safety

TODO: add

### Transaction

Transactions are data structures that encode the transfer of value between participants in the blockchain system.
A transaction consumes previously recorded unspent transaction outputs and creates new transaction output(s) that can be consumed by future transactions.
This is the way in which chunks of value move forward from one owner to the next one in the blockchain network.
The two main parts of the transaction that represent the chain between available value and spent value are outputs and inputs.

### Transaction input

Transaction input is a part of the transaction structure, and it identifies (often by reference) which UTXO will be consumed.
It provides a proof of ownership using an unblocking script.

First part of the input is usually a pointer/reference to the UTXO, and the second part is the unlocking script (usually constructed by the wallet) that satisfies the spending condition set in the UTXO(s) that the transaction is going to consume.
Input data structure can have arbitrary fields inside it depending on the blockchain network implementation, but they generally are:

* `txid` — referencing the transaction id that contains the UTXO
* `index` — referencing which UTXO from that transaction is going to be spent
* `scriptSig` — a script signature that satisfies the spending conditions; known as "unlocking script"
* a sequence number

An input may or may not reference any nominal value or other context depending on the organization.
What it must contain is the unlocking script (witness part) that unlocks the UTXO.

### Transaction Output

The transaction output is the fundamental building block of a blockchain transaction.
Outputs are indivisible chunks of value denominated in the blockchain network exchanged value/currency, recorded on the blockchain and recognized as valid by the entire network.
An output can contain an arbitrary value, but once created it is indivisible — it can be consumed only in its entirety by the transactions.
Outputs are discrete and indivisible units of value.

Transaction outputs consist generally of two parts:

* amount of the UTXO denominated value
* a cryptographic script or a public key that determines the conditions on which the UTXO can be spent; also known as the "locking script".

The locking script is a hash representation of a public key hash (in the simplest situation) or it is a hash of an unlocking script.
In both cases - the public key along with the signature or the unlocking script need to be presented at the time of consuming the output in order the values to be transferred.

### UTXO Set

This is the set of all unspent transaction outputs that are available to the whole network.
The UTXO set grows when a new transaction output is created and shrinks when UTXO is consumed.
At any moment of time the set represents the UTXO that are available to be spent by the network participants.

## Avalanche Concepts

### Ancestor Set

TODO: add

### Chit

A transaction receives a chit (boolean value) when the majority (quorum) of queried nodes respond with an approval for that transaction.

### Confidence

The sum of transaction/vertex's descendants chits and its own chit.
The chit value is a result of a one-time query for its associated transaction and becomes immutable afterwards.
Because chit values can be `0` or `1` — `c ∈ {0,1}` confidence values are monotonic.

### Consecutive Success Number

The number of times a transaction or its descendant received an approval from majority (quorum) of queried nodes.
TODO: add more thorough description

### Progeny

TODO: add

### Slush

TODO: add

## Flow Concepts

### Quorum Certificate

A process by which nodes using the HotStuff consensus algorithm submit signed messages in order to generate a certificate for bootstrapping HotStuff. Each collector cluster runs a mini-version of HotStuff, and since clusters are randomized each epoch, a new quorum certificate is required for each cluster each epoch.

## Other

### Graph

TODO: introduce the idea in one sentence without technicals

A graph is pair of sets `G = (V, E)`.
`V` is a set of elements called vertices (singular: vertex) and `E` is a set of paired vertices and its elements are called edges.
An edge `{x, y}` contains vertices `x` and `y`, and they are its endpoints.
A vertex may belong to no edge and in that case it is not joined to any other vertex.
The set of vertices in a graph is a discrete value, meaning the set of edges is also a discrete value.
The number of vertices — `|V|` is the "order" of the graph and the number of edges — `|E|` is the "size" of the graph.

### Directed Graph

A directed [graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)) consists of set of vertices connected by "directed" edges — also called arcs.
A directed graph is represented as a pair of sets `G = (V, A)`, where:

* `V` is a set of elements (vertices, nodes)
* `A` is a set of ordered pairs of vertices, called _arcs_

### Directed Acyclic Graph (DAG)

A [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) contains no directed cycles — each edge is directed from one vertex to another such that following those directions never forms a closed loop.
A directed graph is a DAG if and only if it can be topologically ordered by arranging the vertices as a linear ordering that is consistent with all edge directions.
A topological ordering of a directed graph is an ordering of its vertices into a sequence, such that for every edge the start vertex of the edge occurs earlier in the sequence than the ending vertex of the edge.
A graph that has a topological ordering does not have any cycles, consequently every graph with a topological ordering is acyclic.
Every acyclic directed graph has at least one topological ordering.

### Distributed System

A system of computers connected by a network — local, hub-based or wide, that work together as one single computer.
Applications built with distributed systems appear to users and other client applications as if they are running on a single machine.
Distributed systems take advantage of each node's physical and virtual resources (horizontal scalability), minimize overload of the whole system and can be designed to achieve a certain performance level, speed or other characteristic levels needed for the application running to overcome.

### Partition of a Set

A set's partition is a group of its elements into non-empty subsets, in such a way that every element is included in exactly one subset.

### Transitive Relation

For a set `T` with elements `a`, `b`, `c`, `d`

```text
A{a, b, c, d}
```

A relation `R` on the elements of the set that relates `a` to `b`, `b` to `c` and `c` to `d`, but also relates `a` to `c`, `a` to `d` and `b` to `d` is called transitive.

```text
a -> b, b -> c, c -> d
AND
a -> c, a -> d, b -> d
```

#### Example of a Transitive Relation `R`

```text
A -> ancestor of B
B -> ancestor of C

=> A -> ancestor of C
```

```text
A ──> B ──> C => A ──> C
```

#### Example of a Non-Transitive Relation `R`

A -> ancestor of B
B -> ancestor of C
A -> ancestor of D
=> D is NOT an ancestor of C

```text
A
├──> B ──> C
└──> D
```

The inverse of a transitive relation is always transitive.
See also [Homogeneous Relation](https://en.wikipedia.org/wiki/Binary_relation#Homogeneous_relation).

### Transitive Closure of a Graph

A transitive [closure](https://en.wikipedia.org/wiki/Closure_(mathematics)) of a directed graph `G` is a directed graph `G'` with an edge `(i, j)` corresponding to each directed path from `i` to `j` in `G`. The resultant directed graph `G'` representation in the form of the adjacency matrix is called the _connectivity matrix_.
