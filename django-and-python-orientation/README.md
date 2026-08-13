<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Django and Python Orientation</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to explain how a React request moves through a Django REST API and read the Python syntax used in this project.

## The application we are building

The React frontend is already complete. Its service functions send HTTP requests and expect JSON responses. Our job is to build the server that receives those requests.

One request will move through the application like this:

1. A React service calls `fetch('http://localhost:8000/hoots')`. (already built)
2. Django's `urls.py` matches the `/hoots` path.
3. A view function decides what to do based on the HTTP method.
4. A model reads from or writes to PostgreSQL.
5. A serializer changes model data into JSON-shaped data.
6. Django REST Framework sends an HTTP response back to React.

The frontend and backend are two separate applications. They can use different programming languages because they communicate using HTTP and JSON.

## Translate the familiar concepts

| Express/Mongoose | Django/DRF | Job |
| --- | --- | --- |
| `server.js` configurations | `settings.py` | Configure the application |
| `server.js` Express routes | Django URL pattern | Match a request path |
| Express Controller function | Django View function | Run request logic |
| `req` | `request` | Hold incoming request data |
| `req.body` | `request.data` | Hold parsed JSON from the body |
| `res.json(data)` | `Response(data)` | Send JSON-shaped data |
| Mongoose schema/model | Django model | Describe and query stored data |
| Mongoose document | Django model instance | Represent one database record |
| `populate()` | Nested serializer | Include related data in a response |
| Middleware | Middleware/authentication/permissions | Run checks around a request |

The names differ, but the responsibilities are familiar.

## Django, Django REST Framework, and PostgreSQL

- **Django** supplies the project structure, models, database tools, users, passwords, and admin site.
- **Django REST Framework (DRF)** adds tools for API requests, JSON responses, serializers, status codes, and permissions.
- **PostgreSQL** stores the data.

Django can render HTML templates, but our React application already handles the interface. This backend will return data instead of pages.

## A Django project contains apps

We will create:

- A Django **project** named `hoot_api`. It holds settings and the root URL file.
- A Django **app** named `api`. It holds the Hoot-specific models, serializers, views, and URLs.

Here, “app” does not mean a separate deployed application. It means one organized feature area inside a Django project.

## The Python we need today

We do not need all of Python before starting Django. We need to recognize a small group of patterns.

### Variables, strings, booleans, and null values

| JavaScript | Python |
| --- | --- |
| `const title = 'Hello'` | `title = 'Hello'` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |
| `console.log(title)` | `print(title)` |

Python does not use `const`, `let`, or semicolons.

### Lists and dictionaries

Python **lists** are similar to JavaScript arrays. Python **dictionaries** are similar to JavaScript objects.

```python
categories = ["News", "Sports", "Games"]

user = {
    "_id": "1",
    "username": "sam",
}
```

### Functions and indentation

```python
def greet(username):
    message = f"Welcome, {username}!"
    return message
```

- `def` begins a function definition.
- A colon starts the function's indented block.
- Indentation is part of Python syntax, not decoration.
- An f-string inserts values into a string, similar to a JavaScript template literal.

The closest JavaScript version is:

```javascript
const greet = (username) => {
  const message = `Welcome, ${username}!`
  return message
}
```

Use four spaces for each Python indentation level. Do not mix tabs and spaces.

### Conditions

```python
if request.method == "GET":
    return Response({"message": "Reading data"})

if request.method == "POST":
    return Response({"message": "Creating data"})
```

Python uses `and`, `or`, and `not` instead of `&&`, `||`, and `!`.

### Imports

```python
from rest_framework.response import Response
from .models import Hoot
```

The first line imports `Response` from an installed package. The dot in `.models` means “the `models.py` file in this same app.”

### Classes

```python
class Hoot(models.Model):
    title = models.CharField(max_length=100)
```

A Django model is a Python class. `Hoot` inherits useful database behavior from `models.Model`.

### Decorators

```python
@api_view(["GET", "POST"])
def hoot_list_create(request):
    # request logic will go here
```

A line beginning with `@` is a **decorator**. It adds behavior to the function immediately below it. This decorator tells DRF which HTTP methods the view accepts and prepares the DRF request/response features.

A Python **decorator is a function that accepts another function as an argument and returns a function**. The `@` syntax is mostly convenient syntax for doing that.

The biggest adjustment from JavaScript is that instead of explicitly writing:

```js
someFunction(callback)
```

Python gives you syntax that says:

```python
@some_function
def callback():
    ...
```

## This is very similar to Express middleware

Suppose Express has:

```js
app.get('/profile', isSignedIn, showProfile)
```

Conceptually:

```text
request
   ↓
isSignedIn
   ↓
showProfile
```

A Python framework might express a similar idea with a decorator:

```python
@login_required
def profile(request):
    ...
```

Conceptually:

```text
request
   ↓
login_required
   ↓
profile
```

They're **not exactly the same mechanism**, but they solve a similar architectural problem: adding behavior around request-handling functions.

## Django uses decorators like this

You may encounter:

```python
from django.contrib.auth.decorators import login_required

@login_required
def profile(request):
    ...
```

Mentally translate that to:

```python
profile = login_required(profile)
```

`login_required` receives the `profile` function and gives us back a wrapped version that checks authentication before allowing the original view to run.

So rather than putting this logic into every view:

```python
def profile(request):
    if not request.user.is_authenticated:
        # redirect

    # actual profile logic
```

we can separate the authentication concern:

```python
@login_required
def profile(request):
    # actual profile logic
```

## There's one slightly more confusing version

You'll also see decorators with parentheses:

```python
@api_view(["GET", "POST"])
def hoot_list(request):
    ...
```

This looks like we're passing `["GET", "POST"]` instead of passing the function.

That's because there's actually **another layer**.  If you want to learn more about decorators, check out [this documentation](https://wiki.python.org/moin/PythonDecorators#What_is_a_Decorator) - otherwise, just remember that they give extra functionality to the functions defined below them.

> "`@api_view(["GET"])` wraps this function and adds DRF's API-view behavior to it. It also says this view accepts GET requests."


## Read the frontend contract

Before building an API, inspect its client. The supplied React services (from our Hoot Frontend application) expect these paths:

```plaintext
POST   /auth/sign-up
POST   /auth/sign-in
GET    /users
GET    /hoots
POST   /hoots
GET    /hoots/:hootId
PUT    /hoots/:hootId
DELETE /hoots/:hootId
POST   /hoots/:hootId/comments
PUT    /hoots/:hootId/comments/:commentId
DELETE /hoots/:hootId/comments/:commentId
```

They also expect IDs named `_id` and dates named `createdAt`. Django normally uses `id` and our Python code will use `created_at`. Later, serializers will translate those names so we do not need to rewrite the frontend.

