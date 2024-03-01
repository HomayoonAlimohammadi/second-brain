#golang #reflection #software_engineering 

### References
https://go.dev/blog/laws-of-reflection

### The Laws of Reflection

This blog post by Rob Pike on 6 September 2011, updated in January 2022, delves into how reflection works in Go. Reflection, a form of metaprogramming, allows a program to examine its own structure through types. The post aims to clarify reflection in Go, highlighting its differences from other languages and explaining its fundamental principles and best practices.

#### Introduction to Reflection
- Reflection enables examining a program's structure, particularly types, at runtime.
- It's based on Go's type system, including static typing and interfaces.

#### Types and Interfaces
- Every variable in Go has a static type known at compile time.
- Interface types represent sets of methods, allowing variables to store any value that implements these methods.
- The empty interface (`interface{}` or `any`) can hold any value, as every value implements zero or more methods.

#### Interface Representation
- An interface variable stores a pair: the concrete value and its type descriptor.
- Type assertions are used to assert the interface's stored value implements other interfaces.

```go
type MyInt int
var x MyInt = 7
v := reflect.ValueOf(x)
```

> the `Kind` of `v` is still `reflect.Int`, even though the static type of `x` is `MyInt`, not `int`. In other words, the `Kind` cannot discriminate an `int` from a `MyInt` even though the `Type` can.

#### Reflection Laws
1. **Reflection goes from interface value to reflection object**: 
    - `reflect.TypeOf` and `reflect.ValueOf` functions retrieve type and value information from an interface value.
    - Example: `reflect.TypeOf(x)` returns the type information of `x`.
    
2. **Reflection goes from reflection object to interface value**: 
    - The `Interface()` method on a `reflect.Value` converts it back to an interface value, retaining type information.
    - Example: `v.Interface().(float64)` converts a reflection object `v` back to its original `float64` value.
    
3. **To modify a reflection object, the value must be settable**: 
    - Settability depends on whether the reflection object holds the original item's address. Only settable objects can be modified.
    - Example: To modify a variable `x` through reflection, one must pass a pointer to `x` to `reflect.ValueOf`.
		```go
		var x float64 = 3.4
		p := reflect.ValueOf(&x) // Note: take the address of x.
		fmt.Println("type of p:", p.Type()) // *float64
		fmt.Println("settability of p:", p.CanSet()) // false because the
		// pointer is not settable, but the underlying value is
		
		v := p.Elem()
		fmt.Println("settability of v:", v.CanSet()) // true
		```
    
#### Practical Examples
- The post illustrates using reflection to inspect and modify variables and struct fields.
- It emphasizes the importance of passing pointers to modify values and the role of exported fields in settability.

#### Conclusion
- Reflection is a powerful but subtle tool in Go, governed by three main laws.
- It should be used cautiously and is most useful when direct access to type or value information is necessary.

For a more in-depth exploration of reflection, including its applications and limitations, read the full blog post on [Go's official blog](https://go.dev/blog/laws-of-reflection).