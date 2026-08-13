<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Connect the React Frontend</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to configure CORS, connect the Vite environment variable to Django, and complete an authenticated request from React.

## Why the browser needs CORS permission

During development, the two applications use different origins:

```plaintext
React:  http://localhost:5173
Django: http://localhost:8000
```

An origin includes the protocol, host, and port. Because the ports differ, the browser treats these as different origins and requires the backend to explicitly allow the frontend.

CORS is a browser rule. Postman is not a browser, so a request can work in Postman while still being blocked in React.

## Confirm the CORS package setup

`hoot_api/settings.py` should include:

```python
INSTALLED_APPS = [
    # Django's generated apps
    "corsheaders",
    "rest_framework",
    "api",
]
```

Its middleware should place CORS before common middleware:

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    # remaining generated middleware
]
```

Add the allowed frontend origins near the bottom of settings:

```python
CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
    "http://127.0.0.1:5173",
]
```

Do not use `CORS_ALLOW_ALL_ORIGINS = True` for this project. Listing the two known development origins makes the rule visible and avoids accidentally allowing every website.

## Configure the React base URL

In the root of the supplied React frontend, create `.env`:

```plaintext
VITE_BACK_END_SERVER_URL=http://localhost:8000
```

Do not add a slash at the end. The existing service files add paths beginning with `/`.

Vite reads `.env` when its development server starts. Stop and restart the React server after creating or changing the file.

## Run both applications

Use two terminal windows.

### Backend terminal

```bash
cd hoot-backend
source .venv/bin/activate
python manage.py runserver
```

Windows Git Bash uses:

```bash
source .venv/Scripts/activate
```

### Frontend terminal

```bash
cd hoot-frontend-solution
npm install
npm run dev
```

Keep both terminals visible. Each request should appear in the Django terminal with its method, path, and status code.

## Complete the first browser flow

1. Open the React URL shown by Vite.
2. Sign up with a username and password.
3. Confirm that the dashboard appears and lists the new user.
4. Open the Hoot list.
5. Create a Hoot.
6. Open its details.
7. Add a comment.
8. Edit the Hoot and comment.
9. Delete the comment and Hoot.
10. Sign out, sign back in, and confirm the data remains in PostgreSQL.

## Compare the network request

Open the browser developer tools and select the Network tab. Click one request to `/hoots` and identify:

- **Request URL:** `http://localhost:8000/hoots`
- **Request method:** `GET` or `POST`
- **Authorization header:** begins with `Bearer`
- **Response status:** normally `200` or `201`
- **Response data:** uses `_id`, `createdAt`, and nested author objects

This view connects the code in the React service to the code in the Django view.

## Diagnose by status code

| Result | Meaning | Check first |
| --- | --- | --- |
| Browser says CORS | Django did not allow this frontend origin | Exact React port and CORS middleware order |
| `401 Unauthorized` | The request is not authenticated | Token exists and header begins with `Bearer`, followed by a space |
| `403 Forbidden` | The user is signed in but does not own the content | Compare `request.user` with the record's author |
| `404 Not Found` | No URL or record matched | Request path, captured IDs, and Django terminal |
| `400 Bad Request` | Submitted data failed validation | Response body and serializer errors |
| `500 Internal Server Error` | Backend code raised an exception | Read the bottom of the Django traceback first |

## Do not change several layers at once

When something fails:

1. Reproduce the same request in Postman.
2. If Postman also fails, inspect URLs, views, serializers, models, and the Django traceback.
3. If Postman works but React fails, inspect the frontend base URL, CORS, Bearer token, request body, and browser network response.

Changing both applications before retesting makes it harder to know which change solved or caused the problem.

## Checkpoint

- [ ] Both servers run at the same time.
- [ ] React's `.env` contains the Django origin with no trailing slash.
- [ ] Django explicitly allows both local Vite origins.
- [ ] A browser request reaches Django without a CORS error.
- [ ] Authenticated services send the Bearer token.
- [ ] The complete CRUD and comment flow works in React.
