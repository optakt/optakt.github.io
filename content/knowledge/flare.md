## Avalanche Consensus Algorithm Concepts

### Snow Algorithm Group of Protocols

A group of protocols with a strong probabilistic safety guarantee in the presence of Byzantine nodes.

#### Description

The core concept of operating is the execution of repeated sampling of the network at random, ultimately leading to correct nodes' behavior to obtain a common statement (decision, transaction value, outcome).
This mechanism effectively brings the system to an irreversible state — meaning that a large potion of the network accepted a decision or a statement.
So any such conflicting to that statement would be accepted with a negligible probability ε or smaller.
TODO: finish

A more thorough description of Snow algorithms specifications and examples can be found in the [Avalanche Whitepaper](https://assets-global.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf).

#### Comparison with Nakamoto Consensus

TODO: add

#### Specification

TODO: add

### Leaderless Byzantine Fault Tolerance (LBFT)

[Leaderless Byzantine Fault Tolerance](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2020/EECS-2020-121.pdf) combines the Snowball and Practical Byzantine Fault Tolerance (pBFT) algorithms.
It aims to take advantage of the decentralized aspect of Snowball and deterministic property of pBFT in order to eliminate the weakness of Snowballs' probabilistic property and pBFT's reliance on a "leader".

#### LBFT Specification

TODO: add

## Avalanche

An internet-scale electronic payment system, evaluated in a large scale deployment, based on the Snow family of algorithms.
Avalanche uses a DAG tree for transactions ancestry relation and some optimizations in terms of recursive query of each transaction parents and children, that are described below.
However, the validation of transaction scripts, UTXO references and related specifics is left to the application implementation.

A more thorough description of Avalanche specifications can be found in the [Avalanche Whitepaper](https://assets-global.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf).

TODO: elaborate more on the introduction

### Characteristics

The Avalanche network operates with multiple single-decree instances (nodes), meaning that each instance can make a decision on only one value.
Nodes use the [Snowball algorithm](#Snow-Algorithm-Group-of-Protocols) for their decision-making process.
In the case of a payment system and Avalanche, the Snowball logic used to determine a value is the process of deciding whether a transaction is valid.

The protocol maintains the set of all known transactions using a dynamic, append-only [Directed Acyclic Graph](glossary.md#Directed-Acyclic-Graph-(DAG)) (DAG) structure.
There are two main positives from this:

* DAG streamlines the path on which there are no conflicting transactions — a single vote on a DAG vertex implicitly votes for all transactions on the path to the genesis vertex
* using the same principle as Bitcoin Network (BTC), the DAG tree renders past decisions on the path of transactions as more and more difficult to undo without approval of the correct nodes

The genesis block in the DAG structure is represented by the graph first link, called "genesis vertex".
This is the point where the first two edges of the graph meet.

When a new transaction is created, its parents are defined in the DAG structure as its ancestors and the transaction is their newly created descendant.
The parent-child relationship in DAG is not mandatory for a child transaction may not have any relation to its parents transactions outputs.
It can actually spend funds received in other transactions from its [ancestor set](glossary.md#Ancestor-Set).
This is the set of all transactions reachable via parent edges throughout history.
Oppositely — the term "progeny" means all existing (and potentially existing) children transactions and their children.

The main purpose of a consensus protocol is to avoid the inclusion of conflicting transactions into the ledger.
Even though the characteristics of a conflicting transaction are defined on an application level (ex. transactions that spend same UTXO), the notion of conflict can be abstracted in order to define the conflicting set.
In Avalanche, every transaction belongs to a conflict set.
This set consists of multiple transactions that are invalid towards each other, or it is a singleton set - then it contains a single transaction (in the case of a virtuous transaction).
Since reaching a consensus means to avoid any conflicts between transactions, only one transaction from each conflict set can be included in the approved path of transactions.

Avalanche instantiates a Snowball instance for each conflict set.
Avalanche treats the concept of repeated queries and multiple counters from Snowball taking advantage of the DAG structure.
Specifically, when a transaction `T` is queried, all of its ancestors reachable through the DAG edges from it are implicitly part of the query.
Consequently, nodes would respond positively to a query for transaction `T` only if `T` and all of its ancestors are the preferred option in their respective conflict sets.
If more than a given threshold of responders vote positively, the transaction gets a [chit](glossary.md#Chit).
After that nodes compute their level of confidence as the sum of transaction chit and the chits its progeny, meaning they query the transaction just once and rely on new vertices and possible chits added to the progeny to build up their confidence.

Described ties are broken in case of preference for first-seen transactions.
Chits can also be decoupled from the DAG structure, making the protocol immune to attacks where the attacker generates large padded subgraphs.

### Consensus Specification

Each node `u` keeps track of all transactions it has learned about in set `Tu`. The set `Tu` is [partitioned](glossary.md#Partition-of-a-Set) into mutually exclusive conflict sets — `PT` (partition of `T`), and `PT ∈ Tu`.
Since conflicts are [transitive](glossary.md#Transitive-Relation), if `PTi` and `PTj` are conflicting, they belong to the same conflict set of transactions — `PTi = PTj`.
Conflicting transactions have the equivalence relation because they are equivocations spending the same UTXO.

The parent connection to `T'` from `T` is expressed as `T' <— T`.
The relation `T' *<— T` is its reflexive [transitive closure](glossary.md#Transitive-Closure-of-a-Graph) meaning that there is a path from `T` to `T'`.
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

#### Transaction Entangling

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

#### Node Main Loop

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
                if T' confidence is larger than the confidence of preferred transaction T` in the P conflict set - update the preferred transaction of the set:
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
When the new transaction is added, the node then select a number of peers `k` and queries them.
If more than `α` of these peers return a positive response, the chit value is set to `1`.
After this step, the loop updates the preferred transaction of each conflict set of the transactions in its ancestry.
Next, `T` is added to the set `Q` of queried transactions, so it is never queried again by the node.

#### Query for Transaction

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

### UTXO Graph

In addition to the DAG structure used for transactions data, another graph is used for [Unspent Transactions Outputs (UTXO)](glossary.md#UTXO-Set) set, that captures the available outputs.
It is used to realize the ledger for the payment system.
In the present specifications transactions that encode the data for money transfer are called "transactions", and transactions `T ∈ Set T` are DAG vertices.

Transaction and address mechanisms are inherited from Bitcoin.
In its simplest form, a transaction consists of multiple inputs and outputs with the corresponding redeem scripts.
Addresses are identified by the hash of their public keys, and signatures are being generated by the corresponding private keys.

UTXO are fully consumed by a valid transaction and may generate new UTXOs spendable by the named recipients.

Multiple-input transactions consume multiple UTXOs and in Avalanche may appear in multiple conflict sets.
The conflict relation of transaction input pairs is transitive, because each pair only spends one unspent output.
Each input of the transaction is checked by the logic of isAccepted(T) step, consequently a transaction is accepted only if all its input pairs are accepted in their respective Snowball conflict sets.
The DAG of transaction-inputs is implemented in such a way that multiple transactions can be batched together per query.

### Evaluation of Performance

TODO: add

### DAG Positives in Context of Avalanche

TODO: add

### Cryptography Bottleneck

TODO: add

### Snowball Comparison

TODO: add

## Stellar Consensus Algorithm Concepts

### Federated Byzantine Agreement (FBA)

Federated Byzantine agreement is a new model of consensus that achieves robustness through quorum slices.
It is an adaptation of Byzantine agreement to systems with open membership, where different nodes have different concepts about which group of nodes are important to them about deciding on a statement.

These groups of nodes let the individual node take decisions based on its own trust criteria.
The groups bind the system together in the same way as individual networks' peering and transit decisions unify the Internet.

#### Description

FBA is a model suitable for a worldwide consensus.
In FBA each participant knows and relies on opinion of others that it considers important.
It waits for the `vast` majority of them to agree on a transaction before considering the transaction settled.
In turn, those important participant do not agree to the transaction until the participants `they` consider important agree as well, and so on.
Eventually — enough of the network accepts a transaction, and consequently it becomes infeasible for an attacker to roll it back.
Only at that point all the participants consider it settled.

#### Stellar Consensus Protocol

Stellar consensus protocol (SCP) is a construction of FBA — an FBA system (FBAS).
It is free from blocked states, in which consensus is no longer possible, unless participant failures make it impossible to satisfy trust dependencies.

Stellar claims to be the first provably safe consensus mechanism to have the following key properties simultaneously:

* Decentralized control — anyone is able to participate and no central authority dictates whose approval is required for consensus.
* Low latency — nodes can reach consensus at timescales humans expect for web payments — a few seconds.
* Flexible trust — users have the freedom to trust any combination of parties they see fit.
* Asymptotic security — safety rests on digital signatures and hash families whose parameters can realistically be tuned to protect against adversaries with vast computing power.

#### FBA Specification

Like non-federated Byzantine agreement, FBA addresses the problem of updating replicated state, such as a transaction ledger or certificate tree.
By agreeing on what updates to apply, nodes avoid contradictory, irreconcilable states.

Each update is identified by a unique `slot` from which the inter-update dependencies can be inferred — for example slots can be consecutively numbered positions in a sequentially applied log.

An FBA system runs a consensus protocol that ensures nodes agree on slot contents.
A node `u` can safely apply update `x` in slot `i` when it has safely applied updates in all slots upon which `i` depends.
Additionally, node `u` considers that all correctly functioning nodes wil eventually agree on `x` for slot `i`.
At this point `u` `externalized` `x` for slot `i`.
The outside world may react to externalized values in irreversible ways, so a node cannot change its mind about them later.

A challenge for FBA is that malicious parties can join many times and outnumber honest nodes.
Consequently, traditional majority-based quorums do not work.

##### Quorum Slices

In a consensus protocol, nodes exchange messages asserting statements about slots.
FBA assumes such assertions cannot be forged, which can be guaranteed if nodes are named by public key, and they digitally sign the messages.

When a node knows about a sufficient set of nodes to assert a statement, it assumes that no other functioning node would contradict that statement.
Such a sufficient set of nodes is called a `quorum slice`, or just a slice.
A node may have multiple slices, any of which is sufficient to convince it of a statement.
At a high level the FBA system consists of a loose federation of nodes each of which has chosen one or more slices.

One node might consider that a quorum needs to contain ≥ 3/4 of the nodes in some set `S`, while another believes that a quorum should contain  > 2/3 of some similar but not identical set `S′`.
Nodes might also have less symmetric requirements on quorums, for example a third node might require that a quorum contain a majority of the nodes run by company `X` and a majority of the nodes run by company `Y`, where X runs many more nodes than Y.

By definition, a Federated Byzantine Agreement System (FBAS) is a pair `{U, Q}` that consists of a set of nodes - `U` and a quorum function `Q`.
`Q` specifies one or more quorum slices for each node in `U`, where a node belongs to all of its quorum slices.

A quorum is a set of nodes sufficient to reach an agreement.
A quorum slice is the subset of a quorum convincing one particular node of this agreement.
A quorum slice can be smaller or the same as the quorum.

Example of a four-node system, where each node has a single slice and arrows point to the other members of that slice:

```text
u1 —> {u2, u3}
Q(u1) = {{u1, u2, u3}}

u2 -> {u2, u3, u4}
u3 —> {u2, u3, u3}
u4 -> {u2, u3, u4}

=> Q(u2) = Q(u3) = Q(u4)
```

The slice of node `u1` is `{u1, u2, u3}` and it is sufficient to convince `u1` of a statement.
But `u2` and `u3` have slices that include also `u4` node.
This means that neither `u2` or `u3` can assert a statement without `u4`.
Consequently, no agreement is possible without the participation of `u4` and the only quorum including `u1` is the set of all nodes.

Traditional non-federated Byzantine agreement requires all nodes to accept the same slices for quorums - meaning quorums for each node should be equal.
Since every member accepts every slice, traditional systems do not distinguish between slices and quorums.

Traditional PBFT typically has 3f+1 nodes, any 2f+1 of which comprises a quorum, and f is the maximum number of Byzantine failures the system can handle.
FBA enables each node `u` to chose its own quorum size `Q(u)`.
In this way system-wide quorums arise from individual preferences any node may have.
In some settings, no individual node may have a complete knowledge of all nodes in the system and yet consensus still should be possible.
Federated Byzantine agreement is thus a generalization of the Byzantine agreement.

##### Safety and Liveness

Nodes in FBA are categorized as well-behaved or ill-behaved.
A well-behaved node chooses sensible quorum slices and obeys the protocol, including eventually responding to all requests.
An ill-behaved node does not.
Ill-behaved nodes may act arbitrarily — they might be compromised, their owner may have maliciously modified the software or may be crashed.
The goal of Byzantine agreement is to ensure that well-behaved nodes externalize the same values despite the presence of ill-behaved nodes.
This goal contains two parts:

* preventing nodes from diverging and externalizing different values of the same slot
* ensuring nodes can actually externalize values as opposed to getting blocked in some dead-end state from which the consensus is no longer possible

These properties concern two terms: liveness and safety.
A set of nodes in FBAS enjoy safety if no two of them ever externalize different values from the same slot.
A node in FBAS enjoys liveness if it can externalize new values without the participation of any failed (including ill-behaved) nodes.
Well-behaved nodes that have both safety and liveness properties are called `correct` nodes.

```text
liveness AND safety = correct nodes
```

Nodes that are not correct are `failed`.

```text
NOT correct = failed nodes
```

All ill-behaved nodes are failed, but a well-behaved node can fail too, by waiting indefinitely for messages from ill-behaved nodes or by having its state poisoned by incorrect messages from ill-behaved nodes.

Nodes that lack liveness are `blocked` and nodes that lack safety are `divergent`.

##### Quorum Intersection

In FBAS there is a quorum intersection, if any two of the quorums share a node that is present in all quorums of the system.

Disjoint quorums can independently agree on contradictory statements that undermine the system-wide agreement.
That is why, when many quorums exist — quorum intersection fails if any two do not intersect.

No protocol can guarantee safety in the absence of quorum intersection, since such a configuration can operate as two different FBA systems that do not exchange any messages.
Even with quorum intersection, safety might be impossible when the node which intersects the distinct quorums is ill-behaved.
In that case the effect of having an ill-behaved shared node is equivalent to lack of quorum intersection.

Since ill-behaved nodes contribute nothing to safety, no protocol can guarantee safety without the well-behaved nodes being intersections points.
In a worst-case scenario of safety, ill-behaved nodes can just always make any possible contradictory statement and two quorums overlapping only at ill-behaved nodes will again be able to operate like two different FBA system.

In short, FBAS can survive Byzantine failure by a number of nodes, if after deleting (ignoring) the ill-behaved nodes from the system and quorum slices, it still has a quorum intersection.
It is the responsibility of each node to ensure that the selected quorum(s) it uses do not violate the quorum intersection.
One way to solve this is to pick conservative slices that lead to large quorums.
Malicious nodes can intentionally pick slices that violate the quorum intersection rule, lie about the value returned by the quorum or ignore it in order to make arbitrary assertions.
In short the value produced by the quorum is not meaningful, if an ill-behaved node is receiving that value.
That is why the necessary property for safety — quorum intersection of well-behaved nodes after deleting ill-behaved nodes — is unaffected by the slices of ill-behaved nodes.

Deletion is conceptual for describing better the optimal safety.
A protocol should guarantee safety for quorum slices without the need to know about the ill-behaved nodes.

##### Dispensable Sets (DSets)

The fault tolerance of slices selected by the nodes is captured by the notion of a `dispensable set` or DSet.
The safety and liveness of nodes outside a DSet can be guaranteed regardless of the behavior of nodes inside the DSet — in an optimally resilient FBAS, if a single DSet encompasses every ill-behaved node, it also contains every failed node, and conversely all nodes outside a DSet are correct.

To prevent a misbehaving DSet from affecting the correctness of other nodes, two properties must hold.

* safety — deleting the DSet cannot undermine quorum intersection.
* liveness — the DSet cannot deny other nodes a functioning quorum.

Quorum availability despite the existence of a Dispensable set that contains ill-behaved nodes, protects against ill-behaved nodes in it that refuse to answer requests and block others' progress.
Quorum intersection despite the existence of a Dispensable set that contains ill-behaved nodes, protects against the opposite — nodes in the DSet making contradictory assertions that enable other nodes to externalize inconsistent values for the same slot.

Node must balance between the two threads in slice selection.
All else equal, bigger slices lead to bigger quorums with greater overlap and that leads to the case when fewer DSets with failed nodes will undermine quorum intersection when deleted.
On the other hand, bigger slices are more likely to contain failed nodes that endanger quorum availability.

The smallest DSet that contains all ill-behaved nodes may encompass well-behaved nodes as well, reflecting the fact that a sufficiently large set of ill-behaved nodes can cause well-behaved nodes to fail.

The DSets in FBA are determined 'a priori' by the Quorum function `Q`.
Which nodes are well-behaved and ill-behaved depends on runtime behavior, such as machines getting compromised.
The DSets that are important are those that encompass all ill-behaved nodes, as they help to distinguish nodes that should be guaranteed correct from ones that cannot.

A node in FBA is intact if there exists a DSet that contains all ill-behaved nodes, and none of its nodes are overlapping with the node.
A node in FBA is befouled if it is not intact.
A befouled node is surrounded by enough failed nodes to block its progress or poison its state, even it is well-behaved itself.
No FBAS can guarantee the correctness of a befouled node.
Optimal FBAS guarantees that every intact node remains correct.

##### Federated Voting

###### Voting

Federated Voting consists of nodes casting vote messages that also specify their quorum slices.
Recipients of these messages can dynamically discover quorums based on voters’ stated slices.
Quorum overlap ensures intertwined nodes will not find themselves in contradictory quorums.

However, knowing that no quorum will contradict `a` is insufficient to confirm `a`
When a member of an intact set confirms `a`, we also need to guarantee that the rest of the set will eventually do so.
This requires ensuring not just that `a` received a quorum, but that a quorum knows that `a` received a quorum.
Furthermore, this quorum must be able to convince other nodes, including ones that voted against `a`, to confirm `a`.

Federated voting employs a three-phase protocol in which nodes first vote for a statement (broadcasting a message to this effect), then accept it (again broadcasting the fact), and finally confirm it.

From the perspective of each node, agreement process on statement is divided in three phases — unknown, accepted and confirmed.
Initially the statement status is completely unknown to the node — it can be `true`, `false` or can become completely stuck in an indeterminate state.
If the first phase (`vote`) succeeds, meaning the statement is accepted as true, the node may accept the statement as well.

A node  may vote for any valid statement `a` that is consistent with its other outstanding votes and accepted statements.
The node accepts `a` when it is a member of a quorum, in which every node either votes for `a` or accepts `a`.
Even if the node did not vote for `a`, if every one of its quorum slices contains a node accepting `a` and the node has accepted nothing contradicting `a`, it also accepts `a`.
No two nodes that are intact cannot accept contradictory statements.
This means that if the node is intact and accepts a statement as true, then the statement cannot be false.

The set of accepting nodes intersecting all the node's quorum slices overrules any contradictory votes that the node may have previously cast by proving these contradictory votes could not have been part of a quorum.
Finally, when the node is a member of a quorum in which every node accepts `a`, then this node confirms `a`.

However, there are reasons for which after the statement is accepted from the node, it might not act upon it.
One is the statement can be stuck for other nodes.
Second is the node could be befouled, which is something it does not know itself, then accepting the statement means nothing (the statement could be false for well-behaved nodes).
Even if the node is befouled, the system might still have quorum intersection of well-behaved nodes.
In this case, for optimal safety, the node would need a greater assurance for the statement.

This is when the confirmation phase takes place.
This phase addresses both problems: if the second voting on the statements succeeds, the node moves to the `confirm` phase and in this phase it can consider the statement as true and act on it (considering a statement as true equals to accepting a value and adding it to own records).

###### Voting with Open Membership

A correct node in a Byzantine agreement system acts on a statement only when it knows that other correct nodes will never agree to other contradictory statements.
Most protocols employ voting for this purpose.

Well-behaved nodes vote for a statement only if it is valid and also never change their votes.
Consequently, in a centralized Byzantine agreement, it is safe to accept a statement as valid, if a quorum comprising a majority of well-behaved nodes has voted for it.
We say the statement is ratified when it received the necessary votes.

The FBA adapts the voting to open membership.
A quorum, as mentioned above, no more corresponds to a majority of well-behaved nodes.
In FBA the majority requirement serves to ensure quorum intersection of well-behaved nodes that is present.

A node votes for a statement if and only if

* the node asserts the statement as valid and consistent with all statements it accepted in the past
* the node asserts it has never voted against this statement implicitly — meaning did not vote for a statement that contradicts the one that it is currently accepting
* the node promises never to vote against that statement in the future — the node will not vote against it implicitly accepting a future contradictory statement

A quorum `ratifies` a statement if and only if all every member of it votes for the statement being true.
A node ratifies a statement if the quorum it is a member of ratified the statement.

Two contradictory statements in FBAS cannot be both ratified if there is a quorum intersection in that system.
