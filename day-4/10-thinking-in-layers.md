# Chapter 10: Thinking in Layers

> ⏱ Estimated time: 45 minutes

## What You'll Learn

- Why layered architecture exists
- The three layers: Controller, Service, Repository
- What each layer is responsible for (and NOT responsible for)
- How data flows through the layers
- How to organize your code into packages

---

## Concepts

### Why Layers?

Imagine a restaurant where one person takes orders, cooks, does inventory, handles payments, and cleans tables. Chaos.

Now imagine the same restaurant with dedicated roles:
- **Host** — greets customers, seats them (Controller)
- **Chef** — cooks the food, knows the recipes (Service)
- **Pantry manager** — stores and retrieves ingredients (Repository)

Each person has a clear job. They communicate through defined channels. If the pantry manager changes suppliers, the chef doesn't need to know. If you hire a new host, the recipes don't change.

This is **Separation of Concerns** — the most important principle in software architecture.

### The Three-Layer Architecture

```
                HTTP Request
                     │
                     ▼
┌──────────────────────────────────────┐
│         CONTROLLER LAYER             │
│                                      │
│  Responsibility:                     │
│  • Receive HTTP requests             │
│  • Extract path variables, params    │
│  • Call the appropriate service      │
│  • Return HTTP response with         │
│    correct status code               │
│                                      │
│  Does NOT:                           │
│  • Contain business logic            │
│  • Talk to the database              │
│  • Know how data is stored           │
└──────────────────┬───────────────────┘
                   │ Java method call
                   ▼
┌──────────────────────────────────────┐
│           SERVICE LAYER              │
│                                      │
│  Responsibility:                     │
│  • Business logic and rules          │
│  • Validation (beyond basic format)  │
│  • Orchestrate operations            │
│  • Transform data between DTOs       │
│    and entities                      │
│                                      │
│  Does NOT:                           │
│  • Know about HTTP (no request/      │
│    response objects)                 │
│  • Write SQL or manage connections   │
│  • Know about JSON                   │
└──────────────────┬───────────────────┘
                   │ Java method call
                   ▼
┌──────────────────────────────────────┐
│         REPOSITORY LAYER             │
│                                      │
│  Responsibility:                     │
│  • Store and retrieve data           │
│  • Database queries                  │
│  • Data access logic                 │
│                                      │
│  Does NOT:                           │
│  • Contain business rules            │
│  • Know about HTTP                   │
│  • Decide what data to return        │
│    (it returns what's asked for)     │
└──────────────────┬───────────────────┘
                   │
                   ▼
              DATABASE
```

### The Rules

**Each layer only talks to the layer directly below it:**
- Controller → Service ✅
- Service → Repository ✅
- Controller → Repository ❌ (skip layers = chaos)
- Repository → Controller ❌ (wrong direction)

**Each layer is ignorant of the layers above it:**
- The service doesn't know it's being called by a controller (it could be called by a test, a scheduled job, or another service)
- The repository doesn't know it's being called by a service

### Why This Matters in Practice

**Scenario**: Your boss says "We need to add a rule: users can't create more than 100 books."

With layers:
- You add the rule in `BookService.createBook()` — one place
- The controller doesn't change
- The repository doesn't change
- If you have a scheduled import job that also creates books, it goes through the same service — the rule applies everywhere

Without layers:
- The rule might need to be added in the controller, the import job, the admin panel, and anywhere else that creates books
- Miss one? The rule is inconsistently applied

**Scenario**: You need to switch from an in-memory list to a real database.

With layers:
- You change the repository implementation
- The service doesn't change (it calls the same repository methods)
- The controller doesn't change

Without layers:
- Database code is mixed with business logic and HTTP handling
- Changes ripple everywhere

### Data Flow Example

Let's trace `POST /api/books` with a body `{"title": "Dune", "pages": 412}`:

```
1. CONTROLLER receives the HTTP request
   - Extracts: POST method, /api/books path
   - Deserializes body: JSON → BookRequest object
   - Calls: bookService.createBook(bookRequest)

2. SERVICE receives the BookRequest
   - Validates: title is not empty ✓, pages > 0 ✓
   - Checks: does this book already exist? (asks repository)
   - Transforms: BookRequest → Book entity
   - Calls: bookRepository.save(book)
   - Transforms: Book entity → BookResponse
   - Returns: BookResponse

3. REPOSITORY receives the Book entity
   - Saves it to the database
   - Returns the saved entity (with generated ID)

4. SERVICE returns BookResponse to controller

5. CONTROLLER returns ResponseEntity<BookResponse> with status 201
```

> 🧠 **Think Like a Backend Engineer**: When you get a new requirement, ask yourself: "Which layer does this belong to?"
> - "Show me the data differently" → Controller (change the response format)
> - "Add a business rule" → Service (add validation/logic)
> - "Query the data differently" → Repository (change the query)

---

## Code Examples

### BookShelf v2: Package Organization

Restructure your project into packages:

```
src/main/java/com/bookshelf/
├── BookshelfApplication.java
├── controller/
│   └── BookController.java
├── service/
│   └── BookService.java
├── repository/
│   └── BookRepository.java
└── model/
    └── Book.java
```

**Create `BookRepository.java`** at `src/main/java/com/bookshelf/repository/BookRepository.java`:

```java
package com.bookshelf.repository;

import com.bookshelf.model.Book;
import org.springframework.stereotype.Repository;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class BookRepository {

    private final List<Book> books = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    public List<Book> findAll() {
        return new ArrayList<>(books);  // Return a copy, not the original
    }

    public Optional<Book> findById(Long id) {
        return books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst();
    }

    public Book save(Book book) {
        if (book.getId() == null) {
            // New book — assign an ID
            book.setId(idCounter.getAndIncrement());
            books.add(book);
        } else {
            // Existing book — update in place
            for (int i = 0; i < books.size(); i++) {
                if (books.get(i).getId().equals(book.getId())) {
                    books.set(i, book);
                    break;
                }
            }
        }
        return book;
    }

    public boolean deleteById(Long id) {
        return books.removeIf(b -> b.getId().equals(id));
    }

    public List<Book> findByTitleContaining(String title) {
        return books.stream()
                .filter(b -> b.getTitle().toLowerCase().contains(title.toLowerCase()))
                .toList();
    }
}
```

**Move `Book.java`** to `src/main/java/com/bookshelf/model/Book.java`:

```java
package com.bookshelf.model;

// Same Book class as before, just in a new package
public class Book {
    private Long id;
    private String title;
    private String author;
    private int pages;

    public Book() {}

    public Book(Long id, String title, String author, int pages) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.pages = pages;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public int getPages() { return pages; }
    public void setPages(int pages) { this.pages = pages; }
}
```

**Update `BookService.java`** at `src/main/java/com/bookshelf/service/BookService.java`:

```java
package com.bookshelf.service;

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

    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Optional<Book> getBookById(Long id) {
        return bookRepository.findById(id);
    }

    public Book createBook(Book book) {
        // Business logic could go here:
        // - Validate that the title isn't empty
        // - Check for duplicates
        // - Apply default values
        return bookRepository.save(book);
    }

    public Optional<Book> updateBook(Long id, Book updatedBook) {
        return bookRepository.findById(id)
                .map(existingBook -> {
                    updatedBook.setId(id);
                    return bookRepository.save(updatedBook);
                });
    }

    public boolean deleteBook(Long id) {
        return bookRepository.deleteById(id);
    }

    public List<Book> searchByTitle(String title) {
        return bookRepository.findByTitleContaining(title);
    }
}
```

**Update `BookController.java`** at `src/main/java/com/bookshelf/controller/BookController.java`:

```java
package com.bookshelf.controller;

import com.bookshelf.model.Book;
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
    public ResponseEntity<List<Book>> getAllBooks() {
        return ResponseEntity.ok(bookService.getAllBooks());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        return bookService.getBookById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        Book created = bookService.createBook(book);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(
            @PathVariable Long id,
            @RequestBody Book updatedBook
    ) {
        return bookService.updateBook(id, updatedBook)
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
    public ResponseEntity<List<Book>> searchBooks(@RequestParam String title) {
        return ResponseEntity.ok(bookService.searchByTitle(title));
    }
}
```

Notice the clean dependency chain:
```
BookController → BookService → BookRepository
     ↓                ↓              ↓
 HTTP stuff      Business rules   Data storage
```

---

## Exercise: Restructure BookShelf into Layers

**Goal**: Reorganize your existing code into the three-layer architecture.

### Tasks

1. Create the package structure: `controller/`, `service/`, `repository/`, `model/`
2. Move `Book.java` to `model/`
3. Create `BookRepository` with `@Repository` in `repository/`
4. Move data management code from `BookService` to `BookRepository`
5. Update `BookService` to use `BookRepository` via DI
6. Move `BookController` to `controller/`
7. Update imports everywhere
8. Run and test — everything should work exactly the same

### Verification

Run all the same curl commands from Chapter 9. The API behavior should be identical — you've only reorganized the internal structure.

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Controller calling Repository directly | Always go through the Service layer. The controller should never know about data storage. |
| Putting business logic in the controller | "A book must have at least 10 pages" belongs in the service, not the controller. |
| Putting HTTP concepts in the service | The service should never import `HttpServletRequest`, `ResponseEntity`, or `@PathVariable`. It knows nothing about HTTP. |
| Having the repository make business decisions | The repository stores and retrieves. It doesn't decide *whether* to store — that's the service's job. |
| Skipping the service layer for "simple" operations | Even simple pass-through services are valuable — they give you a place to add logic later without restructuring. |

---

## Key Takeaways

- [ ] The three layers are Controller (HTTP), Service (logic), Repository (data)
- [ ] Each layer only talks to the layer below it
- [ ] Separation of concerns makes code easier to change, test, and understand
- [ ] The controller is thin — it translates HTTP to method calls
- [ ] The service contains all business rules
- [ ] The repository handles data storage and retrieval
- [ ] Package structure reflects the layered architecture

---

## Quick Quiz

1. A new requirement says "books with more than 1000 pages get a 'tome' badge." Which layer handles this?
2. You need to switch from ArrayList storage to a PostgreSQL database. Which layer(s) change?
3. Why shouldn't the controller call the repository directly, even if the service just passes through?
4. Where do you put the rule "only admin users can delete books"?
5. Your `BookService.createBook()` currently just calls `repository.save()`. Is this okay, or should you remove the service?

---

*Next: `11-entities-and-dtos.md` — Why your API and your database need different classes →*
