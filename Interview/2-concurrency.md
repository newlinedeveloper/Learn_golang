Great set of Go concurrency interview questions! Here are **answers with explanations** to help you prepare confidently:

---

### 🧵 1. What are goroutines and how are they different from threads?

**Goroutines** are lightweight functions managed by the Go runtime for concurrency. They are started with the `go` keyword.

```go
go doWork()
```

**Differences from threads:**

| Aspect     | Goroutine                           | OS Thread                   |
| ---------- | ----------------------------------- | --------------------------- |
| Memory     | Uses \~2KB stack (grows/shrinks)    | Uses \~1MB stack (fixed)    |
| Management | Managed by Go runtime scheduler     | Managed by OS kernel        |
| Count      | Can spawn thousands easily          | Limited by system resources |
| Scheduling | Cooperative, multiplexed on threads | Preemptive (by OS)          |

---

### 📦 2. How do channels work in Go?

**Channels** are typed pipes used to communicate between goroutines safely.

```go
ch := make(chan int)

go func() {
    ch <- 10  // send
}()

val := <-ch   // receive
```

* Channels block until the other side is ready (for unbuffered).
* They prevent race conditions by synchronizing data access.

---

### 🔄 3. What is the difference between buffered and unbuffered channels?

| Type           | Behavior                                                                |
| -------------- | ----------------------------------------------------------------------- |
| **Unbuffered** | Sends block until a receiver is ready. Used for strict synchronization. |
| **Buffered**   | Can send up to `n` values without a receiver. Blocks only when full.    |

```go
// Buffered
ch := make(chan int, 2)
ch <- 1 // does not block
ch <- 2 // does not block
// ch <- 3 // would block (buffer full)
```

---

### 🔘 4. What are select statements used for?

`select` lets you wait on multiple channel operations.

```go
select {
case val := <-ch1:
    fmt.Println("Received from ch1", val)
case ch2 <- 42:
    fmt.Println("Sent to ch2")
default:
    fmt.Println("No activity")
}
```

* Only one case runs (random if multiple ready).
* `default` is optional and runs if nothing is ready.

---

### ⚠️ 5. What are race conditions and how do you detect them?

A **race condition** happens when multiple goroutines access shared data concurrently, and at least one modifies it without synchronization.

#### Detecting:

Use the **race detector**:

```bash
go run -race main.go
```

It detects unsynchronized read/writes and outputs warnings.

---

### 👥 6. What is the purpose of `sync.WaitGroup`?

`sync.WaitGroup` waits for a collection of goroutines to finish.

```go
var wg sync.WaitGroup

wg.Add(2)

go func() {
    defer wg.Done()
    // task 1
}()

go func() {
    defer wg.Done()
    // task 2
}()

wg.Wait()  // blocks until counter is zero
```

---

### 🔐 7. Explain the difference between `sync.Mutex` and `sync.RWMutex`.

| Lock Type      | Use Case                          |
| -------------- | --------------------------------- |
| `sync.Mutex`   | Standard lock — exclusive access. |
| `sync.RWMutex` | Multiple readers OR one writer.   |

```go
var mu sync.RWMutex

// For read:
mu.RLock()
// read-only operations
mu.RUnlock()

// For write:
mu.Lock()
// write operations
mu.Unlock()
```

* RWMutex improves performance when many reads but fewer writes are expected.


```
// Online Go compiler to run Golang program online
// Print "Try programiz.pro" message

package main
import (
    "fmt"
    "sync"
)

type Counter struct{
    data int
    mu sync.Mutex
}

func worker(counter *Counter, wg *sync.WaitGroup) {
    defer wg.Done()
    counter.mu.Lock()
    defer counter.mu.Unlock()
    counter.data += 1
}


func main() {
    var wg sync.WaitGroup
    workerCount := 5
    
    counter := Counter{}
    
    for i :=0; i < workerCount; i++ {
        wg.Add(1)
        go worker(&counter, &wg)
    }
    
    wg.Wait()
    fmt.Println("Completed => ", counter.data)
}

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    if result != 5 {
        t.Errorf("expected 5, got %d", result)
    }
}

```

---

Let me know if you’d like example code or mock questions around these!
