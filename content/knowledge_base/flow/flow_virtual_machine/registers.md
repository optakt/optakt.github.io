# Registers

The FVM interacts with the Flow execution state by running atomic operations against a ledger.
What we call a _ledger_ is a type of key/value storage.
It holds an array of key-value pairs called registers.

In Flow, the ledger implementation provides a number of functionalities and guarantees, including speed, memory-efficiency, crash resillience (via write-ahead logs and checkpoints), thread safety etc.
It is also a stateful key/value storage, and every update to the ledger creates a new state.
A limited number of recent states is kept, and updates can be applied to any one of those states.

Each ledger register is referenced by an ID - the *key* and holds a *value* - binary data.

## Register ID to Ledger Path

When referencing storage locations from Flow code, each register is uniquely identified by three components - *owner*, *controller* and *key*.
Owner represents the account to which the register belongs to.
Depending on the register, the *controller* field can be either empty or have the value of the *owner* field.
The *key* field represents the specific storage location of the account.

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

## Common Registers

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

## Flow Contracts

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

These contracts are described in more detail [here](../cadence/contracts/flow_contracts.md).
