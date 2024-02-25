#golang #concurrency #software_engineering 

### Related
[[Go Concurrency Patterns]]
[[Advanced Go Concurrency Patterns]]
### References
https://go.dev/blog/pipelines

This blog by Sameer Ajmani on 13 March 2014, discusses the use of Go's concurrency primitives to construct efficient streaming data pipelines and manage cancellations. Here's a summary with important points and code snippets:

#### Introduction
- Go's concurrency primitives facilitate building streaming data pipelines that efficiently use I/O and CPUs.
- The blog presents examples of pipelines, highlights the challenges with operation failures, and introduces techniques for clean failure handling.

#### What is a Pipeline?
- A pipeline in Go is a series of stages connected by channels, where each stage is a group of goroutines running the same function.
- Stages receive values from upstream via inbound channels, perform functions on that data, and send values downstream via outbound channels.

#### Example Pipeline: Squaring Numbers
- **First Stage (`gen` function)**: Converts a list of integers into a channel that emits these integers.
  ```go
  func gen(nums ...int) <-chan int {
      out := make(chan int)
      go func() {
          for _, n := range nums {
              out <- n
          }
          close(out)
      }()
      return out
  }
  ```
- **Second Stage (`sq` function)**: Receives integers from a channel and emits the square of each received integer.
  ```go
  func sq(in <-chan int) <-chan int {
      out := make(chan int)
      go func() {
          for n := range in {
              out <- n * n
          }
          close(out)
      }()
      return out
  }
  ```
- Demonstrates composition by chaining `sq` functions.

#### Fan-out, Fan-in
- **Fan-out** allows multiple functions to read from the same channel until it's closed, distributing work among multiple workers.
- **Fan-in** involves reading from multiple inputs and proceeding until all are closed, achieved by multiplexing input channels onto a single channel.

#### Handling Early Stage Termination
- Stages should close their outbound channels when all send operations are done and keep receiving values from inbound channels until those channels are closed.
- Discusses the importance of preventing goroutines from blocking indefinitely when earlier stages produce values that later stages do not need.

#### Explicit Cancellation
- Explains how to explicitly cancel pipeline stages by using a `done` channel. Sending a signal over this channel allows upstream stages to abandon values they're attempting to send, preventing blocked sends.
- Demonstrates the use of `select` statements in goroutines for listening on the `done` channel alongside normal operations.

#### Real-world Pipeline Example: File Digests
- Describes a more complex pipeline for calculating MD5 sums of files in a directory, highlighting techniques for parallelization and bounded parallelism to manage resources effectively.

This blog provides a comprehensive guide to building robust and efficient data pipelines in Go, emphasizing the importance of proper handling of failures and cancellations to ensure resource efficiency and program correctness.