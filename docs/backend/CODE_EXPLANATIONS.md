# Backend Code Explanations

## Key Files with Line-by-Line Comments

### 1. Settings Configuration (`tastestack/settings.py`)

```python
# Import required modules for Django configuration
from pathlib import Path
import os
from dotenv import load_dotenv  # Load environment variables from .env file

# Load environment variables from .env file for secure configuration
load_dotenv()

# Security: Get secret key from environment variable, fallback to default for development
SECRET_KEY = os.getenv('SECRET_KEY', 'django-insecure-fallback-key')

# Debug mode: True for development, False for production
DEBUG = os.getenv('DEBUG', 'True').lower() == 'true'

# Custom user model - extends Django's AbstractUser
AUTH_USER_MODEL = 'accounts.User'

# Django REST Framework configuration
REST_FRAMEWORK = {
    # Use JWT for authentication
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    # Require authentication by default
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    # Pagination settings
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 12,  # 12 items per page
}

# JWT token configuration
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),  # Token expires in 1 hour
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),     # Refresh token lasts 7 days
    'ALGORITHM': 'HS256',                            # Signing algorithm
    'AUTH_HEADER_TYPES': ('Bearer',),                # Authorization: Bearer <token>
}
```

### 2. User Model (`accounts/models.py`)

```python
class User(AbstractUser):
    # Custom username validator - allows spaces and more characters
    flexible_username_validator = RegexValidator(
        regex=r'^[\w\s.@+-]+$',  # Allow letters, numbers, spaces, @, ., +, -, _
        message='Enter a valid username. Letters, numbers, spaces, and @/./+/-/_ only.',
    )
    
    # Override username field with custom validator
    username = models.CharField(
        max_length=150,
        unique=True,                                    # Must be unique
        validators=[flexible_username_validator],       # Custom validation
    )
    
    # Email as primary login field
    email = models.EmailField(unique=True)              # Must be unique for login
    
    # Profile picture with upload path
    profile_picture = models.ImageField(
        upload_to='profile_pictures/',  # Uploaded to media/profile_pictures/
        blank=True, 
        null=True
    )
    
    # Authentication configuration
    USERNAME_FIELD = 'email'        # Use email for login instead of username
    REQUIRED_FIELDS = ['username']  # Required fields when creating superuser
```

### 3. Recipe Model (`recipes/models.py`)

```python
class Recipe(models.Model):
    # Basic recipe information
    title = models.CharField(max_length=200)            # Recipe name
    description = models.TextField()                    # Recipe description
    
    # JSON fields for flexible data structure
    ingredients = models.JSONField()                    # List of ingredients
    instructions = models.JSONField()                   # Step-by-step instructions
    
    # Relationship fields
    author = models.ForeignKey(
        User, 
        on_delete=models.CASCADE,       # Delete recipes when user is deleted
        related_name='recipes'          # Access user's recipes via user.recipes
    )
    
    # Computed properties using related models
    @property
    def average_rating(self):
        """Calculate average rating from all ratings"""
        ratings = self.ratings.all()                    # Get all related ratings
        if ratings.exists():
            return sum(rating.rating for rating in ratings) / ratings.count()
        return 0

    @property
    def likes_count(self):
        """Count total likes for this recipe"""
        return self.likes.count()                       # Count related likes
```

### 4. User Registration View (`accounts/views.py`)

```python
@api_view(['POST'])                     # Only accept POST requests
@permission_classes([AllowAny])         # Allow unauthenticated access
def register(request):
    # Validate required fields
    required_fields = ['username', 'email', 'password', 'password_confirm']
    missing_fields = [field for field in required_fields if field not in request.data]
    
    if missing_fields:
        return Response(
            {'error': f'Missing required fields: {", ".join(missing_fields)}'},
            status=status.HTTP_400_BAD_REQUEST
        )
    
    # Use serializer for validation and creation
    serializer = UserRegistrationSerializer(data=request.data)
    
    if serializer.is_valid():
        user = serializer.save()                    # Create user with hashed password
        refresh = RefreshToken.for_user(user)       # Generate JWT tokens
        return Response({
            'user': UserSerializer(user).data,      # Return user data
            'token': str(refresh.access_token),     # Return access token
        }, status=status.HTTP_201_CREATED)
    
    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### 5. Recipe Serializer (`recipes/serializers.py`)

```python
class RecipeSerializer(serializers.ModelSerializer):
    # Read-only nested serializer for author info
    author = UserSerializer(read_only=True)
    
    # Dynamic fields based on current user
    is_liked = serializers.SerializerMethodField()     # Custom method
    
    def get_is_liked(self, obj):
        """Check if current user has liked this recipe"""
        request = self.context.get('request')           # Get request from context
        if request and request.user.is_authenticated:
            # Check if like exists for current user and recipe
            return Like.objects.filter(user=request.user, recipe=obj).exists()
        return False
```