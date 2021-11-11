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

Since the version `1.7` of Go, tests can be run in parallel. This can be done by calling [`t.Parallel`](https://pkg.go.dev/testing#T.Parallel) in each subtest. Calling this function signals that the test is to be run in parallel with (and only with) other parallel tests, and the amount of tests running in parallel is limited by the value of [`runtime.GOMAXPROCS`](https://pkg.go.dev/runtime#GOMAXPROCS). 

There are multiple advantages to parallelizing tests:

* It ensures that regardless of the order in which inputs are given, components behave as expected.
* It maximizes performance, which in turns results in a faster CI and a faster workflow for everyone.
* By making tests run faster, it allows you to write more tests and therefore produce more actionable data to find bugs as well as improve tests cases and coverage.
* It makes it possible to ensure that the components you test are concurrency-safe

!!! danger "Parallelizing table-driven tests"

    When it comes to table-driven tests, a common caveat developers fall into is to call `t.Parallel` in their subtest without capturing the loop variable with their test case.

    Here is an example of how it should be done:

    ```go
    func TestParseCadenceArgument(t *testing.T) {
        tests := []struct {
            name     string
            param    string
            wantArg  cadence.Value
            checkErr assert.ErrorAssertionFunc
        }{
            {
                name:     "parse valid boolean",
                param:    "Bool(true)",
                wantArg:  cadence.Bool(true),
                checkErr: assert.NoError,
            },
            {
                name:     "parse invalid boolean",
                param:    "Bool(horse)",
                checkErr: assert.Error,
            },
            {
                name:     "parse valid normal integer",
                param:    "Int16(1337)",
                wantArg:  cadence.Int16(1337),
                checkErr: assert.NoError,
            },
        }
    
        for _, test := range tests {
            test := test // It is essential to capture the loop variable, or each parallel test will run with the inputs of the last test from the table.
            t.Run(test.name, func(t *testing.T) {
                t.Parallel()
    
                gotArg, err := ParseCadenceArgument(test.param)
                test.checkErr(t, err)
    
                if err == nil {
                    assert.Equal(t, test.wantArg, gotArg)
                }
            })
        }
    }
    ```

### Standard `testing` Package

The standard [`testing`](https://pkg.go.dev/testing) package is very powerful, and does not require additional frameworks to be used efficiently. The only exception we make to that are the `stretchr/testify/assert` and `stretchr/testify/require` packages which we use only for convenience, as they expose assertion functions that produce consistent outputs and make tests easy to understand.

#### Subtests and Sub-benchmarks

The `testing` package exposes a `Run` method on the `T` type which makes it possible to nest tests within tests. This can be very useful, as it enables creating a hierarchical structure within a test.

??? example "index_internal_test.go"

    ```go
    func TestIndex(t *testing.T) {
        // ...
        t.Run("collections", func(t *testing.T) {
            t.Parallel()
    
            collections := mocks.GenericCollections(4)
    
            reader, writer, db := setupIndex(t)
            defer db.Close()
    
            assert.NoError(t, writer.Collections(mocks.GenericHeight, collections))
            // Close the writer to make it commit its transactions.
            require.NoError(t, writer.Close())
    
            // NOTE: The following subtests should NOT be run in parallel, because of the deferral
            // to close the database above.
            t.Run("retrieve collection by ID", func(t *testing.T) {
                got, err := reader.Collection(collections[0].ID())
    
                require.NoError(t, err)
                assert.Equal(t, collections[0], got)
            })
    
            t.Run("retrieve collections by height", func(t *testing.T) {
                got, err := reader.CollectionsByHeight(mocks.GenericHeight)
    
                require.NoError(t, err)
                assert.ElementsMatch(t, mocks.GenericCollectionIDs(4), got)
            })
    
            t.Run("retrieve transactions from collection", func(t *testing.T) {
                // For now this index is not used.
            })
        })
        // ...
    }
    ```

#### Table-Driven Tests

It makes a lot of sense to use subtests when testing the behavior of complex components, but it is better to use [table-driven tests](https://dave.cheney.net/2019/05/07/prefer-table-driven-tests) when testing a simple function with a given output for a given input, and that there are many cases to cover. For such cases, table-driven tests massively improve clarity and readability.

They should not be used blindly for all tests, however. In cases where the tested component is complex and that testing its methods cannot be simplified to a common setup, call to a function and assertion of the output, trying to use table-driven tests at all costs might lead to messy code, where the subtest which runs the test case is full of conditions to try to handle each separate setup. This is usually a sign that using simple tests and subtests would be a better approach.

### Case Study: The Flow DPS Mapper

Sometimes, a core piece of software might seem impossible to test. That was the case for the mapper component in Flow DPS at some point, where its main function consisted of a 453-lines-long loop which orchestrated the use of all the other components of the application.

??? example "mapper_old.go"
    ```go
{!mapper_old.md!}
    ```

As it was, this code was untestable. Covering each possible case from this huge piece of logic would have required immense, complex, unreadable tests, that would break whenever a piece of this logic would change, and this would require a huge amount of maintenance effort.

To solve that massive problem, we refactored our original mapper into a [finite-state machine](https://en.wikipedia.org/wiki/Finite-state_machine) which replicates the same computation logic by applying transitions to a state.

??? example "mapper_new.go"
    ```go
{!mapper_new.md!}
    ```

This refactoring effort allowed us to write simple and concise tests that call a transition function upon the state machine and make assertions upon the resulting state.

??? example "mapper_new_internal_test.go"
    ```go
{!mapper_new_internal_test.md!}
    ```

## Integration Tests

TODO: Explain build tag usage to prevent integration tests from being run at all times.
TODO: Explain how we write integration tests.

## Benchmarks & Examples

TODO: Explain how projects/packages that are meant to be used as libraries should ideally provide benchmarks and examples on top of proper documentation.
