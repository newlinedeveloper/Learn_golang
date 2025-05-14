Great! Let's go through the answers and explanations for each of these **Go (Golang) basic/core language** interview questions:

---

### ✅ **1. What are the key features of Go?**

* **Compiled language** with fast execution (compiles to machine code).
* **Statically typed**, with type inference support.
* **Built-in concurrency** using goroutines and channels.
* **Garbage collected**, automatic memory management.
* **Simplicity and readability** of syntax (influenced by C).
* **Standard library** is powerful and batteries-included.
* **Cross-compilation** built in (compile for other OS/architecture).
* **First-class functions** and support for closures.
* No inheritance, but **interfaces** and composition.

---

### ✅ **2. What is the difference between `var`, `:=`, and `const`?**

| Syntax  | Description                                            | Scope                   |
| ------- | ------------------------------------------------------ | ----------------------- |
| `var`   | Declares a variable with explicit type or value        | Package/global/function |
| `:=`    | Short variable declaration, used only inside functions | Function/local only     |
| `const` | Declares an immutable value (constant)                 | Package/global/local    |

Example:

```go
var a int = 10
b := 20
const pi = 3.14
```

---

### ✅ **3. What are slices and how are they different from arrays?**

* **Arrays**: Fixed length, defined at compile time.

  ```go
  var arr = [3]int{1, 2, 3}
  ```

* **Slices**: Dynamic size, backed by an array, more commonly used.

  ```go
  var s = []int{1, 2, 3}
  s = append(s, 4)
  ```

**Differences**:

| Aspect    | Array    | Slice     |
| --------- | -------- | --------- |
| Length    | Fixed    | Dynamic   |
| Syntax    | `[5]int` | `[]int`   |
| Use cases | Rare     | Preferred |

---

### ✅ **4. What is the zero value in Go?**

When a variable is declared but not initialized, it gets a **default zero value**:

| Type                                 | Zero Value |
| ------------------------------------ | ---------- |
| int                                  | `0`        |
| float64                              | `0.0`      |
| string                               | `""`       |
| bool                                 | `false`    |
| pointer                              | `nil`      |
| slice/map/function/interface/channel | `nil`      |

---

### ✅ **5. What is the purpose of `defer`?**

* `defer` delays the execution of a function until the **surrounding function returns**.
* Useful for **cleanup tasks** like closing files, unlocking mutexes, etc.

Example:

```go
func readFile() {
    f, _ := os.Open("file.txt")
    defer f.Close() // called even if function exits early
}
```

---

### ✅ **6. What is the difference between `new()` and `make()`?**

| Function | Use For                                                | Returns                      |
| -------- | ------------------------------------------------------ | ---------------------------- |
| `new()`  | Allocating memory for value types (int, struct, array) | Pointer to zeroed value      |
| `make()` | Creating and initializing slices, maps, channels       | Initialized (non-nil) object |

Example:

```go
ptr := new(int)      // *int, value is 0
s := make([]int, 0)  // []int, initialized slice
```

---

### ✅ **7. How does garbage collection work in Go?**

* Go uses a **concurrent garbage collector** to free memory that is no longer reachable.
* It is **non-blocking for goroutines**, which means your program keeps running during collection.
* Works with **write barriers** and **generational strategies**.
* Developers don’t manually free memory, but should avoid retaining unnecessary references.

---

### ✅ **8. What is the blank identifier `_` used for?**

* Used to **ignore a value** returned by a function or to avoid compiler errors when a value is unused.

Examples:

```go
_, err := someFunc() // ignore the first return value
_ = "unused"         // suppress unused variable error
```

---

Let me know if you'd like a **mock interview** style quiz or to move on to **Intermediate / Concurrency** questions next.
