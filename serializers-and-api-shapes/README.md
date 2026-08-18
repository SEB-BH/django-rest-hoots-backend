<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Serializers and API Shapes</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to build nested serializers that validate input and return the exact JSON field names expected by React.

## Why we need serializers

A Django model instance is a Python object, not JSON. React cannot directly use it.

A serializer has two main jobs:

1. Convert model instances into simple data that DRF can send as JSON.
2. Validate incoming request data before a model is created or updated.

This is similar to combining part of Mongoose validation with the response-shaping work previously done using `populate()` and `res.json()`.

## Start with the required response

The Hoot details page expects data shaped like this:

```json
{
  "_id": "8",
  "title": "Project presentations",
  "text": "Share what you built this week.",
  "category": "News",
  "author": {
    "_id": "2",
    "username": "sam"
  },
  "comments": [
    {
      "_id": "12",
      "text": "I am ready!",
      "author": {
        "_id": "3",
        "username": "lee"
      },
      "createdAt": "2026-08-13T08:00:00Z"
    }
  ],
  "createdAt": "2026-08-13T07:30:00Z"
}
```

The API must provide nested author and comment data because the React components access values such as:

```javascript
hoot.author.username
hoot.comments.length
comment.author._id
```

## Create the serializer file

Django did not generate this file. Create `api/serializers.py`:

```bash
touch api/serializers.py
```

Add the imports:

```python
from django.contrib.auth.models import User
from rest_framework import serializers

from .models import Comment, Hoot
```

## Serialize users

```python
class UserSerializer(serializers.ModelSerializer):
    _id = serializers.CharField(source="pk", read_only=True)

    class Meta:
        model = User
        fields = ["_id", "username"]
```

### Read this serializer

- `ModelSerializer` can create fields from a Django model.
- The inner `Meta` class says which model and fields to use.
- `source="pk"` reads Django's primary key.
- The serializer sends that value under the frontend's expected name, `_id`.
- `read_only=True` means clients may receive this field but may not choose it when creating or updating a record.

`pk` means primary key. For these models, it is the same value as Django's generated `id`.

## Serialize comments

Add the next class:

```python
class CommentSerializer(serializers.ModelSerializer):
    _id = serializers.CharField(source="pk", read_only=True)
    author = UserSerializer(read_only=True)
    createdAt = serializers.DateTimeField(
        source="created_at",
        read_only=True,
    )

    class Meta:
        model = Comment
        fields = ["_id", "text", "author", "createdAt"]
```

This serializer performs two translations:

| Django value | API field |
| --- | --- |
| `comment.pk` | `_id` |
| `comment.created_at` | `createdAt` |

It also uses `UserSerializer` inside `CommentSerializer`. This produces the nested author object.

The `hoot` relationship is not included in every comment because the comment already appears inside a particular Hoot response.

## Serialize Hoots

Add the final class:

```python
class HootSerializer(serializers.ModelSerializer):
    _id = serializers.CharField(source="pk", read_only=True)
    author = UserSerializer(read_only=True)
    comments = CommentSerializer(many=True, read_only=True)
    createdAt = serializers.DateTimeField(
        source="created_at",
        read_only=True,
    )

    class Meta:
        model = Hoot
        fields = [
            "_id",
            "title",
            "text",
            "category",
            "author",
            "comments",
            "createdAt",
        ]
```

`many=True` matters because one Hoot can have a list of comments, not just one comment.

The author, comments, ID, and creation time are read-only. The frontend only sends these editable values when creating a Hoot:

```json
{
  "title": "Project presentations",
  "text": "Share what you built this week.",
  "category": "News"
}
```

The server will decide the author from the signed-in user and the model will create the timestamp.

## Observe serialization in the shell

The admin checkpoint should have left at least one Hoot in the database. Open the shell:

```bash
python manage.py shell
```

```text
from api.models import Hoot
from api.serializers import HootSerializer

hoot = Hoot.objects.first()
serializer = HootSerializer(hoot)
serializer.data
```

Look for `_id`, `createdAt`, `author`, and `comments`. Exit with `exit()`.

## Observe validation

Open the shell again if needed:

```text
from api.serializers import HootSerializer

bad_data = {
    "title": "Missing text",
    "category": "Not a real category",
}

serializer = HootSerializer(data=bad_data)
serializer.is_valid()
serializer.errors
```

The serializer reports that `text` is required and the category is not a valid choice. No invalid Hoot is saved.

