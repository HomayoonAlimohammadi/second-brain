#software_engineering #golang #concurrency 

### Related:
[[Go Concurrency Patterns]]

### References:
https://www.youtube.com/watch?v=QDDwwePbDtw&t=1537s&ab_channel=GoogleforDevelopers

## Naive Implementation
- In the talk he first implemented a naive rss subscriber with the following loop:

```go
for {
	if s.closed {
		close(s.updates)
		return
	}
	items, next, err := s.fetcher.Fetch()
	if err != nil {
		s.err = err
		time.Sleep(10 * time.Second)
		continue
	}
	for _, item := range items {
		s.updates <- item
	}
	if now := time.Now(); next.After(now) {
		time.Sleep(next.Sub(now))
	}
}

func (s *naiveSub) Close() error {
	s.closed = true
	return s.err
}
```

- But it has three bugs:
	- `s.closed` and `s.err` as accessed by two goroutines (data race, partially written data types in case of error (interface), etc. ...)
	- `time.Sleep` may keep the loop running
	- loop may block for ever on `s.updates <- item` -> leaking goroutine

## Solution
Change the body of loop to a select with three cases
* `Close` was called
* it's time to call `Fetch`
* send an item on `s.updates`
## service channel, reply channels
- We can see goroutines (or processes in different goroutines) as little microservices in our own distributed environment and they can send request and responses via this structure:
```go
chan chan error // or chan chan any
```

- The server sends this channel to its client:
```go
s.closing := make(chan chan error)
errc = make(chan error)
s.closing <- errc
return <-errc
```

the client receives the close signal and sends its response through its channel
```go
for {
	select {
	case errc := <-s.closing:
		errc <- err // sends the response to its caller
		close(s.updates) // tells recievers we are done
		return
	}
}
```

## Nil channels block
- In the implementation we see something that might trigger a bug:
```go
for {
	select {
	case s.updates <- pending[0]:
		pending = pending[1:]
	}
}
```

`select` only checks if the communication can happen (e.g. send or receive on a channel) and not the other logic in the statement, so the `pending[0]` might panic when `len(pending) == 0` 

In order to fix this, when `len(pending) == 0`, we make the `s.updates` channel nil, so that the write will be blocking (read and write on nil channels block)

```go
var pending []Item
for {
	var first Item
	var updates chan Item // default value is nil
	if len(pending) > 0 {
		first = pending[0]
		updates = s.updates // enable send case, e.g. makes non-nil
	}

	select {
	case updates <- first:
		pending = pending[1:]
	}
}
```

Also channels can be nil-ed in order to block reading from them as well. let's say that a request is going to take ~5 seconds and we want to stay responsive and not block while the request is being made. so we create a response channel and try to send the response of the request to that channel upon arrival which is happening in another goroutine.
```go
type fetchResult struct{ fetched []Item; next time.Time; err error }
...
var fetchDone chan fetchResult // nil initially
...
var startFetch <-chan time.Time
if fetchDone == nil {
	startFetch = time.After(fetchDelay) // enable fetch
}
...
select {
case <-startFetch:
	// buffered so the first write is non-blocking
	fetchDone = make(chan fetchResult, 1) 
	go func() {
		fetched, next, err := s.fetcher.Fetch()
		fetchDone <- fetchResult{fetched, next, err}
	}()
case result := <-fetchDone:
	fetchDone = nil // will not try to read anything until the next start
	// use result...
}
```
