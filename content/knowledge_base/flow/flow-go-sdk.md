# Flow Go SDK

Flow Go SDK allows users to interact with the Flow API.
This allows users from the Go Programming Language to send requests directly to the network.
The SDK has the ability to send transactions, scripts and get information about the state of the network.

## Sending a Transaction

Flow SDK exposes the method `SendTransaction` to send transactions to the Flow Network.

### Sending a transaction

In this example the amount of 100 flow tokens will sent between the service account and the `RECEIVER_ACCOUNT`.
In the cadence transaction code, the prepare statement is used to get the service account vault, sender, then it gets the restricted type `FungibleToken.Receiver` is used in order to get access to the deposit function of the vault.

```cadence
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
        let receiverRef =  getAccount(OxRECEIVER_ACCOUNT)
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
  cadenceScript             = `(...)`
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

	err := flowClient.SendTransaction(ctx, tx)
	if err != nil {
		panic(err)
	}
}
```

## Sending a Script

Flow SDK exposes the method `ExecuteScriptAtLatestBlock` to execute script on the latest block.
It also offers two more functions to execute Scripts at a defined height or block id, `ExecuteScriptAtBlockHeight` and `ExecuteScriptAtBlockID` respectavelly.

### Flow Account Balance

To get the current flow token balance from an account the following cadence script should be executed.
In this script the `ACCOUNT` placeholder is the account address of the target balance.

```cadence
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
Using `client.New(ACCESS_NODE_ADDRESS)` a new connection to the access node is created.
With this connection the script can be executed using `ExecuteScriptAtLatestBlock(CONTEXT, CADENCE_SCRIPT, ARGUMENTS)`.

```go
package main

import (
	"context"
	"fmt"

	"github.com/onflow/flow-go-sdk"
)

var (
  cadenceScript = `(...)` // Cadence Script
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
