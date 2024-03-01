#golang #http #server

### References
https://go.dev/blog/routing-enhancements

### Routing Enhancements for Go 1.22

This blog post, written by Jonathan Amsterdam on behalf of the Go team and published on 13 February 2024, introduces two significant enhancements to the `net/http` package’s router in Go 1.22: method matching and wildcards. These features aim to simplify expressing common routing patterns directly in the pattern string, enhancing the usability and functionality of Go's built-in HTTP router. Here are the key points and code examples from the blog:

#### Key Enhancements:
- **Method Matching and Wildcards**: Allows for more expressive route patterns, reducing the need for custom routing code.
- **Backward Compatibility**: Existing code remains functional, with new features adding more capabilities without breaking changes.
- **Community Collaboration**: These enhancements were developed with community input through discussions and proposals, reflecting common needs in web development.

#### Examples and Usage:
- **Basic Route Handling Before Go 1.22**:
  ```go
  http.Handle("/posts/", handlePost)
  ```
  This approach requires additional code to check HTTP methods and extract identifiers.

- **Enhanced Route Handling in Go 1.22**:
  ```go
  http.Handle("GET /posts/{id}", handlePost2)
  ```
  This pattern directly matches GET requests for posts by ID, simplifying method checks and parameter extraction.

- **Wildcard Segment Matching**:
  - `{id}` matches a single path segment.
  - `{pathname...}` matches all remaining segments.

- **New `http.Request` Methods**:
  - `PathValue("id")` for extracting wildcard values.
  - `SetPathValue` allows setting path values, useful for external routers integrating with `net/http`.

#### Precedence Rules:
- Patterns are now selected based on specificity, where the most specific pattern wins when multiple match.
- Literal patterns take precedence over wildcard patterns.
- Patterns ending in a slash (`/posts/`) match all paths beginning with that string. To match only the exact path with the trailing slash, use `/posts/{$}`.

#### Compatibility Considerations:
- The new pattern syntax is designed to be a superset of the old, with efforts made to ensure compatibility with older Go versions.
- Edge cases, such as patterns with braces (`{}`), which were treated literally in older versions but represent wildcards in Go 1.22, can revert to old behavior with the `GODEBUG` setting `httpmuxgo121`.

These enhancements make the `net/http` package more versatile and user-friendly for common web routing tasks, reflecting Go's ongoing improvements to support production system development more effectively. For detailed information, refer to the [`net/http.ServeMux` documentation](https://pkg.go.dev/net/http#ServeMux).