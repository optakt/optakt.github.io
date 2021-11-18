# Avalanche

An internet-scale electronic payment system, evaluated in a large scale deployment, based on the Snow family of algorithms.
The system's bottleneck is transaction verification.
A more thorough description of Avalanche specifications can be found in [Avalanche Whitepaper](https://assets-global.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf)
TODO: elaborate more on the introduction

## Characteristics

The Avalanche network operates with multiple single-decree instances (nodes), meaning that each instance can make a decision on only one value.
Nodes use the [Snowball algorithm](./concepts.md#Snow-Algorithm-Group-of-Protocols) for their decision-making process.
In the case of a payment system and Avalanche, the Snowball logic used to determine a value is the process of deciding whether a transaction is valid.

The protocol maintains the set of all known transactions using a dynamic, append-only Directed Acyclic Graph (DAG) structure.

There are two main positives from this:

* DAG helps streamlining the path on which there are no conflicting transactions — a single vote on a DAG vertex implicitly votes for all transactions on the path to the genesis vertex
* using the same principle as BTC, the DAG tree renders past decisions on the path of transactions as more and more difficult to undo without approval of the correct nodes

The genesis block in the DAG structure is represented by the graph first link, called "genesis vertex".
This is the point where the first two edges of the graph meet.

When a new transaction is created, its parents are defined in the DAG structure as its ancestors and the trascation is their newly created descendant.
The parent-child relationship in DAG is not mandatory for a child transaction may not have any relation to its parents transactions outputs.
It can actually spend funds received in other transactions from its "ancestor set".
The [ancestor set](../../../glossary.md#Ancestor-Set) is the set of all transactions reachable via parent edges throughout history.
Oppositely — the term "progeny" means all existing (and potentially existing) children transactions and their children.

The main purpose of a consensus protocol is to avoid the inclusion of conflicting transactions into the ledger.
Even though the characteristics of a conflicting transaction are defined on an application level (ex. transactions that spend same UTXO), the notion of conflict can be abstracted in order to define the conflicting set.
In Avalanche, every transaction belongs to a conflict set.
This set consists of multiple transactions that are invalid towards each other or it is a singleton set - then it contains a single transaction (each transaction has potential conflict transactions that are going to be created).
Since reaching a consensus means to avoid any conflicts between transactions, only one transaction from each conflict set can be included in the approved path of transactions.

Avalanche instantiates a Snowball instance for each conflict set.
Avalanche treats the concept of repeated queries and multiple counters from Snowball taking advantage of DAG structure.
Specifically, when a transaction `T` is queried, all of its ancestors reachable through the DAG edges from it are implicitly part of the query.
Consequently, nodes would respond positively to a query for transaction `T` only if `T` and all of its ancestors are the preferred option in their respective conflict sets.
If more than a given threshold of responders vote positively, the transactions gets a [chit](../../../glossary.md#Chit).
TODO: finish

## Consensus Specification

TODO: add

## UTXO Graph

TODO: add

## Evaluation of Performance

TODO: add

## DAG Positives in Context of Avalanche

TODO: add

## Cryptography Bottleneck

TODO: add
