<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Connect the React Frontend</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to configure CORS, connect the Vite environment variable to Django, and complete an authenticated request from React.

## Configure CORS permission to test with the Hoot frontend

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

In the root of our React Hoot frontend, change `.env`:

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

## Complete the full user flow

1. Open the React URL shown by Vite.
2. Sign up with a username and password.
3. Confirm that the dashboard appears and lists the new user.
4.  Sign out, sign back in, and confirm the data remains in PostgreSQL.






> **Confirm the user was saved in PostgreSQL**
>
> In a new terminal, connect to the Hoot database:
>
> ```bash
> psql hoot_api
> ```
>
> Then view the users:
>
> ```sql
> SELECT id, username FROM auth_user;
> ```
>
> Confirm that the username you signed up with appears in the results.
> ```text
> id | username
> ----+----------
>  1 | sam
> ```
>
> Exit PostgreSQL with:
>
> ```sql
> \q
> ```
