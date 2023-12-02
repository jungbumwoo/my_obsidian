
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

Channels can be _buffered_. Provide the buffer length as the second argument to `make` to initialize a buffered channel.

Sends to a buffered channel block only when the buffer is full.

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2)
	ch <- 1
	ch <- 2
	ch <- 3  // fatal error: all goroutines are asleep - deadlock!
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}

```


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

### Range and Close

```go
v, ok := <-ch // `ok` is `false` if there are no more values to receive and the channel is closed.
```

A sender can `close` a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receive expression.

The loop `for i := range c` receives values from the channel repeatedly until it is closed.

**Note:** Only the sender should close a channel, never the receiver. Sending on a closed channel will cause a panic.

**Another note:** Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a `range` loop.

### Select

The `select` statement lets a goroutine wait on multiple communication operations.

A `select` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

### Default Selection

The `default` case in a `select` is run if no other case is ready.
Use a `default` case to try a send or receive without blocking.

