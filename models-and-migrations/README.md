<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Models and Migrations</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to create related Hoot and Comment models and apply migrations that create their PostgreSQL tables.

## Plan the data

We will use Django's built-in `User` model and create two models:

| Model | Important fields | Relationship |
| --- | --- | --- |
| User | `username`, password data | Has many Hoots and comments |
| Hoot | `title`, `text`, `category`, `created_at` | Belongs to one user; has many comments |
| Comment | `text`, `created_at` | Belongs to one user and one Hoot |

In PostgreSQL, these become related tables. In Python, Django lets us work with model objects instead of writing SQL for every CRUD operation.

## Build the Hoot model

Open `api/models.py`:

```python
from django.contrib.auth.models import User
from django.db import models


class Hoot(models.Model):
    CATEGORY_CHOICES = [
        ("News", "News"),
        ("Sports", "Sports"),
        ("Games", "Games"),
        ("Movies", "Movies"),
        ("Music", "Music"),
        ("Television", "Television"),
        ("Other", "Other"),
    ]

    title = models.CharField(max_length=100)
    text = models.TextField()
    category = models.CharField(
        max_length=20,
        choices=CATEGORY_CHOICES,
        default="News",
    )
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="hoots",
    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-created_at"]

    def __str__(self):
        return self.title
```

### Read the fields

| Code | Meaning |
| --- | --- |
| `CharField(max_length=100)` | A required short string with a database limit |
| `TextField()` | A required string intended for longer text |
| `choices=CATEGORY_CHOICES` | Only accept one of the listed values |
| `ForeignKey(User, ...)` | Store the ID of one related user |
| `auto_now_add=True` | Set the timestamp once when the Hoot is created |
| `ordering = ["-created_at"]` | Return newer Hoots first by default |

Each choice contains a stored value and a display label. They are identical because the frontend already uses capitalized category names.

### Understand the foreign key options

```python
author = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name="hoots",
)
```

- `User` is the model being referenced.
- `on_delete=models.CASCADE` means deleting a user also deletes that user's Hoots.
- `related_name="hoots"` allows a user instance to access its Hoots with `user.hoots.all()`.

The database stores an `author_id` column. Django lets our Python code use `hoot.author` to access the related user object.

## Build the Comment model

Add this class below `Hoot` in `api/models.py`:

```python
class Comment(models.Model):
    text = models.TextField()
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="comments",
    )
    hoot = models.ForeignKey(
        Hoot,
        on_delete=models.CASCADE,
        related_name="comments",
    )
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["created_at"]

    def __str__(self):
        return self.text[:40]
```

The `Comment.hoot` relationship gives us both directions:

```python
comment.hoot
hoot.comments.all()
```

Deleting a Hoot will also delete its comments because of `CASCADE`.

## Why define `__str__`?

Django calls `__str__` when it needs a readable name for an object, especially in the admin site and terminal.

```python
def __str__(self):
    return self.title
```

Without it, the admin would display a less helpful label such as `Hoot object (1)`.

## Make the migration file

```bash
python manage.py makemigrations
```

`makemigrations` compares the current model definitions with previous migration files and writes instructions for the new database changes. It changes code files, not the database itself.

You should see a new file similar to:

```plaintext
api/migrations/0001_initial.py
```

Do not manually edit the generated migration in this lesson.

## Apply the migration

```bash
python manage.py migrate
```

`migrate` runs unapplied migration instructions against the configured database. PostgreSQL now has tables for Hoots and comments.

The repeatable workflow is:

```plaintext
change model → makemigrations → migrate
```

Commit migration files. Every teammate needs the same record of database structure changes.

## Register the models in admin

Replace `api/admin.py`:

```python
from django.contrib import admin

from .models import Comment, Hoot


@admin.register(Hoot)
class HootAdmin(admin.ModelAdmin):
    list_display = ["title", "category", "author", "created_at"]


@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ["text", "author", "hoot", "created_at"]
```

This registration does not create public API endpoints. It tells Django's built-in admin site to show these models.

Create an admin user:

```bash
python manage.py createsuperuser
```

Start the server, visit [http://localhost:8000/admin](http://localhost:8000/admin), and sign in. Add one Hoot and one comment. This is a quick way to verify the models before building the API.

## You do: inspect the relationship

Open the Django shell:

```bash
python manage.py shell
```

Run these lines one at a time:

```python
from api.models import Hoot

hoot = Hoot.objects.first()
hoot.title
hoot.author.username
hoot.comments.all()
```

Exit with:

```python
exit()
```

## Checkpoint

- [ ] `Hoot` has the exact seven category values used by React.
- [ ] Both `author` fields point to Django's `User` model.
- [ ] `Comment.hoot` uses `related_name="comments"`.
- [ ] `makemigrations` creates a migration.
- [ ] `migrate` applies it successfully.
- [ ] The admin site can create and display both models.
