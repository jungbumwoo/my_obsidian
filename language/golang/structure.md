

## Prefer inversion-of-control dependency injection

Avoid dependency injection frameworks or containers

- Bad case

```go

func BuildContainer() *dig.Container {
	container := dig.New()
	container.Provide(NewConfig)
	container.Provide(ConnectDatabase)
	container.Provide(NewPersonRepository)
	container.Provide(NewPersonService)
	container.Provide(NewServer)
	return container
}

func main() {
	container := buildContainer()
	if err := container.Invoke(func(server *Server) {
		server.Run()
	}); err != nil {
		panic(err)
	}
}

// the func main down at the bottom is compact and the build container function has the appearance of beging pithy and to the point every line does one specific thing but the provide method we see here requires a reflection to interpret its arguments and invoke gives us no clues as to what a server actually needs to do its job it's all kind of hidden away. 



```