<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Release Notes</span>
</h1>

## Version 1.0

### Additions

- Nine beginner-focused microlessons designed for two class days.
- A complete Django 5.2 and Django REST Framework 3.16 solution.
- PostgreSQL, JWT authentication, CORS, Hoot CRUD, comment CRUD, and ownership checks.
- Curriculum config data, layouts, Canvas landing pages, and an instructor guide.

### Design decisions

- Uses function-based views only.
- Uses Django's built-in `User` model.
- Returns Mongo-style `_id` and camelCase `createdAt` values to preserve the supplied React frontend.
- Uses a seven-day access token for classroom convenience; refresh-token handling is deliberately deferred.
