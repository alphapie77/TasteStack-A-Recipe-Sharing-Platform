# API Testing Guide

## Authentication Endpoints

### 1. User Registration
```http
POST /api/auth/register/
Content-Type: application/json

{
    "username": "testuser",
    "email": "test@example.com", 
    "password": "testpass123",
    "password_confirm": "testpass123"
}

Response (201):
{
    "user": {"id": 1, "username": "testuser", "email": "test@example.com"},
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
}
```

### 2. User Login
```http
POST /api/auth/login/
Content-Type: application/json

{
    "email": "test@example.com",
    "password": "testpass123"
}
```

## Recipe Endpoints

### 3. Create Recipe
```http
POST /api/recipes/
Authorization: Bearer <token>
Content-Type: multipart/form-data

title: Chocolate Cake
description: Delicious chocolate cake
ingredients: ["2 cups flour", "1 cup sugar", "3 eggs"]
instructions: ["Mix ingredients", "Bake at 350°F"]
prep_time: 30
cook_time: 30
servings: 8
difficulty: Medium
category: dessert
image: <file>
```

### 4. List Recipes
```http
GET /api/recipes/

Response (200):
{
    "count": 10,
    "results": [
        {
            "id": 1,
            "title": "Chocolate Cake",
            "author": {"username": "testuser"},
            "average_rating": 4.5,
            "likes_count": 12
        }
    ]
}
```

### 5. Search Recipes
```http
GET /api/recipes/search/?q=chocolate&category=dessert&difficulty=Easy&max_time=60
```

## Interaction Endpoints

### 6. Like Recipe
```http
POST /api/interactions/recipes/1/like/
Authorization: Bearer <token>

Response (201):
{"message": "Recipe liked successfully"}
```

### 7. Add Comment
```http
POST /api/interactions/recipes/1/comments/add/
Authorization: Bearer <token>
Content-Type: application/json

{
    "content": "This recipe is amazing!"
}
```

## Testing with cURL

### Register User
```bash
curl -X POST http://localhost:8000/api/auth/register/ \
  -H "Content-Type: application/json" \
  -d '{"username": "testuser", "email": "test@example.com", "password": "testpass123", "password_confirm": "testpass123"}'
```

### Create Recipe
```bash
curl -X POST http://localhost:8000/api/recipes/ \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "title=Test Recipe" \
  -F "description=A test recipe" \
  -F 'ingredients=["flour", "sugar"]' \
  -F "prep_time=15" \
  -F "cook_time=30"
```