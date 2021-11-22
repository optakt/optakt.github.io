# Cadence

## Goals

Cadence Language was designed with this tree goals in mind.

* Safety and security
* Auditability
* Simplicity

## Terminology

* Invalid: Means that the invalid program will not even be allowed to run. The program error is detected and reported statically by the type checker.
* Run-time error: Mean that the erroneous program will run, but bad behavior will result in the execution of the program being aborted.

## Core Concepts

### Documentation Comments

Cadence has a specific way to create documentation comments.
To use it for Single Line Comments use `///`, for Multi Line Comments use the syntax `/** **/`.

```cadence
/// This  a single line documentation comment
/// You can keep them going.

/**
	Multi Line
	Documentation Comment
**/
```

### Conventions

By convention, variables, constants, and functions have lowercase names; and types have title-case names.

### Types

There are several built-in types for cadence.

* Bool
* Integers
    * Int
    * Int8
    * Int16
    * Int32
    * Int64
    * Int128
    * Int256
    * Uint8
    * Uint16
    * Uint32
    * Uint64
    * Uint128
    * Uint256
    * (Uint types which do not check for overflow and underflow are Word)
    * Word8
    * Word16
    * Word32
    * Word64
* String
* Character
* Fixed-Point Numbers
    * (Only Fix64 and UFix64 are available)
    * Fix64
    * UFix64
* Addresses

Each type has some build in functions.

#### Integers

* fun toString(): String
* fun toBigEndianBytes(): [Uint8]
* Uint8.max
* Uint8.min

#### Fixed-Point Number

* fun toString(): String
* fun toBigEndianBytes(): [Uint8]
* Fix64.max
* Fix64.min

#### Address

* fun toString(): String
* fun toBytes(): [Uint8]

#### String

* let length: Int
* let utf8: [Uint8]
* fun concat(_ other: String): String
* fun slice(from: Int, upTo: Int): String
* fun decodeHex(): [Uint8]
* fun toLower(): String
* fun String.encodeHex(_ data: [Uin8]): String

#### Arrays

* let length: Int
* fun concat(_ array: T): T
* fun contains(_ element: T) Bool
* (specific to Variable size Array Functions)
  * fun append(_ element: T): Void
  * fun appendAll(_ array: T) Void
  * fun insert(at index: Int, _ element: T): Void
  * fun remove(at index: Int): T
  * fun removeFirst(): T
  * fun removeLast(): T

#### Dictionary

* let length: Int
* fun insert(key: K, _ value: V) V?
* fun remove(key: K): V?
* let keys: [K]
* let values: [V]
* fun containsKey(key: K): Bool


#### Floating-Point

There is **no** support for floating point numbers.

### AnyStruct and AnyResource

`AnyStruct` is the top type of all non-resource types, i.e., all non-resource types are a subtype of it. (Int8, String, Bool, struct, not type specific)

`AnyResource` is the top type of all resource types.

### Optionals

Optionals are values which can represent the absence of a value.

An optional type is declared using the `?` suffix for another type.
For example, `Int` is a non-optional integer, and `Int?` is an optional integer, i.e. either nothing, or an integer.

The value representing nothing is `nil`.

### Nil-Coalescing Operator

The nil-coalescing operator `??` returns the value inside an optional if it contains a value, or returns an alternative value if the optional has no value, i.e., the optional value is `nil`.

```cadence
let a: Int? = nil
let b: Int = a ?? 42
// `b` is 42, as `a` is nil
```

### Force Unwrap

The force-unwrap operator `!` returns the value inside an optional if it contains a value, or panics and aborts the execution if the optional has no value.

### Never

`Never` can be used as the return type for functions that never return normally.
For example, it is the return type of the function panic.

### String and Characters

Strings are collections of characters.
Strings can be used to work with text in a Unicode-compliant way. Strings are immutable.

### Array Types

Arrays either have a fixed or variable size.

Fixed-size array types have the form `[T; N]`, where `T` is the element type, and `N` is the size of the array.
For example, a fixed-size array of 3 `Int8` elements has the type `[Int8; 3]`.

Variable-size array types have the form `[T]`, where `T` is the element type.
For example, the type `[Int16]` specifies a variable-size array of elements that have type `Int16`.

### Dictionary

Dictionary keys must be hashable and equatable, i.e., must implement the `Hashable` and `Equatable` interfaces.
Most of the built-in types, like booleans and integers, are hashable and equatable, so can be used as keys in dictionaries.

### Swapping

The binary swap operator `<->` can be used to exchange the values of two variables. It is only allowed in a statement and is not allowed in expressions.

```go
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

When arguments are passed to a function, they are copied. Therefore, values that are passed into a function are unchanged in the caller's scope when the function returns.

### Function Preconditions and Postconditions

The functions preconditions and postcondition are block of expressions that check input, outputs, etc.

```cadence
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

Different from the main-stream languages, switches in cadence do not implicit fallthrough.

### Composite Types

Composite types can only be declared within a contract and nowhere else.

There are two types:

* Structures
    * They are copied.
    * They are value types.
* Resources
    * They are moved.
    * They are linear types.

### Resources

Resources are types that can only exist in **one** location at a time and **must** be used **exactly once**. Resources **must** be created (instantiated) by using the `create` keyword.
Resources have the implicit field `let owner: PublicAccount?`, if the resource is currently stored in an account, then the field contains the publicly accessible portion of the account. Otherwise the field is `nil`.
At the end of a function which has resources (variables, constants, parameters) in scope, the resources **must** be either **moved** or **destroyed**.
Resources can be explicitly **destroyed** using the `destroy` keyword.
To make the usage and behaviour of resource types explicit, the prefix `@` must be used in type annotations of variable or constant declarations, parameters, and return types.

#### The Move Operator (←)

In order to move resource from one variable to another the move operator must be used.

```go
// Declare a resource named `SomeResource`, with a variable integer field.
//
pub resource SomeResource {
    pub var value: Int

    init(value: Int) {
        self.value = value
    }
}

// Declare a constant with value of resource type `SomeResource`.
//
let a: @SomeResource <- create SomeResource(value: 0)

// *Move* the resource value to a new constant.
//
let b <- a

// Invalid: Cannot use constant `a` anymore as the resource that it referred to
// was moved to constant `b`.
//
a.value

// Constant `b` owns the resource.
//
b.value // equals 0

// Declare a function which accepts a resource.
//
// The parameter has a resource type, so the type annotation must be prefixed with `@`.
//
pub fun use(resource: @SomeResource) {
    // ...
}

// Call function `use` and move the resource into it.
//
use(resource: <-b)

// Invalid: Cannot use constant `b` anymore as the resource
// it referred to was moved into function `use`.
//
b.value
```

### Equatable Interface

```go
struct interface Equatable {
    pub fun equals(_ other: {Equatable}): Bool
}
```

### Hashable Interface

```go
struct interface Hashable: Equatable {
    pub hashValue: Int
}
```

### Restricted Types

Resources and structs can be used with the restricted types.
For example, a Resource that implements the interface `Balance` can be accessed by accessing only the functions and/or fields available in Balance.

```cadence
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

### Accounts

There are two types of accounts from in cadence.

* PublicAccount
  * The public account is returned from the function `getAccount` and allows the access to the public part of the user account.
* AuthAccount
  * The Auth Account is the authenticated account, only accessible from the `prepare` function of a transaction. This allows access to everything within the account.

#### PublicAccount

```go
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
        // Revoked keys are always returned, but they have \`isRevoked\` field set to true.
        fun get(keyIndex: Int): AccountKey?
    }
}
```

#### AuthAccount

```go
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

There are only three valid domains: `storage`, `private`, and `public`.

Objects in storage are always stored in the `storage` domain. Paths in the storage domain have type `StoragePath`, in the private domain `PrivatePath`, and in the public domain `PublicPath`. `PrivatePath` and `PublicPath` are subtypes of `CapabilityPath`. Both `StoragePath` and `CapabilityPath` are subtypes of `Path`.

### Contracts

Contracts have the implicit field `let account: AuthAccount`, which is the account in which the contract is deployed too. This gives the contract the ability to e.g. read and write to the account's storage.

### Events

Events can only be declared within contracts and cannot use resources as parameters or outputs.

```cadence
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

#### Emitting events

To Emit an event the keyword `emit` is used. For example, emitting an event of the type `Test(field: Int)`: `emit Test(1)`.

#### Core Events

There are some core events built-in in cadence.

* `pub event AccountCreated(address: Address)`
* `pub event AccountKeyAdded(address: Address, publicKey: [UInt8])`
* `pub event AccountKeyRemoved(address: Address, publicKey: [UInt8])`
* `pub event AccountContractAdded(address: Address, codeHash: [UInt8], contract: String)`
* `pub event AccountContractUpdated(address: Address, codeHash: [UInt8], contract: String)`
* `pub event AccountContractRemoved(address: Address, codeHash: [UInt8], contract: String)`
