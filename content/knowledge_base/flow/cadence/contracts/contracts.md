# Contracts

A contract in Cadence is a collection of definitions of interfaces, structs, resources, data (its state) and code (its functions).
Contracts live in the contract storage area of an account and they can be added, updated and removed.

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

## Deploying a Contract

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

## Updating a Contract

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

## Removing a Contract

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
