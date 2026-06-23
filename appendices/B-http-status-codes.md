# Appendix B: HTTP Status Code Reference

## Status Code Categories

| Range | Category | Meaning |
|-------|----------|---------|
| 1xx | Informational | Request received, continuing process |
| 2xx | Success | Request was successfully received and processed |
| 3xx | Redirection | Further action needed to complete the request |
| 4xx | Client Error | Request contains bad syntax or cannot be fulfilled |
| 5xx | Server Error | Server failed to fulfill a valid request |

---

## Codes You'll Use in Spring Boot APIs

### 2xx — Success

| Code | Name | When to Use | Spring Boot |
|------|------|-------------|-------------|
| **200** | OK | Successful GET, PUT, PATCH | `ResponseEntity.ok(body)` |
| **201** | Created | Successful POST (resource created) | `ResponseEntity.status(HttpStatus.CREATED).body(body)` |
| **204** | No Content | Successful DELETE (nothing to return) | `ResponseEntity.noContent().build()` |

### 4xx — Client Errors

| Code | Name | When to Use | Spring Boot |
|------|------|-------------|-------------|
| **400** | Bad Request | Invalid input, validation failed | `ResponseEntity.badRequest().body(error)` |
| **401** | Unauthorized | No credentials or invalid credentials | Spring Security returns this automatically |
| **403** | Forbidden | Authenticated but not authorized | Spring Security returns this automatically |
| **404** | Not Found | Resource doesn't exist | `ResponseEntity.notFound().build()` |
| **405** | Method Not Allowed | Wrong HTTP method for this endpoint | Spring returns this automatically |
| **409** | Conflict | Resource conflict (e.g., duplicate) | `ResponseEntity.status(HttpStatus.CONFLICT).body(error)` |
| **415** | Unsupported Media Type | Missing or wrong Content-Type header | Spring returns this automatically |
| **422** | Unprocessable Entity | Semantically invalid (alternative to 400) | `ResponseEntity.unprocessableEntity().body(error)` |

### 5xx — Server Errors

| Code | Name | When to Use | Spring Boot |
|------|------|-------------|-------------|
| **500** | Internal Server Error | Unhandled exception, bug in your code | Your @ControllerAdvice catch-all handler |
| **502** | Bad Gateway | Upstream service returned invalid response | Rarely set by your app directly |
| **503** | Service Unavailable | Server is overloaded or in maintenance | Rarely set by your app directly |

---

## Quick Decision Guide

```
Did the request succeed?
├── YES → Did it create something?
│         ├── YES → 201 Created
│         └── NO  → Is there a body to return?
│                   ├── YES → 200 OK
│                   └── NO  → 204 No Content
│
└── NO  → Is it the client's fault?
          ├── YES → What went wrong?
          │         ├── Bad/missing data      → 400 Bad Request
          │         ├── Not logged in         → 401 Unauthorized
          │         ├── Not allowed           → 403 Forbidden
          │         ├── Resource not found    → 404 Not Found
          │         └── Duplicate/conflict    → 409 Conflict
          │
          └── NO (server's fault) → 500 Internal Server Error
```

---

## Common Mistakes

| Mistake | Correct Approach |
|---------|-----------------|
| Returning 200 with `{"error": "not found"}` | Return 404 with an error body |
| Returning 200 for a successful POST | Return 201 Created |
| Returning 404 for an empty list | Return 200 with `[]` (the collection exists, it's just empty) |
| Returning 500 for invalid client input | Return 400 (it's the client's fault, not the server's) |
| Returning 403 when the user isn't logged in | Return 401 (they need to authenticate first) |
