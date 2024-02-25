#golang #concurrency #data_race 

### References:
https://dave.cheney.net/2014/06/27/ice-cream-makers-and-data-races
https://research.swtch.com/interfaces
https://www.airs.com/blog/archives/277
https://research.swtch.com/gorace

```go
package main

import "fmt"

type IceCreamMaker interface {
        // Hello greets a customer
        Hello()
}

type Ben struct {
        name string
}

func (b *Ben) Hello() {
        fmt.Printf("Ben says, \"Hello my name is %s\"\n", b.name)
}

type Jerry struct {
        name string
}

func (j *Jerry) Hello() {
        fmt.Printf("Jerry says, \"Hello my name is %s\"\n", j.name)
}

func main() {
        var ben = &Ben{"Ben"}
        var jerry = &Jerry{"Jerry"}
        var maker IceCreamMaker = ben

        var loop0, loop1 func()

        loop0 = func() {
                maker = ben
                go loop1()
        }

        loop1 = func() {
                maker = jerry
                go loop0()
        }

        go loop0()

        for {
                maker.Hello()
        }
}
```

![[Screenshot 2024-02-23 at 8.50.40 in the evening.png]]
![[Screenshot 2024-02-23 at 8.50.50 in the evening.png]]![[Screenshot 2024-02-23 at 8.51.09 in the evening.png]]

This way we might accidentally call `Jerry`‘s `Hello()` function with `ben` as the receiver.