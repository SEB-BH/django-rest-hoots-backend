<h1>
  <span class="headline">Django REST Hoot Backend</span>
  <span class="subhead">Integration Lab</span>
</h1>

**Learning objective:** By the end of this lab, students will be able to demonstrate the complete MVP with two users and diagnose a failure by identifying the responsible application layer.

## Goal

Prove that the Django API is usable from the supplied React frontend. Work independently or in pairs, but each student must be able to explain one request from the React service through Django to PostgreSQL and back.

## Part 1: Run cleanly

From the backend:

```bash
python manage.py check
python manage.py showmigrations api
python manage.py runserver
```

From the frontend:

```bash
npm run dev
```

Resolve startup errors before testing features.

## Part 2: Complete the two-user test

Use two different browser profiles, an incognito window, or sign out between accounts.

### User A

- [ ] Create an account.
- [ ] Create a Hoot in a category other than `News`.
- [ ] Edit the Hoot.
- [ ] Add a comment to the Hoot.

### User B

- [ ] Create a second account.
- [ ] View User A's Hoot.
- [ ] Add a comment to User A's Hoot.
- [ ] Confirm that React does not show edit/delete controls for User A's Hoot.
- [ ] Edit User B's own comment.

### Return to User A

- [ ] Confirm that both comments remain after signing out and back in.
- [ ] Confirm that User A cannot edit User B's comment.
- [ ] Delete User A's Hoot.
- [ ] Confirm that the Hoot and both related comments disappear.

## Part 3: Verify one endpoint directly

Choose one create or update endpoint and reproduce it in Postman with the same Bearer token and JSON body used by React.

Record:

```plaintext
Method:
URL:
Required request fields:
Successful status code:
Important response fields:
One possible error status:
```

## Part 4: Trace one request

Choose `POST /hoots` or `POST /hoots/:hootId/comments`. Complete the table:

| Stage | File or tool | What happens there? |
| --- | --- | --- |
| Request begins | React service file | |
| URL matches | `api/urls.py` | |
| Request logic runs | `api/views.py` | |
| Input is checked/output is shaped | `api/serializers.py` | |
| Data is stored | `api/models.py` and PostgreSQL | |
| Response is used | React component or `App.jsx` | |

## Part 5: Debugging challenge

Ask a partner or instructor to choose one safe temporary mistake from this list:

- Change the React backend port from `8000` to `8001`.
- Remove the word `Bearer` and its following space from one Postman Authorization header.
- Request a Hoot ID that does not exist.
- Submit a category not listed in `CATEGORY_CHOICES`.
- Try to update another user's Hoot.

Diagnose the failure using the browser Network tab, Postman response, status code, and Django terminal. Restore the original value after finding the cause.

Write a short explanation:

```plaintext
Observed symptom:
Status code or browser message:
Responsible layer:
Evidence:
Fix:
```

## Deliverable

Submit one Markdown file named `django-hoot-backend-lab.md` containing:

1. The completed endpoint record from Part 3.
2. The completed request-tracing table from Part 4.
3. The debugging explanation from Part 5.
4. A link to the backend repository.
5. A screenshot of the React Hoot details page showing comments from both users.

## Acceptance criteria

- All core React user flows work against Django and PostgreSQL.
- The API requires authentication for `/users`, Hoot, and comment routes.
- Ownership rules are proven with two accounts.
- The submitted request trace correctly names each backend layer.
- The debugging explanation uses evidence rather than only stating that the code was changed.

## Finished early?

Choose only one small improvement:

- Add a maximum length to comment text and observe the serializer error.
- Add another category in both the Django model choices and React form.
- Add a `GET /health` endpoint that returns `{ "status": "ok" }` without authentication.
- Improve the frontend service error handling so non-`ok` responses throw a useful error.

Do not begin deployment, image uploads, likes, or a custom user model during this lab. Those features require more planning than the remaining session allows.
