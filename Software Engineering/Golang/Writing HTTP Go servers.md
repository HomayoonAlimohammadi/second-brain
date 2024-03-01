#golang #http #server
### References:
https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/

### How I Write HTTP Services in Go After 13 Years

In this insightful blog post on Grafana Labs, the author revisits their approach to writing HTTP services in Go, reflecting on 13 years of evolution in their practices. Here are the key points and technical highlights from the article, including any relevant code snippets:

#### Main Points:
- **Evolution of Practices**: The author discusses how their approach to writing HTTP services in Go has evolved over 13 years, influenced by community feedback and personal experience.
- **Structuring for Maintainability**: Emphasizes structuring servers and handlers in a way that maximizes maintainability, moving away from methods on server structs to more explicit dependency injection.
- **Startup and Shutdown Optimization**: Shares tips for optimizing the startup and graceful shutdown of services, ensuring a quick response and reliability.
- **Common Request Handling**: Covers strategies for handling common functionalities across different request types, focusing on reusability and efficiency.
- **Testing Depth**: Advocates for thorough testing of services, including unit tests, integration tests, and end-to-end tests to ensure robustness and reliability.

#### Key Changes and Insights:
- Shifted from using methods on server structs for handlers to preferring explicit dependency injection for clearer, more testable code.
- Transitioned from a preference for `http.HandlerFunc` to using `http.Handler` as the primary interface, aligning with third-party libraries and enhancing flexibility.
- Expanded focus on testing, emphasizing the importance of a comprehensive testing strategy that covers all aspects of the service.

#### Code Snippets:
- **NewServer Constructor**:
  ```go
  func NewServer(logger *Logger, config *Config, commentStore *commentStore, anotherStore *anotherStore) http.Handler {
      mux := http.NewServeMux()
      // Add routes
      var handler http.Handler = mux
      // Apply middleware
      return handler
  }
  ```
  The `NewServer` function demonstrates the creation of a new server instance, taking dependencies explicitly and setting up routes and middleware.

- **Dependency Injection for Handlers**:
  Handlers now take dependencies explicitly, improving testability and clarity. No specific code snippet provided, but the change emphasizes function arguments for dependencies over struct fields.

- **Middleware Pattern**:
  ```go
  func adminOnly(h http.Handler) http.Handler {
      return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
          // Check if user is admin
          h.ServeHTTP(w, r)
      })
  }
  ```
  This middleware function wraps another `http.Handler` to perform pre-processing, such as authorization checks, before invoking the original handler.

#### Helpers
```go
func encode[T any](w http.ResponseWriter, r *http.Request, status int, v T) error {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(status)
	if err := json.NewEncoder(w).Encode(v); err != nil {
		return fmt.Errorf("encode json: %w", err)
	}
	return nil
}

func decode[T any](r *http.Request) (T, error) {
	var v T
	if err := json.NewDecoder(r.Body).Decode(&v); err != nil {
		return v, fmt.Errorf("decode json: %w", err)
	}
	return v, nil
}
```

```go
// Validator is an object that can be validated.
type Validator interface {
	// Valid checks the object and returns any
	// problems. If len(problems) == 0 then
	// the object is valid.
	Valid(ctx context.Context) (problems map[string]string)
}
```

```go
func decodeValid[T Validator](r *http.Request) (T, map[string]string, error) {
	var v T
	if err := json.NewDecoder(r.Body).Decode(&v); err != nil {
		return v, nil, fmt.Errorf("decode json: %w", err)
	}
	if problems := v.Valid(r.Context()); len(problems) > 0 {
		return v, problems, fmt.Errorf("invalid %T: %d problems", v, len(problems))
	}
	return v, nil, nil
}

decoded, err := decode[CreateSomethingRequest](r)
```

#### Conclusion:
The blog post provides a comprehensive overview of the author's matured approach to building HTTP services in Go, highlighting significant shifts in practices towards better structure, testing, and maintainability. These insights offer valuable guidance for both new and experienced Go developers looking to refine their approach to service development.

For more details, including additional insights and code examples, visit the full blog post on [Grafana Labs](https://grafana.com/blog/2024/02/09/how-i-write-http-services-in-go-after-13-years/).