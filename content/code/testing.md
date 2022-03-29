# Testing Guide

At Optakt, we consistently write tests to ensure a reliable engineering environment where quality is paramount.
Over the course of the product development life cycle, testing saves time and money, and helps developers write better code, more efficiently.
Untested code is fragile, difficult to maintain and becomes questionable as soon as changes are made.

This guide assumes that you are already familiar with Go testing.

!!! warning "External projects"

    This guide assumes that you are dealing with an Optakt project.
    When working on projects for clients, they might have different testing practices.
    In such a case, first make sure to keep your tests consistent with their own, and if you have any doubts please voice them on Slack.
    If the tests of the original project are really problematic, we can discuss refactoring them and changing the way their tests are written in the future to improve code quality and reduce technical debt.

## Unit Tests

This section outlines a few of the rules we try to follow when it comes to Go unit tests.

### Naming Conventions

Unit tests should have consistent names.
The best way to go about it is to follow the [official guidelines of the Go `testing` package](https://pkg.go.dev/testing#hdr-Examples), which states that:

> The naming convention to declare tests for the package, a function `F`, a type `T` and method `M` on type `T` are:
>
> ```go
>   func Test() { ... }
>   func TestF() { ... }
>   func TestT() { ... }
>   func TestT_M() { ... }
> ```

You might sometimes want to have multiple test functions for a single method, in which case the tests should be prefixed as defined above, and followed with the purpose of the test:

```go
func TestMyType_MethodDoesSomethingWhenX(t *testing.T) { ... }
func TestMyType_MethodDoesHandlesXFailure(t *testing.T) { ... }
// And so on.
```

When it comes to subtests, the names of individual subtests should be lowercased and concise. The tests usually start
with a subtest called `nominal case` which verifies that the tested component behaves as expected in a baseline
situation, where no failures occur and no edge cases are handled. Subsequent tests should follow the paths through which
the function can flow from top to bottom.

With the following example:

```go
{!function_example.md!}
```

There are three possible paths through which this function can be traversed:

* The nominal case, where `s.validate.Struct` and `s.index.HeightForBlock` return no errors and the function returns a
  valid response.
* The case where `s.validate.Struct` returns an error.
* The case where `s.index.HeightForBlock` returns an error.

And here is what the tests should look like:

```go
{!function_example_tests.md!}
```

### Internal Unit Tests

In most cases, packages can be tested using external tests only.
When writing tests for a package called `xyz`, the external tests should be in the same folder, but in a package called `xyz_test`.
This case is [handled by Go natively](https://pkg.go.dev/cmd/go@master#hdr-Test_packages) and will therefore not result in complaints about there being two different packages within the same directory.
Of course, there are exceptions. If you need to test some internal logic, those tests must be in a file suffixed with `_internal_test.go`.

### Mocks

When it comes to mocking dependencies for tests, we prefer to use simple hand-made mocks rather than to use testing frameworks to generate them.
The Go language makes it easy to do so elegantly by creating structures that implement the interfaces for dependencies of the tested code and exposing functions that match the interface's signature as attributes that can be overridden externally.

```go
{!mock_example.md!}
```

Using those mocks is as simple as instantiating a baseline version of the mock and setting its attributes to the desired functions:

```go
{!mock_test_example.md!}
```

#### Mocks Package

When the same mock needs to be used within multiple packages, it can make sense to create a package to store common mocks.
This package should be located at `./testing/mocks` from the root of the repository, and each file within it should define a single mock.

### Pseudorandom Generic Values

When using test data for unit tests, it is often a good idea to use random generated data as the inputs.
This avoids the bias where a test passes because it is given a valid set of inputs while some other inputs might have highlighted a flaw in the logic, by using an _unconstrained_ data set.

In order for the tests to be repeatable and for results to be consistent though, the given inputs should not be completely random, but instead they should be pseudorandom, with the same initial seed, to ensure the same sequence of "random" tests.

Here is an example of such a value being generated.

```go
{!generic.md!}
```

!!! warning

    While randomly generating valid inputs makes sense, randomly generating invalid inputs does not. In the case of invalid inputs, it is much better to have an exhaustive list of all types of cases that are expected to be invalid and always test each one of them.

#### Generic Values Package

When the same generic value need to be used within multiple packages, it can make sense to create a package to store common generic values.
This package should be located at `./testing/mocks` from the root of the repository, and values should be in a file called `generic.go`.

### Parallelization

Since the version `1.7` of Go, tests can be run in parallel.
This can be done by calling [`t.Parallel`](https://pkg.go.dev/testing#T.Parallel) in each subtest.
Calling this function signals that the test is to be run in parallel with (and only with) other parallel tests, and the amount of tests running in parallel is limited by the value of [`runtime.GOMAXPROCS`](https://pkg.go.dev/runtime#GOMAXPROCS).

There are multiple advantages to parallelizing tests:

* It ensures that regardless of the order in which inputs are given, components behave as expected.
* It maximizes performance, which in turns results in a faster CI and a faster workflow for everyone, which allows us to write more tests and therefore produce more actionable data to find bugs as well as improve tests cases and coverage.
* It makes it possible to ensure that the components you test are concurrency-safe

!!! danger "Parallelizing table-driven tests"

    When it comes to table-driven tests, a common pitfall developers fall into is to call `t.Parallel` in their subtest without capturing the loop variable with their test case.

    Here is an example of how it should be done:

    ```go
{!table_driven_parallel.md!}
    ```

### Standard `testing` Package

The standard [`testing`](https://pkg.go.dev/testing) package is very powerful, and does not require additional frameworks to be used efficiently.
The only exception we make to that are the `stretchr/testify/assert` and `stretchr/testify/require` packages which we use only for convenience, as they expose assertion functions that produce consistent outputs and make tests easy to understand.

#### Subtests and Sub-benchmarks

The `testing` package exposes a `Run` method on the `T` type which makes it possible to nest tests within tests.
This can be very useful, as it enables creating a hierarchical structure within a test.

??? example "index_internal_test.go"

    ```go
{!index_internal_test.md!}
    ```

##### Structure

In those tests, usually the structure is composed of:

* At the top-level, the common testing variables that are used in multiple subtests.
    * This can be a logger that discards logs, some test input that can be reused, expected return values shared between tests, and so on.
* In each sub-test:
    * The test is defined as parallelized;
    * The inputs and outputs are defined in variables;
    * The mocks are defined and their methods overridden when necessary;
    * If test is for a method, the method's struct (the subject of the test) is created and given the mocks;
    * The tested method is called on the subject, its return values are often stored in `got` and `err`: `got, err := mything.Do(input)`;
    * Assertions are made on the returned values.

#### Table-Driven Tests

It makes a lot of sense to use subtests when testing the behavior of complex components, but it is better to use [table-driven tests](https://dave.cheney.net/2019/05/07/prefer-table-driven-tests) when testing a simple function with an expected output for a given input, and that there are many cases to cover.
For such cases, table-driven tests massively improve clarity and readability.

They should not be used blindly for all tests, however.
In cases where the tested component is complex and that testing its methods cannot be simplified to a common setup, call to a function and assertion of the output, trying to use table-driven tests at all costs might lead to messy code, where the subtest which runs the test case is full of conditions to try to handle each separate setup.
This is usually a sign that using simple tests and subtests would be a better approach.

### Case Study: The Flow DPS Mapper

Sometimes, a core piece of software might seem impossible to test.
That was the case for the mapper component in Flow DPS at some point, where its main function consisted of a 453-lines-long loop which orchestrated the use of all the other components of the application.

??? example "mapper_old.go"

    ```go
{!mapper_old.md!}
    ```

As it was, this code was untestable.
Covering each possible case from this huge piece of logic would have required immense, complex, unreadable tests, that would break whenever a piece of this logic would change, and this would require a huge amount of maintenance effort.

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

Integration tests are essential to ensure that components work together as expected.
Those tests are usually much heavier and slower than unit tests, since they use real components instead of simple mocks, and often might run filesystem or network operations, wait for things to happen, or even run heavy computational tasks.

Integration tests should always be specified in a separate test package and never run internally within the tested package.

### Build Tag

Because integration tests are inherently slower than unit tests, they are placed in specific files that are suffixed with `_integration_test.go` and those files start with a build tag directive which prevents them from running unless the `go test` command is called with the `integration` tag.

Both syntaxes should be specified, the `<go1.17` one which is `+build <tag>` as well as the `>=go1.17` one which is `go:build <tag>`.
The former will be dropped when we feel like it is no longer relevant to support go 1.16 and prior.

```go
//go:build integration
// +build integration

package dps_test
```

## Examples

In Go, good package documentation includes not only comments for each public type and method, but also runnable examples and benchmarks in some cases.
Godoc allows defining examples which are verified by running them as tests and can be manually launched by readers of the documentation on the package's Godoc webpage.

As for typical tests, examples are functions that reside in a package's `_test.go` files.
Unlike normal test functions, though, example functions take no arguments and begin with the word `Example` instead of `Test`.

In order to specify what is the expected output of a given example, a comment has to be written at the end of the `Example` function, in the form of `// Output: <expected output>`.
If this is missing, examples will not be executed and therefore not included in the documentation.

## Benchmarks

When a package exposes a performance-critical piece of code, it should be benchmarked, and benchmark tests must be available for anyone to reproduce the benchmark using their hardware.
Writing benchmark results in a markdown file without providing a way to reproduce them is irrelevant.

```go
{!benchmark_example.md!}
```
