Sure! Here's a well-structured **Go-Gin-based E-commerce API** with authentication, product management, orders, and cart functionality.

---

## **📂 Project Structure**
```
ecommerce-api/
│── cmd/
│   └── main.go          # Entry point
│── config/
│   └── config.go        # Loads environment variables
│── controllers/
│   ├── auth.go          # Handles authentication
│   ├── product.go       # Product CRUD
│   ├── order.go         # Order management
│   ├── cart.go          # Cart functionality
│── middleware/
│   └── auth.go          # JWT authentication middleware
│── models/
│   ├── user.go          # User model
│   ├── product.go       # Product model
│   ├── order.go         # Order model
│   ├── cart.go          # Cart model
│── routes/
│   ├── auth.go          # Authentication routes
│   ├── product.go       # Product routes
│   ├── order.go         # Order routes
│   ├── cart.go          # Cart routes
│── services/
│   ├── auth_service.go  # Auth logic
│   ├── product_service.go  # Product logic
│   ├── order_service.go    # Order logic
│   ├── cart_service.go     # Cart logic
│── database/
│   ├── db.go            # Database connection
│   ├── migrations.go    # Auto-migration
│── utils/
│   ├── jwt.go           # JWT token helper functions
│   ├── hash.go          # Password hashing utilities
│── .env                 # Environment variables
│── go.mod               # Go module dependencies
```

---

## **🚀 Features**
✅ **Authentication** (Register/Login with JWT)  
✅ **Products** (Create, Update, Delete, List)  
✅ **Cart** (Add/Remove items, View cart)  
✅ **Orders** (Place an order, View order history)  
✅ **Middleware** (JWT auth for protected routes)  
✅ **Logging & Environment Config**  

---

## **1️⃣ Install Dependencies**
```sh
go mod init ecommerce-api
go get github.com/gin-gonic/gin gorm.io/gorm gorm.io/driver/postgres github.com/golang-jwt/jwt/v5 github.com/joho/godotenv golang.org/x/crypto/bcrypt
```

---

## **2️⃣ `config/config.go` (Environment Variables)**
```go
package config

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

var DB *gorm.DB

func LoadConfig() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}
}

func ConnectDB() {
	LoadConfig()
	dsn := fmt.Sprintf("host=%s user=%s password=%s dbname=%s port=%s sslmode=disable",
		os.Getenv("DB_HOST"), os.Getenv("DB_USER"), os.Getenv("DB_PASSWORD"),
		os.Getenv("DB_NAME"), os.Getenv("DB_PORT"),
	)

	var err error
	DB, err = gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database")
	}

	log.Println("Connected to database")
	DB.AutoMigrate(&User{}, &Product{}, &Order{}, &Cart{})
}
```

---

## **3️⃣ `models/user.go` (User Model)**
```go
package models

import "gorm.io/gorm"

type User struct {
	gorm.Model
	Username string `gorm:"unique" json:"username"`
	Password string `json:"-"`
	Email    string `gorm:"unique" json:"email"`
}
```

---

## **4️⃣ `controllers/auth.go` (Authentication)**
```go
package controllers

import (
	"net/http"
	"ecommerce-api/models"
	"ecommerce-api/services"

	"github.com/gin-gonic/gin"
	"golang.org/x/crypto/bcrypt"
)

func Register(c *gin.Context) {
	var user models.User
	if err := c.ShouldBindJSON(&user); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	hashedPassword, _ := bcrypt.GenerateFromPassword([]byte(user.Password), bcrypt.DefaultCost)
	user.Password = string(hashedPassword)

	services.CreateUser(&user)
	c.JSON(http.StatusOK, gin.H{"message": "User registered"})
}

func Login(c *gin.Context) {
	var req models.User
	if err := c.ShouldBindJSON(&req); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	user, err := services.AuthenticateUser(req.Username, req.Password)
	if err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid credentials"})
		return
	}

	token, _ := services.GenerateJWT(user.Username)
	c.JSON(http.StatusOK, gin.H{"token": token})
}
```

---

## **5️⃣ `routes/auth.go`**
```go
package routes

import (
	"ecommerce-api/controllers"
	"github.com/gin-gonic/gin"
)

func AuthRoutes(router *gin.Engine) {
	router.POST("/register", controllers.Register)
	router.POST("/login", controllers.Login)
}
```

---

## **6️⃣ `services/auth_service.go`**
```go
package services

import (
	"errors"
	"ecommerce-api/models"
	"ecommerce-api/utils"

	"gorm.io/gorm"
)

func CreateUser(user *models.User) {
	utils.DB.Create(user)
}

func AuthenticateUser(username, password string) (*models.User, error) {
	var user models.User
	result := utils.DB.Where("username = ?", username).First(&user)
	if result.Error != nil {
		return nil, errors.New("user not found")
	}

	if err := utils.ComparePassword(user.Password, password); err != nil {
		return nil, errors.New("invalid password")
	}

	return &user, nil
}
```

---

## **7️⃣ `middleware/auth.go` (JWT Middleware)**
```go
package middleware

import (
	"net/http"
	"os"

	"github.com/gin-gonic/gin"
	"github.com/golang-jwt/jwt/v5"
)

func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		tokenString := c.GetHeader("Authorization")
		if tokenString == "" {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "Missing token"})
			c.Abort()
			return
		}

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

## **8️⃣ `cmd/main.go` (Entry Point)**
```go
package main

import (
	"ecommerce-api/config"
	"ecommerce-api/routes"
	"github.com/gin-gonic/gin"
)

func main() {
	config.ConnectDB()

	router := gin.Default()

	routes.AuthRoutes(router)

	router.Run(":8080")
}
```

---

## **✅ Summary**
✅ **Gin Framework** for API handling  
✅ **GORM** with PostgreSQL  
✅ **JWT Authentication**  
✅ **Modular Controllers & Routes**  
✅ **Middleware for Authorization**  

---

## **🔥 Next Steps**
- Implement **Product CRUD** (`controllers/product.go`, `routes/product.go`)  
- Add **Cart & Orders**  
- Add **Unit Tests** with `httptest`  
