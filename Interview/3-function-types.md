Here are concise explanations and examples for each of the **"Functions and Types"** concepts in Go:

---

### 1. **How do you implement higher-order functions in Go?**

A higher-order function is one that **takes a function as an argument or returns a function**.

```go
func apply(op func(int, int) int, a, b int) int {
    return op(a, b)
}

func add(x, y int) int {
    return x + y
}

func main() {
    fmt.Println(apply(add, 2, 3)) // Output: 5
}
```

---

### 2. **What are first-class functions?**

In Go, functions are **first-class citizens**, meaning:

* You can assign them to variables.
* You can pass them as arguments.
* You can return them from other functions.

Example:

```go
f := func(a, b int) int {
    return a + b
}
fmt.Println(f(3, 4)) // Output: 7
```

---

### 3. **What are variadic functions?**

Functions that accept **a variable number of arguments** of the same type.

```go
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

fmt.Println(sum(1, 2, 3, 4)) // Output: 10
```

---

### 4. **What is a method vs function in Go?**

* A **function** is declared with `func`.
* A **method** is a function with a **receiver type**.

```go
type User struct {
    Name string
}

func greet(u User) {
    fmt.Println("Hello", u.Name)
}

func (u User) greet() {
    fmt.Println("Hello", u.Name)
}
```

Use a **method** when behavior is tightly associated with a type.

---

### 5. **Explain interface types. How is duck typing used in Go?**

An `interface` defines a **set of method signatures**. Any type that implements those methods satisfies the interface — no explicit declaration needed (this is duck typing).

```go
type Speaker interface {
    Speak()
}

type Person struct{}

func (p Person) Speak() {
    fmt.Println("I'm a person")
}

func talk(s Speaker) {
    s.Speak()
}
```

Here, `Person` automatically satisfies `Speaker` by implementing `Speak()`.

---

### 6. **What is type assertion and type switch?**

#### **Type assertion:**

Used to extract the **concrete type** from an interface.

```go
var i interface{} = "hello"
s := i.(string)
fmt.Println(s) // Output: hello
```

#### **Type switch:**

Used when you don't know the type in advance.

```go
switch v := i.(type) {
case string:
    fmt.Println("string:", v)
case int:
    fmt.Println("int:", v)
default:
    fmt.Println("unknown type")
}
```

---

### 7. **What is the empty interface `interface{}` and when is it used?**

`interface{}` means **"any type"**. It's Go’s version of a dynamic type.

Use it when:

* You don’t know the type ahead of time.
* Interfacing with JSON, generic containers, or `fmt` functions.

Example:

```go
func printAnything(v interface{}) {
    fmt.Println(v)
}
```

---
