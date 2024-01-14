
### Generator

Generator: function that returns a channel
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

// Generator: function that returns a channel
func boring(msg string) <-chan string { // Returns received-only channel of strings
	c := make(chan string)
	go func() {
		for i := 0; ; i++ {
			c <- fmt.Sprintf("%s %d", msg, i) // Expression to be sent can be any suitable value.
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		}
	}()
	return c // Return the channel to the caller.
}

func main() {
	joe := boring("I have a sore sore throat")
	ann := boring("Ann")
	for i := 0; i < 5; i++ {
		fmt.Println(<-joe)
		fmt.Println(<-ann)
	}
	fmt.Println("You're boring; I'm leaving.")
}

/*
I have a sore sore throat 0
Ann 0
I have a sore sore throat 1
Ann 1
I have a sore sore throat 2
Ann 2
I have a sore sore throat 3
Ann 3
I have a sore sore throat 4
Ann 4
You're boring; I'm leaving.
*/

```

### Fan - In
![[Pasted image 20231214221556.png]]

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

/*
Multiplexing. the Fan-in pattern.

At generator.go, Joe and Ann count in lockstep.
We can instead use a fan-in function to let whosoever is ready talk.
*/
func fanIn(input1, input2 <-chan string) <-chan string {
	c := make(chan string)
	go func() {
		for {
			c <- <-input1
		}
	}()
	go func() {
		for {
			c <- <-input2
		}
	}()
	return c
}

func boring(msg string) <-chan string { // Returns received-only channel of strings
	c := make(chan string)
	go func() {
		for i := 0; ; i++ {
			c <- fmt.Sprintf("%s %d", msg, i) // Expression to be sent can be any suitable value.
			time.Sleep(time.Duration(rand.Intn(1e3)) * time.Millisecond)
		}
	}()
	return c // Return the channel to the caller.
}

func main() {
	joe := boring("I have a sore sore throat")
	ann := boring("Ann")

	c := fanIn(joe, ann)
	for i := 0; i < 10; i++ {
		fmt.Println(<-c)
	}
	fmt.Println("You're boring; I'm leaving.")
}
```

### Restore sqeuece
외부에서 순서를 저렇게 조절하는게 인상적임.
custom type 으로 messge 주고 받고 순서를 조절함




###  Daisy Chain
```go
package main
import "fmt"

// Daisy chain
// 채널로 기차놀이하는 것 같네
func f(left, right chan int) {
	left <- 1 + <-right
}

func main() {
	const n = 10 // 100000
	leftmost := make(chan int)
	right := leftmost
	left := leftmost
	
	for i := 0; i < n; i++ {
		right = make(chan int)
		go f(left, right)
		left = right
}

	go func(c chan int) { c <- 1 }(right)
	fmt.Println(<-leftmost)
}
```


### Search server example
```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

var (
	Web   = fakeSearch("web")
	Image = fakeSearch("image")
	Video = fakeSearch("video")
)

type Result struct{ string }

// 1.0
func Google(query string) (results []Result) {
	results = append(results, Web(query))
	results = append(results, Image(query))
	results = append(results, Video(query))
	return results
}

// 2.0 
func Google(query string) (results []Result) {
	c := make(chan Result)
	go func() { c <- Web(query) }()
	go func() { c <- Image(query) }()
	go func() { c <- Video(query) }()
	for i := 0; i < 3; i++ {
		result = append(results, <-c)
	}
	return results
}

// 2.1 Don't wait for slow servers. No condition variables. No callbacks.
func Google(query string) (results []Result) {
	c := make(chan Result)
	go func() { c <- Web(query) }()
	go func() { c <- Image(query) }()
	go func() { c <- Video(query) }()
	timeout := time.After(80 * time.Millisecond)
	for i := 0; i < 3; i++ {
		select {
		case result = append(results, <-c):
		case <-timeout:
			fmt.Println("timed out")
			return
		}
	}
	return results
}

type Search func(query string) Result

func fakeSearch(query string) Search {
	return func(query string) Result {
		time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
		return Result(fmt.Sprintf("%s result for %q\n", kind, query))
	}
	}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	results := Google("goglang")
	elapsed := time.Since(start)
	fmt.Println(results)
	fmt.Println(elapsed)
}
```

### Search Server Example 2
```go

/*
Q. How do we avoid discarding results from slow servers?
A. Replicate the servers. Send requests to multiple replicas, and use the first response.
*/

func First(query string, replicas ...Search) Result {
	c := make(chan Result)
	searchReplica := func(i int) { c <- replicas[i](query) }
	for i := range replicas {
		go searchReplica(i)
	}
	return <-c
}

func main() {
	rand.Seed(time.Now().UnixNano())
	start := time.Now()
	result := First("golang",
		fakeSearch("replica 1"),
		fakeSearch("replica 2"))
	elapsed := time.Since(start)
	fmt.Println(result)
	fmt.Println(elapsed)
}

```


### Google Search 3.0
```go
c := make(chan Result)
go func() { c <- First(query, Web1, Web2) } ()
go func() { c <- First(query, Image1, Image2) } ()
go func() { c <- First(query, Vedio1, Video2) } ()

timeout := time.After(80 * time.Millisecond)
for i := 0; i < 3; i ++ {
	select {
	case result := <- c:
		results = append(results, result)
	case <- timeout:
		fmt.Println("timed out")
		return
	}
}
```


---

## Google I/O 2013 - Advanced Go Concurrency Patterns

[](https://www.youtube.com/@GoogleDevelopers)

