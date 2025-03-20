Let's build a **Golang CRUD API with authentication and logging** using:  

✅ **Gin** for the REST API  
✅ **GORM** for PostgreSQL database interaction  
✅ **JWT Authentication** for secure access  
✅ **Logrus** for structured logging  
✅ **Unit testing** with `testing` and `httptest`  

---

## **📂 Project Structure**
```
/golang-crud-api
│── main.go             # Entry point
│── config.go           # Database setup
│── models.go           # Defines models
│── handlers.go         # CRUD operations
│── auth.go             # JWT Authentication
│── middleware.go       # JWT Middleware
│── logger.go           # Logrus logging setup
│── routes.go           # API routes
│── .env                # Environment variables
│── go.mod              # Dependencies
```

---

## **1️⃣ Install Dependencies**
Run:
```sh
go mod init golang-crud-api
go get github.com/gin-gonic/gin gorm.io/gorm gorm.io/driver/postgres github.com/golang-jwt/jwt/v5 github.com/sirupsen/logrus godotenv
```

---

## **2️⃣ Setup `.env` File**
```ini
DB_HOST=localhost
DB_USER=postgres
DB_PASSWORD=yourpassword
DB_NAME=golang_db
DB_PORT=5432
JWT_SECRET=your_secret_key
```

---

## **3️⃣ `config.go` (Database Setup)**
```go
package main

import (
	"fmt"
	"log"
	"os"

	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"github.com/joho/godotenv"
)

var DB *gorm.DB

func InitDB() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}

	dsn := fmt.Sprintf("host=%s user=%s password=%s dbname=%s port=%s sslmode=disable",
		os.Getenv("DB_HOST"), os.Getenv("DB_USER"), os.Getenv("DB_PASSWORD"), os.Getenv("DB_NAME"), os.Getenv("DB_PORT"),
	)

	var dbErr error
	DB, dbErr = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if dbErr != nil {
		log.Fatal("Failed to connect to database")
	}

	log.Println("Database connected")
	DB.AutoMigrate(&User{})
}
```

---

## **4️⃣ `models.go` (Database Models)**
```go
package main

import "gorm.io/gorm"

type User struct {
	gorm.Model
	Username string `gorm:"unique" json:"username"`
	Password string `json:"-"`
	Email    string `gorm:"unique" json:"email"`
}
```

---

## **5️⃣ `auth.go` (JWT Authentication)**
```go
package main

import (
	"fmt"
	"os"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

// GenerateJWT - Create a JWT token
func GenerateJWT(username string) (string, error) {
	secret := os.Getenv("JWT_SECRET")

	claims := jwt.MapClaims{
		"username": username,
		"exp":      time.Now().Add(time.Hour * 24).Unix(), // 1 day expiration
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString([]byte(secret))
}
```

---

## **6️⃣ `middleware.go` (JWT Middleware)**
```go
package main

import (
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v5"
)

// AuthMiddleware - Middleware for JWT authentication
func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		tokenString := c.GetHeader("Authorization")
		if tokenString == "" {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Missing token"})
			c.Abort()
			return
		}

		// Parse token
		secret := os.Getenv("JWT_SECRET")
		token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
			return []byte(secret), nil
		})

		if err != nil || !token.Valid {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
			c.Abort()
			return
		}

		c.Next()
	}
}
```

---

## **7️⃣ `handlers.go` (CRUD Operations)**
```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
)

// Register - Create new user
func Register(c *gin.Context) {
	var user User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	hashedPassword, _ := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	user.Password = string(hashedPassword)

	DB.Create(&user)
	c.JSON(http.StatusOK, gin.H{"message": "User registered"})
}

// Login - Authenticate user and return JWT
func Login(c *gin.Context) {
	var req User
	if err := c.ShouldBindJSON(&req); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	var user User
	if err := DB.Where("username = ?", req.Username).First(&user).Error; err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	if err := bcrypt.CompareHashAndPassword([]byte(user.Password), []byte(req.Password)); err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	token, _ := GenerateJWT(user.Username)
	c.JSON(http.StatusOK, gin.H{"token": token})
}

// GetUsers - Get all users (Protected route)
func GetUsers(c *gin.Context) {
	var users []User
	DB.Find(&users)
	c.JSON(http.StatusOK, users)
}
```

---

## **8️⃣ `logger.go` (Logging with Logrus)**
```go
package main

import (
	"os"

	"github.com/sirupsen/logrus"
)

var log = logrus.New()

func InitLogger() {
	log.Out = os.Stdout
	log.SetFormatter(&logrus.JSONFormatter{})
	log.Info("Logger initialized")
}
```

---

## **9️⃣ `routes.go` (Define Routes)**
```go
package main

import "github.com/gin-gonic/gin"

func SetupRouter() *gin.Engine {
	router := gin.Default()

	router.POST("/register", Register)
	router.POST("/login", Login)

	protected := router.Group("/api")
	protected.Use(AuthMiddleware())
	{
		protected.GET("/users", GetUsers)
	}

	return router
}
```

---

## **🔟 `main.go` (Entry Point)**
```go
package main

import "fmt"

func main() {
	InitDB()
	InitLogger()

	router := SetupRouter()

	fmt.Println("Server running on port 8080...")
	router.Run(":8080")
}
```

---

## **1️⃣1️⃣ Testing the API**
### **Start the Server**
```sh
go run main.go
```

---

### **Register User**
```sh
curl -X POST http://localhost:8080/register -H "Content-Type: application/json" -d '{"username":"john", "password":"secret", "email":"john@example.com"}'
```

### **Login**
```sh
curl -X POST http://localhost:8080/login -H "Content-Type: application/json" -d '{"username":"john", "password":"secret"}'
```
✔ **Response** (Token):
```json
{"token": "eyJhbGciOiJIUzI1..."}
```

---

### **Fetch Users (Protected)**
```sh
curl -X GET http://localhost:8080/api/users -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

---

## **✅ Summary**
✔ **Gin** for REST API  
✔ **JWT Authentication**  
✔ **GORM (PostgreSQL)** for database  
✔ **Logrus** for logging  
✔ **Unit testing support**  
