<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Hoot CRUD</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to implement and test Hoot list, create, show, update, and delete operations using function-based DRF views.

## The two Hoot URL patterns

We can support five CRUD operations with two paths:

| Path | Method | Operation |
| --- | --- | --- |
| `/hoots` | `GET` | Index/list |
| `/hoots` | `POST` | Create |
| `/hoots/:hootId` | `GET` | Show/details |
| `/hoots/:hootId` | `PUT` | Update |
| `/hoots/:hootId` | `DELETE` | Delete |

Django first matches the path. Our view function then checks `request.method`.

## Add imports

Add this Django shortcut to the imports in `api/views.py`:

```python
from django.shortcuts import get_object_or_404
```

Update the app imports so they include Hoot:

```python
from .models import Hoot
from .serializers import HootSerializer, UserSerializer
```

Keep the authentication imports and views from the previous microlesson.

## Build index and create in one view

Add this function below the auth and user views:

```python
@api_view(["GET", "POST"])
def hoot_list_create(request):
    if request.method == "GET":
        hoots = Hoot.objects.all()
        serializer = HootSerializer(hoots, many=True)
        return Response(serializer.data)

    serializer = HootSerializer(data=request.data)

    if serializer.is_valid():
        serializer.save(author=request.user)
        return Response(serializer.data, status=status.HTTP_201_CREATED)

    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

### Follow the GET branch

```python
hoots = Hoot.objects.all()
serializer = HootSerializer(hoots, many=True)
return Response(serializer.data)
```

1. The model retrieves all Hoot records.
2. The serializer converts the collection into frontend-compatible data.
3. `Response` sends it to the client.

The model's `Meta.ordering` makes newer Hoots appear first.

### Follow the POST branch

```python
serializer = HootSerializer(data=request.data)
```

Passing `data=` means “validate incoming data.”

```python
if serializer.is_valid():
    serializer.save(author=request.user)
```

The serializer validates the title, text, and category. The server sets the author from the authenticated request instead of trusting an author ID sent by the browser.

## Build show, update, and delete

```python
@api_view(["GET", "PUT", "DELETE"])
def hoot_detail(request, hoot_id):
    hoot = get_object_or_404(Hoot, pk=hoot_id)

    if request.method == "GET":
        serializer = HootSerializer(hoot)
        return Response(serializer.data)

    if hoot.author != request.user:
        return Response(
            {"err": "You can only change your own hoots."},
            status=status.HTTP_403_FORBIDDEN,
        )

    if request.method == "PUT":
        serializer = HootSerializer(hoot, data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)

        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    deleted_id = str(hoot.id)
    hoot.delete()
    return Response({"message": "Hoot deleted.", "_id": deleted_id})
```

### Find one record or return 404

```python
hoot = get_object_or_404(Hoot, pk=hoot_id)
```

This asks the `Hoot` model for one primary key. If it does not exist, Django immediately returns a `404 Not Found` response.

### Protect changes by author

Any authenticated user may view the Hoot. Only its author may update or delete it:

```python
if hoot.author != request.user:
```

`403 Forbidden` means the server knows who the user is, but that user is not allowed to perform this action.

### Update an existing instance

```python
serializer = HootSerializer(hoot, data=request.data)
```

This serializer receives both:

- `hoot`: the existing model instance to change.
- `data`: the proposed new field values.

Without the first argument, `.save()` would create a new Hoot instead of updating this one.

### Return JSON after delete

DRF APIs often return an empty `204` response after a delete. This endpoint returns a small JSON object because the supplied frontend calls `res.json()` after every service request.

## Add Hoot URLs

Add two patterns to `api/urls.py`:

```python
urlpatterns = [
    # existing auth and user paths
    path("hoots", views.hoot_list_create, name="hoot-list-create"),
    path("hoots/<int:hoot_id>", views.hoot_detail, name="hoot-detail"),
]
```

`<int:hoot_id>` captures one integer from the URL and passes it into the view as the `hoot_id` argument.

## Test the CRUD cycle

Every request needs the Bearer token from the auth lesson.

### Create

`POST http://localhost:8000/hoots`

```json
{
  "title": "Demo day",
  "text": "Share an MVP and one lesson learned.",
  "category": "News"
}
```

Expect `201 Created`. Copy the returned `_id`.

### Index

`GET http://localhost:8000/hoots`

Expect an array with the new Hoot at the beginning.

### Show

`GET http://localhost:8000/hoots/1`

Replace `1` with the copied ID. Expect one object with an empty `comments` array.

### Update

`PUT http://localhost:8000/hoots/1`

```json
{
  "title": "Demo day reminders",
  "text": "Share an MVP, the planning process, and one lesson learned.",
  "category": "News"
}
```

Expect the updated object.

### Delete

Create a temporary second Hoot, then send `DELETE` to its detail URL. Expect:

```json
{
  "message": "Hoot deleted.",
  "_id": "2"
}
```

## You do: prove the ownership rule

1. Sign up a second user and copy that user's token.
2. Use the second token to `GET` the first user's Hoot. It should work.
3. Use the second token to `PUT` or `DELETE` the first user's Hoot.
4. Confirm that the API returns `403 Forbidden` and does not change the record.

## Checkpoint

- [ ] The list response is an array.
- [ ] Create ignores any author supplied by a client and uses `request.user`.
- [ ] Detail returns `404` for a missing ID.
- [ ] Update changes the existing row rather than creating a duplicate.
- [ ] Delete returns JSON that the frontend can parse.
- [ ] A second user can read but cannot change the first user's Hoot.
