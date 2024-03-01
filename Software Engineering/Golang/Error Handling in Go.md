#golang #error

### References
https://go.dev/blog/error-handling-and-go

### Error Handling and Go

This blog post by Andrew Gerrand on 12 July 2011 dives into Go's approach to error handling, focusing on the built-in `error` type and best practices for managing errors in Go programs. Here's a summary with key points and examples:

#### Understanding the Error Type
- **Basic Usage**: In Go, functions often return an `error` type as a second return value to indicate abnormal states or errors. The `error` type is an interface that can describe itself as a string.
  ```go
  func Open(name string) (file *File, err error)
  ```
- **Error Checking**: Checking the returned `error` value is a common Go idiom for error handling.
  ```go
  f, err := os.Open("filename.ext")
  if err != nil {
      log.Fatal(err)
  }
  ```

#### The Error Interface
- **Interface Definition**: The `error` interface requires an `Error()` method that returns a string.
  ```go
  type error interface {
      Error() string
  }
  ```
- **Common Implementation**: The `errors` package provides a basic implementation of the `error` interface with the `errors.New` function, returning an `error` value.
  ```go
  err := errors.New("math: square root of negative number")
  ```

#### Custom Error Types
- **Defining Custom Errors**: You can define custom error types that implement the `error` interface to provide more detailed error information.
  ```go
  type NegativeSqrtError float64

  func (f NegativeSqrtError) Error() string {
      return fmt.Sprintf("math: square root of negative number %g", float64(f))
  }
  ```
- **Using `fmt.Errorf` for Formatting Errors**: The `fmt.Errorf` function allows for formatting error messages with additional context.
  ```go
  if f < 0 {
      return 0, fmt.Errorf("math: square root of negative number %g", f)
  }
  ```

#### Handling Errors
- **Simplifying Repetitive Error Handling**: To reduce repetitive error handling, you can define types or use techniques that abstract away common error handling patterns.
- **Example with HTTP Handlers**: An `appHandler` type is defined to simplify HTTP error handling by including an `error` return value.
  ```go
  type appHandler func(http.ResponseWriter, *http.Request) error
  ```
- **Improving User-Friendly Error Responses**: By defining an `appError` struct, you can provide user-friendly error messages and HTTP status codes while logging detailed error information for debugging.
  ```go
  type appError struct {
      Error   error
      Message string
      Code    int
  }

  func viewRecord(w http.ResponseWriter, r *http.Request) *appError {
      // handle errors and return appError with specific message and code
  }
  ```

#### Conclusion
Effective error handling is crucial for writing reliable and maintainable Go code. This post emphasizes the importance of understanding and utilizing Go's `error` interface, implementing custom error types for detailed error information, and employing strategies to simplify repetitive error handling patterns.

For further exploration of error handling in Go, including more advanced techniques and patterns, refer to the original blog post on the [Go blog](https://go.dev/blog/error-handling-and-go).