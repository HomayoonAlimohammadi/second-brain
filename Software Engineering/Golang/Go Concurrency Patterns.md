#software_engineering #golang #concurrency

### References:
https://www.youtube.com/watch?v=f6kdp27TYZs&ab_channel=GoogleforDevelopers


## Generator
- a function that returns a channel

![[Screenshot 2024-02-14 at 6.27.02 in the evening.png]]

## Fan-In

![[Screenshot 2024-02-14 at 6.38.22 in the evening 1.png]]

## Wait Channels
- This way we preserve the sequencing of the responses. but it's somehow similar to a similar case (multiple simple generators)

![[Screenshot 2024-02-14 at 6.40.20 in the evening.png]]

- Here the `Message` struct is like this:
```
type Message struct {
	str string
	wait chan bool
}
```

## Select
-  blocks until one of them ready to run (if multiple cases are ready it selects on pseudo-randomly)
- default executes immediately if no other case is ready
- if priority is important (ordering of the cases) we should use nested select cases like this:
```go
select {
case <-mostImportant:
	doSomething()
default:
	select {
	case <-lessImportant:
		doSomething()
	default:
		select {
		case <-notImportant:
			doSomething()
		default:
			fmt.Println("nothing happened")
		} 
	}
}
```

- We can do fan-in via select:
```go
func fanIn(input1, input2 <-chan string) <-chan string {
	c := make(chan string)
	go func() {
		for {
			select {
			case s := <-input1: c <- s
			case s := <-input2: c <- s
			}
		}
	}()

	return c
}
```


## Timeout using select

- We can either time out for each receive or the whole conversation (with a channel) or both

```go
func main() {
	c := generator() // returns a <-chan string
	convTimeout := time.After(5 * time.Second)
	for {
		select {
		case s := <- c: fmt.Printf("generator said %s\n", s)
		case <-time.After(1 * time.Second): // single receive timeout
			fmt.Println("too late")
			return
		case <-convTimeout: // conversation timeout
			fmt.Println("time is up!")
			return
		}
	}
}
```


## Quit channel
- someway of stopping a running goroutine. first we see how to stop it without any precautions on any cleanup or other final tasks that might have, then we see how we can make sure it's gracefully stopped
```go
func main() {
	quit := make(chan struct{})
	c := generator(quit) 
	timeout := time.After(5 * time.Second)
	for {
		select {
		case s := <-c: fmt.Println("got message from c:", s)
		case <-timeout:
			quit <- struct{}{} // this signals the stop to the goroutine
			// but the break and the eventual termination of the program
			// might prevent some cleanup that the generator needs
			break
		}
	}
}

func generator(quit chan struct{}) <-chan string {
	c := make(chan string)
	go func() {
		for {
			select {
				case c <- "some message": // do nothing
				case <-quit:
					// what if it needs to do some cleanup?
					return
			}
		}
	}()
}
```

- Now let's make the cleanup possible (but I think there is a race in reading from "quit" channel in the main function and in the generator function. as soon as the main function sends a signal on quit, isn't it possible for the main function to read its own signal and interpret it as generator's? In other words the generator might never receive the signal and quit). SHIIIT! this won't happen because this scenario will cause a deadlock (which is not our case). the send only succeeds if a receiver is ready to receive.
```go
func main() {
	quit := make(chan struct{})
	c := generator(quit) 
	timeout := time.After(5 * time.Second)
	for {
		select {
		case s := <-c: fmt.Println("got message from c:", s)
		case <-timeout:
			quit <- struct{}{}
			<-quit
			break
		}
	}
}

func generator(quit chan struct{}) <-chan string {
	c := make(chan string)
	go func() {
		for {
			select {
				case c <- "some message": // do nothing
				case <-quit:
					cleanup()
					quit <- struct{}{}
					return
			}
			}
	}()
}
```

## Daisy-Chain
- The idea is that we chain channels together, meaning that we read from one in order to write into another one
```go
func pass(left, right chan int) {
	right <- 1 + <-left // read from left and write to right
}

func main() {
	n := 100000 // number of links (~goroutines and channels)
	rightmost := make(chan int)
	right := rightmost
	left := rightmost
	for i := 0; i < n; i++ {
		left = make(chan int)
		go pass(left, right)
		right = left
	}

	go func(c chan int) {c <- 1}(left)
	fmt.Println(<-rightmost)
}
```

## Replicating server calls
- probably the most important point of this talk at least for me is to use a mechanism to replicate calls to a server and use the first one in order to reduce tail latency
```go
func First(query string, replicas ...Search) Result {
	c := make(chan Result)
	searchReplica := func(i int) {c <- replicas[i](query)}
	for i := range replicas {
		searchReplica(i)
	}
	return <-c // return the first fetched result from the search replicas
	// notice that it's better to also implement a mechanism to cancel ongoing
	// requests when we returned the response (prevent unnecessary processing)
}
```

- an implementation of this "First" function in Rob's presentation is like this:
```go
c := make(func Result)
go func() {c <- First(query, Web1, Web2)} ()
go func() {c <- First(query, Image1, Image2)} ()
go func() {c <- First(query, Video1, Video2)} ()

timeout := time.After(80 * time.Millisecond)
var results [3]Result
for i := 0; i < 3; i++ {
	select {
	case <-timeout:
		fmt.Println("timed out")
		return
	case result := <- c:
		results[i] = result 
	}
}

fmt.Println(results)
```

## Bonus: Concurrent Prime Sieve (golang)
- https://gist.github.com/lyricat/3359265

