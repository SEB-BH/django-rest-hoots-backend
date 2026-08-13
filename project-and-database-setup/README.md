<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Project and Database Setup</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to create and verify a Django REST Framework project connected to PostgreSQL.

## Verify the prerequisites

Python and PostgreSQL should already be installed. Check both before creating the project:

```bash
python3 --version
psql --version
createdb --version
```

Windows students may need to use `python --version` instead of `python3 --version`.

Do not spend core lesson time installing PostgreSQL. If one of these commands fails, use the installfest instructions or work with an instructor.

## Create the project folder

```bash
mkdir hoot-backend
cd hoot-backend
```

Open the folder in VS Code:

```bash
code .
```

## Create a virtual environment

macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Windows Git Bash:

```bash
python -m venv .venv
source .venv/Scripts/activate
```

The virtual environment is a project-local Python installation. Packages installed while it is active go inside `.venv` instead of being mixed with packages from other projects.

Your terminal prompt should now begin with `(.venv)`. Once the environment is active, the command `python` should use the environment on both operating systems.

## Add project dependencies

Create `requirements.txt`:

```plaintext
Django>=5.2,<5.3
djangorestframework>=3.16,<4
django-cors-headers>=4,<5
djangorestframework-simplejwt>=5.5,<6
psycopg[binary]>=3.2,<4
python-dotenv>=1,<2
```

Install them:

```bash
pip install -r requirements.txt
```

| Package | Purpose |
| --- | --- |
| Django | Project structure, ORM, users, passwords, migrations, and admin |
| Django REST Framework | API requests, serializers, responses, and permissions |
| django-cors-headers | Allow the separate React development server to call Django |
| Simple JWT | Read and create Bearer tokens |
| psycopg | Connect Python/Django to PostgreSQL |
| python-dotenv | Load local configuration from `.env` |

## Add `.gitignore`

```plaintext
.DS_Store
.venv/
__pycache__/
*.py[cod]
.env
db.sqlite3
```

The virtual environment and `.env` belong to each developer's computer and should not be committed.

## Create the PostgreSQL database

```bash
createdb hoot_api
```

If PostgreSQL requires the default EDB user, use:

```bash
createdb -U postgres hoot_api
```

You only create the database once. Django migrations will create its tables later.

## Create the Django project and app

```bash
django-admin startproject hoot_api .
python manage.py startapp api
```

The dot in the first command means “create the project in the current folder.” Without it, Django creates an extra outer folder.

## Django Project Structure

After creating our Django **project** and **app**, our backend currently looks something like this:

```plaintext
hoot-backend/
├── .venv/
├── api/
│   ├── migrations/
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   └── views.py
├── hoot_api/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── manage.py
└── requirements.txt
```

Remember that Django separates our code into a **project** and one or more **apps**.

* `hoot_api/` is our **Django project**. It contains configuration for the backend as a whole.
* `api/` is our **Django app**. This is where most of the code specific to our API will live.

### `.venv/`

This is our **virtual environment**. It contains the Python packages installed specifically for this project.

We won't edit anything inside this folder, and we won't commit it to GitHub.

### `api/`

This is the Django app we created. Most of the code we write for our API will live here.

#### `models.py`

```plaintext
api/models.py
```

This is where we'll define our **models**.

Models describe the data our application stores in the database. If you're coming from Mongoose, Django models serve a similar purpose to our Mongoose schemas and models.

#### `views.py`

```plaintext
api/views.py
```

This is where we'll write code that handles **requests and responses**.

For our REST API, our views will eventually contain much of our CRUD logic.

#### `migrations/`

```plaintext
api/migrations/
```

Django uses migrations to keep track of changes we make to our database structure.

Django will generate files inside this directory when we run commands such as:

```bash
python manage.py makemigrations
```

We generally **don't write these files ourselves**.

#### `admin.py`

```plaintext
api/admin.py
```

This file lets us register models with Django's built-in **admin site**.

We may use this later, but it isn't necessary for building our API.

#### `apps.py`

```plaintext
api/apps.py
```

This contains configuration for the `api` app.

Django generates it for us, and we generally won't need to change it.

### `manage.py`

```plaintext
manage.py
```

`manage.py` gives us access to Django's **command-line tools**.

We'll use it frequently:

```bash
python manage.py runserver
python manage.py makemigrations
python manage.py migrate
```

We use this file often, but we generally **don't edit it**.

### `requirements.txt`

```plaintext
requirements.txt
```

This keeps track of the Python packages our project depends on.

For example:

```txt
Django
djangorestframework
django-cors-headers
psycopg
```

Other developers can then install the project's dependencies with:

```bash
pip install -r requirements.txt
```

### `hoot_api/`

This is the **project configuration** directory.

It contains settings that affect the entire Django project rather than one specific app.

#### `urls.py`

```plaintext
hoot_api/urls.py
```

This is the project's main URL configuration.

Requests enter Django through this URL configuration, which can then send them to URLs and views belonging to our `api` app.

We'll modify this file when we connect our API's routes.

#### Files We Probably Won't Touch

There are several generated files and folders that Django needs, but that we likely won't edit during this project:

```plaintext
.venv/
api/migrations/
api/apps.py
hoot_api/asgi.py
hoot_api/wsgi.py
```

We may also make little or no use of:

```plaintext
api/admin.py
```

Django creates these because they support features that Django applications *can* use, even if our particular application doesn't need them.

That leaves a relatively small number of files we'll spend most of our time working with:

```plaintext
api/models.py
api/views.py
hoot_api/urls.py
hoot_api/settings.py
manage.py
requirements.txt
```

We'll also create a few additional files as we build our REST API.

### Next: `settings.py`

Before we start building, let's look at:

```plaintext
hoot_api/settings.py
```

`settings.py` is the main **configuration file for our Django project**. It tells Django which apps we're using, how to connect to our database, which middleware should run, and other project-wide settings.

We don't need to understand every setting Django generated. Instead, let's look at the pieces that will matter for our application.


## Django `settings.py`

The `settings.py` file is the main **configuration file for a Django project**. It tells Django how the overall project should behave.

You usually won't need to understand every setting right away. For now, focus on the sections we will interact with most often.

### `BASE_DIR`

```python
BASE_DIR = Path(__file__).resolve().parent.parent
```

`BASE_DIR` represents the **root folder of our Django project**.

Django can use this when it needs to build paths to other files or folders in our project.


### `SECRET_KEY`

```python
SECRET_KEY = 'django-insecure-...'
```

Django uses the secret key internally for security-related features.

The generated key is fine while we're developing locally, but **a real deployed application should not store its secret key directly in `settings.py` or commit it to GitHub**.

We can eventually move this into an environment variable.


### `DEBUG`

```python
DEBUG = True
```

When `DEBUG` is `True`, Django gives us detailed error pages that are very useful during development.

In production, this should be:

```python
DEBUG = False
```


### `ALLOWED_HOSTS`

```python
ALLOWED_HOSTS = []
```

This controls which hostnames are allowed to serve our Django application.

An empty list works for normal local development when `DEBUG = True`.

When we eventually deploy our application, we'll add our deployed domain here.


### `INSTALLED_APPS`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

`INSTALLED_APPS` tells Django **which apps are part of this project**.

Django starts us off with several built-in apps for things like authentication, sessions, static files, and the admin site.

When we create our own app, we'll add it here:

```python
INSTALLED_APPS = [
    # Django apps...
    'inventory',
]
```

If we're using packages such as Django REST Framework, they may also need to be added here:

```python
'rest_framework',
'corsheaders',
```


### `MIDDLEWARE`

```python
MIDDLEWARE = [
    ...
]
```

Middleware is code that runs **between an incoming request and our Django application**, and/or between our application and the outgoing response.

For example, middleware can handle:

* security
* authentication
* sessions
* CORS

For now, we generally won't change this unless a package tells us to add something.


### `ROOT_URLCONF`

```python
ROOT_URLCONF = 'config.urls'
```

This tells Django where the project's **main URL configuration** lives.

You can think of this file as the starting point Django uses when deciding which code should handle an incoming URL.


### `DATABASES`

Django starts with SQLite:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

This tells Django **which database to use and how to connect to it**.

If we're using PostgreSQL, we'll replace this configuration with our PostgreSQL connection information.


### `LANGUAGE_CODE` and `TIME_ZONE`

```python
LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'
```

These control Django's default language and timezone.


### Static Files

```python
STATIC_URL = 'static/'
```

Static files are files such as:

* CSS
* JavaScript
* icons
* other assets

This setting tells Django how static files should be referenced.


### The Big Picture

Think of `settings.py` as the project's **configuration panel**.

It answers questions like:

> What apps belong to this project?
> What database are we using?
> Are we in development mode?
> What security features should run?
> Where are important project resources located?

We won't normally put our application's actual CRUD logic in `settings.py`. Instead, we'll return here whenever we need to **configure Django or a package we're adding to the project**.

## Create environment variables

Create `.env` beside `manage.py`:

```plaintext
SECRET_KEY=replace-this-with-a-long-random-string
DATABASE_ENGINE=django.db.backends.postgresql
DATABASE_NAME=hoot_api
DATABASE_USER=
DATABASE_PASSWORD=
DATABASE_HOST=localhost
DATABASE_PORT=5432
```

Many macOS PostgreSQL setups can leave the username and password blank. EDB installations commonly use `postgres`; those students should fill in `DATABASE_USER=postgres` and their PostgreSQL password.

Also create a safe `.env.example` with the same variable names but no real password. Commit `.env.example`, not `.env`.

Use the below command to generate a random string for your `SECRET_KEY`:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

## Load the environment in settings

At the top of `hoot_api/settings.py`, add the new imports:

```python
import os
from pathlib import Path

from dotenv import load_dotenv
```

Immediately after Django defines `BASE_DIR`, load `.env`:

```python
BASE_DIR = Path(__file__).resolve().parent.parent
load_dotenv(BASE_DIR / ".env")
```

`BASE_DIR / ".env"` essentially means the .env file inside the `BASE_DIR` directory.  `BASE_DIR` isn't a normal string.  It's a Python Path object from the built-in pathlib module. Path objects let you construct file paths using `/`.

Change `SECRET_KEY`:

```python
SECRET_KEY = os.getenv(
    "SECRET_KEY",
    "django-insecure-classroom-only-not-for-production-12345",
)
```

The fallback keeps a classroom setup from crashing if `.env` is briefly missing. It is not a production secret.

## Register the packages and app

Add three items to the end of `INSTALLED_APPS`:

```python
INSTALLED_APPS = [
    # Django's existing apps remain here
    "corsheaders",
    "rest_framework",
    "api",
]
```

Django does not automatically use an installed package or a newly created app. Registering them tells Django to load them when the project starts.

Add the CORS middleware before Django's common middleware:

```python
MIDDLEWARE = [
    "corsheaders.middleware.CorsMiddleware",
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    # the remaining generated middleware stays here
]
```

Middleware order matters because CORS needs an opportunity to add headers to responses.

## Configure PostgreSQL

Replace the generated `DATABASES` setting:

```python
DATABASES = {
    "default": {
        "ENGINE": os.getenv(
            "DATABASE_ENGINE", "django.db.backends.postgresql"
        ),
        "NAME": os.getenv("DATABASE_NAME", "hoot_api"),
        "USER": os.getenv("DATABASE_USER", ""),
        "PASSWORD": os.getenv("DATABASE_PASSWORD", ""),
        "HOST": os.getenv("DATABASE_HOST", "localhost"),
        "PORT": os.getenv("DATABASE_PORT", "5432"),
    }
}
```

`os.getenv()` reads one variable from `.env`. The second argument is a fallback value.

## Create Django's built-in tables

```bash
python manage.py migrate
```

Django includes models for users, permissions, sessions, and the admin site. The first `migrate` command creates their tables.

Run a system check:

```bash
python manage.py check
```

Start the server:

```bash
python manage.py runserver
```

Visit [http://localhost:8000](http://localhost:8000). The Django success page confirms that the project starts. A database or import error in the terminal must be fixed before continuing.

## Checkpoint

- [ ] The terminal shows `(.venv)`.
- [ ] `pip install -r requirements.txt` completes.
- [ ] The `hoot_api` PostgreSQL database exists.
- [ ] `api` is listed in `INSTALLED_APPS`.
- [ ] `python manage.py migrate` completes.
- [ ] `python manage.py check` reports no issues.
- [ ] The Django development server starts.
