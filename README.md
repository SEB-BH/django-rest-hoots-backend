<h1>
  <span class="prefix"></span>
  <span class="headline">Django REST Hoot Backend</span>
</h1>

## About

In this lesson, students build a Django REST Framework API for the supplied Hoot React frontend. The course assumes students already understand CRUD and React from the MERN stack, but have very little Python experience and no previous Django experience.

Every API endpoint is a function, every HTTP method has an explicit branch, and Django's built-in `User` model is reused. The finished MVP supports sign-up, sign-in, a user list, Hoot CRUD, nested comment CRUD, JWT authorization, ownership rules, PostgreSQL, and CORS.

## Contents

| Lesson | Skill |
| --- | ---: |
|[Django and Python Orientation](./django-and-python-orientation/README.md) | Relate Express concepts to Django and read the Python used in the API. |
|[Project and Database Setup](./project-and-database-setup/README.md) | Create a virtual environment, Django project, app, and PostgreSQL connection. |
|[Models and Migrations](./models-and-migrations/README.md) | Define related Django models and create database tables with migrations. |
|[Serializers and API Shapes](./serializers-and-api-shapes/README.md) | Use serializers to validate input and shape model data as frontend-compatible JSON. |
|[JWT Authentication](./jwt-authentication/README.md) | Create sign-up and sign-in endpoints and protect routes with Bearer tokens. |
|[Connect the React Frontend](./connect-the-react-frontend/README.md) | Configure CORS and connect a Vite environment variable to Django. |
|[Hoot CRUD](./hoot-crud/README.md) | Build explicit list, create, show, update, and delete route handlers. |
|[Comments and Ownership](./comments-and-ownership/README.md) | Build nested comment routes and enforce author-only changes. |


<!-- |[Integration Lab](./integration-lab/README.md) | 60 min | -->


