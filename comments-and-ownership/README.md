<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Comments and Ownership</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to create, update, and delete nested comments while enforcing author-only changes.

## Why comments use nested paths

A comment belongs to a Hoot. The frontend therefore includes the parent Hoot ID in every comment URL:

```plaintext
POST   /hoots/:hootId/comments
PUT    /hoots/:hootId/comments/:commentId
DELETE /hoots/:hootId/comments/:commentId
```

The path communicates the relationship and lets the server confirm that a comment belongs to the requested Hoot.

## Add imports

Update the local imports in `api/views.py`:

```python
from .models import Comment, Hoot
from .serializers import CommentSerializer, HootSerializer, UserSerializer
```

Keep all other imports and existing views.

## Build comment create

```python
@api_view(["POST"])
def comment_create(request, hoot_id):
    hoot = get_object_or_404(Hoot, pk=hoot_id)
    serializer = CommentSerializer(data=request.data)

    if serializer.is_valid():
        serializer.save(author=request.user, hoot=hoot)
        return Response(serializer.data, status=status.HTTP_201_CREATED)

    return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

This follows the same pattern as Hoot create:

1. Find the parent Hoot.
2. Validate the incoming comment text.
3. Set both relationships on the server.
4. Save and return the nested response shape.

The frontend sends only:

```json
{
  "text": "This is helpful!"
}
```

It does not choose the author or parent Hoot.

## Build comment update and delete

```python
@api_view(["PUT", "DELETE"])
def comment_detail(request, hoot_id, comment_id):
    comment = get_object_or_404(
        Comment,
        pk=comment_id,
        hoot_id=hoot_id,
    )

    if comment.author != request.user:
        return Response(
            {"err": "You can only change your own comments."},
            status=status.HTTP_403_FORBIDDEN,
        )

    if request.method == "PUT":
        serializer = CommentSerializer(comment, data=request.data)

        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)

        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    deleted_id = str(comment.id)
    comment.delete()
    return Response({"message": "Comment deleted.", "_id": deleted_id})
```

### Verify both IDs

```python
comment = get_object_or_404(
    Comment,
    pk=comment_id,
    hoot_id=hoot_id,
)
```

This looks for a comment with the requested comment ID **and** parent Hoot ID. A mismatched nested URL returns `404`.

### Repeat the ownership check

```python
if comment.author != request.user:
```

We intentionally repeat the ownership logic instead of creating a custom permission class. At this stage, the visible repetition makes the rule and its location easier to understand.

## Add comment URLs

Add these patterns to `api/urls.py`:

```python
urlpatterns = [
    # existing auth, user, and Hoot paths
    path(
        "hoots/<int:hoot_id>/comments",
        views.comment_create,
        name="comment-create",
    ),
    path(
        "hoots/<int:hoot_id>/comments/<int:comment_id>",
        views.comment_detail,
        name="comment-detail",
    ),
]
```

## Test comment create

Use a valid Bearer token:

`POST http://localhost:8000/hoots/1/comments`

```json
{
  "text": "This is helpful!"
}
```

Expect a response similar to:

```json
{
  "_id": "1",
  "text": "This is helpful!",
  "author": {
    "_id": "2",
    "username": "sam"
  },
  "createdAt": "2026-08-13T08:00:00Z"
}
```

Now `GET` the parent Hoot. Its `comments` array should contain the comment because `HootSerializer` already nests `CommentSerializer`.

## Test update and delete

`PUT http://localhost:8000/hoots/1/comments/1`

```json
{
  "text": "This is very helpful!"
}
```

Then create a temporary comment and delete it:

`DELETE http://localhost:8000/hoots/1/comments/2`

Expect a JSON confirmation rather than an empty response.

## You do: test with two users

Use User A and User B from the Hoot ownership activity.

1. User A creates a Hoot.
2. User B comments on User A's Hoot. This should work.
3. User A tries to edit User B's comment. This should return `403`.
4. User B edits the comment. This should work.
5. User A deletes the Hoot. The related comment should also disappear because `Comment.hoot` uses `CASCADE`.

This proves that Hoot ownership and comment ownership are separate rules.

## Final backend file structure

```plaintext
hoot-backend/
├── api/
│   ├── migrations/
│   │   └── 0001_initial.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── serializers.py
│   ├── urls.py
│   └── views.py
├── hoot_api/
│   ├── settings.py
│   └── urls.py
├── .env
├── .env.example
├── .gitignore
├── manage.py
└── requirements.txt
```

## Complete the full user flow

1. Open the React URL shown by Vite.
2. Sign up with a username and password.
3. Confirm that the dashboard appears and lists the new user.
4. Open the Hoot list.
5. Create a Hoot.
6. Open its details.
7. Add a comment.
8. Edit the Hoot and comment.
9. Delete the comment and Hoot.
10. Sign out, sign back in, and confirm the data remains in PostgreSQL.

## Compare the network request

Open the browser developer tools and select the Network tab. Click one request to `/hoots` and identify:

- **Request URL:** `http://localhost:8000/hoots`
- **Request method:** `GET` or `POST`
- **Authorization header:** begins with `Bearer`
- **Response status:** normally `200` or `201`
- **Response data:** uses `_id`, `createdAt`, and nested author objects

This view connects the code in the React service to the code in the Django view.

## Diagnose by status code

| Result | Meaning | Check first |
| --- | --- | --- |
| Browser says CORS | Django did not allow this frontend origin | Exact React port and CORS middleware order |
| `401 Unauthorized` | The request is not authenticated | Token exists and header begins with `Bearer`, followed by a space |
| `403 Forbidden` | The user is signed in but does not own the content | Compare `request.user` with the record's author |
| `404 Not Found` | No URL or record matched | Request path, captured IDs, and Django terminal |
| `400 Bad Request` | Submitted data failed validation | Response body and serializer errors |
| `500 Internal Server Error` | Backend code raised an exception | Read the bottom of the Django traceback first |

## Do not change several layers at once

When something fails:

1. Reproduce the same request in Postman.
2. If Postman also fails, inspect URLs, views, serializers, models, and the Django traceback.
3. If Postman works but React fails, inspect the frontend base URL, CORS, Bearer token, request body, and browser network response.

Changing both applications before retesting makes it harder to know which change solved or caused the problem.