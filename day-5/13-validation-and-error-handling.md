# Chapter 13: Validation and Error Handling

> ⏱ Estimated time: 60 minutes

## What You'll Learn

- Why you should never trust client input
- How to validate request data using Bean Validation annotations
- How to create custom error responses
- How to use `@ExceptionHandler` and `@ControllerAdvice` for global error handling
- How to make your API return meaningful, consistent error messages

---

## Concepts

### Never Trust the Client

This is the #1 rule of backend development:

> **Every piece of data from the client is potentially wrong, malicious, or missing.**

The client could send:
- An empty title: `{"title": "", "pages": 412}`
- Negative pages: `{"title": "Dune", "pages": -5}`
- Missing fields: `{"title": "Dune"}`
- Garbage: `{"title": "x".repeat(10000), "pages": 999999999}`

Your frontend might validate before sending, but:
1. Someone can bypass the frontend and call your API directly with curl
2. Bugs in the frontend can send bad data
3. You might have multiple clients (web, mobile, third-party) and can't control them all

**The backend is the last line of defense.** Always validate.

### Bean Validation (Jakarta Validation)

Spring Boot supports **Bean Validation** — annotations you put on your DTO fields to declare rules:

```java
public record BookRequest(
    @NotBlank(message = "Title is required")
    String title,
    
    @NotBlank(message = "Author is required")
    String author,
    
    @Min(value = 1, message = "Pages must be at least 1")
    @Max(value = 10000, message = "Pages must be at most 10000")
    int pages
) {}
```

To activate validation, add `@Valid` to the controller parameter:

```java
@PostMapping
public ResponseEntity<BookResponse> createBook(@Valid @RequestBody BookRequest request) {
    // If validation fails, this method is never called
    // Spring returns 400 Bad Request automatically
}
```

### Common Validation Annotations

| Annotation | Applies To | Rule |
|-----------|-----------|------|
| `@NotNull` | Any type | Must not be null |
| `@NotBlank` | String | Must not be null, empty, or whitespace |
| `@NotEmpty` | String, Collection | Must not be null or empty |
| `@Size(min=, max=)` | String, Collection | Length/size must be in range |
| `@Min(value)` | Number | Must be ≥ value |
| `@Max(value)` | Number | Must be ≤ value |
| `@Positive` | Number | Must be > 0 |
| `@Email` | String | Must be a valid email format |
| `@Pattern(regexp=)` | String | Must match the regex |
| `@Past` | Date/Time | Must be in the past |
| `@Future` | Date/Time | Must be in the future |

### The Problem with Default Error Responses

When validation fails, Spring returns something like this:

```json
{
    "timestamp": "2025-01-15T10:30:00.000+00:00",
    "status": 400,
    "error": "Bad Request",
    "path": "/api/books"
}
```

This is unhelpful — the client doesn't know *what* was wrong. We need custom error responses.

### Custom Error Response

Define a consistent error format:

```json
{
    "status": 400,
    "error": "Validation Failed",
    "message": "Request contains invalid fields",
    "details": [
        "title: Title is required",
        "pages: Pages must be at least 1"
    ],
    "timestamp": "2025-01-15T10:30:00"
}
```

### Exception Handling with @ControllerAdvice

`@ControllerAdvice` creates a global exception handler — a single class that catches exceptions from ANY controller and converts them to proper HTTP responses.

```
Request → Controller → throws exception
                           ↓
                    @ControllerAdvice catches it
                           ↓
                    Returns structured error response
```

---

## Code Examples

### Step 1: Add Validation Dependency

If you're using `spring-boot-starter-web`, validation is already included. If not, add:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### Step 2: Add Validation to BookRequest

Update `src/main/java/com/bookshelf/dto/BookRequest.java`:

```java
package com.bookshelf.dto;

import jakarta.validation.constraints.*;

public record BookRequest(
    @NotBlank(message = "Title is required")
    @Size(max = 255, message = "Title must be at most 255 characters")
    String title,

    @NotBlank(message = "Author is required")
    @Size(max = 255, message = "Author must be at most 255 characters")
    String author,

    @Min(value = 1, message = "Pages must be at least 1")
    @Max(value = 10000, message = "Pages must be at most 10,000")
    int pages
) {}
```

### Step 3: Add @Valid to Controller

Update your controller methods that accept request bodies:

```java
@PostMapping
public ResponseEntity<BookResponse> createBook(@Valid @RequestBody BookRequest request) {
    BookResponse created = bookService.createBook(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}

@PutMapping("/{id}")
public ResponseEntity<BookResponse> updateBook(
        @PathVariable Long id,
        @Valid @RequestBody BookRequest request
) {
    return bookService.updateBook(id, request)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
}
```

### Step 4: Create Error Response DTO

Create `src/main/java/com/bookshelf/dto/ErrorResponse.java`:

```java
package com.bookshelf.dto;

import java.time.LocalDateTime;
import java.util.List;

public record ErrorResponse(
    int status,
    String error,
    String message,
    List<String> details,
    LocalDateTime timestamp
) {
    // Factory method for easy creation
    public static ErrorResponse of(int status, String error, String message, List<String> details) {
        return new ErrorResponse(status, error, message, details, LocalDateTime.now());
    }

    public static ErrorResponse of(int status, String error, String message) {
        return new ErrorResponse(status, error, message, List.of(), LocalDateTime.now());
    }
}
```

### Step 5: Create a Custom Exception

Create `src/main/java/com/bookshelf/exception/BookNotFoundException.java`:

```java
package com.bookshelf.exception;

public class BookNotFoundException extends RuntimeException {
    
    public BookNotFoundException(Long id) {
        super("Book not found with id: " + id);
    }
}
```

### Step 6: Create Global Exception Handler

Create `src/main/java/com/bookshelf/exception/GlobalExceptionHandler.java`:

```java
package com.bookshelf.exception;

import com.bookshelf.dto.ErrorResponse;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import java.util.List;

@ControllerAdvice
public class GlobalExceptionHandler {

    // Handle validation errors (when @Valid fails)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> details = ex.getBindingResult().getFieldErrors().stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .toList();

        ErrorResponse response = ErrorResponse.of(
                400,
                "Validation Failed",
                "Request contains invalid fields",
                details
        );
        return ResponseEntity.badRequest().body(response);
    }

    // Handle "not found" errors
    @ExceptionHandler(BookNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleBookNotFound(BookNotFoundException ex) {
        ErrorResponse response = ErrorResponse.of(
                404,
                "Not Found",
                ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }

    // Handle everything else (safety net)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse response = ErrorResponse.of(
                500,
                "Internal Server Error",
                "An unexpected error occurred"
                // Don't expose the real error message — it might contain sensitive info
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

### Step 7: Use the Exception in the Service

Update `BookService` to throw `BookNotFoundException`:

```java
public BookResponse getBookById(Long id) {
    return bookRepository.findById(id)
            .map(this::toResponse)
            .orElseThrow(() -> new BookNotFoundException(id));
}

public BookResponse updateBook(Long id, BookRequest request) {
    Book existingBook = bookRepository.findById(id)
            .orElseThrow(() -> new BookNotFoundException(id));

    existingBook.setTitle(request.title());
    existingBook.setAuthor(request.author());
    existingBook.setPages(request.pages());

    return toResponse(bookRepository.save(existingBook));
}

public void deleteBook(Long id) {
    if (!bookRepository.existsById(id)) {
        throw new BookNotFoundException(id);
    }
    bookRepository.deleteById(id);
}
```

Update the Controller (simpler now — no Optional handling):

```java
@GetMapping("/{id}")
public ResponseEntity<BookResponse> getBookById(@PathVariable Long id) {
    return ResponseEntity.ok(bookService.getBookById(id));
    // If not found, BookNotFoundException is thrown → caught by GlobalExceptionHandler → 404
}

@PutMapping("/{id}")
public ResponseEntity<BookResponse> updateBook(
        @PathVariable Long id,
        @Valid @RequestBody BookRequest request
) {
    return ResponseEntity.ok(bookService.updateBook(id, request));
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
    bookService.deleteBook(id);
    return ResponseEntity.noContent().build();
}
```

### Testing Error Responses

```bash
# Validation error — empty title
curl -v -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "", "author": "Test", "pages": 100}'
# Response: 400
# {"status":400,"error":"Validation Failed","message":"Request contains invalid fields",
#  "details":["title: Title is required"],"timestamp":"..."}

# Validation error — negative pages
curl -v -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "author": "Test", "pages": -5}'
# Response: 400
# {"status":400,"error":"Validation Failed","details":["pages: Pages must be at least 1"],...}

# Not found error
curl -v http://localhost:8080/api/books/999
# Response: 404
# {"status":404,"error":"Not Found","message":"Book not found with id: 999",...}
```

---

## Exercise: Add Validation and Error Handling to BookShelf

**Goal**: Make your API reject bad input and return helpful error messages.

### Tasks

1. Add validation annotations to `BookRequest`
2. Add `@Valid` to controller methods
3. Create `ErrorResponse` DTO
4. Create `BookNotFoundException`
5. Create `GlobalExceptionHandler` with `@ControllerAdvice`
6. Update the service to throw `BookNotFoundException`
7. Simplify controller (remove Optional handling — let exceptions flow)

### Test These Scenarios

| Request | Expected |
|---------|----------|
| POST with empty title | 400 with "Title is required" |
| POST with pages = 0 | 400 with "Pages must be at least 1" |
| POST with empty title AND pages = 0 | 400 with BOTH errors listed |
| GET /api/books/999 | 404 with "Book not found" |
| DELETE /api/books/999 | 404 with "Book not found" |
| POST with valid data | 201 (still works!) |

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Forgetting `@Valid` on the controller parameter | Validation annotations on the DTO won't do anything without `@Valid` on the controller method. |
| Using `@NotNull` for strings | `@NotNull` allows empty strings `""`. Use `@NotBlank` for strings to also reject empty and whitespace. |
| Exposing stack traces in error responses | Never send `ex.getStackTrace()` to the client. It reveals internal implementation details. Log it, return a generic message. |
| Catching exceptions in the controller | Let exceptions propagate to `@ControllerAdvice`. That's why it exists — centralized error handling. |
| Not handling the generic `Exception` case | Always have a catch-all handler. Unhandled exceptions return ugly Spring defaults. |

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 35](../appendices/E-coding-exercises.md#exercise-35) | Add Validation Annotations | ⭐⭐ |
| [Exercise 36](../appendices/E-coding-exercises.md#exercise-36) | Global Exception Handler | ⭐⭐ |
| [Exercise 37](../appendices/E-coding-exercises.md#exercise-37) | Structured Validation Errors | ⭐⭐ |
| [Exercise 38](../appendices/E-coding-exercises.md#exercise-38) | Handle Multiple Exception Types | ⭐⭐⭐ |
| [Exercise 39](../appendices/E-coding-exercises.md#exercise-39) | Test Validation with curl | ⭐⭐ |

Solutions are in [Appendix F](../appendices/F-exercise-solutions.md).

---

## Key Takeaways

- [ ] Never trust client input — always validate on the backend
- [ ] Use Bean Validation annotations (`@NotBlank`, `@Min`, `@Size`, etc.) on DTOs
- [ ] Add `@Valid` to controller parameters to activate validation
- [ ] Use `@ControllerAdvice` + `@ExceptionHandler` for global error handling
- [ ] Create custom exceptions (like `BookNotFoundException`) for specific error cases
- [ ] Always return structured, consistent error responses

---

## Quick Quiz

1. What's the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`?
2. Why should you validate on the backend even if the frontend already validates?
3. What annotation activates Bean Validation in a controller method?
4. What does `@ControllerAdvice` do?
5. Why shouldn't you include `ex.getMessage()` or stack traces in a 500 error response?

---

*Next: `14-relationships-and-queries.md` — Connect your entities together →*
