# API reference

The API is served under `/api/v1`. Unless noted otherwise, request and response
bodies use JSON. Resource IDs are UUIDs.

## Authentication

Register or log in to receive an access token. Send it on protected routes as
`Authorization: Bearer <accessToken>`. Public routes do not require a token.

```http
POST /api/v1/auth/register
Content-Type: application/json

{"email":"user@example.com","displayName":"User","password":"password123"}
```

Registration returns `201` with `user`, `accessToken`, and `expiresAt`. Login
uses the same fields except `displayName` and returns `200` with the same shape.

## Routes

| Method | Path | Access | Description |
| --- | --- | --- | --- |
| `POST` | `/auth/register` | Public | Create an account |
| `POST` | `/auth/login` | Public | Authenticate and issue a token |
| `GET` | `/posts` | Public | List posts |
| `GET` | `/posts/{id}` | Public | Get a post |
| `GET` | `/posts/{id}/comments` | Public | List a post's comments |
| `GET` | `/users/me` | User | Get the authenticated user's profile |
| `PATCH` | `/users/me` | User | Update the display name |
| `DELETE` | `/users/me` | User | Soft-delete the authenticated user's profile |
| `POST` | `/posts` | User | Create a post |
| `PUT` | `/posts/{id}` | Post author or admin | Replace a post's title, body, and category |
| `DELETE` | `/posts/{id}` | Post author or admin | Soft-delete a post |
| `POST` | `/posts/{id}/comments` | User | Add a comment to a post |
| `PUT` | `/comments/{id}` | Comment author or admin | Replace a comment's body |
| `DELETE` | `/comments/{id}` | Comment author or admin | Soft-delete a comment |
| `GET` | `/admin/users` | Admin | List users |
| `PATCH` | `/admin/users/{id}/status` | Admin | Activate or deactivate a user |

Prefix each path above with `/api/v1` when making a request.

## Posts and comments

Create or update a post with all three fields. Categories are `GENERAL` and
`QUESTION`.

```json
{"title":"Example","body":"Post text","category":"GENERAL"}
```

Create or update a comment with `{"body":"Comment text"}`. Successful creates
return `201`; reads and updates return `200`; deletes return `204` with no body.
Post and comment responses include their UUIDs, author ID and public author
details, text fields, and creation/update timestamps. Password hashes and
deleted records are not exposed.

## Listing and pagination

`GET /posts` accepts `page`, `size`, `search`, `category`, and `sort` query
parameters. `category` may be `GENERAL` or `QUESTION`; `sort` currently supports
only `latest`. `GET /posts/{id}/comments` accepts `page` and `size`.

Page numbers below 1 use page 1. Sizes below 1 use 20, and sizes above 100 are
capped at 100. List responses have the shape:

```json
{"items":[],"page":1,"size":20,"total":0}
```

## Errors and health checks

Errors return a JSON object containing `error.code`, `error.message`, and
`requestId`. Common status codes are `400` for invalid input, `401` for missing
or invalid authentication, `403` for insufficient permissions, `404` for missing
resources, `409` for conflicts, and `500` for unexpected failures.

- `GET /health/live` reports whether the process is running.
- `GET /health/ready` checks database readiness and returns `503` if unavailable.
