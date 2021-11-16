# Go Code Style Guidelines

## Guidelines

### Copy Slices and Maps at Boundaries

Slices and maps contain pointers to the underlying data so be wary of scenarios when they need to be copied.

#### Receiving Slices and Maps

Keep in mind that users can modify a map or slice you received as an argument if you store a reference to it.

<table>
<thead><tr><th>Bad</th> <th>Good</th></tr></thead>
<tbody>
<tr>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

</td>
<td>

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

</td>
</tr>

</tbody>
</table>

#### Returning Slices and Maps

Similarly, be wary of user modifications to maps or slices exposing internal state.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  return s.counters
}

// snapshot is no longer protected by the mutex, so any
// access to the snapshot is subject to data races.
snapshot := stats.Snapshot()
```

</td><td>

```go
type Stats struct {
  mu sync.Mutex
  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()
```

</td></tr>
</tbody></table>

### Start Enums at One

The standard way of introducing enumerations in Go is to declare a custom type and a `const` group with `iota`.
Since variables have a 0 default value, you should usually start your enums on a non-zero value.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

</td><td>

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

</td></tr>
</tbody></table>

There are cases where using the zero value makes sense, for example when the zero value case is the desirable default behavior.

```go
type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

### Error Types

There are various options for declaring errors:

* [errors.New] for errors with simple static strings
* [fmt.Errorf] for formatted error strings
* Custom types that implement an `Error()` method
* Wrapped errors using ["pkg/errors".Wrap]

When returning errors, consider the following to determine the best choice:

* Is this a simple error that needs no extra information? If so, [errors.New] should suffice.
* Do the clients need to detect and handle this error? If so, you should use a custom type, and implement the `Error()` method.
* Are you propagating an error returned by a downstream function? If so, check the [section on error wrapping](#error-wrapping).
* Otherwise, [fmt.Errorf] is okay.

  [errors.New]: https://golang.org/pkg/errors/#New
  [fmt.Errorf]: https://golang.org/pkg/fmt/#Errorf
  ["pkg/errors".Wrap]: https://godoc.org/github.com/pkg/errors#Wrap

If the client needs to detect a specific error case, a sentinel error should be created using [errors.New].

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  err := foo.Open()
  if err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

err := foo.Open()
if err != nil {
  if errors.Is(err, foo.ErrCouldNotOpen) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

</td></tr>
</tbody></table>

If you have an error that clients may need to detect, and you would like to add more information to it (e.g., it is not a static string), then you should use a custom type.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  err := open("testfile.txt")
  if err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td><td>

```go
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  err := open("testfile.txt")
  if err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

</td></tr>
</tbody></table>

Be careful with exporting custom error types directly since they become part of the public API of the package.
It is preferable to expose matcher functions to check the error instead.

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

err := foo.Open("foo")
if err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

### Error Wrapping

There are three main options for propagating errors if a call fails:

* Return the original error if there is no additional context to add and you want to maintain the original error type.
* Add context using ["pkg/errors".Wrap] so that the error message provides more context and ["pkg/errors".Cause] can be used to extract the original error.
* Use [fmt.Errorf] if the callers do not need to detect or handle that specific error case.

It is recommended to add context where possible so that instead of a vague error such as "connection refused", you get more useful errors such as "call service foo: connection refused".

See also [Don't just check errors, handle them gracefully].

["pkg/errors".Cause]: https://godoc.org/github.com/pkg/errors#Cause
[Don't just check errors, handle them gracefully]: https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully

### Handle Type Assertion Failures

The single return value form of a [type assertion] will panic on an incorrect type.
Therefore, always use the "comma ok" idiom.

[type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

</td></tr>
</tbody></table>

### Avoid Mutable Globals

Avoid mutating global variables, instead opting for dependency injection.
This applies to function pointers as well as other kinds of values.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// sign.go

var _timeNow = time.Now

func sign(msg string) string {
  now := _timeNow()
  return signWithTime(msg, now)
}
```

</td><td>

```go
// sign.go

type signer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *signer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```
</td></tr>
<tr><td>

```go
// sign_test.go

func TestSign(t *testing.T) {
  oldTimeNow := _timeNow
  _timeNow = func() time.Time {
    return someFixedTime
  }
  defer func() { _timeNow = oldTimeNow }()

  assert.Equal(t, want, sign(give))
}
```

</td><td>

```go
// sign_test.go

func TestSigner(t *testing.T) {
  s := newSigner()
  s.now = func() time.Time {
    return someFixedTime
  }

  assert.Equal(t, want, s.Sign(give))
}
```

</td></tr>
</tbody></table>

### Avoid `init()`

Avoid `init()` where possible.
When `init()` is unavoidable or desirable, code should attempt to:

1. Be completely deterministic, regardless of program environment or invocation.
2. Avoid depending on the ordering or side effects of other `init()` functions.
   While `init()` ordering is well-known, code can change, and thus relationships between `init()` functions can make code brittle and error-prone.
3. Avoid accessing or manipulating global or environment state, such as machine information, environment variables, working directory, program arguments/inputs, etc.
4. Avoid I/O, including both filesystem, network, and system calls.

Code that cannot satisfy these requirements likely belongs as a helper to be called as part of `main()` (or elsewhere in a program's lifecycle), or be written as part of `main()` itself.
In particular, libraries that are intended to be used by other programs should take special care to be completely deterministic and not perform "init magic".

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Foo struct {
    // ...
}

var _defaultFoo Foo

func init() {
    _defaultFoo = Foo{
        // ...
    }
}
```

</td><td>

```go
var _defaultFoo = Foo{
    // ...
}

// or, better, for testability:

var _defaultFoo = defaultFoo()

func defaultFoo() Foo {
    return Foo{
        // ...
    }
}
```

</td></tr>
<tr><td>

```go
type Config struct {
    // ...
}

var _config Config

func init() {
    // Bad: based on current directory
    cwd, _ := os.Getwd()

    // Bad: I/O
    raw, _ := ioutil.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )

    yaml.Unmarshal(raw, &_config)
}
```

</td><td>

```go
type Config struct {
    // ...
}

func loadConfig() Config {
    cwd, err := os.Getwd()
    // handle err

    raw, err := ioutil.ReadFile(
        path.Join(cwd, "config", "config.yaml"),
    )
    // handle err

    var config Config
    yaml.Unmarshal(raw, &config)

    return config
}
```

</td></tr>
</tbody></table>

Considering the above, some situations in which `init()` may be preferable or necessary might include:

* Complex expressions that cannot be represented as single assignments.
* Pluggable hooks, such as `database/sql` dialects, encoding type registries, etc.

### Exit in Main

Go programs use [os.Exit] or [log.Fatal*] to exit immediately.
(Panicking is not a good way to exit programs, please don't panic.)

[os.Exit]: https://golang.org/pkg/os/#Exit
[log.Fatal*]: https://golang.org/pkg/log/#Fatal

Call one of `os.Exit` or `log.Fatal*` **only in `main()`**.
All other functions should return errors to signal failure.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func main() {
  body := readFile(path)
  fmt.Println(body)
}

func readFile(path string) string {
  file, err := os.Open(path)
  if err != nil {
    log.Fatal(err)
  }

  payload, err := ioutil.ReadAll(file)
  if err != nil {
    log.Fatal(err)
  }

  return string(payload)
}
```

</td><td>

```go
func main() {
  body, err := readFile(path)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(body)
}

func readFile(path string) (string, error) {
  file, err := os.Open(path)
  if err != nil {
    return "", err
  }

  payload, err := ioutil.ReadAll(file)
  if err != nil {
    return "", err
  }

  return string(payload), nil
}
```

</td></tr>
</tbody></table>

Rationale: Programs with multiple functions that exit present a few issues:

* Non-obvious control flow: Any function can exit the program, so it becomes difficult to reason about the control flow.
* Difficult to test: A function that exits the program will also exit the test calling it.
  This makes the function difficult to test and introduces risk of skipping other tests that have not yet been run by `go test`.
* Skipped cleanup: When a function exits the program, it skips function calls enqueued with `defer` statements.
  This adds risk of skipping important cleanup tasks.

#### Exit Once

If possible, prefer to call `os.Exit` or `log.Fatal` **at most once** in your `main()`.
If there are multiple error scenarios that halt program execution, put that logic under a separate function and return errors from it.

This has the effect of shortening your `main()` function and putting all key business logic into a separate, testable function.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
package main

func main() {
  args := os.Args[1:]
  if len(args) != 1 {
    log.Fatal("missing file")
  }
  name := args[0]

  file, err := os.Open(name)
  if err != nil {
    log.Fatal(err)
  }
  defer file.Close()

  // If we call log.Fatal after this line,
  // file.Close will not be called.

  payload, err := ioutil.ReadAll(file)
  if err != nil {
    log.Fatal(err)
  }

  // ...
}
```

</td><td>

```go
package main

func main() {
  err := run()
  if err != nil {
    log.Fatal(err)
  }
}

func run() error {
  args := os.Args[1:]
  if len(args) != 1 {
    return errors.New("missing file")
  }
  name := args[0]

  file, err := os.Open(name)
  if err != nil {
    return err
  }
  defer file.Close()

  payload, err := ioutil.ReadAll(file)
  if err != nil {
    return err
  }

  // ...
}
```

</td></tr>
</tbody></table>

## Performance

Performance-specific guidelines apply only to the hot path(s) of the application.

### Prefer strconv over fmt

When converting primitives to/from strings, `strconv` is faster than `fmt`.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}
```

</td><td>

```go
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}
```

</td></tr>
<tr><td>

```text
BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

</td><td>

```text
BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

</td></tr>
</tbody></table>

### Avoid string-to-byte conversion

Do not create byte slices from a fixed string repeatedly.
Instead, perform the conversion once and capture the result.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}
```

</td><td>

```go
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}
```

</tr>
<tr><td>

```text
BenchmarkBad-4   50000000   22.2 ns/op
```

</td><td>

```text
BenchmarkGood-4  500000000   3.25 ns/op
```

</td></tr>
</tbody></table>

### Prefer Specifying Container Capacity

Specify container capacity where possible in order to allocate memory for the container up front.
This minimizes subsequent allocations (by copying and resizing of the container) as elements are added.

#### Specifying Map Capacity Hints

Where possible, provide capacity hints when initializing maps with `make()`.

```go
make(map[T1]T2, hint)
```

Providing a capacity hint to `make()` tries to right-size the map at initialization time, which reduces the need for growing the map and allocations as elements are added to the map.

Note that, unlike slices, map capacity hints do not guarantee complete, preemptive allocation, but are used to approximate the number of hashmap buckets required.
Consequently, allocations may still occur when adding elements to the map, even up to the specified capacity.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[string]os.FileInfo)

files, _ := ioutil.ReadDir("./files")
for _, f := range files {
    m[f.Name()] = f
}
```

</td><td>

```go

files, _ := ioutil.ReadDir("./files")

m := make(map[string]os.FileInfo, len(files))
for _, f := range files {
    m[f.Name()] = f
}
```

</td></tr>
<tr><td>

`m` is created without a size hint; there may be more allocations at assignment time.

</td><td>

`m` is created with a size hint; there may be fewer allocations at assignment time.

</td></tr>
</tbody></table>

#### Specifying Slice Capacity

Where possible, provide capacity hints when initializing slices with `make()`, particularly when appending.

```go
make([]T, length, capacity)
```

Unlike maps, slice capacity is not a hint: the compiler will allocate enough memory for the capacity of the slice as provided to `make()`, which means that subsequent `append()` operations will incur zero allocations (until the length of the slice matches the capacity, after which any appends will require a resize to hold additional elements).

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td><td>

```go
for n := 0; n < b.N; n++ {
  data := make([]int, 0, size)
  for k := 0; k < size; k++{
    data = append(data, k)
  }
}
```

</td></tr>
<tr><td>

```text
BenchmarkBad-4    100000000    2.48s
```

</td><td>

```text
BenchmarkGood-4   100000000    0.21s
```

</td></tr>
</tbody></table>

## Style

### Import Grouping

In order to make imports orderly and clear, imported packages should be grouped in the following order from top to bottom:

* Standard library
* External dependencies
* External dependencies from our org
* Internal dependencies

Within each import group, imports should be sorted alphabetically.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
    "errors"
    "fmt"
    "github.com/rs/zerolog"
    "github.com/onflow/flow-go/ledger"
    "github.com/onflow/flow-go/ledger/complete/mtrie/trie"
    "github.com/onflow/flow-go/model/flow"
    "github.com/optakt/flow-dps/models/dps"
	"sync"
	"time"
)
```

</td><td>

```go
import (
    "errors"
    "fmt"
    "sync"
    "time"

    "github.com/rs/zerolog"

    "github.com/onflow/flow-go/ledger"
    "github.com/onflow/flow-go/ledger/complete/mtrie/trie"
    "github.com/onflow/flow-go/model/flow"

    "github.com/optakt/flow-dps/models/dps"
)
```

</td></tr>
</tbody></table>

### Package Names

When naming packages, choose a name that is:

* All lower-case. No capitals or underscores.
* Does not need to be renamed using named imports at most call sites.
* Short and succinct. Remember that the name is identified in full at every call site.
* Not plural. For example, `net/url`, not `net/urls`.
* Not "common", "util", "shared", or "lib". These are bad, uninformative names.

See also [Package Names] and [Style guideline for Go packages].

[Package Names]: https://blog.golang.org/package-names
[Style guideline for Go packages]: https://rakyll.org/style-packages/

### Function Names

We follow the Go community's convention of using [MixedCaps for function names].
An exception is made for test functions, which may contain underscores for the purpose of grouping related test cases, e.g., `TestMyFunction_WhatIsBeingTested`.

[MixedCaps for function names]: https://golang.org/doc/effective_go.html#mixed-caps

### Function Grouping and Ordering

* Functions should be sorted in rough call order.
* Functions in a file should be grouped by receiver.

Therefore, exported functions should appear first in a file, after `struct`, `const`, `var` definitions.

A `newXYZ()`/`NewXYZ()` may appear after the type is defined, but before the rest of the methods on the receiver.

Since functions are grouped by receiver, plain utility functions should appear towards the end of the file.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n []int) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

</td><td>

```go
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n []int) int {...}
```

</td></tr>
</tbody></table>

### Reduce Nesting

Code should reduce nesting where possible by handling error cases/special conditions first and returning early or continuing the loop.
Reduce the amount of code that is nested multiple levels.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    err := v.Call()
    if err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

</td><td>

```go
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  err := v.Call()
  if err != nil {
    return err
  }
  v.Send()
}
```

</td></tr>
</tbody></table>

### Unnecessary Else

If a variable is set in both branches of an if, it can be replaced with a single if.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var a int
if b {
  a = 100
} else {
  a = 10
}
```

</td><td>

```go
a := 10
if b {
  a = 100
}
```

</td></tr>
</tbody></table>

### Top-level Variable Declarations

At the top level, use the standard `var` keyword.
Do not specify the type, unless it is not the same type as the expression.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var _s string = F()

func F() string { return "A" }
```

</td><td>

```go
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

</td></tr>
</tbody></table>

Specify the type if the type of the expression does not match the desired type exactly.

```go
type myError struct{}

func (myError) Error() string { return "error" }

func F() myError { return myError{} }

var _e error = F()
// F returns an object of type myError, but we want error.
```

### Local Variable Declarations

Short variable declarations (`:=`) should be used if a variable is being set to some value explicitly.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var s = "foo"
```

</td><td>

```go
s := "foo"
```

</td></tr>
</tbody></table>

However, there are cases where the default value is clearer when the `var`keyword is used.
[Declaring Empty Slices], for example.

[Declaring Empty Slices]: https://github.com/golang/go/wiki/CodeReviewComments#declaring-empty-slices

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td><td>

```go
func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

</td></tr>
</tbody></table>

### Reduce Scope of Variables

Where possible, reduce scope of variables.
Do not reduce the scope if it conflicts with [Reduce Nesting](#reduce-nesting).

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
data, err := ioutil.ReadFile(name)
if err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

</td><td>

```go
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

err = cfg.Decode(data)
if err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

</td></tr>
</tbody></table>

### Avoid Naked Parameters

Naked parameters in function calls can hurt readability.
Add C-style comments(`/* ... */`) for parameter names when their meaning is not obvious.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true, true)
```

</td><td>

```go
// func printInfo(name string, isLocal, done bool)

printInfo("foo", true /* isLocal */, true /* done */)
```

</td></tr>
</tbody></table>

Better yet, replace naked `bool` types with custom types for more readable and type-safe code.
This allows more than just two states (true/false) for that parameter in the future.

```go
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady Status = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

### Initializing Structs

#### Use Field Names to Initialize Structs

You should almost always specify field names when initializing structs.
This is enforced by [go vet].

[go vet]: https://golang.org/cmd/vet/

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
k := User{"John", "Doe", true}
```

</td><td>

```go
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

</td></tr>
</tbody></table>

Exception: Field names *may* be omitted in test tables when there are 3 or fewer fields.

```go
tests := []struct{
  op Operation
  want string
}{
  {Add, "add"},
  {Subtract, "subtract"},
}
```

#### Omit Zero Value Fields in Structs

When initializing structs with field names, omit fields that have zero values unless they provide meaningful context.
Otherwise, let Go set these to zero values automatically.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
  MiddleName: "",
  Admin: false,
}
```

</td><td>

```go
user := User{
  FirstName: "John",
  LastName: "Doe",
}
```

</td></tr>
</tbody></table>

This helps reduce noise for readers by omitting values that are default in that context.
Only meaningful values are specified.

Include zero values where field names provide meaningful context.
For example, test cases in test tables can benefit from names of fields even when they are zero-valued.

```go
tests := []struct{
  give string
  want int
}{
  {give: "0", want: 0},
  // ...
}
```

#### Use `var` for Zero Value Structs

When all the fields of a struct are omitted in a declaration, use the `var` form to declare the struct.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
user := User{}
```

</td><td>

```go
var user User
```

</td></tr>
</tbody></table>

This differentiates zero valued structs from those with non-zero fields similar to the distinction created for [map initialization], and matches how we prefer to [declare empty slices][Declaring Empty Slices].

[map initialization]: #initializing-maps

#### Initializing Struct References

Use `&T{}` instead of `new(T)` when initializing struct references so that it is consistent with the struct initialization.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

</td><td>

```go
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

</td></tr>
</tbody></table>

### Initializing Maps

Prefer `make(..)` for empty maps, and maps populated programmatically.
This makes map initialization visually distinct from declaration, and it makes it easy to add size hints later if available.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = map[T1]T2{}
  m2 map[T1]T2
)
```

</td><td>

```go
var (
  // m1 is safe to read and write;
  // m2 will panic on writes.
  m1 = make(map[T1]T2)
  m2 map[T1]T2
)
```

</td></tr>
<tr><td>

Declaration and initialization are visually similar.

</td><td>

Declaration and initialization are visually distinct.

</td></tr>
</tbody></table>

Where possible, provide capacity hints when initializing maps with `make()`.
See [Specifying Map Capacity Hints](#specifying-map-capacity-hints) for more information.

On the other hand, if the map holds a fixed list of elements, use map literals to initialize the map.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
  k1: v1,
  k2: v2,
  k3: v3,
}
```

</td></tr>
</tbody></table>

The basic rule of thumb is to use map literals when adding a fixed set of elements at initialization time, otherwise use `make` (and specify a size hint if available).

## Patterns

### Functional Options

Functional options is a pattern in which you declare an opaque `Option` type that records information in some internal struct.
You accept a variadic number of these options and act upon the full information recorded by the options on the internal struct.

Use this pattern for optional arguments in constructors and other public APIs that you foresee needing to expand, especially if you already have three or more arguments on those functions.

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// package db

func Open(
  addr string,
  cache bool,
  logger *zap.Logger
) (*Connection, error) {
  // ...
}
```

</td><td>

```go
// package db

type Option interface {
  // ...
}

func WithCache(c bool) Option {
  // ...
}

func WithLogger(log *zap.Logger) Option {
  // ...
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  // ...
}
```

</td></tr>
<tr><td>

The cache and logger parameters must always be provided, even if the user wants to use the default.

```go
db.Open(addr, db.DefaultCache, zap.NewNop())
db.Open(addr, db.DefaultCache, log)
db.Open(addr, false /* cache */, zap.NewNop())
db.Open(addr, false /* cache */, log)
```

</td><td>

Options are provided only if needed.

```go
db.Open(addr)
db.Open(addr, db.WithLogger(log))
db.Open(addr, db.WithCache(false))
db.Open(
  addr,
  db.WithCache(false),
  db.WithLogger(log),
)
```

</td></tr>
</tbody></table>

Our suggested way of implementing this pattern is with an `Option` interface that holds an unexported method, recording options on an unexported `options` struct.

```go
type options struct {
  cache  bool
  logger *zap.Logger
}

type Option interface {
  apply(*options)
}

type cacheOption bool

func (c cacheOption) apply(opts *options) {
  opts.cache = bool(c)
}

func WithCache(c bool) Option {
  return cacheOption(c)
}

type loggerOption struct {
  Log *zap.Logger
}

func (l loggerOption) apply(opts *options) {
  opts.logger = l.Log
}

func WithLogger(log *zap.Logger) Option {
  return loggerOption{Log: log}
}

// Open creates a connection.
func Open(
  addr string,
  opts ...Option,
) (*Connection, error) {
  options := options{
    cache:  defaultCache,
    logger: zap.NewNop(),
  }

  for _, o := range opts {
    o.apply(&options)
  }

  // ...
}
```

Note that there's a method of implementing this pattern with closures, but we believe that the pattern above provides more flexibility for authors and is easier to debug and test for users.
In particular, it allows options to be compared against each other in tests and mocks, versus closures where this is impossible.
Further, it lets options implement other interfaces, including `fmt.Stringer` which allows for user-readable string representations of the options.

See also,

* [Self-referential functions and the design of options]
* [Functional options for friendly APIs]

  [Self-referential functions and the design of options]: https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html
  [Functional options for friendly APIs]: https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis
