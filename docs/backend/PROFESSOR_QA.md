# Professor Q&A - Backend Implementation

## Technical Questions & Answers

### Q1: "Explain your backend architecture"
**Answer**: My backend follows Django's MVT pattern with REST API layer:
- **Models**: Define database structure (User, Recipe, Like, Comment)
- **Views**: Handle HTTP requests and business logic using DRF
- **Serializers**: Convert between JSON and Python objects
- **URLs**: Route requests to appropriate views

### Q2: "How do you handle authentication?"
**Answer**: I use JWT token-based authentication:
```python
# Login process:
1. User sends email/password
2. Server validates and generates JWT token
3. Client stores token, sends in Authorization header
4. Server validates token for protected endpoints

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
}
```

### Q3: "Describe your database relationships"
**Answer**: 
- **User → Recipe**: One-to-Many (author relationship)
- **User ↔ Recipe**: Many-to-Many via Like, Rating, Comment
- **User ↔ User**: Many-to-Many via Follow (self-referencing)

### Q4: "How do you validate user input?"
**Answer**: Multi-layer validation:
```python
class UserRegistrationSerializer(serializers.ModelSerializer):
    def validate_password(self, value):
        if len(value) < 8:
            raise ValidationError("Password too short")
        return value
    
    def validate_email(self, value):
        if User.objects.filter(email=value).exists():
            raise ValidationError("Email already exists")
        return value
```

### Q5: "Explain your API design principles"
**Answer**: I follow REST conventions:
- Resource-based URLs: `/api/recipes/`, `/api/recipes/1/`
- HTTP methods: GET (read), POST (create), PUT (update), DELETE (remove)
- Status codes: 200 (OK), 201 (Created), 400 (Bad Request), 404 (Not Found)
- JSON content negotiation

### Q6: "How do you handle file uploads securely?"
**Answer**: 
```python
# Model field with secure upload path
profile_picture = models.ImageField(upload_to='profile_pictures/')

# Security measures:
1. File type validation via Pillow (images only)
2. Django handles secure file naming
3. Absolute URLs in API responses
```

### Q7: "Describe your search implementation"
**Answer**: Multi-field search with filters:
```python
def search_recipes(request):
    query = request.GET.get('q', '')
    recipes = Recipe.objects.all()
    
    if query:
        words = query.lower().split()
        for word in words:
            recipes = recipes.filter(
                Q(title__icontains=word) |
                Q(description__icontains=word) |
                Q(ingredients__icontains=word)
            )
```

### Q8: "How do you optimize database performance?"
**Answer**: 
```python
# Avoid N+1 queries
recipes = Recipe.objects.select_related('author').all()

# Use pagination
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'PageNumberPagination',
    'PAGE_SIZE': 12
}

# Database indexes
class Recipe(models.Model):
    title = models.CharField(db_index=True)
```

### Q9: "Explain your error handling strategy"
**Answer**: Consistent error responses:
```json
{
    "error": "General error message",
    "field_errors": {
        "email": ["This field is required"],
        "password": ["Password too short"]
    }
}
```

### Q10: "How would you deploy this to production?"
**Answer**: 
- **Environment**: Use environment variables for secrets
- **Database**: PostgreSQL with connection pooling
- **Web Server**: Gunicorn + Nginx
- **Security**: HTTPS, proper CORS configuration
- **Monitoring**: Logging and error tracking