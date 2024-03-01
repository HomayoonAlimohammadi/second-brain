#golang #context #software_engineering 

### References
https://zenhorace.dev/blog/context-control-go/

### Context Control in Go

This blog post emphasizes the best practices for handling context ("context plumbing") in Go programming. It outlines rules for creating, passing, and managing contexts across API boundaries, focusing on control-flow operations and request-scoped data management.

#### Key Points:
- **Purpose of Context**: Serves two main roles in Go:
  1. Providing a control-flow mechanism across API boundaries with signals.
  2. Carrying request-scoped data across API boundaries.
  
- **Rules for Handling Context**:
  1. **Context Creation**: Only entry-point functions should create a new context (`context.Background()`). Mid-chain functions may create child contexts for sharing data or flow control but should not initiate new base contexts.
  2. **Passing Contexts Down**: Contexts are passed down the call chain. If you need to call a function that accepts a context and you're not at the top of the call chain, your function should also accept a context and pass it along. Use `context.TODO()` as a placeholder if the context isn't available yet, indicating that further integration is needed.
  3. **Avoid Storing Contexts**: Do not store contexts inside structs or use them after the function returns. Doing so breaks the lifecycle expectations of contexts and can lead to obscure bugs.

#### Practical Example:
- The author shares a learning experience with context management in a long-running routine scenario. Initially, a worker struct was designed to run tasks until explicitly stopped, with context managed in a way that inadvertently broke context lifecycle rules.

#### Revised Approach:
- After feedback, the author shifted to a more straightforward, synchronous approach that respects Go's best practices for context management. The final solution relies solely on the context passed to control the lifecycle of the worker routine, avoiding the anti-pattern of managing goroutines within libraries.

```go
type worker struct {
  // internal details. no stop channel.
}

// Start configures and runs the worker.
// Blocks until context cancellation.
func Start(ctx context.Context, ...){
  // Configure setup. Details elided.
  w := worker{...}
  // blocking call to run
  w.run(ctx context.Context)
}

func (w *worker) run(ctx context.Context) {
  ticker := time.NewTicker(time.Minute)
  defer ticker.Stop()
  for {
    select {
    case <- ctx.Done:
      // perform cleanup
    case <-ticker.C:
      // do work
    }
  }
}
```

#### Final Rule Emphasized:
- **Don’t Store Contexts**: Contexts should be used only for the duration of a function call and not after it returns. This ensures that cancellations are managed correctly and avoids potential errors from prematurely canceled operations.

> When a function takes a context parameter, that context should only be used for the duration of the call, not after it returns.
#### Conclusion:
The post concludes with a stronger understanding of context handling in Go, stressing the importance of adhering to Go's context management conventions to write clean, reliable, and maintainable code.

**References and Further Reading**:
- The blog offers additional resources for deepening the understanding of context usage in Go, including Go's official blog and style guides.

For more details on context control in Go, visit [Horace's Blog](https://zenhorace.dev/blog/context-control-go/).