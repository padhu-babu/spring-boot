# Chapter 17: Logging and API Documentation

> ⏱ Estimated time: 50 minutes

## What You'll Learn

- Why logging matters and how to do it in Spring Boot (SLF4J + Logback)
- Log levels and when to use each one
- How to add Spring Boot Actuator for health monitoring
- How to generate API documentation with Swagger/OpenAPI
- How to make your API self-documenting

---

## Concepts

### Why Logging Matters

When something goes wrong in production at 3 AM, you can't attach a debugger. Logs are your eyes and ears.

**Without logging**: "The app is broken." → "I have no idea why."
**With logging**: "The app is broken." → Check logs → "BookService.createBook failed at line 42: Author not found for ID 999."

### SLF4J: The Logging API

Spring Boot uses **SLF4J** (Simple Logging Facade for Java) as the logging API and **Logback** as the implementation. They're included automatically — no extra dependencies.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class BookService {

    private static final Logger log = LoggerFactory.getLogger(BookService.class);

    public BookResponse createBook(BookRequest request) {
        log.info("Creating book with title: {}", request.title());
        // ... create the book ...
        log.info("Book created with id: {}", saved.getId());
        return toResponse(saved);
    }
}
```

**Key points:**
- One `Logger` per class, using the class name
- Use `{}` as placeholders — SLF4J fills them in (don't use string concatenation)
- The logger is `static final` — created once per class

### Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| `ERROR` | Something failed that shouldn't have. Needs attention. | `log.error("Failed to save book: {}", ex.getMessage(), ex)` |
| `WARN` | Something unexpected but recoverable. | `log.warn("Author not found for id {}, using default", authorId)` |
| `INFO` | Important business events. | `log.info("Book created: id={}", book.getId())` |
| `DEBUG` | Detailed info for troubleshooting. | `log.debug("Searching for books with title containing: {}", title)` |
| `TRACE` | Very detailed, rarely used. | `log.trace("Entering method getAllBooks()")` |

**Rule of thumb:**
- `ERROR` → "Wake someone up at 3 AM"
- `WARN` → "Look at this during business hours"
- `INFO` → "This is what the app is doing"
- `DEBUG` → "I need to troubleshoot a specific issue"

### Configuring Log Levels

In `application.properties`:

```properties
# Set log level for your app
logging.level.com.bookshelf=DEBUG

# Set log level for Spring framework
logging.level.org.springframework.web=INFO

# Set log level for Hibernate SQL
logging.level.org.hibernate.SQL=DEBUG
```

### Spring Boot Actuator

**Actuator** exposes operational endpoints for monitoring your application:

Add the dependency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Configure in `application.properties`:
```properties
management.endpoints.web.exposure.include=health,info,metrics
management.endpoint.health.show-details=always
```

Now you get:
```bash
curl http://localhost:8080/actuator/health
# {"status":"UP","components":{"db":{"status":"UP"},"diskSpace":{"status":"UP"}}}

curl http://localhost:8080/actuator/info
# Application info
```

### Swagger / OpenAPI: API Documentation

**OpenAPI** is a specification for describing REST APIs. **Swagger UI** is a web interface that reads an OpenAPI spec and creates interactive documentation.

With `springdoc-openapi`, documentation is generated **automatically** from your controller code.

Add the dependency:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

That's it. Run your app and visit:
- **Swagger UI**: `http://localhost:8080/swagger-ui.html`
- **OpenAPI JSON**: `http://localhost:8080/v3/api-docs`

You'll see all your endpoints listed with their parameters, request bodies, and response types — generated from your Java code.

---

## Code Examples

### Adding Logging to BookService

```java
package com.bookshelf.service;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
// ... other imports ...

@Service
public class BookService {

    private static final Logger log = LoggerFactory.getLogger(BookService.class);

    private final BookRepository bookRepository;
    private final AuthorRepository authorRepository;

    public BookService(BookRepository bookRepository, AuthorRepository authorRepository) {
        this.bookRepository = bookRepository;
        this.authorRepository = authorRepository;
    }

    public List<BookResponse> getAllBooks() {
        log.debug("Fetching all books");
        List<BookResponse> books = bookRepository.findAll().stream()
                .map(this::toResponse)
                .toList();
        log.info("Retrieved {} books", books.size());
        return books;
    }

    public BookResponse getBookById(Long id) {
        log.debug("Fetching book with id: {}", id);
        return bookRepository.findById(id)
                .map(this::toResponse)
                .orElseThrow(() -> {
                    log.warn("Book not found with id: {}", id);
                    return new BookNotFoundException(id);
                });
    }

    public BookResponse createBook(BookRequest request) {
        log.info("Creating book: title='{}', authorId={}", request.title(), request.authorId());

        Author author = authorRepository.findById(request.authorId())
                .orElseThrow(() -> {
                    log.error("Author not found with id: {} while creating book '{}'", 
                              request.authorId(), request.title());
                    return new RuntimeException("Author not found with id: " + request.authorId());
                });

        Book book = new Book();
        book.setTitle(request.title());
        book.setAuthor(author);
        book.setPages(request.pages());

        Book saved = bookRepository.save(book);
        log.info("Book created successfully: id={}, title='{}'", saved.getId(), saved.getTitle());
        return toResponse(saved);
    }

    public void deleteBook(Long id) {
        log.info("Deleting book with id: {}", id);
        if (!bookRepository.existsById(id)) {
            log.warn("Attempted to delete non-existent book: id={}", id);
            throw new BookNotFoundException(id);
        }
        bookRepository.deleteById(id);
        log.info("Book deleted: id={}", id);
    }

    // ... rest of methods ...
}
```

### Adding Logging to GlobalExceptionHandler

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> details = ex.getBindingResult().getFieldErrors().stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .toList();

        log.warn("Validation failed: {}", details);
        // ... return error response ...
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error occurred", ex);  // Log the full stack trace
        // ... return generic error response (don't expose internals to client) ...
    }
}
```

### Swagger Annotations (Optional Enhancement)

You can add descriptions to make the documentation richer:

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@RequestMapping("/api/books")
@Tag(name = "Books", description = "Book management endpoints")
public class BookController {

    @Operation(
        summary = "Get all books",
        description = "Returns a list of all books in the library"
    )
    @ApiResponse(responseCode = "200", description = "List of books")
    @GetMapping
    public ResponseEntity<List<BookResponse>> getAllBooks() {
        return ResponseEntity.ok(bookService.getAllBooks());
    }

    @Operation(summary = "Get a book by ID")
    @ApiResponse(responseCode = "200", description = "Book found")
    @ApiResponse(responseCode = "404", description = "Book not found")
    @GetMapping("/{id}")
    public ResponseEntity<BookResponse> getBookById(
            @Parameter(description = "ID of the book to retrieve")
            @PathVariable Long id
    ) {
        return ResponseEntity.ok(bookService.getBookById(id));
    }
}
```

---

## Exercise: Add Logging and Documentation to BookShelf

**Goal**: Make your application observable and self-documenting.

### Tasks

1. Add logging to `BookService` (INFO for creates/deletes, DEBUG for reads, WARN for not-found, ERROR for unexpected failures)
2. Add logging to `GlobalExceptionHandler`
3. Add Spring Boot Actuator and verify `/actuator/health`
4. Add `springdoc-openapi` and view your API at `/swagger-ui.html`
5. (Optional) Add Swagger annotations for richer documentation

### Verification

```bash
# Check health
curl http://localhost:8080/actuator/health

# Create a book and watch the console for log messages
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "authorId": 1, "pages": 412}'

# Open Swagger UI in browser
open http://localhost:8080/swagger-ui.html
```

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Using `System.out.println()` | Use a logger. `System.out` has no levels, no configuration, no context. |
| String concatenation in log messages | Use `{}` placeholders: `log.info("Book: {}", id)` not `log.info("Book: " + id)`. Placeholders are only evaluated if the log level is active. |
| Logging sensitive data | Never log passwords, tokens, credit card numbers, or personal data. |
| Logging at the wrong level | Don't use ERROR for expected business cases (like "book not found"). That's WARN at most. |
| Not including exceptions in error logs | `log.error("Failed", ex)` — always include the exception as the last argument for the full stack trace. |

---

## Key Takeaways

- [ ] Use SLF4J for logging — one Logger per class
- [ ] Use `{}` placeholders, not string concatenation
- [ ] Choose the right level: ERROR (broken) > WARN (unexpected) > INFO (events) > DEBUG (details)
- [ ] Actuator provides operational endpoints (/health, /info, /metrics)
- [ ] springdoc-openapi generates interactive API documentation automatically
- [ ] Never log sensitive information

---

## Quick Quiz

1. Why use `log.info("Book: {}", id)` instead of `log.info("Book: " + id)`?
2. When should you use ERROR vs. WARN?
3. What does `management.endpoints.web.exposure.include=health` do?
4. Where do you access Swagger UI?
5. Should you log the full exception object for a 404 "not found" error? Why or why not?

---

*Next: `18-security-basics.md` — Protect your API endpoints →*
