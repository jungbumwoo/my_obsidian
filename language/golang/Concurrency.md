
## Goroutines

A _goroutine_ is a lightweight thread managed by the Go runtime.

## Channels

Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`.

By default, sends and receives block until the other side is ready. This allows goroutines to synchronize without explicit locks or condition variables.

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)  // -5, 17, 12 - why alwalys print this result? I thought (17, -5, 12) is possible. 
	// -> for i := 100 fail. for i := 10000 success. I got different result.
}

```


### Buffered Channels

Channels can be _buffered_. Provide the buffer length as the second argument to `make` to initialize a buffered channel


```go
package main

import "fmt"

func countTo(max int) (<-chan int, func()) {
	ch := make(chan int)
	done := make(chan struct{})
	cancel := func() {
		close(done)
	}

	go func() {
		for i := 0; i < max; i++ {
			select {
			case <-done:
				return
			case ch <- i:
			}
		}
		close(ch)
	}()
	return ch, cancel
}

func main() {
	ch, cancel := countTo(10)
	for i := range ch {
		if i > 5 {
			break
		}
		fmt.Println(i)
	}
	cancel() 
}

```

Rather than return the done channel directly, create a closure that closes the done channel and return the closure instead. Cancelliong with a closure allows us to perform additional clean-up with, if needed.