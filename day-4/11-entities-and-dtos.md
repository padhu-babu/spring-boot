# Chapter 11: Entities and DTOs

> ⏱ Estimated time: 45 minutes

## What You'll Learn

- What an Entity is (database representation)
- What a DTO is (API representation)
- Why you need both — and when using one class backfires
- Java records for DTOs
- How to map between Entities and DTOs

---

## Concepts

### The Problem with One Class for Everything

Right now, our `Book` class does everything:
- It represents data in the database
- It represents what the client sends (request)
- It represents what the client receives (response)

This seems efficient but creates real problems:

**Problem 1: Exposing internal data**

Your database might have fields the client should never see:

```java
public class User {
    private Long id;
    private String name;
    private String email;
    private String passwordHash;   // ← NEVER send this to the client!
    private LocalDateTime createdAt;
    private boolean isDeleted;     // ← Internal flag, client doesn't need this
}
```

If you return the `User` entity directly from your API, the password hash goes to the client.

**Problem 2: Client sends different data than what's stored**

When creating a book, the client sends:
```json
{ "title": "Dune", "author": "Frank Herbert", "pages": 412 }
```

But the database stores:
```json
{ "id": 1, "title": "Dune", "author": "Frank Herbert", "pages": 412, 
  "createdAt": "2025-01-15T10:00:00", "updatedAt": "2025-01-15T10:00:00" }
```

The client doesn't send `id` (the server generates it) or `createdAt` (the server sets it). Using one class means accepting fields that shouldn't be in the request and hiding fields that shouldn't be in the response.

**Problem 3: Changing the API changes the database (and vice versa)**

If you rename a field in your entity to match a new database column, the API field name changes too. The API and database should evolve independently.

### The Solution: Separate Classes

Use different classes for different purposes:

```
CLIENT                    YOUR CODE                    DATABASE
──────                    ─────────                    ────────

BookRequest ──────► Controller
  (what the           │
   client sends)      │ converts to
                      ▼
                   Book (Entity)  ◄──────────► books table
                      │                        (database)
                      │ converts to
                      ▼
               BookResponse ─────────────────► Client
                (what the
                 client receives)
```

### Entity

An **Entity** represents a row in a database table. It maps directly to the database schema.

```java
// Entity — represents the "books" table in the database
public class Book {
    private Long id;          // Primary key
    private String title;
    private String author;
    private int pages;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Constructors, getters, setters
}
```

We'll add JPA annotations (`@Entity`, `@Id`, etc.) in Chapter 12.

### DTO (Data Transfer Object)

A **DTO** is a simple class that carries data between layers — specifically between the controller and the client.

You typically have two DTOs per resource:

**Request DTO** — what the client sends:

```java
// Only the fields the client should provide
public class BookRequest {
    private String title;
    private String author;
    private int pages;
    // No id, no createdAt — the server handles those
}
```

**Response DTO** — what the client receives:

```java
// Only the fields the client should see
public class BookResponse {
    private Long id;
    private String title;
    private String author;
    private int pages;
    private LocalDateTime createdAt;
    // No passwordHash, no internal flags
}
```

### Java Records: Perfect for DTOs

Java 16+ introduced **records** — immutable data classes that are perfect for DTOs:

```java
// 1 line instead of 30
public record BookRequest(String title, String author, int pages) {}

public record BookResponse(Long id, String title, String author, int pages, LocalDateTime createdAt) {}
```

A record automatically generates:
- Constructor with all fields
- Getter methods (`.title()`, `.author()`, etc. — no `get` prefix)
- `equals()`, `hashCode()`, `toString()`
- Fields are `final` (immutable)

```java
// Using a record
BookRequest request = new BookRequest("Dune", "Frank Herbert", 412);
String title = request.title();    // "Dune" (note: no "get" prefix)
```

> Records are ideal for DTOs because DTOs are inherently "carry data, don't change it" objects.

### Mapping Between Entities and DTOs

The service layer is responsible for converting between them:

```
Controller receives BookRequest (DTO)
    ↓
Service converts BookRequest → Book (Entity)
Service calls Repository to save
Repository returns Book (Entity)
Service converts Book → BookResponse (DTO)
    ↓
Controller returns BookResponse (DTO)
```

You can do this manually (simple and clear) or use a mapping library (for large projects).

---

## Code Examples

### BookShelf: Adding DTOs

**Create `BookRequest.java`** at `src/main/java/com/bookshelf/dto/BookRequest.java`:

```java
package com.bookshelf.dto;

public record BookRequest(
    String title,
    String author,
    int pages
) {}
```

**Create `BookResponse.java`** at `src/main/java/com/bookshelf/dto/BookResponse.java`:

```java
package com.bookshelf.dto;

import java.time.LocalDateTime;

public record BookResponse(
    Long id,
    String title,
    String author,
    int pages,
    LocalDateTime createdAt
) {}
```

**Update `Book.java`** entity (add timestamp fields):

```java
package com.bookshelf.model;

import java.time.LocalDateTime;

public class Book {
    private Long id;
    private String title;
    private String author;
    private int pages;
    private LocalDateTime createdAt;

    public Book() {}

    public Book(Long id, String title, String author, int pages) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.pages = pages;
        this.createdAt = LocalDateTime.now();
    }

    // All getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public int getPages() { return pages; }
    public void setPages(int pages) { this.pages = pages; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**Update `BookService.java`** to handle mapping:

```java
package com.bookshelf.service;

import com.bookshelf.dto.BookRequest;
import com.bookshelf.dto.BookResponse;
import com.bookshelf.model.Book;
import com.bookshelf.repository.BookRepository;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
public class BookService {

    private final BookRepository bookRepository;

    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }

    public List<BookResponse> getAllBooks() {
        return bookRepository.findAll().stream()
                .map(this::toResponse)
                .toList();
    }

    public Optional<BookResponse> getBookById(Long id) {
        return bookRepository.findById(id)
                .map(this::toResponse);
    }

    public BookResponse createBook(BookRequest request) {
        Book book = toEntity(request);
        Book saved = bookRepository.save(book);
        return toResponse(saved);
    }

    public Optional<BookResponse> updateBook(Long id, BookRequest request) {
        return bookRepository.findById(id)
                .map(existingBook -> {
                    existingBook.setTitle(request.title());
                    existingBook.setAuthor(request.author());
                    existingBook.setPages(request.pages());
                    return toResponse(bookRepository.save(existingBook));
                });
    }

    public boolean deleteBook(Long id) {
        return bookRepository.deleteById(id);
    }

    public List<BookResponse> searchByTitle(String title) {
        return bookRepository.findByTitleContaining(title).stream()
                .map(this::toResponse)
                .toList();
    }

    // ---- Mapping methods ----

    private Book toEntity(BookRequest request) {
        return new Book(null, request.title(), request.author(), request.pages());
    }

    private BookResponse toResponse(Book book) {
        return new BookResponse(
                book.getId(),
                book.getTitle(),
                book.getAuthor(),
                book.getPages(),
                book.getCreatedAt()
        );
    }
}
```

**Update `BookController.java`** to use DTOs:

```java
package com.bookshelf.controller;

import com.bookshelf.dto.BookRequest;
import com.bookshelf.dto.BookResponse;
import com.bookshelf.service.BookService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping
    public ResponseEntity<List<BookResponse>> getAllBooks() {
        return ResponseEntity.ok(bookService.getAllBooks());
    }

    @GetMapping("/{id}")
    public ResponseEntity<BookResponse> getBookById(@PathVariable Long id) {
        return bookService.getBookById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<BookResponse> createBook(@RequestBody BookRequest request) {
        BookResponse created = bookService.createBook(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<BookResponse> updateBook(
            @PathVariable Long id,
            @RequestBody BookRequest request
    ) {
        return bookService.updateBook(id, request)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        if (bookService.deleteBook(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }

    @GetMapping("/search")
    public ResponseEntity<List<BookResponse>> searchBooks(@RequestParam String title) {
        return ResponseEntity.ok(bookService.searchByTitle(title));
    }
}
```

### Updated Project Structure

```
src/main/java/com/bookshelf/
├── BookshelfApplication.java
├── controller/
│   └── BookController.java
├── dto/
│   ├── BookRequest.java       ← NEW
│   └── BookResponse.java      ← NEW
├── model/
│   └── Book.java              ← UPDATED (added createdAt)
├── repository/
│   └── BookRepository.java
└── service/
    └── BookService.java       ← UPDATED (handles mapping)
```

### Testing

```bash
# Create a book — notice request has no 'id' or 'createdAt'
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "author": "Frank Herbert", "pages": 412}'

# Response includes id and createdAt (set by server):
# {"id":1,"title":"Dune","author":"Frank Herbert","pages":412,"createdAt":"2025-01-15T10:30:00"}
```

---

## Exercise: Add DTOs to BookShelf

**Goal**: Separate your API representation from your data representation.

### Tasks

1. Create `BookRequest` record in `dto/` package
2. Create `BookResponse` record in `dto/` package
3. Add `createdAt` field to the `Book` entity
4. Update `BookService` to convert between DTOs and entities
5. Update `BookController` to use `BookRequest` and `BookResponse`
6. Test all endpoints — behavior should be the same, but responses now include `createdAt`

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Using the entity directly in the API | Couples your API to your database schema. Any DB change breaks the API. |
| Creating DTOs for every single class | Not everything needs a DTO. Start with DTOs for classes exposed through your API. Internal classes can stay simple. |
| Doing mapping in the controller | Mapping belongs in the service layer. The controller shouldn't know about entities. |
| Forgetting that records are immutable | You can't do `request.setTitle("new")`. Records are read-only by design. |

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 27](../appendices/E-coding-exercises.md#exercise-27) | Create a JPA Entity | ⭐ |
| [Exercise 28](../appendices/E-coding-exercises.md#exercise-28) | Create Request and Response DTOs | ⭐⭐ |

Solutions are in [Appendix F](../appendices/F-exercise-solutions.md).

---

## Key Takeaways

- [ ] Entities represent database rows; DTOs represent API data
- [ ] Separating them prevents leaking internal data and couples API ↔ database
- [ ] Request DTOs define what clients send; Response DTOs define what they receive
- [ ] Java records are perfect for DTOs (concise, immutable)
- [ ] The service layer handles Entity ↔ DTO conversion

---

## Quick Quiz

1. Why is returning a `User` entity with a `passwordHash` field dangerous?
2. When creating a book, should the `BookRequest` include an `id` field? Why or why not?
3. What Java feature makes DTOs concise? What do you get for free?
4. Which layer is responsible for converting between DTOs and entities?
5. A `BookResponse` has a `createdAt` field but `BookRequest` doesn't. Why?

---

*Next: `12-databases-with-jpa.md` — Time to persist data for real →*
