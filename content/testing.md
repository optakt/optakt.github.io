# Testing Guide

At Optakt, we consistently write tests to ensure a reliable engineering environment where quality is paramount. Over the course of the product development life cycle, testing saves time and money, and helps developers write better code, more efficiently.

This guide assumes that you are already familiar with Go testing.

## Unit Tests

This section outlines a few of the rules we try to follow when it comes to Go unit tests.

### Internal Unit Tests

In most cases, packages can be tested using external tests only. When writing tests for a package called `xyz`, the external tests should be in the same folder, but in a package called `xyz_test`. This case is [handled by Go natively](https://pkg.go.dev/cmd/go@master#hdr-Test_packages) and will therefore not result in complaints about there being two different packages within the same directory. Of course, there are exceptions. If you need to test some internal logic, those tests must be in a file suffixed with `_internal_test.go`.

### Mocks

When it comes to mocking dependencies for tests, we prefer to use simple hand-made mocks rather than to use testing frameworks to generate them. The Go language makes it easy to do so elegantly by creating structures that implement the interfaces for dependencies of the tested code and exposing functions that match the interface's signature as attributes that can be overridden externally.

```go
package mocks

import (
	"testing"

	"github.com/onflow/flow-go/ledger/complete/mtrie/trie"
)

type Loader struct {
	TrieFunc func() (*trie.MTrie, error)
}

func BaselineLoader(t *testing.T) *Loader {
	t.Helper()

	l := Loader{
		TrieFunc: func() (*trie.MTrie, error) {
			return GenericTrie, nil
		},
	}

	return &l
}

func (l *Loader) Trie() (*trie.MTrie, error) {
	return l.TrieFunc()
}
```

Using those mocks is as simple as instantiating a baseline version of the mock and setting its attributes to the desired functions:

```go
// ...
    t.Run("handles failure to load checkpoint", func(t *testing.T) {
        t.Parallel()

        tr, st := baselineFSM(t, StatusEmpty)

        load := mocks.BaselineLoader(t)
        load.CheckpointFunc = func() (*trie.MTrie, error) {
            return nil, mocks.GenericError
        }

        tr.load = load

        err := tr.BootstrapState(st)
        assert.Error(t, err)
    })
// ...
```

### Pseudorandom Generic Values

When using test data for unit tests, it is always a good idea to use random generated data as the inputs. This avoids the bias where a test passes because it is given a valid set of inputs while some other inputs might have highlighted a flaw in the logic, by using an _unconstrained_ data set.

In order for the tests to be repeatable and for results to be consistent though, the given inputs should not be completely random, but instead they should be pseudorandom, with the same initial seed, to ensure the same sequence of "random" tests.

Here is an example of such a value being generated.

```go
func GenericAddresses(number int) []flow.Address {
	// Ensure consistent deterministic results.
	random := rand.New(rand.NewSource(5))

	var addresses []flow.Address
	for i := 0; i < number; i++ {
		var address flow.Address
		binary.BigEndian.PutUint64(address[0:], random.Uint64())

		addresses = append(addresses, address)
	}

	return addresses
}

func GenericAddress(index int) flow.Address {
	return GenericAddresses(index + 1)[index]
}
```

!!! warning
    While randomly generating valid inputs makes sense, randomly generating invalid inputs does not. In the case of invalid inputs, it is much better to have an exhaustive list of all types of cases that are expected to be invalid and always test each one of them.

### Parallelization

### Standard `testing` Package

TODO: Explain subtest and structure of nested tests.
TODO: Explain when to use subtests vs when to use table-driven tests.

### Case Study: The Flow DPS Mapper

TODO: Example of how we made the Flow DPS mapper testable.

## Integration Tests

TODO: Explain build tag usage to prevent integration tests from being run at all times.
TODO: Explain how we write integration tests.

## Benchmarks & Examples

TODO: Explain how projects/packages that are meant to be used as libraries should ideally provide benchmarks and examples on top of proper documentation.
