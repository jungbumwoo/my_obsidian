

## Prefer inversion-of-control dependency injection

Avoid dependency injection frameworks or containers

- Bad case

```go

func BuildContainer() *dig.Container {
	container := dig.New()
	container.Provide(New)
}


```