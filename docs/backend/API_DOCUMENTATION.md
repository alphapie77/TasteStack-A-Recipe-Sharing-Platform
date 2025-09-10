# TasteStack Backend API Documentation

## Project Overview
**TasteStack** is a recipe sharing platform with Django REST Framework backend providing APIs for user management, recipe CRUD operations, and social interactions.

## Architecture Components

### 1. Technology Stack
- **Framework**: Django 5.2.1 with Django REST Framework 3.16.1
- **Database**: SQLite (development) / PostgreSQL (production)
- **Authentication**: JWT tokens using SimpleJWT
- **File Storage**: Local media files with Pillow for image processing
- **CORS**: django-cors-headers for frontend integration

### 2. Project Structure
```
backend/
├── tastestack/          # Main project settings
├── accounts/            # User authentication & profiles
├── recipes/             # Recipe management
├── interactions/        # Likes, comments, ratings
└── media/              # Uploaded files
```

## API Endpoints Overview

### Authentication APIs (`/api/auth/`)
- `POST /register/` - User registration
- `POST /login/` - User login
- `GET /user/` - Get current user profile
- `PUT /user/update/` - Update user profile
- `GET /dashboard-stats/` - User dashboard statistics
- `POST /follow/{user_id}/` - Follow/unfollow user

### Recipe APIs (`/api/recipes/`)
- `GET /` - List all recipes (with pagination)
- `POST /` - Create new recipe
- `GET /{id}/` - Get recipe details
- `PUT /{id}/` - Update recipe (author only)
- `DELETE /{id}/` - Delete recipe (author only)
- `GET /search/` - Advanced recipe search
- `POST /{id}/rate/` - Rate a recipe

### Interaction APIs (`/api/interactions/`)
- `POST /recipes/{id}/like/` - Like recipe
- `POST /recipes/{id}/unlike/` - Unlike recipe
- `GET /recipes/{id}/comments/` - Get recipe comments
- `POST /recipes/{id}/comments/add/` - Add comment

## Database Models

### User Model (Custom)
```python
class User(AbstractUser):
    email = models.EmailField(unique=True)           # Primary login field
    username = models.CharField(max_length=150)      # Display name
    first_name = models.CharField(max_length=150)    # Optional
    last_name = models.CharField(max_length=150)     # Optional
    bio = models.TextField(max_length=500)           # User description
    location = models.CharField(max_length=200)      # User location
    website = models.URLField()                      # Personal website
    profile_picture = models.ImageField()            # Profile image
    date_joined = models.DateTimeField()             # Registration date
```

### Recipe Model
```python
class Recipe(models.Model):
    title = models.CharField(max_length=200)         # Recipe name
    description = models.TextField()                 # Recipe description
    ingredients = models.JSONField()                 # List of ingredients
    instructions = models.JSONField()                # Step-by-step instructions
    prep_time = models.PositiveIntegerField()        # Preparation time (minutes)
    cook_time = models.PositiveIntegerField()        # Cooking time (minutes)
    servings = models.PositiveIntegerField()         # Number of servings
    difficulty = models.CharField()                  # Easy/Medium/Hard
    category = models.CharField()                    # Recipe categories
    image = models.ImageField()                      # Recipe image
    author = models.ForeignKey(User)                 # Recipe creator
    created_at = models.DateTimeField()              # Creation timestamp
    updated_at = models.DateTimeField()              # Last update timestamp
```

## Security Implementation

### 1. Authentication Security
- JWT tokens with expiration
- Password hashing using Django's built-in system
- Email-based login (more secure than username)

### 2. Authorization Controls
- Recipe CRUD: Only authors can edit/delete
- Comment moderation: Authors and recipe owners can hide/delete
- Profile updates: Users can only update their own profiles

### 3. Input Validation
- Serializer validation for all API inputs
- HTML escaping for comment content (XSS prevention)
- File upload validation for images only