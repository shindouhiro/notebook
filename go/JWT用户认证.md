# Go JWT 用户认证

> 🔗 **关联笔记**：[[留言系统项目实战]]
> 📅 **创建时间**：2024-12-24

---

## 📖 概述

JWT (JSON Web Token) 是一种无状态的认证方式，适合 RESTful API 的身份验证。

---

## 📦 安装依赖

```bash
go get -u github.com/golang-jwt/jwt/v5
```

---

## 🛠️ 实现步骤

### 1. 定义用户模型

```go
// models/user.go
package models

import (
	"golang.org/x/crypto/bcrypt"
	"gorm.io/gorm"
)

// User 用户模型
type User struct {
	ID       uint   `json:"id" gorm:"primaryKey"`
	Username string `json:"username" gorm:"size:50;uniqueIndex;not null"`
	Password string `json:"-" gorm:"size:255;not null"` // json:"-" 不返回密码
	Email    string `json:"email" gorm:"size:100"`
	gorm.Model
}

// SetPassword 设置密码 (加密)
func (u *User) SetPassword(password string) error {
	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
	if err != nil {
		return err
	}
	u.Password = string(hashedPassword)
	return nil
}

// CheckPassword 验证密码
func (u *User) CheckPassword(password string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(u.Password), []byte(password))
	return err == nil
}
```

### 2. JWT 工具类

```go
// utils/jwt.go
package utils

import (
	"errors"
	"time"

	"github.com/golang-jwt/jwt/v5"
)

var jwtSecret = []byte("your-secret-key-here") // 生产环境请使用环境变量

// Claims 自定义 JWT Claims
type Claims struct {
	UserID   uint   `json:"user_id"`
	Username string `json:"username"`
	jwt.RegisteredClaims
}

// GenerateToken 生成 JWT Token
func GenerateToken(userID uint, username string) (string, error) {
	claims := Claims{
		UserID:   userID,
		Username: username,
		RegisteredClaims: jwt.RegisteredClaims{
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(24 * time.Hour)), // 24小时过期
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			NotBefore: jwt.NewNumericDate(time.Now()),
			Issuer:    "message-board",
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString(jwtSecret)
}

// ParseToken 解析 JWT Token
func ParseToken(tokenString string) (*Claims, error) {
	token, err := jwt.ParseWithClaims(tokenString, &Claims{}, func(token *jwt.Token) (interface{}, error) {
		return jwtSecret, nil
	})

	if err != nil {
		return nil, err
	}

	if claims, ok := token.Claims.(*Claims); ok && token.Valid {
		return claims, nil
	}

	return nil, errors.New("invalid token")
}
```

### 3. 认证中间件

```go
// middleware/auth.go
package middleware

import (
	"message-board/utils"
	"net/http"
	"strings"

	"github.com/gin-gonic/gin"
)

// AuthMiddleware JWT 认证中间件
func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 获取 Authorization Header
		authHeader := c.GetHeader("Authorization")
		if authHeader == "" {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "未提供认证信息"})
			c.Abort()
			return
		}

		// 检查 Bearer 格式
		parts := strings.SplitN(authHeader, " ", 2)
		if len(parts) != 2 || parts[0] != "Bearer" {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "认证格式错误"})
			c.Abort()
			return
		}

		// 解析 Token
		claims, err := utils.ParseToken(parts[1])
		if err != nil {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "无效的 Token"})
			c.Abort()
			return
		}

		// 将用户信息存入上下文
		c.Set("userID", claims.UserID)
		c.Set("username", claims.Username)
		c.Next()
	}
}
```

### 4. 登录/注册处理器

```go
// handlers/auth.go
package handlers

import (
	"message-board/config"
	"message-board/models"
	"message-board/utils"
	"net/http"

	"github.com/gin-gonic/gin"
)

// RegisterInput 注册输入
type RegisterInput struct {
	Username string `json:"username" binding:"required,min=3,max=50"`
	Password string `json:"password" binding:"required,min=6"`
	Email    string `json:"email"`
}

// LoginInput 登录输入
type LoginInput struct {
	Username string `json:"username" binding:"required"`
	Password string `json:"password" binding:"required"`
}

// Register 用户注册
func Register(c *gin.Context) {
	var input RegisterInput
	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	// 检查用户名是否已存在
	var existingUser models.User
	if config.DB.Where("username = ?", input.Username).First(&existingUser).Error == nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": "用户名已存在"})
		return
	}

	// 创建用户
	user := models.User{
		Username: input.Username,
		Email:    input.Email,
	}
	user.SetPassword(input.Password)

	if err := config.DB.Create(&user).Error; err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "注册失败"})
		return
	}

	c.JSON(http.StatusCreated, gin.H{
		"message": "注册成功",
		"user":    user,
	})
}

// Login 用户登录
func Login(c *gin.Context) {
	var input LoginInput
	if err := c.ShouldBindJSON(&input); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}

	// 查找用户
	var user models.User
	if err := config.DB.Where("username = ?", input.Username).First(&user).Error; err != nil {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "用户名或密码错误"})
		return
	}

	// 验证密码
	if !user.CheckPassword(input.Password) {
		c.JSON(http.StatusUnauthorized, gin.H{"error": "用户名或密码错误"})
		return
	}

	// 生成 Token
	token, err := utils.GenerateToken(user.ID, user.Username)
	if err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": "生成 Token 失败"})
		return
	}

	c.JSON(http.StatusOK, gin.H{
		"message": "登录成功",
		"token":   token,
		"user":    user,
	})
}

// GetProfile 获取当前用户信息
func GetProfile(c *gin.Context) {
	userID := c.GetUint("userID")
	
	var user models.User
	if err := config.DB.First(&user, userID).Error; err != nil {
		c.JSON(http.StatusNotFound, gin.H{"error": "用户不存在"})
		return
	}

	c.JSON(http.StatusOK, gin.H{"user": user})
}
```

### 5. 配置路由

```go
// routes/routes.go
package routes

import (
	"message-board/handlers"
	"message-board/middleware"

	"github.com/gin-gonic/gin"
)

func SetupRoutes(r *gin.Engine) {
	api := r.Group("/api")
	{
		// 公开路由 (无需认证)
		api.POST("/register", handlers.Register)
		api.POST("/login", handlers.Login)

		// 受保护路由 (需要认证)
		protected := api.Group("/")
		protected.Use(middleware.AuthMiddleware())
		{
			protected.GET("/profile", handlers.GetProfile)
			
			// 留言相关 (需要登录)
			messages := protected.Group("/messages")
			{
				messages.POST("", handlers.CreateMessage)
				messages.PUT("/:id", handlers.UpdateMessage)
				messages.DELETE("/:id", handlers.DeleteMessage)
			}
		}
		
		// 留言列表 (公开)
		api.GET("/messages", handlers.GetMessages)
		api.GET("/messages/:id", handlers.GetMessage)
	}
}
```

---

## 🧪 测试示例

```bash
# 1. 注册用户
curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "123456", "email": "test@example.com"}'

# 2. 登录获取 Token
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "password": "123456"}'

# 3. 使用 Token 访问受保护接口
curl http://localhost:8080/api/profile \
  -H "Authorization: Bearer <your-token-here>"

# 4. 使用 Token 创建留言
curl -X POST http://localhost:8080/api/messages \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-token-here>" \
  -d '{"content": "这是一条需要登录才能发的留言"}'
```

---

## 🔐 安全建议

1. **密钥管理**：使用环境变量存储 JWT 密钥
2. **Token 过期**：设置合理的过期时间
3. **HTTPS**：生产环境必须使用 HTTPS
4. **刷新 Token**：实现 Refresh Token 机制

---

## 📚 相关笔记

- [[留言系统项目实战]] - 主项目
- [[文件上传实现]] - 文件上传

---

#Go #JWT #认证 #安全 #后端开发
