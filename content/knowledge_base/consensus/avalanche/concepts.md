# Avalanche Consensus Algorithm Concepts

## Snow Algorithm Group of Protocols

A group of protocols with a strong probabilistic safety guarantee in the presence of Byzantine nodes.

### Description

The core concept of operating is execution of repeated sampling the network at random, ultimately leading to correct nodes behavior to obtain a common statement (decision, transacation value, outcome).
This mechanism effectively brings the system to an irreversible state — meaning that a large potion of the network accepted a decision or a statement.
So any such conflicting to that statement would be accepted with a negligible probability ε or smaller.
TODO: finish
A more thorough description of Snow Algorithms specifications and examples can be found in [Avalanche Whitepaper](https://assets-global.website-files.com/5d80307810123f5ffbb34d6e/6009805681b416f34dcae012_Avalanche%20Consensus%20Whitepaper.pdf)

### Comparison with Nakamoto Consensus

TODO: add

### Specification

TODO: add

## Leaderless Byzantine Fault Tolerance

[Leaderless Byzantine Fault Tolerance](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2020/EECS-2020-121.pdf) combines the Snowball and Practical Byznatine Fault Tolerance (pBFT) algorithms.
It aims to take advantage of the decentralized aspect of Snowball and deterministic property of pBFT in order to eliminate the weakness of Snowballs's probabilistic property and pBFT's reliance on a "leader".

### LBFT Specification

TODO: add
