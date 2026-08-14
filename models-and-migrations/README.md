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
from django.db import models
from django.contrib.auth.models import User


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
| `CharField(max_length=100)` | A required short string with a database limit (note that Django requires `max_length` to be defined whenever `CharField` is used) |
| `TextField()` | A required string intended for longer text |
| `choices=CATEGORY_CHOICES` | Only accept one of the listed values |
| `ForeignKey(User, ...)` | Store the ID of one related user |
| `auto_now_add=True` | Set the timestamp once when the Hoot is created |
| `ordering = ["-created_at"]` | Return newer Hoots first by default |


Django’s `choices` on the `category` field is very similar to Mongoose’s `enum`. **Both restrict a field to a predefined set of values.**
`CATEGORY_CHOICES` is a list of tuples. Each tuple contains two values:

```python
("value stored in database", "label shown to user")
```

The two values don’t have to match.
For example:

```python
("TV", "Television")
```

This would store `"TV"` in the database but show **Television** to the user.

Later, you would connect these choices to a field:

```python
category = models.CharField(
    max_length=20,
    choices=CATEGORY_CHOICES,
    default="Other"
)
```

Django can then create a dropdown containing only those category options. `CATEGORY_CHOICES` itself does not create a database field—it only defines the options that the `category` field can use.


### Understand the foreign key options

This code is basically saying:

> **Every Hoot belongs to one User.**

```python
author = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name="hoots",
)
```

Let's break down each piece.

### `author =`

This creates a field called `author` on every Hoot.

So if we have:

```python
hoot = Hoot.objects.get(id=1)
```

we can access the user who wrote it with:

```python
hoot.author
```

### `ForeignKey` 

`ForeignKey` creates a **many-to-one relationship**.

It means:

> Each Hoot has **one author**, but one User can author **many Hoots**.

For example:

```text
User: aisha1
   ↑
   ├── Hoot #1
   ├── Hoot #2
   └── Hoot #3
```

### `on_delete=models.CASCADE`

> What should happen to a user's Hoots if we delete that User?

`CASCADE` means **delete their Hoots too**.

So:

```text
Delete User
    ↓
Delete all of their Hoots
```

Django requires you to explicitly decide what should happen when the related object is deleted.

### `related_name="hoots"`

This is for going **the other direction**.

We already know we can go from a Hoot → User.  But what if we have a User and want **all the Hoots they wrote?**

Because we specified:

```python
related_name="hoots"
```

we can do:

```python
user.hoots.all()
```

So you get this nice two-way relationship:

```python
# Hoot → User
hoot.author

# User → their Hoots
user.hoots.all()
```

> **A Hoot has one author. The author must be a User. A User can have many Hoots. If the User is deleted, delete their Hoots too. Let me access a user's Hoots with `user.hoots.all()`.**

So this is essentially how we're representing the **one-to-many relationship** you would have handled with an `ObjectId` + `ref` in Mongoose.

### Why define `__str__`?

Django calls `__str__` when it needs a readable name for an object, especially in the admin site and terminal.

```python
def __str__(self):
    return self.title
```

Without it, the admin would display a less helpful label such as `Hoot object (1)`.

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

#### `__str__` is a bit different on `Comment`

The `[:40]` is Python slicing. It means: "Give me everything from the beginning up to the first 40 characters."

## Wait, where's the `User` model?

We never defined the `User` model because **Django already defined one for us**.

This import:

```python
from django.contrib.auth.models import User
```

is essentially saying:

> "Give me Django's built-in `User` model so I can use it in this file."

Django's `User` model already has common authentication fields such as:

```text
User
├── username
├── password
├── email
├── first_name
├── last_name
├── is_staff
├── is_active
└── ...
```
_Check out [these docs](https://docs.djangoproject.com/en/6.0/ref/contrib/auth/#fields) for the full list of built-in fields_.

So when we write:

```python
author = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    related_name="hoots",
)
```

we're creating a relationship between our `Hoot` model and **Django's existing `User` model**.

### Compared with our previous Mongoose apps

In Mongoose, we wrote something like:

```javascript
const userSchema = new mongoose.Schema({
  username: String,
  password: String,
})

const User = mongoose.model('User', userSchema)
```

We had to define what a user was ourselves.

Django is more **"batteries included."** Authentication is such a common requirement that Django ships with a complete user/authentication system.

That's also why Django gives us things like `request.user` and `user.is_authenticated` without us having to build all of that ourselves.

### One small caveat

In larger Django projects, developers often create a **custom User model** so they can add things like avatars, roles, bios, etc.

But for our Hoot app, using Django's built-in `User` keeps the things simple.


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
