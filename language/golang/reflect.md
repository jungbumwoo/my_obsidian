
Go is statically typed. Every variable has a static type, that is, exactly one type known and fixed at compile time.

One important category of type is interface types, which represent fixed sets of methods. (When discussing reflection, we can ignore the use of interface definitions as constraints within polymorphic code.) An interface variable can store any concrete (non-interface) value as long as that value implements the interface’s methods.

Some people say that Go’s interfaces are dynamically typed, but that is misleading. They are statically typed: a variable of interface type always has the same static type, and even though at run time the value stored in the interface variable may change type, that value will always satisfy the interface.

https://go.dev/blog/laws-of-reflection