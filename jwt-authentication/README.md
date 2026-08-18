<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">JWT Authentication</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to issue a frontend-compatible JWT during sign-up/sign-in and use it to authenticate protected API requests.

<!-- ## Begin Day 2

Activate the environment and verify yesterday's work:

macOS:

```bash
source .venv/bin/activate
```

Windows Git Bash:

```bash
source .venv/Scripts/activate
```

```bash
python manage.py check
python manage.py showmigrations api
``` -->

## Review the frontend's auth flow

After sign-up or sign-in, the React service expects:

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
    eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSCI6MTUxNjIzOTAyMn0.
    KMUFsIDTnFmyG3nMiGM6H9FNFUROf3wh7SmqJp-QV30"
}
```

React saves the token in `localStorage`. Later services send it in a request header:

```js
Authorization: `Bearer ${localStorage.getItem('token')}`
```

React also decodes the middle part of the JWT and reads a `payload` object:

```json
{
  "payload": {
    "_id": "2",
    "username": "sam"
  }
}
```

Our token must both authenticate requests and include that exact frontend data.

> A JWT is signed, not encrypted. Do not place passwords or other secret user data inside it.

## Configure DRF authentication

At the top of `hoot_api/settings.py`, add the `timedelta` import:

```python
import os
from pathlib import Path
from datetime import timedelta
```

Add these settings near the bottom:

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(days=7),
}
```

The defaults now say:

1. Look for a valid JWT in a Bearer header.
2. Require an authenticated user for every DRF view unless a view explicitly opts out.

## Create the token helper

Replace the generated imports in `api/views.py` with:

```python
from django.contrib.auth import authenticate
from django.contrib.auth.models import User
from rest_framework import status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import AllowAny
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken

from .serializers import UserSerializer
```

Add this helper:

```python
def create_access_token(user):
    token = RefreshToken.for_user(user).access_token
    token["payload"] = {
        "_id": str(user.id),
        "username": user.username,
    }
    return str(token)
```

Simple JWT creates an access token that its authentication class knows how to verify. We add only the two safe fields React needs.

Why convert the ID with `str()`? Route parameters and MongoDB IDs were strings in the earlier frontend. Using strings preserves strict comparisons such as:

```javascript
hoot.author._id === user._id
```

## Build sign-up

Add the first view function:

```python
@api_view(["POST"])
@permission_classes([AllowAny])
def sign_up(request):
    username = request.data.get("username", "").strip()
    password = request.data.get("password", "")
    confirm_password = request.data.get("confirmPassword", "")

    if not username or not password:
        return Response(
            {"err": "Username and password are required."},
            status=status.HTTP_400_BAD_REQUEST,
        )

    if password != confirm_password:
        return Response(
            {"err": "Passwords do not match."},
            status=status.HTTP_400_BAD_REQUEST,
        )

    if User.objects.filter(username=username).exists():
        return Response(
            {"err": "That username is already taken."},
            status=status.HTTP_400_BAD_REQUEST,
        )

    user = User.objects.create_user(username=username, password=password)
    token = create_access_token(user)

    return Response({"token": token}, status=status.HTTP_201_CREATED)
```

### Read the request logic

- `request.data` is DRF's parsed request body.
- `.get("username", "")` safely returns an empty string if a key is missing.
- `User.objects.filter(...).exists()` checks whether the username is taken.
- `create_user()` hashes the password before saving. Never create a user with `User.objects.create(...)` and a plain password.
- Error responses use the key `err` because the supplied frontend checks `data.err`.

`AllowAny` overrides the global authenticated-only rule for this view. A new user cannot already have a token.

## Build sign-in

```python
@api_view(["POST"])
@permission_classes([AllowAny])
def sign_in(request):
    username = request.data.get("username", "")
    password = request.data.get("password", "")
    user = authenticate(username=username, password=password)

    if user is None:
        return Response(
            {"err": "Invalid username or password."},
            status=status.HTTP_401_UNAUTHORIZED,
        )

    token = create_access_token(user)
    return Response({"token": token})
```

`authenticate()` checks the submitted password against Django's stored password hash. It returns the user when the credentials are valid and `None` when they are not.

## Build the dashboard's user endpoint

The signed-in dashboard also requests a user list:

```python
@api_view(["GET"])
def user_list(request):
    users = User.objects.all().order_by("username")
    serializer = UserSerializer(users, many=True)
    return Response(serializer.data)
```

This view has no `AllowAny` decorator, so the global permission requires a valid token. `many=True` tells the serializer it received a collection.

## Create app URLs

Create `api/urls.py`:

```python
from django.urls import path

from . import views


urlpatterns = [
    path("auth/sign-up", views.sign_up, name="sign-up"),
    path("auth/sign-in", views.sign_in, name="sign-in"),
    path("users", views.user_list, name="user-list"),
]
```

Do not add trailing slashes to these paths. The existing frontend requests the exact URLs shown.

Open `hoot_api/urls.py` and include the app URLs at the root:

```python
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("api.urls")),
]
```

## Test sign-up in Postman

Make a `POST` request to:

```plaintext
http://localhost:8000/auth/sign-up
```

Choose raw JSON and send:

```json
{
  "username": "sam",
  "password": "test1234",
  "confirmPassword": "test1234"
}
```

Expect status `201` and a token. Copy the token for the next request.

## Test the protected user list

Make a `GET` request to:

```plaintext
http://localhost:8000/users
```

Without a token, expect `401 Unauthorized`.

Then add an Authorization header using Postman's **Bearer Token** option. Paste the token only—the UI adds the word `Bearer`. Expect an array similar to:

```json
[
  {
    "_id": "1",
    "username": "sam"
  }
]
```

