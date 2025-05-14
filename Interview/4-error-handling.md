Here’s a detailed yet clear explanation of **Error Handling in Go**, covering each of your interview question points:

---

### 1. **How does Go handle errors?**

Go handles errors **explicitly** using the built-in `error` type. Functions that may fail typically return an `error` as a second return value.

#### Example:

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}
```

---

### 2. **Why doesn't Go use exceptions?**

Go deliberately **avoids exceptions** for control flow to keep error handling **explicit, simple, and predictable**.

* Exceptions make flow harder to reason about.
* Go prefers **explicit checks** rather than `try/catch`.

> Panic/recover exists, but it's meant for truly unexpected scenarios like runtime errors or system crashes — **not for normal error handling**.

---

### 3. **What is the idiomatic way to handle multiple errors?**

Check each error after each call. It may seem verbose, but it ensures **clear and local** handling.

#### Example:

```go
file, err := os.Open("data.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

data, err := io.ReadAll(file)
if err != nil {
    log.Fatal(err)
}
```

You can also use helper functions to reduce boilerplate if needed.

---

### 4. **How do you define custom error types?**

Implement the `error` interface by defining a `Error() string` method.

#### Example:

```go
type NotFoundError struct {
    Resource string
}

func (e NotFoundError) Error() string {
    return fmt.Sprintf("%s not found", e.Resource)
}

// Usage
err := NotFoundError{Resource: "User"}
fmt.Println(err.Error()) // Output: User not found
```

Useful for returning structured or typed errors.

---

### 5. **What is the use of `errors.Is()` and `errors.As()`?**

These were added in Go 1.13 for **error inspection** with wrapped errors.

#### ✅ `errors.Is(err, target)`

Checks whether `err` is or wraps a target error.

```go
if errors.Is(err, os.ErrNotExist) {
    fmt.Println("File does not exist")
}
```

#### ✅ `errors.As(err, &target)`

Unwraps and **extracts** a specific error type.

```go
var pathError *os.PathError
if errors.As(err, &pathError) {
    fmt.Println("Failed path:", pathError.Path)
}
```

These functions are preferred over manual type assertions for **robust and layered error handling**.

---
