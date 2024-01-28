
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

// Compiler error:
// r.String undefined
	// (type []int32 has no field or method String)

```

```go
// 들어오는 S를 기반으로 E가 추론 됨
func Scale[S ~[]E, E constraints.Integer](s S, sc E) S {
	r := make(S, len(s))
	for i, v := range s {
		r[i] = v * c
	}
	return r
}

func ScaleAndPrint(p Point) {
	r := Scale(p, 2)
	fmt.Println(r.String())
}

// Why don't we have to write 
// r := Scale[Point, int32](p, 2)
// ?
```

Constraint Type Inference - Deduce type arguments from type parameter constraints

## When to use generics

### When are type parameters useful?

- Functions that work on slices, maps, and channels of any element type.
	- if a function has parameters with those types and function code doesn't make any particular assumptions about the element types then it may be useful to use a type parameter.
	- alternative to using type parameters for  this kind of function is typically to use reflection. but that's more awkward programming model. it not statically type checked, it's often slower.
- General Purpose data structure.
	- replacing an interface type with type parameter can often permit the data to be stored more efficiently.
	- can avoid type assertions and can instead be fully type checked at complie time.
	- When operating on type parameters, prefer functions to methods. (for general purpose data types, use fuctions rather than writing constraint that require method )
- When a method looks the same for all types.
	
```go
// example of (When a method looks the same for all types.)

type SliceFn[T any] struct {
	s []T
	cmp func(T, T) bool
}

func (s SliceFn[T]) Len() int { return len(s.s)}
func (s SliceFn[T]) Swap(i, j int) {
	s.s[i], s.s[j] = s.s[j], s.s[i]
}
func (s SliceFn[T]) Less(i, j int) bool {
	return s.cmp(s.s[i], s.s[j])
}
```

---

https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md

https://www.youtube.com/watch?v=Pa_e9EeCdy8
