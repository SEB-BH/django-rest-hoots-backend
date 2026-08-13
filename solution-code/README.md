# Hoot Django REST Backend - Solution

This folder contains the finished classroom MVP built by the lesson. It is designed to work with the supplied Hoot React frontend without changing the frontend services.

## Run the solution

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
createdb hoot_api
python manage.py migrate
python manage.py runserver
```

Windows Git Bash uses this activation command:

```bash
source .venv/Scripts/activate
```

If PostgreSQL requires its default user, create the database with `createdb -U postgres hoot_api` and fill in `DATABASE_USER` and `DATABASE_PASSWORD` in `.env`.

In the React project, create `.env` containing:

```plaintext
VITE_BACK_END_SERVER_URL=http://localhost:8000
```

Then restart the Vite server.
