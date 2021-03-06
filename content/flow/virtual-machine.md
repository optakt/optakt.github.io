# Flow

## Registers

The FVM interacts with the Flow execution state by running atomic operations against a ledger.
What we call a _ledger_ is a type of key/value storage.
It holds an array of key-value pairs called registers.

In Flow, the ledger implementation provides a number of functionalities and guarantees, including speed, memory-efficiency, crash resilience (via write-ahead logs and checkpoints), thread safety etc.
It is also a stateful key/value storage, and every update to the ledger creates a new state.
A limited number of recent states is kept, and updates can be applied to any one of those states.

Each ledger register is referenced by an ID — the _key_ and holds a _value_ — binary data.

### Register ID to Ledger Path

When referencing storage locations from Flow code, each register is uniquely identified by three components — _owner_, _controller_ and _key_.
Owner represents the account to which the register belongs to.
Depending on the register, the _controller_ field can be either empty or have the value of the _owner_ field.
The _key_ field represents the specific storage location of the account.

Definition of a register ID ([source](https://github.com/onflow/flow-go/blob/master/model/flow/ledger.go)):

```go
type RegisterID struct {
    Owner      string
    Controller string
    Key        string
}
```

Register ID is converted to a `ledger.Key` by concatenating specific key parts ([source](https://github.com/onflow/flow-go/blob/master/engine/execution/state/state.go)).

```go
const (
    KeyPartOwner      = uint16(0)
    KeyPartController = uint16(1)
    KeyPartKey        = uint16(2)
)

// ...

func RegisterIDToKey(reg flow.RegisterID) ledger.Key {
    return ledger.NewKey([]ledger.KeyPart{
        ledger.NewKeyPart(KeyPartOwner, []byte(reg.Owner)),
        ledger.NewKeyPart(KeyPartController, []byte(reg.Controller)),
        ledger.NewKeyPart(KeyPartKey, []byte(reg.Key)),
    })
}
```

In order to map a Ledger key to a ledger path, the key can be converted to a corresponding path using [pathfinder.KeyToPath](https://pkg.go.dev/github.com/onflow/flow-go/ledger/common/pathfinder#KeyToPath).
Depending on the version of the ledger pathfinder version, ledger path is either `SHA256` or `SHA3-256` (current version) hash of the canonical form of the ledger key.

The canonical form of the ledger key is the concatenation of the individual key parts, taking into account the key part type ([source](https://github.com/onflow/flow-go/blob/master/ledger/ledger.go))

```go
func (k *Key) CanonicalForm() []byte {
    ret := ""
    for _, kp := range k.KeyParts {
        ret += fmt.Sprintf("/%d/%v", kp.Type, string(kp.Value))
    }
    return []byte(ret)
}
```

Effectively, this means that for a given register, the ledger path is the `SHA3-256` hash of the `/0/<owner>/1/<controller>/2/<key>`, where `<controller>` is typically either the `<owner>` field or an empty string, depending on the register.

### Common Registers

There's a number of registers that have specific meaning and are often encountered in Flow.

The following registers can be typically found in a Flow account:

- `exists` - does the account exist or not
- `frozen` - is the account frozen or not
- `contract_names` - list of the names of contracts deployed to an account
- `public_key_count` - number of public keys an account has
- `public_key_<N>` - location of the N-th public key of an account
- `storage_used` - amount of storage used by the account, in bytes
- `storage_index` - used to keep track of a number of registers in the owner account
- `code.<name>` - register where the `name` Cadence contract is stored

### Flow Contracts

There are core Cadence contracts that are deployed on the Flow network as soon as it is bootstrapped.
These contracts are essential to the normal operation of the Flow network.
These contracts are:

- `FlowToken`
- `FungibleToken`
- `FlowServiceAccount`
- `FlowEpoch`
- `FlowFees`
- `FlowStorageFees`
- `FlowClusterQC`
- `FlowDKG`
- `FlowIDTableStaking`
- `FlowStakingCollection`
- `LockedTokens`
- `StakingProxy`

These contracts are described in more detail in the [Cadence document](cadence.md#flow-contracts).

## Flow Virtual Machine

The Flow Virtual Machine (FVM) augments the Cadence runtime with the domain-specific functionality required by the Flow protocol.

There are two types of operations that can be executed in the FVM - scripts and transactions.
_Scripts_ are operations that have read-only access to the ledger, while _transactions_ can read or write to it.

Cadence [runtime.Interface](https://github.com/onflow/cadence/blob/master/runtime/interface.go) defines a set of functions Cadence will use to query state from the FVM/ledger.

The `fvm` package provides two execution environments that satisfy the `runtime.Interface` - the [scriptEnv](https://github.com/onflow/flow-go/blob/master/fvm/scriptEnv.go) for scripts, and [transactionEnv](https://github.com/onflow/flow-go/blob/master/fvm/transactionEnv.go) for transaction execution.
These environments allow reading and writing values to the underlying storage (ledger), as expected by the Cadence interpreter.

## Case study - execution of a Cadence script

This section is a detailed walkthrough for a simple scenario, in which a simple Cadence script is executed.
It includes the creation of a Flow Virtual machine, Cadence interpreter and runtime, as well as the interaction with the underlying storage (ledger) provided by the `ScriptEnv`.

```rust
// In this example 0x01 represents the address where the contract is located.
import FungibleToken from 0x01

pub fun main(): UFix64 {
    return 1.0
}
```

To execute this, we first need to create the FVM interpreter runtime in order to execute Cadence scripts and procedures.
Default implementation of the Cadence interpreter runtime can be found in Cadence [runtime/runtime.go](https://github.com/onflow/cadence/blob/master/runtime/runtime.go).
Then, the Flow Virtual Machine is created.

```go
runtime := fvm.NewInterpreterRuntime()
vm := fvm.NewVirtualMachine(runtime)
```

Now, we will create a `fvm.ScriptProcedure`.
A `Procedure` in the context of the FVM is an operation that reads or updates the ledger state.
Both Cadence scripts and transactions are procedures, via the [ScriptProcedure](https://github.com/onflow/flow-go/blob/master/fvm/script.go) and [TransactionProcedure](https://github.com/onflow/flow-go/blob/master/fvm/transaction.go) types.

```go
// Create a fvm.ScriptProcedure with the given script.
// script argument is a byte slice with the Cadence script text.
procedure := fvm.Script(script)

// The fvm.ScriptProcedure is then run using the Flow Virtual Machine.
err = vm.Run(ctx, procedure, view, programs)
```

When using the FVM to run a procedure, it invokes the procedure's `Run()` method.
During the preparation for running the script, first a new `ScriptEnv` is created, and then the `ExecuteScript()` method of the runtime is executed.

```go
func (i ScriptInvocator) Process(vm *VirtualMachine, ctx Context, proc *ScriptProcedure, sth *state.StateHolder, programs *programs.Programs) error {

    env := NewScriptEnvironment(ctx, vm, sth, programs)

    value, err := vm.Runtime.ExecuteScript(
        // ...
        runtime.Context{
            Interface: env,
            // ...
        },
    )
}
```

The `ScriptEnv` created earlier is used as the runtime storage provider.

```go
func (r *interpreterRuntime) ExecuteScript(script Script, context Context) (cadence.Value, error) {
    context.InitializeCodesAndPrograms()

    runtimeStorage := newRuntimeStorage(context.Interface)
    // ...
}
```

This runtime storage handler is in charge of I/O operations when it comes to the ledger.

The Cadence runtime then parses the provided script and check its correctness (via the `parseAndCheckProgram()` method of the runtime).
One of the things that the check does is look at any imports found, and it uses the interpreter's `GetAccountContractCode()` method to load it.
As the interpreter's storage provider was previously initialized to `ScriptEnv`, it's effectively invoking `ScriptEnv`s `GetAccountContractCode()` method.

`GetAccountContractCode()` checks whether the account is frozen, and, if not, tries to read the contract code from the appropriate location.
Skipping few levels down the call stack, `GetAccountContractCode()` will result in a call to [fvm/state/accounts.go](https://github.com/onflow/flow-go/blob/master/fvm/state/accounts.go) `getValue()` method.

The code of `getValue()` shown below demonstrates two distinct solutions for resolving parameters for the register getter, the difference being the `isController` boolean flag.

```go
func (a *StatefulAccounts) getValue(address flow.Address, isController bool, key string) (flow.RegisterValue, error) {
    if isController {
        return a.stateHolder.State().Get(string(address.Bytes()), string(address.Bytes()), key)
    }
    return a.stateHolder.State().Get(string(address.Bytes()), "", key)
}
```

The `State`s `Get()` method shown there can be simplified to this:

```go
func (s *State) Get(owner, controller, key string) (flow.RegisterValue, error) {
    return s.view.Get(owner, controller, key);
}
```

To get to the origin of the `StateHolder`/`State` objects, we need to look back at the FVM creation.
For instance, in the Flow DPS [invoker](https://pkg.go.dev/github.com/optakt/flow-dps/service/invoker) code, the `state` provider, named `view` below, is created like this:

```go
read := readRegister(i.index, i.cache, height)
view := delta.NewView(read)
vm.Run(ctx, procedure, view, programs)
```

What's important to note is that the `readRegister` function returns a `GetRegisterFunc`, in charge of returning value for an appropriate register.
In the Flow DPS ecosystem, it in essence means translating the `owner`, `controller` and `key` combination into a ledger path and reading it from the DPS index at the appropriate height.

```go
type GetRegisterFunc func(owner, controller, key string) (flow.RegisterValue, error)
```

What's done with the `ledger.Value`/`flow.RegisterValue` later depends on the context.
In the case of previous script example, the contents of the register `/0/0x01/1/0x01/2/code.FungibleToken` are shown as plaintext and can be imported to the Cadence program that is to be executed.
The data in the register itself may be formatted differently based on the specific register, and it is up to the caller to unpack it correctly.
For instance, the `code.FungibleToken` register contains a plaintext Cadence script.
On the other hand, the `contract_names` register contains a CBOR-encoded array of strings.
Registers can also contain complex types such as dictionaries, structs or arrays.

## Flow Localnet

TODO

### Starting a Localnet

- `git clone https://github.com/onflow/flow-go.git`
- `cd crypto ; go generate`
- `cd ../internal/localnet`
- `make init`
- `make start`

### Indexing Localnet's past Sporks with Flow DPS

TODO

### Using Flow Insights to inspect some Registers

TODO
