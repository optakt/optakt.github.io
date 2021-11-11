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
