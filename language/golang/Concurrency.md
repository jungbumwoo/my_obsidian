

```golang

```

Rather than return the done channel directly, create a closure that closes the done channel and return the closure instead. Cancelliong with a closure allows us to perform additional clean-up with, if needed.