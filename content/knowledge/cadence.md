# Cadence

## Introduction

[Cadence](https://docs.onflow.org/cadence/) is a resource-oriented programming language specifically designed for smart-contract programming.

### Goals

Cadence Language was designed with these three goals in mind:

* Safety and security
    * Safety is the underlying reliability of any smart contract. Security is the prevention of attacks on the network or smart contracts.
* Clarity
    * Code needs to be easy to read, and its meaning should be as unambiguous as possible.
* Simplicity
    * Writing code and creating programs should be as approachable as possible.

### Features

Some features of the Cadence programming language are:

* type safety and a strong static type system
* resource-oriented programming
    * resources are types that can only exist in one location at a time and cannot be copied, lost or stolen; Thus there is a concept of scarcity and ownership over these objects
* built-in pre-conditions and post-conditions for functions and transactions
* capability-based security
    * access to objects is restricted only to the owner of the object and those who have a valid reference to it

### Terminology

* **Invalid**: The invalid program is not even be allowed to run. The program error is detected and reported statically by the type checker.
* **Run-time error**: The erroneous program can run, but bad behavior will result in the execution of the program being aborted.

## Account

Accounts are address on the flow network, e.g. `0xf233dcee88fe0abe`.
Every account can be accessed through two types: [`PublicAccount`](#publicaccount) and [`AuthAccount`](#authaccount), each corresponding to a level of accessibility.

Cadence transactions consist of four optional phases: `prepare`, `precondition`, `execution` and `postconditions`.
Phases can be omitted, but, if present, they must appear in that order.
Each account signing the transaction appears as an argument in the `prepare()` phase.
These signers appear as arguments of `AuthAccount` type, and their number must match the number of signers of the transaction.
The _prepare_ phase is the only place where direct access to the signing accounts is possible.

```rust
transaction {
    prepare(account: AuthAccount, account2: AuthAccount, (...), accountN: AuthAccount) {
        // ...
    }
}
```

### PublicAccount

_PublicAccount_ represents the publicly available information about an account.
This information can be retrieved about any account using the `getAccount()` function.

```rust
struct PublicAccount {

    let address: Address
    // The FLOW balance of the default vault of this account
    let balance: UFix64
    // The FLOW balance of the default vault of this account that is available to be moved
    let availableBalance: UFix64
    // Amount of storage used by the account, in bytes
    let storageUsed: UInt64
    // storage capacity of the account, in bytes
    let storageCapacity: UInt64

    // Contracts deployed to the account
    let contracts: PublicAccount.Contracts

    // Keys assigned to the account
    let keys: PublicAccount.Keys

    // Storage operations

    fun getCapability<T>(_ path: PublicPath): Capability<T>
    fun getLinkTarget(_ path: CapabilityPath): Path?

    struct Contracts {

        let names: [String]

        fun get(name: String): DeployedContract?
    }

    struct Keys {
        // Returns the key at the given index, if it exists.
        // Revoked keys are always returned, but they have `isRevoked` field set to true.
        fun get(keyIndex: Int): AccountKey?
    }
}
```

### AuthAccount

On the other hand, _AuthAccount_ represents an **authorized** account.
Authorized accounts can only be encountered in the `prepare` function of a signed transaction.

```rust
struct AuthAccount {

    let address: Address
    // The FLOW balance of the default vault of this account
    let balance: UFix64
    // The FLOW balance of the default vault of this account that is available to be moved
    let availableBalance: UFix64
    // Amount of storage used by the account, in bytes
    let storageUsed: UInt64
    // storage capacity of the account, in bytes
    let storageCapacity: UInt64

    // Contracts deployed to the account

    let contracts: AuthAccount.Contracts

    // Keys assigned to the account

    let keys: AuthAccount.Keys

    // Key management

    // Adds a public key to the account.
    // The public key must be encoded together with their signature algorithm, hashing algorithm and weight.
    // This method is currently deprecated and is available only for the backward compatibility.
    // `keys.add` method can be use instead.
    fun addPublicKey(_ publicKey: [UInt8])

    // Revokes the key at the given index.
    // This method is currently deprecated and is available only for the backward compatibility.
    // `keys.revoke` method can be use instead.
    fun removePublicKey(_ index: Int)

    // Account storage API (see the section below for documentation)

    fun save<T>(_ value: T, to: StoragePath)
    fun load<T>(from: StoragePath): T?
    fun copy<T: AnyStruct>(from: StoragePath): T?

    fun borrow<T: &Any>(from: StoragePath): T?

    fun link<T: &Any>(_ newCapabilityPath: CapabilityPath, target: Path): Capability<T>?
    fun getCapability<T>(_ path: CapabilityPath): Capability<T>
    fun getLinkTarget(_ path: CapabilityPath): Path?
    fun unlink(_ path: CapabilityPath)

    struct Contracts {

        // The names of each contract deployed to the account
        let names: [String]

        fun add(
            name: String,
            code: [UInt8],
            ... contractInitializerArguments
        ): DeployedContract

        fun update__experimental(name: String, code: [UInt8]): DeployedContract

        fun get(name: String): DeployedContract?

        fun remove(name: String): DeployedContract?
    }

    struct Keys {
        // Adds a new key with the given hashing algorithm and a weight, and returns the added key.
        fun add(
            publicKey: PublicKey,
            hashAlgorithm: HashAlgorithm,
            weight: UFix64
        ): AccountKey

        // Returns the key at the given index, if it exists, or nil otherwise.
        // Revoked keys are always returned, but they have `isRevoked` field set to true.
        fun get(keyIndex: Int): AccountKey?

        // Marks the key at the given index revoked, but does not delete it.
        // Returns the revoked key if it exists, or nil otherwise.
        fun revoke(keyIndex: Int): AccountKey?
    }
}

struct DeployedContract {
    let name: String
    let code: [UInt8]
}
```

### Account Storage

Each account has storage, and this is where resources and structs can be persisted.
Authorized accounts have full access to the account storage.

Objects in storage are stored under paths.
Each storage path location corresponds to a single register.
Paths correspond to the `Key` part of the register ID.
Paths have a format of `/<domain>/<identifier>`.
There are three valid domains &#0151 `storage`, `public` and `private`.
Objects in storage are **always** stored in the `storage` domain &#0151 meaning this is where resources and any other data is kept.
The `public` domain allows the account to let other accounts access the objects, links to `storage`, inside it.

## Resources

Resources are types that can exist only in one memory location at a time.
At the end of a function which has resources in scope, resources must either be **moved** or **destroyed**.

After it was _destroyed_, the resource can no longer be used.
_Moving_ a resource means either assigning it to a different constant or a variable, passing it as an argument to another function, or returning it from a function.
After the resource was moved &#0151 e.g. assigned to a `const`, the previous reference to the resource is invalid and cannot be used anymore.

To make the resource behavior clear and explicit, the prefix `@` must be used in all type annotations dealing with resources, and the resource movement is denoted with the _move_ operator &#0151 `<-`.

```rust
// Resource definition.
pub resource SomeResource {

    // The resource has a single field &#0151 the `value` integer.
    pub var value: Int

    // Define the resource initializer
    init(value: Int) {
        self.value = value
    }
}

// Resource is created using the `create` keyword.
// Note that the const `a` has type of `@SomeResource`.
let a: @SomeResurce <- create SomeResource(value: 2)

// Resource is moved from const a to const b.
// `a` can no longer be used to access the resource.
let b <- a

// Resource is destroyed.
destroy b

// Neither `a` or `b` can now be used to access the resource as it was destroyed.
```

When a resource is returned from a function, the function caller has the responsibility to use the returned resource.

```rust
// This function is invalid as it does not use the resource.
pub fun invalid_use(r: @SomeResource) {
}

// This function uses the resource by moving it to a new const.
// This new const is used by being returned from the function.
pub fun use(res: @SomeResource): @SomeResource {
    let moved <- res
    // Note that the return call still uses the `move` operator.
    return <-moved
}

// Create a new resource.
let a: @SomeResource <- create SomeResource(value: 3)

// Move the function return value to a new const.
// The const `a` can no longer be used to access the resource.
let result <- use(res: <- a)

// Destroy the resource.
destroy result
```

Resource can have a destructor, which is called when the resource is destroyed.

```rust
pub resource SomeResource {

    destroy() {
        // Some logic here, e.g. decrement the variable indicating the number of resources in existence.
    }
}
```

Since the resource variables cannot be assigned to, there are two options to replace the values of resource variables &#0151 the _swap_ operator (`<->`) and the _shift_ operator (`<- target <-`)

```rust
pub resource SomeResource{}

var x <- create SomeResource()
var y <- create SomeResource()

// Swap the resources.
x <-> y

// Alternatively, use the shift operator.
// The shift operator moves the resource from `x` to `oldX`.
// At the same time, `x` receives the value of the new resource.
let oldX <- x <- create SomeResource()

// oldX still needs to be used.
```

Resources have an implicit unique identifier in the form of the predeclared public field `uuid` of type `UInt64`.
This field is incremented on each resource creation, and can never be the same for two resources, even if some of them were destroyed.

## Reference

### Comments

Single-line comments in Cadence use `//`, multi-line comments use `/* */`.

```rust
// This is a comment on a single line.

/* This is a comment which
spans multiple lines. */
```

#### Documentation Comments

Cadence has a specific way to create documentation comments.
For single-line comments use `///`, for multi-line comments use the syntax `/** **/`.

```rust
/// This is a single-line documentation comment
/// You can keep them going.

/**
	Multi-line
	Documentation Comment
**/
```

### Names

Names can have letters, underscores and numbers, but can only start with any letter or underscore.
By convention, variables, constants, and functions have lowercase names; and types have title-case names.

```markdown
// Valid: title-case
//
PersonID

// Valid: with underscore
//
token_name

// Valid: leading underscore and characters
//
_balance

// Valid: leading underscore and numbers
_8264

// Valid: characters and number
//
account2

// Invalid: leading number
//
1something

// Invalid: invalid character #
_#1

// Invalid: various invalid characters
//
!@#$%^&*
```

### Types

There are several built-in types for Cadence.

#### Integer

Cadence supports the following integer types: `Int`, `Int8`, `Int16`, `Int32`, `Int64`, `Int128`, `Int256`, `UInt`, `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`, `UInt256`, `Word8`, `Word16`, `Word32`, `Word64`.

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBigEndianBytes(): [Uint8]`
* `Uint8.max`
* `Uint8.min`

#### Fixed-Point Number

Cadence supports the following fixed-point types: `Fix64`, `UFix64`.

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBigEndianBytes(): [Uint8]`
* `Fix64.max`
* `Fix64.min`

#### Address

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBytes(): [Uint8]`

#### String

Strings are collections of characters.
Strings can be used to work with text in a Unicode-compliant way.
Strings are immutable.

The supported built-ins for this type family are:

* `let length: Int`
* `let utf8: [Uint8]`
* `fun concat(_ other: String): String`
* `fun slice(from: Int, upTo: Int): String`
* `fun decodeHex(): [Uint8]`
* `fun toLower(): String`
* `fun String.encodeHex(_ data: [Uin8]): String`

#### Arrays

The supported built-ins for this type family are:

* `let length: Int`
* `fun concat(_ array: T): T`
* `fun contains(_ element: T) Bool`

Specific to Variable size Array Functions:

* `fun append(_ element: T): Void`
* `fun appendAll(_ array: T) Void`
* `fun insert(at index: Int, _ element: T): Void`
* `fun remove(at index: Int): T`
* `fun removeFirst(): T`
* `fun removeLast(): T`

#### Dictionary

The supported built-ins for this type family are:

* `let length: Int`
* `fun insert(key: K, _ value: V) V?`
* `fun remove(key: K): V?`
* `let keys: [K]`
* `let values: [V]`
* `fun containsKey(key: K): Bool`

#### Floating-Point

There is **no** support for floating point numbers. These kinds of numbers, not natural/cardinal, are handle by the Fixed-Point type.

### AnyStruct and AnyResource

`AnyStruct` is the base type of all non-resource types, i.e., all non-resource types are a subtype of it. (`Int8`, `String`, `Bool`, `struct`, not type specific)
`AnyResource` is the base type of all resource types.

### Optionals

Optionals are values which can represent the absence of a value.

An optional type is declared using the `?` suffix for another type.
For example, `Int` is a non-optional integer, and `Int?` is an optional integer, i.e. either an integer, or nothing.

The value representing the absence of a value is `nil`.

### Nil-Coalescing Operator

The nil-coalescing operator `??` returns the value inside an optional if it contains a value, or returns an alternative value if the optional has no value.

```rust
let a: Int? = nil
let b: Int = a ?? 42
// `b` is 42, as `a` is nil
```

### Force Unwrap

The force-unwrap operator `!` returns the value inside an optional if it contains a value, or panics and aborts the execution if the optional has no value.

### Never

`Never` can be used as the return type for functions that never return normally.
For example, it is the return type of the `panic` function.

### Array Types

Arrays either have a fixed or variable size.

Fixed-size array types have the form `[T; N]`, where `T` is the element type, and `N` is the size of the array.
For example, a fixed-size array of 3 `Int8` elements has the type `[Int8; 3]`.

Variable-size array types have the form `[T]`, where `T` is the element type.
For example, the type `[Int16]` specifies a variable-size array of elements that have type `Int16`.

### Dictionary Types

Dictionary keys must be hashable and equatable, i.e., must implement the `Hashable` and `Equatable` interfaces.
Most of the built-in types, like booleans and integers, are hashable and equatable, so can be used as keys in dictionaries.

### Swapping

The binary swap operator `<->` can be used to exchange the values of two variables. It is only allowed in a statement and is not allowed in expressions.

```rust
var a = 1
var b = 2
var c = 3

// Invalid: The swap operation cannot be used in an expression.
a <-> b <-> c

// Instead, the intended swap must be
// written in multiple statements.
b <-> c
a <-> b
```

### Argument Passing Behavior

Arguments are passed to functions by value. Therefore, values passed to a function are unchanged in the caller's scope when that function returns.

### Function Preconditions and Postconditions

The functions `preconditions` and `postconditions` are blocks of expressions that check input and outputs.

```rust
fun factorial(_ n: Int): Int {
    pre {
        // Require the parameter `n` to be greater than or equal to zero.
        //
        n >= 0:
            "factorial is only defined for integers greater than or equal to zero"
    }
    post {
        // Ensure the result will be greater than or equal to 1.
        //
        result >= 1:
            "the result must be greater than or equal to 1"
    }

    if n < 1 {
       return 1
    }

    return n * factorial(n - 1)
}

factorial(5)  // is `120`

// Run-time error: The given argument does not satisfy
// the precondition `n >= 0` of the function, the program aborts.
//
factorial(-2)
```

### Switches

As opposed to some languages, Cadence does not feature "fall through" in switch statements.

### Composite Types

Composite types can only be declared within a contract and nowhere else.
There are two types:

* Structures
    * They are copied.
    * They are value types.
* Resources
    * They are moved.
    * They are linear types.

### Equatable Interface

```rust
struct interface Equatable {
    pub fun equals(_ other: {Equatable}): Bool
}
```

### Hashable Interface

```rust
struct interface Hashable: Equatable {
    pub hashValue: Int
}
```

### Restricted Types

Resources and structs can be used with the restricted types.
For example, a `Resource` that implements the interface `Balance` can be accessed by accessing only the functions and/or fields available in `Balance`.

```rust
resource interface HasCount {
    pub let count: Int
}

pub resource Counter: HasCount {
    pub var count: Int

    init(count: Int) {
        self.count = count
    }

    pub fun increment() {
        self.count = self.count + 1
    }
}

let counter: @Counter <- create Counter(count: 42)

counter.count  // is `42`

counter.increment()

counter.count  // is `43`

// Move the resource in variable `counter` to a new variable `restrictedCounter`,
// but typed with the restricted type `Counter{HasCount}`:
// The variable may hold any `Counter`, but only the functionality
// defined in the given restriction, the interface `HasCount`, may be accessed
//
let restrictedCounter: @Counter{HasCount} <- counter

// Invalid: Only functionality of restriction `Count` is available,
// i.e. the read-only field `count`, but not the function `increment` of `Counter`
//
restrictedCounter.increment()
```

### Events

Events can only be declared within contracts and cannot use resources as parameters or outputs.

```rust
// Invalid: An event cannot be declared globally
//
event GlobalEvent(field: Int)

pub contract Events {
    // Event with explicit argument labels
    //
    event BarEvent(labelA fieldA: Int, labelB fieldB: Int)

    // Invalid: A resource type is not allowed to be used
    // because it would be moved and lost
    //
    event ResourceEvent(resourceField: @Vault)
}
```

#### Emitting Events

To emit an event, the keyword `emit` is used.
For example, emitting an event of the type `Test(field: Int)`: `emit Test(1)`.

#### Core Events

There are some core events built-in in Cadence.

* [`pub event AccountCreated(address: Address)`](https://docs.onflow.org/cadence/language/core-events/#account-created)
* [`pub event AccountKeyAdded(address: Address, publicKey: [UInt8])`](https://docs.onflow.org/cadence/language/core-events/#account-key-added)
* [`pub event AccountKeyRemoved(address: Address, publicKey: [UInt8])`](https://docs.onflow.org/cadence/language/core-events/#account-key-removed)
* [`pub event AccountContractAdded(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-added)
* [`pub event AccountContractUpdated(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-updated)
* [`pub event AccountContractRemoved(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-removed)

## Contracts

A contract in Cadence is a collection of definitions of interfaces, structs, resources, data (its state) and code (its functions).
Contracts live in the contract storage area of an account, and they can be added, updated and removed.

Composite types &#0151 structs, resources, events and interfaces for these types have to be defined in a contract.
However, there is an exception to this rule, since there is a number of native event types that are not defined in a contract, but built into the Cadence runtime itself.
These events are emitted on user account creation, account key addition or removal, and contract deployment, update or removal.

Contracts themselves are types, similar to composite types, but cannot be used as values, copied or moved like resources or structs.

Example of a simple contract is given below:

```rust
// Name of this contract is HelloWorld.
pub contract HelloWorld {

    // Public constants &#0151 state fields.
    pub let description: String
    pub let greeting: String

    // init function, called when a contract is created.
    init(description: String) {
        self.description = description
        self.greeting = "Hello World!"
    }

    // Public function hello(), returning a string.
    // This function can be called by anyone importing the contract.
    pub fun hello(): String {
        return self.greeting
    }
}
```

This contract can be imported by Cadence scripts, transactions or other contracts at the beginning of the script, transaction or contract definition.

```rust
// 0x01 is the account address where the contract is deployed.
import HelloWorld from 0x01

// Invoke the hello() function of the imported contract.
log(HelloWorld.hello())
```

Any number of contracts can be present on an account, which could include an arbitrary amount of data.

Each contract has an implicit field &#0151 `let account: AuthAccount`, which is the account in which the contract is deployed.

### Deploying a Contract

A new contract can be deployed to an account using the `add` function.
An example transaction deploying a `HelloWorld` contract is given below:

```rust
transaction(code: String) {
    prepare(signer: AuthAccount) {

        signer.contracts.add(
            name: "HelloWorld",
            code: code.decodeHex(),
            description: "This is a new contract on an existing account"
        )
    }
}
```

When executing a transaction with this Cadence script, given a string argument that is a hex-encoded text of the Cadence contract, a new contract gets deployed to the account that signed the transaction.
The contract's text can then be found in the `code.HelloWorld` account register.
An event of type `flow.AccountContractAdded` is emitted as a result of contract deployment.
This and a number of other events are built into the Cadence [standard library](https://docs.onflow.org/cadence/language/built-in-functions/).

Another register is created in the given account, prefixed `contract`, followed by the `\x1F` character and the contract name.
This register holds the state of the contract.

### Updating a Contract

The contract's updates are currently experimental.
Updates to existing contracts are done using the `update__experimental` function.
Contract updates are very similar to the contract deployments, except the `init()` function of the contract is not invoked on update.

```rust
transaction(code: String) {
    prepare(signer: AuthAccount) {

        signer.contracts.update__experimental(
            name: "HelloWorld",
            code: code.decodeHex()
        )
    }
}
```

There is a number of [limitations](https://docs.onflow.org/cadence/language/contract-updatability/#updating-a-contract) to the types of updates that can be made to a contract.
For example, it is valid to remove a field or change its access modifier from public to private, but it is invalid to add a field, or change its type.
This is done in order to ensure data consistency, since changing a contract changes how the program interprets the data, but does not change the actual stored data.
After a successful contract update, an event of `flow.AccountContractUpdated` type is emitted.

### Removing a Contract

An existing contract can be deleted from an account using the `remove` function.
Example of a transaction removing a contract:

```rust
transaction(name: String) {
    prepare(signer: AuthAccount) {

        // name is the name of the contract that should be removed
        signer.contracts.remove(
            name: name,
        )
    }
}
```

After a successful contract removal, an event of `flow.AccountContractRemoved` type is emitted.

## Flow Contracts

There are core Cadence contracts that are deployed on the Flow network as soon as it is bootstrapped.
These contracts are essential to the normal operation of the Flow network.

### FungibleToken

The `Flow Fungible Token` standard is defined in [0xf233dcee88fe0abe](https://flowscan.org/contract/A.f233dcee88fe0abe.FungibleToken) for the mainnet.
It defines the `FungibleToken` contract interface, to which all fungible token contracts need to conform to.

The `Vault` resource is also defined in the same contract, which is the resource that each account needs to have in storage in order to own tokens.

Resources and interfaces defined here allow sending and receiving of tokens peer-to-peer, by withdrawing tokens from one user's `Vault` and depositing them to another user's `Vault.`

### FlowToken

Implementation of the `FungibleToken` interface for the FLOW token.

### FlowServiceAccount

The Flow Service Account is an account like any other on Flow except it is responsible for managing core network operations. The Service Account can be referred to as the keeper of core network parameters which will be managed algorithmically over time but are currently set by a group of core contributors to ensure ease of updates to the network in this early stage of its development.

### FlowEpoch

Top-level smart contract that manages the lifecycle of [epochs](https://docs.onflow.org/staking/epoch-preparation/#epochs-overview).
Epochs are the smallest unit of time during which the set of network operators is either static or can decrease due to nodes election or malicious node presence.
Nodes may only leave or join the network at epoch boundaries.
The list of participating nodes is kept in the **Identity Table**.

Epochs have three phases &#0151 [staking](#staking-phase), [setup](#setup-phase) and [committed](#committed-phase).

#### Staking Phase

During the [staking phase](https://docs.onflow.org/staking/epoch-preparation/#phase-0-staking-auction) (or staking Auction), nodes submit staking requests for the **next** epoch.
At the end of this phase, the identity table for the next epoch is determined.

#### Setup Phase

During the [setup phase](https://docs.onflow.org/staking/epoch-preparation/#phase-1-epoch-setup), the participants are preparing for the next epoch.
Collection nodes submit votes for their cluster's root **quorum certificate** and consensus nodes run the [**distributed key generation** protocol](https://docs.onflow.org/staking/qc-dkg/#consensus-committee-distributed-key-generation-protocol-dkg) (DKG) to set up the random beacon.

#### Committed Phase

When the [committed phase](https://docs.onflow.org/staking/epoch-preparation/#phase-2-epoch-committed) begins, the network is fully prepared to move to the next epoch.
Failure to enter this phase before transitioning to the next epoch would be a critical failure and would cause the chain to halt.

### FlowFees

Contract that manages the fee `Vault` and deposits and withdrawal of fees to and from the fee vault.

### FlowStorageFees

Storage capacity of an account determines how much storage on chain it can use.
There is a set of minimum amount of tokens reserved for storage capacity that is paid during account creation, by the creator.
If any transaction that results in account's storage use greater that the storage capacity, the transaction will fail.

### FlowClusterQC

This contract manages the process of collecting votes for the root [Quorum Certificate (QC)](glossary.md#quorum-certificate) of the upcoming epoch for all Collection node clusters assigned for the next epoch.
During the setup phase of the epoch, collection nodes in a specific cluster generate a root block for the cluster.
The nodes then need to submit a vote for the root block.
Once enough cluster nodes have voted with the same unique vote, the cluster is considered complete.
Once all clusters are complete, the QC is complete.

### FlowDKG

This contract manages the process of generating a group key with the participation of all consensus nodes for the upcoming epoch (DKG).

### FlowIDTableStaking

The Flow ID Table and Staking contract manages the node operators' and delegators' information and Flow tokens that are staked as part of the protocol.
Nodes submit their stakes during the [staking phase](#staking-phase) of the epoch.
When staking, nodes receive an object they can use to stake, unstake and withdraw rewards.
Tokens are held in multiple token buckets depending on their status — staked, unstaking, unstaked or rewarded.

This is also where the `Delegator` functionality, which manages delegation of FLOW to node operators, is specified.

### FlowStakingCollection

This contract defines a collection for staking and delegating objects which allows users to stake and delegate to as many nodes as they want.

### LockedTokens

This contract implements the functionality required to manage FLOW buyers locked tokens from the token sale.

Each token holder gets two accounts.
The first account is the locked token account, jointly controlled by the user and the token administrator.
Token administrator cannot interact with the account without approval from the token holder except for depositing additional tokens, or to unlock existing tokens at the appropriate milestones.

Second account is the unlocked user account, which is in full possession of the user.
This account stores a capability which allows the account owner to withdraw tokens when they become unlocked, and also to perform staking operations with the locked tokens.

### StakingProxy

This contract defines an interface for node stakers to use to be able to perform common staking actions.

### Contract Addresses

* FungibleToken
    * Mainnet: [0xf233dcee88fe0abe](https://flowscan.org/contract/A.f233dcee88fe0abe.FungibleToken)
* NonFungibleToken
    * Mainnet: [0x1d7e57aa55817448](https://flowscan.org/contract/A.1d7e57aa55817448.NonFungibleToken)
* DapperUtilityCoin
    * Mainnet: [0xead892083b3e2c6c](https://flowscan.org/contract/A.ead892083b3e2c6c.DapperUtilityCoin)
