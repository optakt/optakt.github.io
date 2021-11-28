# Case study - execution of a Cadence script

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

// The fvm.ScriptProcedure is then run using the Flow Virtual Marchine.
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
