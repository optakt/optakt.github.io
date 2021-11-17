# Stellar Consensus Algorithm Concepts

## Federated Byzantine Agreement (FBA)

Federated Byzantine agreement is a new model of consensus that achieves robustness through quorum slices.
It is an adaptation of Byzantine agreement to systems with open membership, where different nodes have different concepts about which group of nodes are important to them about deciding on a statement.

These groups of nodes let the inidividual node take decisions based on its own trust criteria.
The groups bind the system together in the same way as inidividual networks' peering and transit decisions unifiy the Internet.

### Description

FBA is a model suitable for a worldwide consensus.
In FBA each particiapnt knows and relies on opinion of others that it considers important.
It waits for the `vast` majority of them to agree on a transaction before considering the transaction settled.
In turn, those important participant do not agree to the transaction until the participants `they` consider important agree as well, and so on.
Eventually — enough of the network accepts a transaction and consequently it becomes infeasible for an attacker to roll it back.
Only at that point all of the participants consider it settled.

### Stellar Consensus Protocol

Stellar consensus protocol (SCP) is a construction of FBA — an FBA system (FBAS).
It is free from blocked states, in which consensus is no longer possible, unless participant failures make it impossible to satisfy trust dependencies.

Stellar claims to be the first provably safe consensusm mechanism to have the following key properties simultaneously:

* Decentralized control — anyone is able to participate and no central authority dictates whose approval is required for consensus.
* Low latency — nodes can reach consensus at timescales humans expect for web payments — a few seconds.
* Flexible trust — users have the freedom to trust any combination of parties they see fit.
* Asymptotic security — safety rests on digital signatures and hash families whose parameteres can reallistically be tuned to protect against adversaries with vast computing power.

### FBA Specification

Like non-federated Byzantine agreement, FBA addresses the problem of updating replicated state, such as a transaction ledger or certificate tree.
By agreeing on what updates to apply, nodes avoid contradictory, irreconcilable states.

Each update is identified by a unque `slot` from which the inter-update dependencies can be inferred — for example slots can be consequtively numbered positions in a sequentially applied log.

An FBA system runs a consensus protocol that ensures nodes agree on slot contents.
A node `u` can safely apply update `x` in slot `i` when it has safely applied updates in all slots upon which `i` depends.
Additionally, node `u` considers that all correctly functioning nodes wil eventually agree on `x` for slot `i`.
At this point `u` `externalized` `x` for slot `i`.
The outside world may react to externalized values in irreversible ways, so a node cannot change its mind about them later.

A chanllenge for FBA is that malicious parties can join many times and outnumber honest nodes.
Consequently, tranditional majority-based quorums do not work.

#### Quorum Slices

In a consensus protocol, nodes exchange messages asserting statements about slots.
FBA assumes such assertions cannot be forged, which can be guaranteed if nodes are named by public key and they digitally sign the messages.

When a node knows about a sufficient set of nodes to assert a statement, it assumes that no other functioning node would contradict that statement.
Such a sufficient set of nodes is called a `quorum slice`, or just a slice.
A node may have multiple slices, any of which is sufficient to convince it of a statement.
At a high level the FBA system consists of a loose federation of nodes each of which has chosen one or more slices.

One node might consider that a quorum needs to contain ≥ 3/4 of the nodes in some set `S`, while another believes that a quorum should contain  > 2/3 of some similar but not identical set `S′`.
Nodes might also have less symmetric requirements on quorums, for example a third node might require that a quorum contain a majority of the nodes run by company `X` and a majority of the nodes run by company `Y`, where X runs many more nodes than Y.

By definition, a Federated Byzantine Agreement System (FBAS) is a pair `{U, Q}` that comprises of a set of nodes - `U` and a quorum function `Q`.
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

Traditional non-federated Byzantine agreement requires all nodes to accept the same sllices for quorums - meaning quorums for each node should be equal.
Since every member accepts every slice, traditional systems do not distinguish between slices and quorums.

Traditional PBFT typically has 3f+1 nodes, any 2f+1 of which comprises a quorum, and f is the maximum number of Byzantine failures the system can handle.
FBA enables each node `u` to chose its own quorum size `Q(u)`.
In this way system-wide quorums arise from individual preferences any node may have.
In some settings, no individual node may have a complete knowledge of all nodes in the system and yet consensus still should be possible.
Federated Byzantine agreement is thus a generalization of the Byzantine agreement.

#### Safety and Liveness

Nodes in FBA are categorized as well-behaved or ill-behaved.
A well-behaved node choses sensisble quorum slices and obeys the protocol, including eventually repsonding to all requests.
An ill-behaved node does not.
Ill-behaved nodes may act arbitrarily — they might be compromised, their owner may have maliciously modified the software or may be crashed.
The goal of Byzantine agreement is to ensure that well-behaved nodes extrenalize the same values despite the presence of ill-behaved nodes.
This goal contains two parts:

* preventing nodes from diverging and externalizing different values of the same slot
* ensuring nodes can actually externalize values as opposed to getting blocked in some dead-end state from which the consensus is no loger possible

These properties concern two terms: liveness and safety.
A set of nodes in FBAS enjoy safety if no two of them ever externalize different values from the same slot.
A node in FBAS enjoys liveness if it can externalize new values without the participation of any failed (including ill-behaved) nodes.
Well-behaved nodes that has noth safety and liveness properties are called `correct` nodes.

```text
liveness AND safety = correct nodes
```

Nodes that are not correct are `failed`.

```text
NOT correct = failed nodes
```

All ill-behaved nodes are failed, but a well-behaved node can fail too, by waiting indefinitely for messages from ill-behaved nodes or by having its state poisoned by incorrect messages from ill-behaved nodes.

Nodes that lack liveness are `blocked` and nodes that lack safety are `divergent`.

#### Quorum Intersection

In FBAS there is a quorum intersection, if any two of the quorums share a node that is present in all quorums of the system.

Disjoint quorums can independently agree on contradictionary statements that undermine the system wide agreement.
That is why, when many quorums exists - quorum intersection fails if any two do not intersect.

No protocol can guarantee safety in the absense of quorum intersection, since such a configuration can operate as two different FBA systems that do not exchange any messages.
Even with quorum intersection, safety might be impossible when the node which intersects the distinct quorums is ill-behaved.
In that case the effect of having an ill-behaved shared node is equivalent to lack of quorum intersection.

Since ill-behaved nodes contribute nothing to safety, no protocol can guarantee safety without the well-behaved nodes being intersections points.
In a worst-case scenario of safety, ill-behaved nodes can just always make any possible contradictionary statement and two quorums overlapping only at ill-behaved nodes will again be able to operate like two different FBA system.

In short, FBAS can survive Byzantine failure by a number of nodes, if after deleting (ignoring) the ill-behaved nodes from the system and quorum slices, it still has a quorum intersection.
It is the responsibility of each node to ensure that the selected quorum(s) it uses do not violate the quorum intersection.
One way to solve this is to pick conservative slices that lead to large quorums.
Malicious nodes can intentionally pick slices that violate the quorum intersection rule, lie about the value returned by the quorum or ingnore it in order to make arbitrary assertions.
In short the value produced by the quorum is not meaningful, if an ill-behaved node is receiving that value.
That is why the necessary property for safety — quorum intersection of well-behaved nodes after deleting ill-behaved nodes — is unaffected by the slices of ill-behaved nodes.

Deletion is conceptual for describing better the optimal safety.
A protocol should guarantee safety for quorum slices without the need to know about the ill-behaved nodes.

#### Dispensable Sets (DSets)

The fault tolerance of slices selected by the nodes is captured by the notion of a `dispensable set` or DSet.
The safety and liveness of nodes outside a DSet can be guaranteed regardless of the behavior of nodes inside the the DSet — in an optimally resilient FBAS, if a single DSet encompasses every ill-behaved node, it also contains every failed node, and conversely all nodes outside a DSet are correct.

To prevent a misbehaving DSet from affecting the correctness of other nodes, two properties must hold.

* safety — deleting the DSet cannot undermine quorum intersection.
* liveness — the DSet cannot deny other nodes a functioning quorum.

Quorum availability despite the existense of a Dispensable set that contains ill-behaved nodes, protects against ill-behaved nodes in it that refuse to answer requests and block others' progress.
Quorum intersection despite the existense of a Dispensable set that contains ill-behaved nodes, protects against the opposite — nodes in the DSet making contradictory assertions that enable other nodes to externalize inconsistent values for the same slot.

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

#### Federated Voting

##### Voting

Federated Voting consists of nodes casting vote messages that also specify their quorum slices.
Recipients of these messages can dynamically discover quorums based on voters’ stated slices.
Quorum overlap ensures intertwined nodes will not find themselves in contradictory quorums.

However, knowing that no quorum will contradict `a` is insufficient to confirm `a`
When a member of an intact set confirms `a`, we also need to guarantee that the rest of the set will eventually do so.
This requires ensuring not just that `a` received a quorum, but that a quorum knows that `a` received a quorum.
Furthermore, this quorum must be able to convince other nodes, including ones that voted against `a`, to confirm `a`.

Federated voting employs a three-phase protocol in which nodes first vote for a statement (broadcasting a message to this effect), then accept it (again broadcasting the fact), and finally confirm it.

From the perspective of each node, agreement process on statement is divided in three phases — unknown, accepted and confirmed.
Initially the statement status is completely unknown to the node — it can be `true`, `false` or can become completely stuck in a indeterminate state.
If the first phase (`vote`) succeeds, meaning the statement is accepted as true, the node may accept the statement as well.

A node  may vote for any valid statement `a` that is consistent with its other outstanding votes and accepted statements.
The node accepts `a` when it is a member of a quorum, in which every node either votes for `a` or accepts `a`.
Even if the node did not vote for `a`, if every one of its quorum slices contains a node accepting `a` and the node has accepted nothing contradicting `a`, it also accepts `a`.
No two nodes that are intact cannot accept contradictory statements.
This means that if the node is intact and accepts a statement as true, then the statement cannot be false.

The set of accepting nodes intersecting all of the node's quorum slices overrules any contradictory votes that the node may have previously cast by proving these contradictory votes could not have been part of a quorum.
Finally, when the node is a member of a quorum in which every node accepts `a`, then this node confirms `a`.

However, there are reasons for which after the statement is accepted from the node, it might not act upon it.
One is the statement can be stuck for other nodes.
Second is the node could be befouled, which is something it does not know itself, then accepting the statement means nothing (the statement could be false for well-behaved nodes).
Even if the node is befouled, the system might still have quorum intersection of well-behaved nodes.
In this case, for optimal safety, the node would need a greater assurance for the statement.

This is when the confirmation phase takes place.
This phase addresses both problems: if the second voting on the statements succeeds, the node moves to the `confirm` phase and in this phase it can consider the statement as true and act on it (considering a statement as true equals to acceptin a value and addiing it to own records).

##### Voting with Open Membership

A correct node in a Byzantine agreement system acts on a statement only when it knows that other corect nodes will never agree to other contraditory statements.
Most protocols employ voting for this purpose.

Well-behaved nodes vote for a statement only if it is valid and also never change their votes.
Consequently in a centralized Byzantine agreement, it is safe to accept a statement as valid, if a quorum comprising a majority of well-behaved nodes has voted for it.
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
