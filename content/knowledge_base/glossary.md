# Glossary

## General Blockchain Concepts

### Blockchain

This technology is based on a chain of blocks linked to each other.
Once a new block is created and inserted on the chain, the new block will have a link to the previous block, creating a chain.
They all include the hash of the previous block except for the first one, which is called the genesis block, its a zero hash value.

There are two types of blockchain, permissionless and permissioned.
The permissionless blockchain, also called public blockchain, is available to everyone, because of that, anyone can create and track down blocks.
This type of blockchain comes with some limitations, mainly slow performance and bad scaling.

On the other side, the permissioned blockchain, or often referred to as private blockchain, has limited availability, only being accessible to a certain group instead of being publicly available, they are also faster and better to scale.
On this type of blockchain, there is no anonymity like in a public blockchain.

The core concents of the a blockchain network are:

* Transparency
    * Every full node in the network has a copy of the whole chain of blocks, this means that every transaction is available to each member, making the transactions traceable. This makes every transaction more transparent and removes the need for intermediaries, allowing the process to be more efficient.
* Immutable
    * Information in the blockchain is immutable. If a block in the middle of the chain is changed, the chain is no longer valid. To be valid, the blocks in the furthen down the chain must be recomputed. To apply the previous changes in every node in the network, a consensus algorithm should be used in every block that was modified.
* Decentralization
    * This is one of the biggest advantages of the blockchain, being a decentralized system allows for the lack of a central authority to control the transactions. Every full node has a copy of the chain, which they can update with new information.  If someone wants to update the chain, they need to have the consensus of the network. This is done with a consensus algorithm, in the case of the public blockchain, the most popular is Proof Of Work (PoW). One of the main disadvantages of this type of consensus algorithm in the public blockchain is the huge amount of energy required to perform it.
    
### Consensus

TODO: add

### Gossip Algorithms

TODO: add

### Liveness Threshold

The number of malicious participants that can be tolerated before the consensus protocol stops functioning.

### Nakamoto Type of Consensus

TODO: introduce (for BTC)
A consensus algorithm that provides a probabilistic safety guarantee; its decisions may revet with some probability ε.
A permissionless protocol allowing each node to join or leave the system at any time.
TODO: finish

### Safety

TODO: add

## Avalanche Concepts

### Ancestor Set

TODO: add

### Chit

A transaction receives a chit (boolean value) when the majority (quorum) of queried nodes respond with an approval for that transaction.

### Confidence

The sum of transaction/vertex's decendants cihts and its own chit.
The chit value is a result of a one-time query for its associated transaction and becomes immutable afterwards.
Because chit values can be 0 or 1 - c ∈ {0,1} confidence values are monotonic.

### Consecutive Success Number

The number of times a transaction or its decendant received an approval from majority (quorum) of queried nodes.
TODO: add more thorough description

### Progeny

TODO: add

### Slush

TODO: add

## Other

### Graph

TODO: introduce the idea in one sentence without technicals

A graph is pair of sets G = (V, E).
V is a set of elements called vertices (singular: vertex) and E is a set of paired vertices and its elements are called edges.
An edge {x, y} contains vertices x and y and they are its endpoints.
A vertex may belong to no edge and in that case it is not joined to any other vertex.
The set of vertices in a graph is a discrete value, meaning the set of edges is also a discrete value.
The number of vertices - |V| is the "order" of the graph and the number of edges - |E| is the "size" of the graph.

### Directed Graph

A directed graph comprises of set of vertices connected by "directed" edges - also called arcs.
A directed graph is represented as a pair of sets G = (V, A), where

* V is a set of elements (vertices, nodes)
* A is a set of ordered pairs of vertices, called arcs

### Directed Acyclic Graph (DAG)

A [Directed Acyclic Graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) contains no directed cycles - each edge is directed from one vertex to another such that following those directions will never form a closed loop.
A directed graph is a DAG if and only if it can be topolgically ordered by arranging the vertices as a linear ordering that is consistent with all edge directions.
A toplogical ordering of a directed graph is an ordering of its vertices into a sequence, such that for every edge the start vertex of the edge occurs earlier in the sequence than the ending vertex of the edge.
A graph that has a topological ordering does not have any cycles, consequently every graph with a topological ordering is acyclic.
Every acyclic directed graph has at least one topological ordering.

### Distributed System

A system of computers connected by a network - local, hub-based or wide, that work together as one single computer.
Applications built with distributed systems "apear" to users and other client applications as if they are running on a single machine.
Distributed systems take advantage of each node's physical and virtual resources (horizontal scalability), minimize overload of the whole system and can have architecture aiming to achieve a certain performance level, speed or other characteristic levels needed for the application running to overcome.

### Partition of a Set

Partiotion of a set is grouping of its elements into non-empty subsets, in such a way that every element is included in exactly one subset.

### Transitive Relation

A set T with elements a, b, c, d -> A{a, b, c, d}
A relation R on the elements of X that relates
a to b,
b to c,
c to d
but also relates
a to c,
a to d,
b to d
is called transitive.

#### Example of a transitive relation R

A -> ancestor of B
B -> ancestor of C
=> A -> ancestor of C

```text
A ──> B ──> C => A ──> C
```

#### Example of a non-transitive relation R

A -> ancestor of B
B -> ancestor of C
A -> ancestor of D
=> D is NOT an ancestor of C

```text
A
├──> B ──> C
└──> D
```

The inverse of a transtive relation is always transitive.
See also [Homogeneous Relation](https://en.wikipedia.org/wiki/Binary_relation#Homogeneous_relation)

### Transitive Closuse of a Graph

A transitive [closure](https://en.wikipedia.org/wiki/Closure_(mathematics)) of a directed graph G is a directed graph G' with an edge (i, j) correspoding to each directed path from i to j in G. The resultant directed graph G' representation in the form of the adjancecy matrix is called the connectivity matrix.
