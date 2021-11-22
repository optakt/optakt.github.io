# Account

Accounts are address on the flow network, e.g. `0xf233dcee88fe0abe`.
Every account can be accessed through two types &#0151 [`PublicAccount`](#publicaccount) and [`AuthAccount`](#authaccount), each corresponding to a level of accessabilty.

Cadence transactions consist of four optional phases &#0151 prepare, precondition, execution and postconditions phase.
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

## PublicAccount

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

## AuthAccount

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

## Account Storage

Each account has storage, and this is where resources and structs can be persisted.
Authorized accounts have full access to the accounts storage.

Objects in storage are stored under paths.
Each storage path location corresponds to a single register.
Paths correspond to the `Key` part of the register ID.
Paths have a format of `/<domain>/<identifier>`.
There are three valid domains &#0151 `storage`, `public` and `private`.
Objects in storage are **always** stored in the `storage` domain &#0151 meaning this is where resources and any other data is kept.
The `public` domain allows the account to let other accounts access the objects, links to `storage`, inside it.
