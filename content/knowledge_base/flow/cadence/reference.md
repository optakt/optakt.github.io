# Reference

## Comments

Single-line comments in Cadence use `//`, multi-line comments use `/* */`.

```rust
// This is a comment on a single line.

/* This is a comment which
spans multiple lines. */
```

### Documentation Comments

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

## Names

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

## Types

There are several built-in types for Cadence.

### Integer

Cadence supports the following integer types: `Int`, `Int8`, `Int16`, `Int32`, `Int64`, `Int128`, `Int256`, `UInt`, `UInt8`, `UInt16`, `UInt32`, `UInt64`, `UInt128`, `UInt256`, `Word8`, `Word16`, `Word32`, `Word64`.

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBigEndianBytes(): [Uint8]`
* `Uint8.max`
* `Uint8.min`

### Fixed-Point Number

Cadence supports the following fixed-point types: `Fix64`, `UFix64`.

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBigEndianBytes(): [Uint8]`
* `Fix64.max`
* `Fix64.min`

### Address

The supported built-ins for this type family are:

* `fun toString(): String`
* `fun toBytes(): [Uint8]`

### String

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

### Arrays

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

### Dictionary

The supported built-ins for this type family are:

* `let length: Int`
* `fun insert(key: K, _ value: V) V?`
* `fun remove(key: K): V?`
* `let keys: [K]`
* `let values: [V]`
* `fun containsKey(key: K): Bool`

### Floating-Point

There is **no** support for floating point numbers. These kind of numbers, not natural/cardinal, are handle by the Fixed-Point type.

## AnyStruct and AnyResource

`AnyStruct` is the base type of all non-resource types, i.e., all non-resource types are a subtype of it. (`Int8`, `String`, `Bool`, `struct`, not type specific)
`AnyResource` is the base type of all resource types.

## Optionals

Optionals are values which can represent the absence of a value.

An optional type is declared using the `?` suffix for another type.
For example, `Int` is a non-optional integer, and `Int?` is an optional integer, i.e. either an integer, or nothing.

The value representing the absence of a value is `nil`.

## Nil-Coalescing Operator

The nil-coalescing operator `??` returns the value inside an optional if it contains a value, or returns an alternative value if the optional has no value.

```rust
let a: Int? = nil
let b: Int = a ?? 42
// `b` is 42, as `a` is nil
```

## Force Unwrap

The force-unwrap operator `!` returns the value inside an optional if it contains a value, or panics and aborts the execution if the optional has no value.

## Never

`Never` can be used as the return type for functions that never return normally.
For example, it is the return type of the `panic` function.

## Array Types

Arrays either have a fixed or variable size.

Fixed-size array types have the form `[T; N]`, where `T` is the element type, and `N` is the size of the array.
For example, a fixed-size array of 3 `Int8` elements has the type `[Int8; 3]`.

Variable-size array types have the form `[T]`, where `T` is the element type.
For example, the type `[Int16]` specifies a variable-size array of elements that have type `Int16`.

## Dictionary Types

Dictionary keys must be hashable and equatable, i.e., must implement the `Hashable` and `Equatable` interfaces.
Most of the built-in types, like booleans and integers, are hashable and equatable, so can be used as keys in dictionaries.

## Swapping

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

## Argument Passing Behavior

Arguments are passed to functions by value. Therefore, values passed to a function are unchanged in the caller's scope when that function returns.

## Function Preconditions and Postconditions

The functions preconditions and postconditions are blocks of expressions that check input and outputs.

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

## Switches

As opposed to some languages, Cadence does not feature "fall through" in switch statements.

## Composite Types

Composite types can only be declared within a contract and nowhere else.
There are two types:

* Structures
    * They are copied.
    * They are value types.
* Resources
    * They are moved.
    * They are linear types.

## Equatable Interface

```rust
struct interface Equatable {
    pub fun equals(_ other: {Equatable}): Bool
}
```

## Hashable Interface

```rust
struct interface Hashable: Equatable {
    pub hashValue: Int
}
```

## Restricted Types

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

## Events

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

### Emitting Events

To emit an event, the keyword `emit` is used.
For example, emitting an event of the type `Test(field: Int)`: `emit Test(1)`.

### Core Events

There are some core events built-in in Cadence.

* [`pub event AccountCreated(address: Address)`](https://docs.onflow.org/cadence/language/core-events/#account-created)
* [`pub event AccountKeyAdded(address: Address, publicKey: [UInt8])`](https://docs.onflow.org/cadence/language/core-events/#account-key-added)
* [`pub event AccountKeyRemoved(address: Address, publicKey: [UInt8])`](https://docs.onflow.org/cadence/language/core-events/#account-key-removed)
* [`pub event AccountContractAdded(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-added)
* [`pub event AccountContractUpdated(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-updated)
* [`pub event AccountContractRemoved(address: Address, codeHash: [UInt8], contract: String)`](https://docs.onflow.org/cadence/language/core-events/#account-contract-removed)
