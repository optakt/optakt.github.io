# Avalanche

An internet-scale electronic payment system, evaluated in a large scale deployment, based on the Snow family of algorithms.
Avalanche uses a DAG tree for transactions ancestry relation and some optimizations in terms of recursive query of each transaction parents and children, that are described below.
However, the validation of transaction scripts, UTXO references and related specifics is left to the application implementation.

A more thorough description of Avalanche specifications can be found in the [Avalanche Whitepaper](https://assets-global.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf).

TODO: elaborate more on the introduction

## Characteristics

The Avalanche network operates with multiple single-decree instances (nodes), meaning that each instance can make a decision on only one value.
Nodes use the [Snowball algorithm](./concepts.md#Snow-Algorithm-Group-of-Protocols) for their decision-making process.
In the case of a payment system and Avalanche, the Snowball logic used to determine a value is the process of deciding whether a transaction is valid.

The protocol maintains the set of all known transactions using a dynamic, append-only [Directed Acyclic Graph](../../../glossary.md#Directed-Acyclic-Graph-(DAG)) (DAG) structure.
There are two main positives from this:

* DAG streamlines the path on which there are no conflicting transactions — a single vote on a DAG vertex implicitly votes for all transactions on the path to the genesis vertex
* using the same principle as Bitcoin Network (BTC), the DAG tree renders past decisions on the path of transactions as more and more difficult to undo without approval of the correct nodes

The genesis block in the DAG structure is represented by the graph first link, called "genesis vertex".
This is the point where the first two edges of the graph meet.

When a new transaction is created, its parents are defined in the DAG structure as its ancestors and the transcation is their newly created descendant.
The parent-child relationship in DAG is not mandatory for a child transaction may not have any relation to its parents transactions outputs.
It can actually spend funds received in other transactions from its [ancestor set](../../../glossary.md#Ancestor-Set).
This is the set of all transactions reachable via parent edges throughout history.
Oppositely — the term "progeny" means all existing (and potentially existing) children transactions and their children.

The main purpose of a consensus protocol is to avoid the inclusion of conflicting transactions into the ledger.
Even though the characteristics of a conflicting transaction are defined on an application level (ex. transactions that spend same UTXO), the notion of conflict can be abstracted in order to define the conflicting set.
In Avalanche, every transaction belongs to a conflict set.
This set consists of multiple transactions that are invalid towards each other or it is a singleton set - then it contains a single transaction (in the case of a virtuous transaction).
Since reaching a consensus means to avoid any conflicts between transactions, only one transaction from each conflict set can be included in the approved path of transactions.

Avalanche instantiates a Snowball instance for each conflict set.
Avalanche treats the concept of repeated queries and multiple counters from Snowball taking advantage of the DAG structure.
Specifically, when a transaction `T` is queried, all of its ancestors reachable through the DAG edges from it are implicitly part of the query.
Consequently, nodes would respond positively to a query for transaction `T` only if `T` and all of its ancestors are the preferred option in their respective conflict sets.
If more than a given threshold of responders vote positively, the transaction gets a [chit](../../../glossary.md#Chit).
After that nodes compute their level of confidence as the sum of transaction chit and the chits its progeny, meaning they query the transaction just once and rely on new vertices and possible chits added to the progeny to build up their confidence.

Described ties are broken in case of preference for first-seen transactions. TODO: explain this
Chits can also be decoupled from the DAG structure, making the protocol immune to attacks where the attacker generates large padded subgraphs.

## Consensus Specification

Each node `u` keeps track of all transactions it has learned about in set `Tu`. The set `Tu` is [partitioned](../../../glossary.md#Partition-of-a-Set) into mutually exclusive conflict sets — `PT` (partition of `T`), and `PT ∈ Tu`.
Since conflicts are [transitive](../../../glossary.md#Transitive-Relation), if `PTi` and `PTj` are conflicting, they belong to the same conflict set of transactions — `PTi = PTj`.
Conflicting transactions have the equivalence relation because they are equivocations spending the same UTXO.

The parent connection to `T'` from `T` is expressed as `T' <— T`.
The relation `T' *<— T` is its reflexive [transitive closure](../../../glossary.md#Transitive-Closuse-of-a-Graph) meaning that there is a path from `T` to `T'`.
If `T' <— T`, then every node in the system that has `T` also has `T'` and knows about the same relation `T' <— T`.
If such a relation between these transactions do not exist, then no node ends up with `T' <— T`.

DAGs built in different nodes are guaranteed to be compatible, though at any moment in time they might not have a complete view of all vertices in the system.
Each node can compute a confidence value `d U (T)` (confidence value for node `u` about transaction `T`) from the progeny as the sum of transaction T descendants chits and its own chit.
Each transaction initially has a chit of 0 before the node gets the query results.
If the node collects a threshold of α 'yes' votes from the query the value of transaction `T`, chit is set to `1`, otherwise it will remain `0`.
A chit value reflect the result from a single query of its associated transaction and becomes immutable afterwards, while confidence `d` can increase as the DAG grows by collecting more chits in its progeny.
Additionally, each node maintains its own local list of known nodes: Nu — nodes known to `u` is a subset of the set of all existing nodes: `Nu ⊆ N`.

Each node implements an event-driven state machine, centered around a query that serves both to solicit votes on each transaction and to notify other nodes of the existence of newly discovered transactions.
When a node `u` discovers a transaction `T` through a query, it starts a one-time query process by sampling `k` random peers and sending a message to them, after `T` is delivered to the node by the `onReceiveTx` callback.
Node `u` answers a query by checking whether each `T'` such that `T' *<— T` is currently preferred among competing transactions `∀T'' ∈ PT'` from conflict set `P` of `T'`.
If every single ancestor `T'` fulfills this criterion, the transaction is said to be strongly preferred and receives a yes-vote (1).
A failure on this criterion at any `T'` yields a no-vote (0).
When `u` accumulates `k` responses, it checks whether there are `α` yes-votes for `T`, and if so grants the chit value (`cT = 1`).
This process yields a labeling of the DAG with a chit value and associated confidence for each transaction `T`.

Example of (chit,confidence) values:

```text
T1(1,6)
├── T2(1,5)
│    ├── T4(1,2) ── T8(1,1)
│    │               │
│    └────────────── T5(1,3)
│                    └──── T9(1,1)
│
└── T3(0,0)
    ├── T6(0,0)
    └── T7(0,0)
```

Conflict set `P` for `T1` has one element `T1`.
Conflict set `P` for `T2` and `T3` has two elements `{T2, T3}`.
Conflict set `P` for `T7`, `T6` and `T9` has 3 elements `{T6, T7, T9}`.

Similar to Snowball, sampling in Avalanche creates a positive feedback for the preference of a single transaction in its conflict set.
When a transaction has a larger confidence, its descendants are likely to collect chits in the future compared to others with lower confidence.
For example `T2` has higher confidence than `T3` and its descendants are more likely to collect chits in the future.

Similar to Bitcoin, Avalanche leaves determining the acceptance point of a transaction to the application.
Committing a transaction can be performed through a safe early commitment.
For virtuous transactions, `T` is accepted when it is the only transaction in its conflict set and has a confidence no less than a threshold of `β1`.
If a virtuous transaction fails to get accepted due to a problem with parents, it could be accepted if reissued with different parents.

### Transaction Entangling

How Avalanche entangles transactions:

```text
init:
    `Set T` = Ø set of known transactions (empty set)
    Q = Ø set of queried transactions (empty set)

generateTx:
    edges = {T' <— T: T' ∈ parentSelection(`Set T`)}
    T = Tx(data, edges)
    onReceiveEvent(T)

onReceiveEvent(t):
    if T ∉ `Set T` then
        if PT (conflict set of T) = Ø then
            PT = {T}, Pt.pref = T
            PT.last = T, PT.cnt = 0
        else PT = PT U {T}

        `Set T` = T U {T}, cT = 0
```

Because transactions that generate and consume the same UTXO do not conflict with each other, any transaction can be reissued with different parents.

### Node Main Loop

Protocol main loop executed by each node:

```text
AvalancheLoop:
    while true do
        find T that satisfies T ∈ `Set T` AND T ∉ Q
        K = sample(N\u, k)
        P = SUM(v ∈ k) query(v, T)

        if P ≥ α then
            cT = 1 (chit for transaction T)
            update preferences for ancestors:
            for T' ∈ `Set T`: T' *<— T do
                if T' confidence is larger than the confidence of preferred transation T` in the P conflict set - update the prefered transaction of the set:
                if d(T') > d(PT'.pref) then
                    PT'.pref = T'
                if T' is not the last transaction in the T' conflict set - update the last place with it
                if T' ≠ PT'.last then
                    PT'.last = T', PT'.cnt=1
                else
                    ++PT'.cnt
        else
            for T' ∈ `Set T`: T' *<— T do
                PT'.cnt = 0

            otherwise confidence cnt remains 0 forever
            Q = Q U {T} - mark T as queried
```

In each iteration the node tries to select a transaction that has not yet been queried.
If no such transaction exists, the loop stalls until a new transaction is added to `Set T`.
When a the new transaction is added, the node then select a number of peers `k` and queries them.
If more than `α` of these peers return a positive response, the chit value is set to `1`.
After this step, the loop updates the preferred transaction of each conflict set of the transactions in its ancestry.
Next, `T` is added to the set `Q` of queried transactions so it is never queried again by the node.

### Query for Transaction

What happens when a node receives a query for transaction `T` from peer `j`:

```text
isPreferred(T):
    return T = PT.pref

isStronglyPreferred(T):
    return ∀T' ∈ `Set T`, T' *<— T : isPreferred(T')

isAccepted(T):
    return
        ((∀T' ∈ `Set T`, T' <— T: isAccepted(T'))
            ∩ |PT|=1 ∩ PT.cnt ≥ β1)  safe early commitment
        ∪ (PT.cnt ≥ β2)              consecutive counter

onQuery(j, T):
    onReceiveTx(T)
    respond(, isStronglyPreferred(T))
```

The first step is to add transaction `T` to `Set T` unless it is already included.
After that step, `u` determines whether `T` is currently strongly preferred, in which case `u` returns a positive response to peer `j`, otherwise it returns a negative response.
In the example pseudocode it is assumed that if a node knows about `T`, then it recursively knows about `T`'s ancestry.
This can be achieved by postponing the delivery of `T` until its entire ancestry is recursively fetched (in practice an additional gossip process that disseminates the transaction is used in parallel).

## UTXO Graph

In addition to the DAG structure used for transactions data, another graph is used for [Unspent Transactions Outputs (UTXO)](../../../glossary.md#UTXO-Set) set, that captures the available outputs.
It is used to realize the ledger for the payment system.
In the present specifications transactions that encode the data for money transfer are called "transactions", and transactions `T ∈ Set T` are DAG vertices.

Transaction and address mechanisms are inherited from Bitcoin.
In its simplest form, a transaction consists of multiple inputs and outputs with the corresponding redeem scripts.
Addresses are identified by the hash of their public keys, and signatures are being generated by the correponding private keys.

UTXO are fully consumed by a valid transaction and may generate new UTXOs spendable by the named recipients.

Multiple-input transactions consume multiple UTXOs and in Avalanche may appear in multiple conflict sets.
The conflict relation of transaction input pairs is transitive, because each pair only spends one unspent output.
Each input of the transaction is checked by the logic of isAccepted(T) step, consequently a transaction is accepted only if all its input pairs are accepted in their respective Snowball conflict sets.
The DAG of transaction-inputs is implemented in such a way that multiple transactions can be batched together per query.

## Evaluation of Performance

TODO: add

## DAG Positives in Context of Avalanche

TODO: add

## Cryptography Bottleneck

TODO: add

## Snowball Comparison

TODO: add
