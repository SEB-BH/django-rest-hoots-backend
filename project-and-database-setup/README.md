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

The important structure is now:

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

- `manage.py` runs project commands.
- `hoot_api` is the project configuration folder.
- `api` is the app where we will write Hoot features.

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
