**fundamentals of Golang**
---
## **1. Variables, Constants, and Data Types**

### **Variables**
Golang is statically typed, meaning variable types are determined at compile time.

```go
package main

import "fmt"

func main() {
	// Explicit declaration
	var name string = "John"

	// Implicit declaration (type inferred)
	age := 30

	// Multiple variable declaration
	var x, y int = 10, 20

	fmt.Println(name, age, x, y)
}
```

### **Constants**
Use `const` for values that do not change.

```go
const Pi = 3.14159
const Greeting = "Hello, Go!"
```

### **Data Types**
- **Primitive Types**: `int`, `float64`, `string`, `bool`
- **Composite Types**: `array`, `slice`, `map`, `struct`

```go
package main

import "fmt"

func main() {
	// Integer and Float
	var a int = 42
	var b float64 = 3.14
	var isActive bool = true

	// String
	var message string = "Golang"

	// Array (Fixed size)
	var arr [3]int = [3]int{1, 2, 3}

	// Slice (Dynamic array)
	slice := []string{"apple", "banana", "cherry"}

	// Map (Key-Value Store)
	person := map[string]int{"Alice": 25, "Bob": 30}

	fmt.Println(a, b, isActive, message, arr, slice, person)
}
```

---

## **2. Functions**
Functions in Go can return multiple values.

```go
package main

import "fmt"

// Function with return type
func add(a int, b int) int {
	return a + b
}

// Function returning multiple values
func divide(x, y int) (int, int) {
	quotient := x / y
	remainder := x % y
	return quotient, remainder
}

func main() {
	result := add(5, 7)
	fmt.Println("Sum:", result)

	q, r := divide(10, 3)
	fmt.Println("Quotient:", q, "Remainder:", r)
}
```

---

## **3. Pointers (`*`, `&`, passing by reference)**
Pointers store the memory address of a variable.

```go
package main

import "fmt"

func main() {
	x := 10
	ptr := &x // Get memory address of x

	fmt.Println("Value of x:", x)
	fmt.Println("Address of x:", ptr)
	fmt.Println("Value at ptr:", *ptr) // Dereferencing
}
```

### **Passing by Reference**
When you pass a pointer to a function, it modifies the original variable.

```go
package main

import "fmt"

func modifyValue(num *int) {
	*num = 50 // Changing the value at the memory address
}

func main() {
	value := 10
	fmt.Println("Before:", value)

	modifyValue(&value)
	fmt.Println("After:", value)
}
```

---

## **4. Control Structures**

### **If-Else**
```go
package main

import "fmt"

func main() {
	age := 20

	if age >= 18 {
		fmt.Println("You are an adult.")
	} else {
		fmt.Println("You are a minor.")
	}
}
```

### **For Loop**
Golang only has `for` loops (no `while` or `do-while`).

```go
package main

import "fmt"

func main() {
	// Traditional loop
	for i := 0; i < 5; i++ {
		fmt.Println(i)
	}

	// Range-based loop (for slices, arrays, maps)
	nums := []int{10, 20, 30}
	for index, value := range nums {
		fmt.Println("Index:", index, "Value:", value)
	}
}
```

### **Switch Case**
```go
package main

import "fmt"

func main() {
	day := "Monday"

	switch day {
	case "Monday":
		fmt.Println("Start of the week!")
	case "Friday":
		fmt.Println("Weekend is near!")
	default:
		fmt.Println("It's a normal day.")
	}
}
```

---

## **5. Error Handling**

### **Using `error` Interface**
```go
package main

import (
	"errors"
	"fmt"
)

// Custom error function
func divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("cannot divide by zero")
	}
	return a / b, nil
}

func main() {
	result, err := divide(10, 0)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("Result:", result)
	}
}
```

### **Using `fmt.Errorf`**
```go
package main

import (
	"fmt"
)

// Using fmt.Errorf to format error messages
func getUser(id int) (string, error) {
	if id != 1 {
		return "", fmt.Errorf("user with ID %d not found", id)
	}
	return "John Doe", nil
}

func main() {
	user, err := getUser(2)
	if err != nil {
		fmt.Println("Error:", err)
	} else {
		fmt.Println("User:", user)
	}
}
```

### **Panic and Recover**
- `panic` stops the normal execution of a function.
- `recover` is used to catch a panic and prevent program crash.

```go
package main

import "fmt"

func safeFunction() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Recovered from panic:", r)
		}
	}()
	
	panic("Something went wrong!")
}

func main() {
	safeFunction()
	fmt.Println("Program continues execution.")
}
```

---
