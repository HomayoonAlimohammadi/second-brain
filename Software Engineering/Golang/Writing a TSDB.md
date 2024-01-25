#gophercon #golang #tsdb #software_engineering #optimization 

pick the right data structure:
- lsm (reduce write)
- column oriented storage (??) 3 separate columns, metric names and metadata, values, timestamps (3 separate columns??, sorted and compressed differently)
- ts data does not change that much, use append-only writes

optimizations:
- string interning
	- store only on unique copy in memory
	- allocate a new object for a string only once and others refer to it
	- basically a `map[string]string` map (s to s)
	```go
	func ptr (s string) uintptr {
		return (*reflect.StringHeader)(unsafe.Pointer(&s)).Data
	}
	
	func main() {
		s1 := intern("42")
		s2 := intern("fmt.Sprintf("%d", 42))
		fmt.Println(ptr(s1) == ptr(s2)) // true
	}
	```
	
	- map is not thread safe, locks and mutexes are not scalable with lots of contention
	- use sync.Map{} (LoadOrStore), this is good for when you want to write once and to read many times (high cache hit rate), it is optimized for concurrent access for multiple goroutines (TODO: benchmark)
	- our map becomes less effective over time, how to avoid:
		- rotate maps once in a while (replace it with a blank one)
		- or add ttl logic to purge cold entries
	- sanity check of arguments
		- *(*string)(unsafe.Pointer(&b)) or str[:n]
		- bytes slice or substring (mutable), TODO: check what it means
		- if you intern a substring (slice) it will point to the original object (wrong, TODO: check)
		- use strings.Clone(s)
		
	- summary: 
		- intern is used to store time series metadata
		- helps reduce memory usage during parsing
		- good for read intesive work flows with limited number of variants with high hit rate
		
- func result caching
	- ex: relabling (some_thing -> some-thing)
	- this tranformation happens on scrape for same label
	- using caching transformer (struct with Transform method and a sync.Map cache)
	- a Transformer struct can have transformerFunc and a cache to have NewTransformer with custom func
	- this help save CPU, consts: increased memory
	- works best for string transformations and regex matching (TODO: why?)
	- like any cache needs high hit rat
	
- limiting concurrency for cpu-bound load
	- concurLimit := runtime.NumCPU() and x2 number of workers (6 cpu 12 workers) (TODO: check was is NumCPU)
	- maybe an easier approach is to have a global channel with len of NumCPU and try to push into that channel (blocking) in the beginning of a process func, and then try to read (empty) from it at the end
	- beware of limiting IO bound operations (they should not be blocked)
	- this prevents memory spikes as well
	

