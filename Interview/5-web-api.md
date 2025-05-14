Here’s a complete overview of how to handle **Web and API development in Go using the Gin framework**, with **examples** and **explanations** for each interview question:

---

### ✅ 1. **How do you create an HTTP server in Go (using Gin)?**

```go
package main

import (
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default() // includes Logger and Recovery middleware by default
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{"message": "pong"})
    })
    r.Run() // listens on :8080 by default
}
```

---

### ✅ 2. **How do you handle routes using Gin?**

```go
r := gin.Default()

// Basic routes
r.GET("/users", getUsers)
r.POST("/users", createUser)

// Path parameters
r.GET("/users/:id", getUserByID)

// Grouped routes (e.g., for /api/v1/)
v1 := r.Group("/api/v1")
{
    v1.GET("/items", getItems)
}
```

---

### ✅ 3. **How do you parse JSON in Go (Gin automatically binds JSON)?**

```go
type User struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

func createUser(c *gin.Context) {
    var user User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    c.JSON(200, gin.H{"user": user})
}
```

---

### ✅ 4. **How do you handle middleware in Go (Gin)?**

```go
func LoggerMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // before request
        fmt.Println("Before handler")

        c.Next()

        // after request
        fmt.Println("After handler")
    }
}

func main() {
    r := gin.Default()
    r.Use(LoggerMiddleware())
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{"message": "pong"})
    })
    r.Run()
}
```

---

### ✅ 5. **How do you write unit tests for HTTP handlers (Gin)?**

You can use `httptest` from the standard library along with `net/http`.

```go
// handler.go
func PingHandler(c *gin.Context) {
    c.JSON(200, gin.H{"message": "pong"})
}
```

```go
// handler_test.go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"

    "github.com/gin-gonic/gin"
    "github.com/stretchr/testify/assert"
)

func TestPingHandler(t *testing.T) {
    w := httptest.NewRecorder()
    c, _ := gin.CreateTestContext(w)

    PingHandler(c)

    assert.Equal(t, 200, w.Code)
    assert.Contains(t, w.Body.String(), "pong")
}
```

---

### 📌 Summary

| Task         | Tool/Method                               |
| ------------ | ----------------------------------------- |
| HTTP Server  | `gin.Default()` + `Run()`                 |
| Routing      | `GET`, `POST`, Groups                     |
| JSON Parsing | `ShouldBindJSON`                          |
| Middleware   | Custom `gin.HandlerFunc`                  |
| Unit Testing | `httptest`, `CreateTestContext`, `assert` |

Let me know if you want to build a full CRUD API with Gin as a reference.
