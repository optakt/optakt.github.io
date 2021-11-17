# Glossary

## General Blockchain Concepts

### Consensus

TODO: add

### Blockchain

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

Transaction input is a part of the transaction structure and it identifies (often by reference) which UTXO will be consumed.
It provides a proof of ownership using an unblocking script.

First part of the input is usually a pointer/reference to the UTXO, and the second part is the unlocking script (usually contructed by the wallet) that satisfies the spending condition set in the UTXO(s) that the transaction is going to consume.
Input data structure can have arbitrary fields inside it depending on the blockchain network implementation, but they generally are:

* txid - referencing the transaction id that contains the UTXO
* index referencing which UTXO from that transaction is going to be spent
* scriptSig - a script signature that satisfies the spending conditions; known as "unlocking script"
* a sequence number

An input may or may not reference any nominal value or other context depending on the organization.
What it must contain is the unlocking script (witness part) that unclocks the UTXO.

### Transaction Output

The transaction output is the fundamental building block of a blockchain transaction.
Outputs are indivisible chunks of value denominated in the blockchain network exchanged value/currency, recorded on the blockchain and recognized as valid by the entire network.
An output can contain an arbitrary value, but once created it is indivisible — it can be consumed only in its entirety by the transactions.
Outputs are discrete and indivisible units of value.

Transaction outputs consist generally of two parts:

* amount of the UTXO denominated value
* a cryptographic script or a public key that determines the conditions on which the UTXO can be spent; also known as the "locking script".

The locking script is a hash representaion of the a public key hash (in the simplest situation) or it is a hash of an unlocking script.
In both cases - the public key along with the signature or the unlocking script need to be presented at the time of consuming the output in order the values to be transfered.

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

A directed [graph](https://en.wikipedia.org/wiki/Graph_(discrete_mathematics)) comprises of set of vertices connected by "directed" edges — also called arcs.
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

The inverse of a transtive relation is always transitive.
See also [Homogeneous Relation](https://en.wikipedia.org/wiki/Binary_relation#Homogeneous_relation).

### Transitive Closure of a Graph

A transitive [closure](https://en.wikipedia.org/wiki/Closure_(mathematics)) of a directed graph `G` is a directed graph `G'` with an edge `(i, j)` corresponding to each directed path from `i` to `j` in `G`. The resultant directed graph `G'` representation in the form of the adjacency matrix is called the _connectivity matrix_.
