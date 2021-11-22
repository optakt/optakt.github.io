# Flow Go SDK

Flow Go SDK allows users to interact with the Flow Access API.
This allows Go code applications to send requests directly to the network.
The SDK has the ability to send transactions, scripts and get information about the state of the network.

## Sending a Transaction

Flow SDK exposes the method `SendTransaction` to send transactions to the Flow Network.

### Sending a transaction

In this example the amount of 100 FLOW tokens will be sent between the service account and the `RECEIVER_ACCOUNT`.
In the `prepare` phase of the transaction, the reference to the sender's vault is retrieved and the specified amount of tokens is withdrawn to a temporary vault.
In the `execute` phase  of the transaction, the `FungibleToken.Receiver` capability is used to get access to the deposit function of the receiver's vault.
The tokens are then deposited to the receiver from the temporary vault.

```rust
/// 0xFUNGIBLE_TOKEN and 0xFLOW_TOKEN are place for the actual addresses of the contracts that differ from network to network.
import FungibleToken from 0xFUNGIBLE_TOKEN
import FlowToken from 0xFLOW_TOKEN

transaction() {
    let sentVault: @FungibleToken.Vault

    prepare(signer: AuthAccount) {
        // Get a reference to the signer's stored vault
        let vaultRef = signer.borrow<&FlowToken.Vault>(from: /storage/flowTokenVault)
			?? panic("Could not borrow reference to the owner's Vault!")

        // Withdraw tokens from the signer's stored vault
        self.sentVault <- vaultRef.withdraw(amount: 100)
    }

    execute {
        // Get a reference to the recipient's Receiver
        let receiverRef =  getAccount(0xRECEIVER_ACCOUNT)
            .getCapability(/public/flowTokenReceiver)
            .borrow<&{FungibleToken.Receiver}>()
			      ?? panic("Could not borrow receiver reference to the recipient's Vault")

        // Deposit the withdrawn tokens in the recipient's receiver
        receiverRef.deposit(from: <-self.sentVault)
    }
}
```

Using the SDK the script connects to the access node using `client.New(ADDRESS)`, then gets the private key and the first account key of the service account and uses it to pay and sign the transaction.
Finally the transaction is sent.

```go
package main

import (
	"context"

	"github.com/onflow/flow-go-sdk"
	"github.com/onflow/flow-go-sdk/crypto"
)

var (
  cadenceScript             = "Cadence transaction code describing token transfer..."
  serviceAccountAdress      = "0xADDRESS"
  serviceAccountPrivateKey  = "privateKey"
)

func main() {
  ctx := context.Background()

  cli, err := client.New("127.0.0.1", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}

  privateKey, err := crypto.DecodePrivateKeyHex(crypto.ECDSA_P256, serviceAccountPrivateKey)
	if err != nil {
    panic(err)
  }

	addr := flow.HexToAddress(serviceAccountAdress)
	acc, err := cli.GetAccount(ctx, addr)
	if err != nil {
    panic(err)
  }

	accountKey := acc.Keys[0]
	signer := crypto.NewInMemorySigner(privateKey, accountKey.HashAlgo)

	tx := flow.NewTransaction().
		SetPayer(addr).
		SetProposalKey(addr, accountKey.Index, accountKey.SequenceNumber).
		SetScript([]byte(cadenceScript))

	err := tx.SignEnvelope(addr, accountKey.Index, signer)
	if err != nil {
    panic(err)
  }

  err := cli.SendTransaction(ctx, tx)
  if err != nil {
    panic(err)
  }
}
```

## Sending a Script

Flow SDK exposes the method `ExecuteScriptAtLatestBlock` to execute a script on the latest block.
It also offers two more functions to execute scripts on a block at a defined height or referenced by its id: `ExecuteScriptAtBlockHeight` and `ExecuteScriptAtBlockID` respectavelly.

### Flow Account Balance

To get the current FLOW token balance from an account the following cadence script should be executed.
In this script the `ACCOUNT` placeholder is the account address of the target balance.

```rust
import FlowToken from 0xFLOW_TOKEN

pub fun main() {
    let account = getAccount(0xACCOUNT)

    let balanceRef = account.getCapability<&FlowToken.Vault{FlowToken.Balance}>(/public/flowTokenBalance)
        .borrow()
        ?? panic("Could not borrow a reference to the account balance")

    return balanceRef.balance
}
```

To execute the script from the Go SDK first it requires to create a new connection to an access node.
This is done using `client.New(ACCESS_NODE_ADDRESS)`.
With this connection the script can be executed using `ExecuteScriptAtLatestBlock(CONTEXT, CADENCE_SCRIPT, ARGUMENTS)`.

```go
package main

import (
	"context"
	"fmt"

	"github.com/onflow/flow-go-sdk"
)

var (
  cadenceScript = "Cadence script code describing retrieving token balance..."
)

func main() {
  ctx := context.Background()

  cli, err := client.New("127.0.0.1", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}

	script := []byte(cadenceScript)
	value, err := cli.ExecuteScriptAtLatestBlock(ctx, script, nil)
  if err != nil {
    panic(err)
  }
	fmt.Printf("\nValue: %s", value.String())
}
```
