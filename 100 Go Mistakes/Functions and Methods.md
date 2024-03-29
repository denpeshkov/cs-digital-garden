---
tags: Go
---

# Receiver Type

A receiver should be a value:

- If the receiver is a map, function or channel, because these types are already pointers
- If the receiver is a slice that is not resliced/reallocated
- If the receiver is a small array or struct that is naturally a value type without mutable fields, e.g. `time.Time`. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap
- If we have to enforce a receiver's immutability
- If the receiver is a basic type

A receiver should be a pointer:

- If the method needs to mutate the receiver
- If the receiver contains a field that cannot be copied, e.g. `sync.Mutex`. Note that this field can be declared in the nested object, rather than directly in the receiver
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, as it will make the intention clearer to the reader
- If the receiver is a large object

In short, we should almost always use pointer receivers unless we have compelling reasons not to

If some of the methods of the type must have pointer receivers, the rest should too, so the method set is consistent regardless of how the type is used

# Defer Arguments Evaluation

Each time a `defer` statement executes, the function value and parameters to the call are **evaluated right-away** and saved anew but the actual function is not invoked

Deferred functions are invoked immediately before the surrounding function returns, in the reverse order they were deferred. That is, deferred functions are executed **after** any result parameters are set by that return statement but **before** the function returns to its caller

If a deferred function value evaluates to `nil`, execution panics when the function is invoked, not when the "defer" statement is executed

If the deferred function has any return values, they are discarded when the function completes

If we want to evaluate arguments to `defer` **during** deferred function execution we call a closure as a `defer` statement. The arguments passed to a `defer` function are still evaluated right away. But the variables referenced by a `defer` closure are evaluated during the closure execution

For example:

```go
var v int
defer func() {
	f(v) // initial deffered function call
}()
```

Here, we wrap the call to both `f` within a closure. This closure references the `v` from outside its body. Therefore, `v` is evaluated once the closure is executed, not when we call `defer`

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Go Code Review Comments - The Go Programming Language](https://go.dev/wiki/CodeReviewComments)
- [receivers | Dave Cheney](https://dave.cheney.net/tag/receivers)
- [Frequently Asked Questions (FAQ) - The Go Programming Language](https://go.dev/doc/faq)
- [Why not a mixture of value and pointer receiver?](https://groups.google.com/g/golang-nuts/c/xOsuXPe1IUo)
- [The Go Programming Language Specification - The Go Programming Language](https://tip.golang.org/ref/spec)
