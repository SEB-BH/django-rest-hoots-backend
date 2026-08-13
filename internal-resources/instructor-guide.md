<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Instructor Guide</span>
</h1>

The times leave room on Day 1 for review and practice. Day 2 is a full build-and-integration session.

## What students build

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/auth/sign-up` | Create a user and return a token |
| `POST` | `/auth/sign-in` | Check credentials and return a token |
| `GET` | `/users` | List usernames for the dashboard |
| `GET`, `POST` | `/hoots` | List or create Hoots |
| `GET`, `PUT`, `DELETE` | `/hoots/:hootId` | Read, update, or delete one Hoot |
| `POST` | `/hoots/:hootId/comments` | Add a comment |
| `PUT`, `DELETE` | `/hoots/:hootId/comments/:commentId` | Update or delete one comment |



## Deliberately outside this lesson

- Refresh tokens and token revocation
- Deployment and production settings
- Custom user models
- Class-based views, generic views, routers, and viewsets
- Automated test-writing as a new topic
- Pagination, search, likes, images, and notifications

These are all reasonable later improvements, but including them would make the first Django API harder to follow and push the lesson beyond two days.

## Included solution

The complete working API is in [`solution-code`](./solution-code/README.md). Instructors can use it as a checkpoint reference or run it against the supplied React frontend.

## Internal

### Prerequisites

- Students have completed React CRUD and frontend JWT lessons.
- Python and PostgreSQL are installed before this lesson begins.
- Each student can run `python3 --version`, `psql --version`, and `createdb --version`.
- Postman or another API client is available.

### Course landing pages

- [SEB](./canvas-landing-pages/seb.md)
- [Fallback](./canvas-landing-pages/fallback.md)

### Resources

- [Instructor Guide](./internal-resources/instructor-guide.md)
- [Release Notes](./internal-resources/release-notes.md)
- [Video Hub](./internal-resources/video-hub.md)


## Teaching approach

This lesson is a translation exercise as much as a Django lesson. Keep returning to concepts students already know from Express, Mongoose, and React services. Avoid introducing viewsets, routers, generic views, custom permissions, custom user models, refresh-token flows, or deployment during the core build.

The finished code repeats ownership checks and serializer validation on purpose. The repetition lets students see where each decision occurs before they learn abstractions.

## Suggested pacing

### Day 1

1. **Orientation - 30 minutes:** Draw the request path from React to URL, view, serializer, model, and PostgreSQL. Complete the short Python translation activity.
2. **Setup - 60 minutes:** Pair-check virtual environments, packages, database creation, project/app creation, and settings.
3. **Models - 75 minutes:** Build `Hoot` first, then `Comment`. Pause for the one-to-many relationship. Run migrations and inspect data in admin.
4. **Serializers - 60 minutes:** Start with `UserSerializer`, then `CommentSerializer`, then `HootSerializer`. Emphasize that `_id` and `createdAt` are frontend compatibility names.
5. **Checkpoint/recovery - 45 to 75 minutes:** Students compare file trees and run `python manage.py check` before leaving.

### Day 2

1. **Authentication - 75 minutes:** Build one helper and two auth endpoints. Decode a token at jwt.io or in the browser console only to inspect its payload; never paste a real production token.
2. **Hoot CRUD - 75 minutes:** Test every method in Postman before opening React.
3. **Comments and ownership - 75 minutes:** Use two accounts to prove ownership rules.
4. **React connection - 45 minutes:** Add CORS, create the Vite `.env`, and restart both servers.
5. **Integration lab - 60 minutes:** Students complete the checklist independently or in pairs.

## Checkpoints

| Checkpoint | Evidence |
| --- | --- |
| End of setup | Django starts with no configuration error and PostgreSQL migrations complete. |
| End of models | Hoots and comments can be added in Django admin. |
| End of serializers | Serializer output contains `_id`, `createdAt`, nested `author`, and nested `comments`. |
| End of auth | Sign-up returns a token whose payload contains `_id` and `username`. |
| End of API | All seven endpoint patterns respond correctly in Postman. |
| End of integration | A user can complete sign-up, CRUD, comments, and sign-out in React. |

## Common problems

| Symptom | Most likely cause | First check |
| --- | --- | --- |
| `command not found: python` | Student's system uses `python3`, or the environment is inactive. | Run `python3 --version` and inspect the prompt for `(.venv)`. |
| `role ... does not exist` | PostgreSQL is trying the computer username, but that role was never created. | Use the configured PostgreSQL role in `.env`, often `postgres` on EDB installs. |
| `password authentication failed` | Incorrect `DATABASE_USER` or `DATABASE_PASSWORD`. | Confirm the same credentials work in `psql`. |
| `No module named ...` | The environment is inactive or packages were installed outside it. | Run `which python` or `where python`, then `pip list`. |
| `relation api_hoot does not exist` | Migrations were not made/applied. | Run `makemigrations` and `migrate`. |
| `401 Unauthorized` | Missing, expired, or malformed Bearer token. | Inspect the request's `Authorization` header. |
| React shows no edit button | `_id` is numeric or the nested author is missing. | Compare the API JSON with the serializer lesson's target shape. |
| Browser CORS error | Origin mismatch or middleware order. | Confirm port `5173` and place CORS middleware before common middleware. |
| Frontend still calls the old URL | Vite loaded `.env` only when it started. | Restart `npm run dev`. |

## Optional instructor extension

If the class finishes early, add `select_related("author")` and `prefetch_related("comments__author")` to the Hoot query, then explain that this improves database efficiency without changing the response. Do not make this part of the required build.
