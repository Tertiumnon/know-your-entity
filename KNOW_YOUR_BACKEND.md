# Backend / REST API Style Guide

This document describes REST API design recommendations and typical patterns used in the project.

## Typical CRUD endpoints

- GET `/user` — get all users
- GET `/user/{id}` — get one user by ID
- POST `/user` — create a new user
- PUT `/user/{id}` — update a user (replace)
- PATCH `/user/{id}` — update a user (partial)
- DELETE `/user/{id}` — delete a user

## Design notes

- Keep endpoints resource-oriented and predictable.
- Use plural or singular consistently across the API (team decision).
- Use standard HTTP status codes and meaningful error responses.

## Documentation & Tools

- Generate API docs with OpenAPI/Swagger when possible.
- Use API versioning in the URL or via headers for breaking changes.

## References

- [Swagger / OpenAPI](https://swagger.io/)
