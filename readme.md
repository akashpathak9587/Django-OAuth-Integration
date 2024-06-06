# Django OAuth2 Project

This guide will help you set up OAuth2 authentication in a Django project using `djangorestframework` and `django-oauth-toolkit`.

## Steps

### 1. Install Django and Create a New Project

```bash
pip install django
django-admin startproject django_oauth2
cd django_oauth2
django-admin startapp oauth_test
```

### 2. Install Required Packages

```bash
pip install djangorestframework django-oauth-toolkit
```

### 3. Update `settings.py`

Add the following apps to the `INSTALLED_APPS` section:

```python
INSTALLED_APPS = [
    ...
    'oauth_test',
    'oauth2_provider',
    'rest_framework',
]
```

Add the available scopes:

```python
OAUTH2_PROVIDER = {
    'SCOPES': {
        'read': 'Read scope',
        'write': 'Write scope',
        'groups': 'Access to your groups'
    }
}
```

Add authentication classes to the REST framework:

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'oauth2_provider.contrib.rest_framework.OAuth2Authentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
}
```

### 4. Create `serializers.py` in `oauth_test` app

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('username', 'email', 'first_name', 'last_name')

class GroupSerializer(serializers.ModelSerializer):
    class Meta:
        model = Group
        fields = ('name',)
```

### 5. Create `views.py` in `oauth_test` app

```python
from django.contrib.auth.models import User, Group
from oauth2_provider.contrib.rest_framework import TokenHasReadWriteScope, TokenHasScope
from rest_framework import generics, permissions

from oauth_test.serializers import UserSerializer, GroupSerializer

class UserList(generics.ListCreateAPIView):
    permission_classes = [permissions.IsAuthenticated, TokenHasReadWriteScope]
    queryset = User.objects.all()
    serializer_class = UserSerializer

class UserDetails(generics.RetrieveAPIView):
    permission_classes = [permissions.IsAuthenticated, TokenHasReadWriteScope]
    queryset = User.objects.all()
    serializer_class = UserSerializer

class GroupList(generics.ListAPIView):
    permission_classes = [permissions.IsAuthenticated, TokenHasScope]
    required_scopes = ['groups']
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```

### 6. Update `urls.py` in `django_oauth2` project

```python
from django.contrib import admin
from django.urls import path, include

from oauth_test.views import UserList, UserDetails, GroupList

urlpatterns = [
    path('admin/', admin.site.urls),
    path('o/', include('oauth2_provider.urls', namespace='oauth2_provider')),
    path('users/', UserList.as_view()),
    path('users/<pk>/', UserDetails.as_view()),
    path('groups/', GroupList.as_view()),
]
```

### 7. Migrate Changes

```bash
python manage.py makemigrations
python manage.py migrate
```

### 8. Create a Superuser

```bash
python manage.py createsuperuser
```

### 9. Run the Server

```bash
python manage.py runserver
```

### 10. Register an App for OAuth

Go to [http://localhost:8000/admin/](http://localhost:8000/admin/) and log in with your superuser credentials. Then, navigate to [http://localhost:8000/o/applications/](http://localhost:8000/o/applications/) to register a new application with the following details:

- **Name:** Your Application Name
- **Client type:** Confidential
- **Authorization grant type:** Resource owner password-based

### 11. Generate OAuth Token

Open Postman and send a POST request to [http://localhost:8000/o/token/](http://localhost:8000/o/token/) with the following details:

- **Authorization tab:** Provide the client ID and client secret of your app.
- **Body (form-data):**
  - `grant_type`: `password`
  - `username`: Your superuser username
  - `password`: Your superuser password

### 12. Access Resources

Use the generated access token to access resources. In Postman, select "Bearer Token" in the Authorization tab and paste your access token. Now you can send requests to endpoints like `/users/` and `/groups/`.

### Example Token Response

```json
{
  "access_token": "VQVXQNlQFKmZPABygyDH8Wir8fwp0f",
  "expires_in": 36000,
  "token_type": "Bearer",
  "scope": "read write groups",
  "refresh_token": "oCrgBS6lDgqZcC2hXKdM2zh0jhd3MM"
}
```

With this token, you can access the user and group information provided by your API. Make sure to include the token in the Authorization header for every request.
