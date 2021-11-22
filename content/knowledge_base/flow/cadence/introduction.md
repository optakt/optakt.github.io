# Introduction

[Cadence](https://docs.onflow.org/cadence/) is a resource-oriented programming language specifically designed for smart-contract programming.

## Goals

Cadence Language was designed with these tree goals in mind:

* Safety and security
    * Safety is the underlying reliability of any smart contract. Security is the prevention of attacks on the network or smart contracts.
* Clarity
    * Code needs to be easy to read, and its meaning should be as unambiguous as possible.
* Simplicity
    * Writing code and creating programs should be as approachable as possible.

## Features

Some of the features of the Cadence programming language are:

* type safety and a strong static type system
* resource-oriented programming
    * resources are types that can only exist in one location at a time and cannot be copied, lost or stolen; Thus there is a concept of scarcity and ownership over these objects
* built-in pre-conditions and post-conditions for functions and transactions
* capability-based security
    * access to objects is restricted only to the owner of the object and those who have a valid reference to it

## Terminology

* **Invalid**: The invalid program is not even be allowed to run. The program error is detected and reported statically by the type checker.
* **Run-time error**: The erroneous program can run, but bad behavior will result in the execution of the program being aborted.
