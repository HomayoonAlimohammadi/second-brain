#golang #json 

### References
https://www.cockroachlabs.com/blog/high-performance-json-parsing/
### Summary

The blog post from Cockroach Labs discusses the journey and challenges of optimizing JSON parsing performance in CockroachDB, focusing on the need for high-performance JSON parsing in Go. Here are the key points and technical details:

- **Initial State**: CockroachDB initially used Go's standard `encoding/json` library for JSON parsing, which was identified as inefficient due to its two-phase parsing process: first parsing the string into raw `interface{}` types using `encoding/json`, then transforming that into a CockroachDB-specific JSON representation.
    
- **Problem Identification**: The inefficiency became apparent when a customer reported significant performance degradation when inserting JSON data into JSONB columns compared to BYTES columns. Benchmarking confirmed the customer's observations, showing that the `ParseJSON` function was a bottleneck, especially for large inputs or inputs with escaped characters.
    
- **Benchmarking and Results**: Initial benchmarks on various input sizes revealed that the parsing speed and memory allocation efficiency of the `encoding/json` library were suboptimal. For example, parsing a 524,288-byte input took about 2.8 milliseconds, with significant memory allocation overhead.
    
- **Exploration of Alternatives**: The team explored several third-party JSON parsing libraries seeking better performance and memory allocation behavior. `jsoniter` emerged as a promising alternative, offering significant improvements in parsing throughput but still not optimal in terms of memory allocation, especially for inputs with escaped characters.
    
- **Further Optimization**: Further research led to the discovery of the `pkg/json` library, which promised over 1GB/s parsing throughput and minimal allocations through API design. Although initially not production-ready, Cockroach Labs decided to fork and enhance this library, adding missing functionality and optimizing it for their use case.
    
- **Outstanding Improvements**: The optimized JSON parser significantly outperformed both the standard library and `jsoniter` in benchmarks, achieving up to 8 times faster parsing speeds and more efficient memory allocation. This was particularly evident in the handling of large inputs and inputs with a high percentage of escaped characters.
    
- **Technical Insights**: The blog post highlights the importance of API design in achieving high performance, specifically how returning a byte array for the next token in the input stream can reduce allocations. It also discusses the benefits of method expressions and compile-time escape analysis in optimizing performance.
    
- **Conclusion**: The optimization project was a success, demonstrating the value of exploring open-source alternatives and the importance of targeted optimization efforts. Cockroach Labs also considers contributing their improvements back to the `pkg/json` package.
    

This case study underscores the complexities of JSON parsing performance in Go and illustrates how a combination of benchmarking, exploring alternatives, and targeted optimization can lead to significant performance gains.