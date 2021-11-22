# Flow Virtual Machine

The Flow Virtual Machine (FVM) augments the Cadence runtime with the domain-specific functionality required by the Flow protocol.

There are two types of operations that can be executed in the FVM - scripts and transactions.
_Scripts_ are operations that have read-only access to the ledger, while _transactions_ can read or write to it.

Cadence [runtime.Interface](https://github.com/onflow/cadence/blob/master/runtime/interface.go) defines a set of functions Cadence will use to query state from the FVM/ledger.

The `fvm` package provides two execution environments that satisfy the `runtime.Interface` - the [scriptEnv](https://github.com/onflow/flow-go/blob/master/fvm/scriptEnv.go) for scripts, and [transactionEnv](https://github.com/onflow/flow-go/blob/master/fvm/transactionEnv.go) for transaction execution.
These environments allow reading and writing values to the underlying storage (ledger), as expected by the Cadence interpreter.
