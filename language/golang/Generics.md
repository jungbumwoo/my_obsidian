
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

#### The types of type parameters

```go
func min[T constraints.Ordered](x, y T) T
```

Type parameter lists also have a type for each type parameter.
This type defines a set of types. It is called the type constraint.


## Type constraints are interfaces

Interface define method sets. but also, Interface define type sets
 
#### ~ Token
```go
package constraints

type Ordered interface {
	Integer | Float | ~string
}
```

`~` is a new token added to Go.
`~T` means the set of all types with underlying type T.

#### Constraint literals
```go
// 아래 셋 다 같음.
[S interface{~[]E}, E interface{}]
[S ~[]E, E interface{}]
[S ~[]E, E any]
```

## Type inference

#### calling with type inference
```go
func min[T constraints.Ordered](x, y T) T

var a, b, m float64

m = min[float64](a, b)

m = min(a, b) // complier가 타입추론으로 바로 위에꺼를 아래와 같이 general func 같이 사용되게 해줌. looks like ordinary func.
```


## How to use

#### Scale a slice of any integer type
```go
// This implementation has a problem
func Scale[E constraints.Integer](s []E, c E) []E {
	r := make([]E, len(s))
	for i, v := range s {
		r[i] = v * c
	}
	return r
}

type Point []int32

func (p Point) String() string {
	// ...
}

func ScaleAndPrint(p Point) {
	r := Scale(p, 2)
	fmt.Println(r.String()) // Does Not COMPILE!
}

```



---

https://www.youtube.com/watch?v=Pa_e9EeCdy8