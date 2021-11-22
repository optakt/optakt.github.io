# Resources

Resources are types that can exist only in one memory location at a time.
At the end of a function which has resources in scope, resources must either be **moved** or **destroyed**.

After it was _destroyed_, the resource can no longer be used.
_Moving_ a resource means either assigning it to a different constant or a variable, passing it as an argument to another function, or returning it from a function.
After the resource was moved &#0151 e.g. assigned to a `const`, the previous reference to the resource is invalid and cannot be used anymore.

To make the resource behavior clear and explicit, the prefix `@` must be used in all type annotations dealing with resources, and the resource movement is denoted with the _move_ operator &#0151 `<-`.

```rust
// Resource definition.
pub resource SomeResource {

    // The resource has a single field &#0151 the `value` integer.
    pub var value: Int

    // Define the resource initializer
    init(value: Int) {
        self.value = value
    }
}

// Resource is created using the `create` keyword.
// Note that the const `a` has type of `@SomeResource`.
let a: @SomeResurce <- create SomeResource(value: 2)

// Resource is moved from const a to const b.
// `a` can no longer be used to access the resource.
let b <- a

// Resource is destroyed.
destroy b

// Neither `a` or `b` can now be used to access the resource as it was destroyed.
```

When a resource is returned from a function, the function caller has the responsibility to use the returned resource.

```rust
// This function is invalid as it does not use the resource.
pub fun invalid_use(r: @SomeResource) {
}

// This function uses the resource by moving it to a new const.
// This new const is used by being returned from the function.
pub fun use(res: @SomeResource): @SomeResource {
    let moved <- res
    // Note that the return call still uses the `move` operator.
    return <-moved
}

// Create a new resource.
let a: @SomeResource <- create SomeResource(value: 3)

// Move the function return value to a new const.
// The const `a` can no longer be used to access the resource.
let result <- use(res: <- a)

// Destroy the resource.
destroy result
```

Resource can have a destructor, which is called when the resource is destroyed.

```rust
pub resource SomeResource {

    destroy() {
        // Some logic here, e.g. decrement the variable indicating the number of resources in existence.
    }
}
```

Since the resource variables cannot be assigned to, there are two options to replace the values of resource variables &#0151 the _swap_ operator (`<->`) and the _shift_ operator (`<- target <-`)

```rust
pub resource SomeResource{}

var x <- create SomeResource()
var y <- create SomeResource()

// Swap the resources.
x <-> y

// Alternatively, use the shift operator.
// The shift operator moves the resource from `x` to `oldX`.
// At the same time, `x` receives the value of the new resource.
let oldX <- x <- create SomeResource()

// oldX still needs to be used.
```

Resources have an implicit unique identifier in the form of the predeclared public field `uuid` of type `UInt64`.
This field is incremented on each resource creation, and can never be the same for two resources, even if some of them were destroyed.
