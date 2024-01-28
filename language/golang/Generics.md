
### Instantiation
1. Substitute type arguments for type parameters.
2. Check that type arguments implement their constraints.

 #### Instantiating generic min
```go
fmin := min[float64]

m := fmin(2.71, 3.14)


// min[float64](2.71, 3.14) is evaluated as (min[float64])(2.71, 3.14)
// Instantiatioin produces a non-generic function.
```