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

Okay, picture this. You've been coding along, feeling great about your `Book` class. It does *everything*:
- It represents data in the database
- It represents what the client sends (request)
- It represents what the client receives (response)

One class to rule them all! Efficient, right? Elegant, even?

**Yeah... no.** This is the "I'll just use one suitcase for ALL my trips" approach. Weekend getaway? Same suitcase. Two-week European vacation? Same suitcase. Moving across the country? SAME SUITCASE.

Let's see how this blows up in your face.

**Problem 1: Exposing internal data**

Your database might have fields the client should **never, ever, EVER** see:

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

> ⚠️ **Watch it!** Returning entities directly from your API is like handing someone your diary when they asked for your phone number. Sure, the phone number is *in* there, but so is... everything else.

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

The client doesn't send `id` (the server generates it) or `createdAt` (the server sets it). Using one class means accepting fields that shouldn't be in the request and hiding fields that shouldn't be in the response. It's a mess.

**Problem 3: Changing the API changes the database (and vice versa)**

If you rename a field in your entity to match a new database column, the API field name changes too. One tiny database migration, and suddenly every frontend developer on your team is filing bug reports.

> 🗣️ **Overheard at the coffee shop:** *"I just renamed a column in my database and now the mobile app is crashing."* — A developer who used one class for everything.

The API and database should evolve independently. Full stop.

### The Solution: Separate Classes

Here's the big idea: **use different classes for different purposes.** Each one has *exactly* the fields it needs. No more, no less.

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

> 🧠 **Brain Power:** Look at the diagram above. The Entity sits right in the middle, touching the database. The DTOs sit on the edges, touching the client. Why do you think the conversion happens in the service layer and not the controller?

### Entity

An **Entity** represents a row in a database table. It maps directly to the database schema. It's the "truth" — the full, unfiltered, warts-and-all representation of your data.

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

A **DTO** is a simple class that carries data between layers — specifically between the controller and the client. Think of it as a *curated view* of your data. You pick exactly which fields get in.

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

> 🎯 **Key Point:** An Entity says "here's EVERYTHING that's in the database." A DTO says "here's ONLY what you need to know."

---

### 🗣️ Fireside Chat: Entity vs. DTO

*Tonight's episode: "Who's More Important?" We sit down with two Java classes who have some... strong opinions about each other.*

---

**Interviewer:** Thanks for being here, both of you. Let's start with introductions. Entity, you first.

**Entity:** Thank you. I AM the data. Let's be clear about that. When something gets saved to the database, it goes through *me*. When it comes back out, it comes through *me*. Without me, there is no application. There's just... a nice-looking form with nowhere to put anything.

**DTO:** Oh, here we go.

**Entity:** What? It's true!

**Interviewer:** DTO, your turn.

**DTO:** Sure. I PROTECT the data. That's my whole job. See, Entity over here is an open book — pun intended. Every dirty little secret the database has? Entity will happily parade it right out the front door. Password hashes, internal flags, soft-delete markers...

**Entity:** That's not fair! I'm just being *honest*. I represent the truth!

**DTO:** And sometimes the truth needs an editor. Look, nobody's saying you're not important. But you shouldn't be walking up to clients and showing them everything. That's *my* job — I decide what gets shown and what stays hidden.

**Interviewer:** Entity, what do you say to that?

**Entity:** I say... fine, maybe I'm not great at boundaries. But you know what? Without me, you're nothing. You're literally *made from me*. Someone takes my fields, picks the ones they like, and creates you. You're a... a highlight reel!

**DTO:** A *carefully curated* highlight reel. And I'm proud of that.

**Interviewer:** Can you two work together?

**Entity:** We literally have to. The service layer converts between us. I just wish I got more credit.

**DTO:** And I just wish you'd stop trying to talk to clients directly. It always ends badly.

**Entity:** *sighs* Fair enough.

---

### Java Records: Perfect for DTOs

Now here's where Java gives you a gift. Java 16+ introduced **records** — immutable data classes that are basically DTOs on autopilot:

```java
// 1 line instead of 30
public record BookRequest(String title, String author, int pages) {}

public record BookResponse(Long id, String title, String author, int pages, LocalDateTime createdAt) {}
```

Wait, WHAT?! That tiny thing replaces a whole class with fields, constructors, getters, equals, hashCode, and toString?

Yep. A record automatically generates:
- Constructor with all fields
- Getter methods (`.title()`, `.author()`, etc. — no `get` prefix)
- `equals()`, `hashCode()`, `toString()`
- Fields are `final` (immutable)

```java
// Using a record
BookRequest request = new BookRequest("Dune", "Frank Herbert", 412);
String title = request.title();    // "Dune" (note: no "get" prefix)
```

> 💡 **There are no Dumb Questions**
>
> **Q: Why are records "perfect" for DTOs but not for entities?**
>
> A: Records are immutable — once you create one, you can't change its fields. That's exactly what you want for DTOs: data comes in, you read it, done. But entities often need to be *modified* (like updating a book's title), so they need setters. Records don't have setters. Also, JPA (which we'll see in Chapter 12) needs to be able to create empty objects and fill in fields, which records don't support well.
>
> **Q: What if my request and response have the same fields? Do I still need two DTOs?**
>
> A: You *could* use one, but you probably shouldn't. They tend to diverge over time. Today they look the same; next month you add `createdAt` to the response but not the request. Start with two. Future-you will thank present-you.
>
> **Q: Is "DTO" a Spring thing?**
>
> A: Nope! DTOs are a general design pattern used across all languages and frameworks. Spring doesn't even have a `@DTO` annotation. It's just a plain Java class (or record) that you use by convention.

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

You can do this manually (simple and clear) or use a mapping library (for large projects). For our BookShelf app, manual mapping is the way to go — it's easy to read and understand.

> 🧠 **Brain Power:** Why does the controller never touch the Entity directly? What would happen if the controller created Entity objects and passed them straight to the repository, skipping the service layer entirely?

---

## Code Examples

### BookShelf: Adding DTOs

Alright, time to get your hands dirty. Here's the plan: we're going to split our `Book` class into three roles — the entity stays as `Book`, and we'll add `BookRequest` and `BookResponse` records.

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

See what happened there? The client sent three fields. The server added `id` and `createdAt` on its own and sent back five fields. The `BookRequest` record only accepted what the client *should* send. The `BookResponse` record only returned what the client *should* see. Boundaries, people. Boundaries.

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

> 🧠 **Brain Power:** Before you start coding, sketch out on paper (yes, actual paper!) which fields belong in `BookRequest`, which belong in `BookResponse`, and which belong in the `Book` entity. Are there any fields that appear in all three? Any that appear in only one?

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Using the entity directly in the API | Couples your API to your database schema. Any DB change breaks the API. |
| Creating DTOs for every single class | Not everything needs a DTO. Start with DTOs for classes exposed through your API. Internal classes can stay simple. |
| Doing mapping in the controller | Mapping belongs in the service layer. The controller shouldn't know about entities. |
| Forgetting that records are immutable | You can't do `request.setTitle("new")`. Records are read-only by design. |

> ⚠️ **Watch it!** The most common mistake beginners make? Putting `toEntity()` and `toResponse()` methods in the controller. Don't do it! The controller's job is to handle HTTP concerns (status codes, headers, request parameters). Mapping is business logic — it belongs in the service.

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 27](../../appendices/E-coding-exercises.md#exercise-27) | Create a JPA Entity | ⭐ |
| [Exercise 28](../../appendices/E-coding-exercises.md#exercise-28) | Create Request and Response DTOs | ⭐⭐ |

Solutions are in [Appendix F](../../appendices/F-exercise-solutions.md).

---

## Key Takeaways

- [ ] Entities represent database rows; DTOs represent API data
- [ ] Separating them prevents leaking internal data and couples API ↔ database
- [ ] Request DTOs define what clients send; Response DTOs define what they receive
- [ ] Java records are perfect for DTOs (concise, immutable)
- [ ] The service layer handles Entity ↔ DTO conversion

> 🎯 **Key Point:** If you take away just ONE thing from this chapter, let it be this: **your database is not your API.** The moment you treat them as the same thing, you've created a ticking time bomb. Separate your concerns. Use Entities for the database. Use DTOs for the API. Let the service layer translate between the two worlds.

---

## Quick Quiz

1. Why is returning a `User` entity with a `passwordHash` field dangerous?
2. When creating a book, should the `BookRequest` include an `id` field? Why or why not?
3. What Java feature makes DTOs concise? What do you get for free?
4. Which layer is responsible for converting between DTOs and entities?
5. A `BookResponse` has a `createdAt` field but `BookRequest` doesn't. Why?

> 💡 **There are no Dumb Questions**
>
> **Q: This feels like a lot of extra code just to avoid returning one or two fields. Is it really worth it?**
>
> A: Right now, with a simple Book, it might feel like overkill. But applications grow. Fields get added. Security requirements tighten. Six months from now, when someone asks "can we add an `internalNotes` field to the database without exposing it to clients?" — you'll be glad the separation is already in place. The cost of adding DTOs early is small. The cost of retrofitting them later is... not.

---

*Next: `12-databases-with-jpa.md` — Time to persist data for real →*
