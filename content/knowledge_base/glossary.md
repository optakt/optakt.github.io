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
