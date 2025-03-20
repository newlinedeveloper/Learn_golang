Great! Let’s dive into **Advanced Golang Concepts** step by step. 🚀  

---

## **1. Concurrency & Goroutines**
Golang provides lightweight threads called **goroutines** that run independently.

### **Goroutines (`go` keyword)**
A goroutine runs a function concurrently.

```go
package main

import (
	"fmt"
	"time"
)

func printNumbers() {
	for i := 1; i <= 5; i++ {
		fmt.Println(i)
		time.Sleep(500 * time.Millisecond)
	}
}

func main() {
	go printNumbers() // Run in a separate goroutine
	fmt.Println("Main function executed")

	time.Sleep(3 * time.Second) // Wait to see goroutine output
}
```
⚡ Without `time.Sleep`, the program may exit before `printNumbers()` runs.

---

### **WaitGroup (`sync.WaitGroup`)**
When launching multiple goroutines, use `sync.WaitGroup` to wait for all to complete.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, wg *sync.WaitGroup) {
	defer wg.Done() // Mark task as done
	fmt.Printf("Worker %d starting\n", id)
	time.Sleep(1 * time.Second)
	fmt.Printf("Worker %d done\n", id)
}

func main() {
	var wg sync.WaitGroup

	for i := 1; i <= 3; i++ {
		wg.Add(1)         // Add a task
		go worker(i, &wg) // Run as goroutine
	}

	wg.Wait() // Wait for all workers
	fmt.Println("All workers finished")
}
```

---

### **Mutex (`sync.Mutex`)**
Use `sync.Mutex` to prevent **race conditions** when multiple goroutines modify shared data.

```go
package main

import (
	"fmt"
	"sync"
)

var counter int
var mutex sync.Mutex

func increment(wg *sync.WaitGroup) {
	defer wg.Done()
	mutex.Lock()
	counter++
	mutex.Unlock()
}

func main() {
	var wg sync.WaitGroup

	for i := 0; i < 5; i++ {
		wg.Add(1)
		go increment(&wg)
	}

	wg.Wait()
	fmt.Println("Final Counter:", counter)
}
```

---

### **Channels**
Channels allow goroutines to **communicate safely**.

```go
package main

import "fmt"

func sendData(ch chan string) {
	ch <- "Hello from Goroutine!"
}

func main() {
	ch := make(chan string)
	go sendData(ch)

	msg := <-ch // Receive from channel
	fmt.Println(msg)
}
```

---

### **Buffered Channels**
Buffered channels allow sending multiple values before a receiver picks them up.

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 2) // Buffer size 2

	ch <- 1
	ch <- 2

	fmt.Println(<-ch) // Receive first value
	fmt.Println(<-ch) // Receive second value
}
```

---

### **Select Statement**
Used to wait on multiple channels.

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go func() {
		time.Sleep(2 * time.Second)
		ch1 <- "Message from channel 1"
	}()

	go func() {
		time.Sleep(1 * time.Second)
		ch2 <- "Message from channel 2"
	}()

	select {
	case msg1 := <-ch1:
		fmt.Println(msg1)
	case msg2 := <-ch2:
		fmt.Println(msg2) // This executes first
	}
}
```

---

## **2. Interfaces & Structs**
Interfaces allow writing **flexible, reusable** code.

### **Structs**
```go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func main() {
	p := Person{Name: "Alice", Age: 30}
	fmt.Println("Person:", p)
}
```

### **Interfaces**
```go
package main

import "fmt"

type Shape interface {
	Area() float64
}

type Rectangle struct {
	Width, Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

func printArea(s Shape) {
	fmt.Println("Area:", s.Area())
}

func main() {
	r := Rectangle{Width: 5, Height: 10}
	printArea(r)
}
```

---

## **3. Worker Pool (Efficient Goroutines)**
A **worker pool** limits the number of concurrent goroutines to avoid high memory usage.

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func worker(id int, jobs <-chan int, wg *sync.WaitGroup) {
	defer wg.Done()
	for job := range jobs {
		fmt.Printf("Worker %d processing job %d\n", id, job)
		time.Sleep(time.Second) // Simulate work
	}
}

func main() {
	numWorkers := 3
	numJobs := 5

	jobs := make(chan int, numJobs)
	var wg sync.WaitGroup

	// Start workers
	for i := 1; i <= numWorkers; i++ {
		wg.Add(1)
		go worker(i, jobs, &wg)
	}

	// Send jobs
	for j := 1; j <= numJobs; j++ {
		jobs <- j
	}
	close(jobs) // Close channel to stop workers

	wg.Wait() // Wait for all workers to finish
}
```

---

## **4. Context Package**
Context is used for **timeouts, cancellations**, and **propagating deadlines** in Goroutines.

### **Context with Timeout**
```go
package main

import (
	"context"
	"fmt"
	"time"
)

func longRunningTask(ctx context.Context) {
	select {
	case <-time.After(3 * time.Second):
		fmt.Println("Task completed")
	case <-ctx.Done():
		fmt.Println("Task cancelled:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel() // Free up resources

	go longRunningTask(ctx)

	time.Sleep(3 * time.Second)
}
```
⏳ Here, `longRunningTask` runs for **3 seconds**, but the context **cancels** it after **2 seconds**.

---

### **Context with Cancel**
Used to stop multiple goroutines.

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Worker", id, "stopped")
			return
		default:
			fmt.Println("Worker", id, "working...")
			time.Sleep(1 * time.Second)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go worker(ctx, 1)
	go worker(ctx, 2)

	time.Sleep(3 * time.Second)
	cancel() // Stop all workers

	time.Sleep(1 * time.Second) // Allow logs to print
}
```

---


I'll create a **Concurrent File Uploader/Downloader** using **Go-Gin**, Goroutines, Channels, and Worker Pools.  

## **Features**
✅ Upload multiple files concurrently  
✅ Download files using worker pools  
✅ Use **Gin** for API endpoints  
✅ Use **Goroutines** for concurrent execution  
✅ Store files in a local directory  

---

## **Project Structure**
```
/concurrent-file-server
│── main.go             # Entry point
│── handlers.go         # Upload and Download handlers
│── worker_pool.go      # Worker pool for concurrent processing
│── uploads/           # Directory for storing uploaded files
```

---

## **1. Install Dependencies**
Run:
```sh
go mod init concurrent-file-server
go get github.com/gin-gonic/gin
```

---

## **2. `main.go` (Entry Point)**
```go
package main

import (
	"fmt"
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
)

func main() {
	// Create uploads directory if it doesn't exist
	if _, err := os.Stat("uploads"); os.IsNotExist(err) {
		os.Mkdir("uploads", 0755)
	}

	router := gin.Default()

	// Routes
	router.POST("/upload", UploadFiles)
	router.GET("/download/:filename", DownloadFile)

	// Start the server
	fmt.Println("Server running on port 8080...")
	router.Run(":8080")
}
```

---

## **3. `handlers.go` (Upload & Download Handlers)**
```go
package main

import (
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"

	"github.com/gin-gonic/gin"
)

// UploadFiles - Upload multiple files concurrently
func UploadFiles(c *gin.Context) {
	form, err := c.MultipartForm()
	if err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid request"})
		return
	}

	files := form.File["files"]
	results := make(chan string, len(files)) // Channel to collect responses

	// Launch goroutines to upload files concurrently
	for _, file := range files {
		go func(f *gin.FileHeader) {
			dst := filepath.Join("uploads", f.Filename)
			if err := c.SaveUploadedFile(f, dst); err != nil {
				results <- fmt.Sprintf("Failed: %s", f.Filename)
			} else {
				results <- fmt.Sprintf("Uploaded: %s", f.Filename)
			}
		}(file)
	}

	// Collect results
	var response []string
	for i := 0; i < len(files); i++ {
		response = append(response, <-results)
	}

	c.JSON(http.StatusOK, gin.H{"message": response})
}

// DownloadFile - Download a file using worker pool
func DownloadFile(c *gin.Context) {
	filename := c.Param("filename")
	filePath := filepath.Join("uploads", filename)

	if _, err := os.Stat(filePath); os.IsNotExist(err) {
		c.JSON(http.StatusNotFound, gin.H{"error": "File not found"})
		return
	}

	c.Header("Content-Disposition", "attachment; filename="+filename)
	c.Header("Content-Type", "application/octet-stream")

	file, err := os.Open(filePath)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "Cannot open file"})
		return
	}
	defer file.Close()

	// Use a worker pool for concurrent downloading
	downloadWorkerPool(file, c.Writer)
}
```

---

## **4. `worker_pool.go` (Worker Pool for Downloads)**
```go
package main

import (
	"io"
	"net/http"
	"sync"
)

// Worker pool to process file downloads in chunks
func downloadWorkerPool(file io.Reader, writer http.ResponseWriter) {
	const chunkSize = 1024 // 1KB chunks
	var wg sync.WaitGroup

	// Channel for sending chunks
	ch := make(chan []byte, 5)

	// Worker goroutine
	wg.Add(1)
	go func() {
		defer wg.Done()
		for chunk := range ch {
			writer.Write(chunk) // Send chunk to response writer
		}
	}()

	// Read and send file in chunks
	buf := make([]byte, chunkSize)
	for {
		n, err := file.Read(buf)
		if n > 0 {
			ch <- buf[:n]
		}
		if err == io.EOF {
			break
		}
	}

	close(ch) // Close the channel
	wg.Wait() // Wait for workers to finish
}
```

---

## **5. Testing the API**
### **Start the server**
Run:
```sh
go run main.go
```
---
### **Test File Upload**
Use **cURL**:
```sh
curl -X POST http://localhost:8080/upload -F "files=@test1.txt" -F "files=@test2.txt"
```

Or use **Postman**:
- Select **POST**
- Enter URL: `http://localhost:8080/upload`
- Choose **Form-Data**
- Key: `files` (Select **File** type)
- Upload multiple files

**Response:**
```json
{
  "message": ["Uploaded: test1.txt", "Uploaded: test2.txt"]
}
```
---
### **Test File Download**
```sh
curl -X GET http://localhost:8080/download/test1.txt -o downloaded_test1.txt
```

Or visit:
```
http://localhost:8080/download/test1.txt
```

---

## **Summary**
✔ **Gin** used for REST API  
✔ **Goroutines** for concurrent uploads  
✔ **Worker Pool** for efficient downloads  
✔ **Channels** for safe communication  
