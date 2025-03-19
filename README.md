# Learn_golang
Learn Golang basics and advanced concepts

---

## **1️⃣ Concurrency and Goroutines**  
### 🔹 **Key Topics:**  
- Goroutines & Channels  
- WaitGroups & Mutexes  
- Deadlocks & Race Conditions  
- Context for cancellation & timeouts  

### **✅ Sample Questions:**  
1. How does **Goroutine scheduling** work in Go?  
2. What is the difference between **sync.Mutex** and **sync.RWMutex**?  
3. Explain **channel deadlocks**. How can you prevent them?  
4. Why do we use **context.Context** in concurrent programs?  
5. Implement a **worker pool** using Goroutines.  

### **💡 Sample Code: Worker Pool with Goroutines**  
```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, results chan<- int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job)
		time.Sleep(time.Second) // Simulate processing time
		results <- job * 2
	}
}

func main() {
	const numWorkers = 3
	jobs := make(chan int, 5)
	results := make(chan int, 5)
	var wg sync.WaitGroup

	// Start worker goroutines
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go worker(i, jobs, results, &wg)
	}

	// Send jobs to workers
	for i := 1; i <= 5; i++ {
		jobs <- i
	}
	close(jobs) // Close the channel to signal no more jobs

	// Wait for all workers to complete
	wg.Wait()
	close(results)

	// Print results
	for res := range results {
		fmt.Println("Result:", res)
	}
}
```
---
## **2️⃣ Performance Optimization & Memory Management**  
### 🔹 **Key Topics:**  
- Garbage Collection (GC)  
- Escape Analysis  
- Profiling & Benchmarking  

### **✅ Sample Questions:**  
1. How does **Garbage Collection** work in Go?  
2. Explain **Escape Analysis**. How can you optimize memory allocation?  
3. How can you detect **memory leaks** in a Go application?  
4. How would you optimize a high-performance **REST API** in Go?  
5. What is the difference between **sync.Pool** and **normal allocation**?  

### **💡 Sample Code: Detecting Escape Analysis**  
```go
package main

import "fmt"

type Data struct {
	value int
}

func escape() *Data {
	d := Data{value: 42} // Allocated on heap (escapes)
	return &d
}

func main() {
	fmt.Println(escape())
}
```
**🔹 Why?** Since the returned `Data` instance **escapes the function scope**, it is allocated **on the heap** instead of the stack.

---
## **3️⃣ System Design & Scalability**  
### 🔹 **Key Topics:**  
- Microservices & Monolith vs Microservices  
- API Rate Limiting & Caching  
- Distributed Systems  

### **✅ Sample Questions:**  
1. How would you design a **rate-limiting system** in Go?  
2. Explain **CQRS** and **Event Sourcing**. How would you implement them in Go?  
3. How would you scale a Go-based **microservice**?  
4. Implement a **caching mechanism** for a REST API using Go.  
5. How would you handle **database connection pooling** in Go?  

### **💡 Sample Code: Implementing API Rate Limiter**  
```go
package main

import (
	"fmt"
	"net/http"
	"sync"
	"time"
)

var rateLimiter = struct {
	sync.Mutex
	requests map[string]int
}{requests: make(map[string]int)}

func rateLimitMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		clientIP := r.RemoteAddr

		rateLimiter.Lock()
		rateLimiter.requests[clientIP]++
		count := rateLimiter.requests[clientIP]
		rateLimiter.Unlock()

		if count > 5 {
			http.Error(w, "Rate limit exceeded", http.StatusTooManyRequests)
			return
		}

		next.ServeHTTP(w, r)
	})
}

func mainHandler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello, World!")
}

func main() {
	go func() {
		for {
			time.Sleep(1 * time.Minute)
			rateLimiter.Lock()
			rateLimiter.requests = make(map[string]int) // Reset rate limits
			rateLimiter.Unlock()
		}
	}()

	http.Handle("/", rateLimitMiddleware(http.HandlerFunc(mainHandler)))
	http.ListenAndServe(":8080", nil)
}
```
---
## **4️⃣ Golang Internals & Advanced Topics**  
### 🔹 **Key Topics:**  
- Reflection  
- Custom Error Handling  
- Embedding & Interfaces  

### **✅ Sample Questions:**  
1. What is **reflection**, and when should you use it?  
2. How does **interface dispatch** work in Go?  
3. What is the difference between `error` and `panic`?  
4. Explain the difference between **struct embedding** vs **interface composition**.  
5. How would you implement **custom logging middleware** in Go?  

### **💡 Sample Code: Custom Error Handling**  
```go
package main

import "fmt"

type MyError struct {
	Message string
	Code    int
}

func (e *MyError) Error() string {
	return fmt.Sprintf("Error %d: %s", e.Code, e.Message)
}

func doSomething() error {
	return &MyError{"Something went wrong", 500}
}

func main() {
	err := doSomething()
	if err != nil {
		fmt.Println(err)
	}
}
```
---
## **5️⃣ Testing & CI/CD Best Practices**  
### 🔹 **Key Topics:**  
- Unit Testing (Testify)  
- Mocking (Mockery)  
- Benchmarking  

### **✅ Sample Questions:**  
1. How would you write **unit tests** for a Golang service?  
2. How do you **mock dependencies** using `mockery`?  
3. What is `httptest`, and how is it used in API testing?  
4. Explain **table-driven tests** in Go.  
5. How would you set up **CI/CD for a Go project**?  

### **💡 Sample Code: Unit Test with `testify`**
```go
package main

import (
	"testing"

	"github.com/stretchr/testify/assert"
)

func add(a, b int) int {
	return a + b
}

func TestAdd(t *testing.T) {
	result := add(2, 3)
	assert.Equal(t, 5, result, "they should be equal")
}
```
---
## **🚀 Final Thoughts**
A **Senior Golang Developer** should have **strong problem-solving skills** and **deep system knowledge**. Expect questions on:  
✔️ **Concurrency & Performance**  
✔️ **Scalability & System Design**  
✔️ **Memory Management & Profiling**  
✔️ **Testing & CI/CD**  

