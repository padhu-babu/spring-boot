# Chapter 7: Controllers — Handling HTTP Requests

> ⏱ Estimated time: 70 minutes

## What You'll Learn

- What a controller is and how it works
- All the request mapping annotations (`@GetMapping`, `@PostMapping`, etc.)
- How to read path variables, query parameters, and request bodies
- How to return different types of data from your endpoints
- How to build BookShelf v1 with hardcoded data

---

## Concepts

### What Is a Controller?

A controller is a Java class that handles incoming HTTP requests. It's the **front door** of your application — every request enters through a controller.

When a request arrives at your Spring Boot app, the framework looks at the HTTP method and URL path and finds the matching controller method. This process is called **routing**.

```
GET /api/books/42
    ↓
Spring looks for a method annotated with @GetMapping("/api/books/{id}")
    ↓
Calls that method with id=42
    ↓
Takes the return value, converts it to JSON, sends it back
```

### The Key Annotations

#### `@RestController`

Marks a class as a controller that returns **data** (not HTML pages). Every public method in this class can handle HTTP requests.

```java
@RestController
public class BookController {
    // methods here handle HTTP requests
}
```

`@RestController` = `@Controller` + `@ResponseBody`. It means "every method's return value is the response body" (usually converted to JSON).

#### `@RequestMapping`

Sets a **base path** for all endpoints in this controller:

```java
@RestController
@RequestMapping("/api/books")  // All paths in this controller start with /api/books
public class BookController {

    @GetMapping           // Handles GET /api/books
    public List<Book> getAll() { ... }

    @GetMapping("/{id}")  // Handles GET /api/books/{id}
    public Book getById(@PathVariable Long id) { ... }
}
```

#### HTTP Method Annotations

| Annotation | HTTP Method | Typical Use |
|------------|-------------|-------------|
| `@GetMapping` | GET | Read/retrieve data |
| `@PostMapping` | POST | Create new data |
| `@PutMapping` | PUT | Update/replace data |
| `@PatchMapping` | PATCH | Partially update data |
| `@DeleteMapping` | DELETE | Remove data |

### Reading Data from Requests

There are three ways clients send data to your API:

#### 1. Path Variables — `@PathVariable`

Data embedded in the URL path:

```
GET /api/books/42
              ↑
              This is a path variable
```

```java
@GetMapping("/{id}")
public Book getById(@PathVariable Long id) {
    // id = 42
    return findBook(id);
}
```

You can have multiple path variables:

```java
@GetMapping("/{authorId}/books/{bookId}")
public Book getAuthorBook(
    @PathVariable Long authorId,
    @PathVariable Long bookId
) {
    // authorId and bookId extracted from the URL
}
```

#### 2. Query Parameters — `@RequestParam`

Data after the `?` in the URL:

```
GET /api/books?genre=fiction&sort=title
               ↑              ↑
               Query parameters
```

```java
@GetMapping
public List<Book> search(
    @RequestParam String genre,                    // required by default
    @RequestParam(defaultValue = "title") String sort,  // optional with default
    @RequestParam(required = false) String author       // optional, can be null
) {
    // genre = "fiction", sort = "title", author = null
}
```

**When to use path variables vs. query parameters:**
- **Path variables** → identifying a specific resource (`/books/42`)
- **Query parameters** → filtering, sorting, pagination (`/books?genre=fiction&page=2`)

#### 3. Request Body — `@RequestBody`

Data sent in the HTTP body (typically JSON):

```http
POST /api/books
Content-Type: application/json

{
    "title": "Dune",
    "author": "Frank Herbert",
    "pages": 412
}
```

```java
@PostMapping
public Book create(@RequestBody BookRequest request) {
    // Spring automatically converts the JSON to a BookRequest object
    // request.getTitle() = "Dune"
    // request.getAuthor() = "Frank Herbert"
    // request.getPages() = 412
}
```

The `BookRequest` class just needs matching field names:

```java
public class BookRequest {
    private String title;
    private String author;
    private int pages;

    // Getters and setters (or use a record — covered in Chapter 11)
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public int getPages() { return pages; }
    public void setPages(int pages) { this.pages = pages; }
}
```

Spring uses **Jackson** (a JSON library) to automatically convert between JSON and Java objects. The JSON field names must match the Java field names (or getter/setter names).

### Return Types

What you return from a controller method becomes the response body:

```java
// Return a String → sent as plain text
@GetMapping("/hello")
public String hello() {
    return "Hello!";
}

// Return an object → automatically converted to JSON
@GetMapping("/{id}")
public Book getById(@PathVariable Long id) {
    return new Book(id, "Dune", "Frank Herbert", 412);
}
// Response: {"id":42,"title":"Dune","author":"Frank Herbert","pages":412}

// Return a List → converted to a JSON array
@GetMapping
public List<Book> getAll() {
    return List.of(
        new Book(1L, "Dune", "Frank Herbert", 412),
        new Book(2L, "1984", "George Orwell", 328)
    );
}
// Response: [{"id":1,...},{"id":2,...}]
```

> 🧠 **Think Like a Backend Engineer**: The controller's only job is to **receive the request** and **send the response**. It should NOT contain business logic. That belongs in the service layer (Chapter 10). For now, we'll put everything in the controller, then refactor.

---

## Code Examples

### BookShelf v1: Complete Controller

Delete the `HelloController.java` from Chapter 6 and create `BookController.java`:

**First, create the Book class** at `src/main/java/com/bookshelf/Book.java`:

```java
package com.bookshelf;

public class Book {
    private Long id;
    private String title;
    private String author;
    private int pages;

    // No-arg constructor (needed by Jackson for JSON deserialization)
    public Book() {}

    // All-arg constructor
    public Book(Long id, String title, String author, int pages) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.pages = pages;
    }

    // Getters and setters
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

**Then, create the controller** at `src/main/java/com/bookshelf/BookController.java`:

```java
package com.bookshelf;

import org.springframework.web.bind.annotation.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/books")
public class BookController {

    // In-memory storage (temporary — we'll replace with a database in Chapter 12)
    private final List<Book> books = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    // GET /api/books — get all books
    @GetMapping
    public List<Book> getAllBooks() {
        return books;
    }

    // GET /api/books/{id} — get one book by ID
    @GetMapping("/{id}")
    public Book getBookById(@PathVariable Long id) {
        return books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst()
                .orElse(null);  // Returns null → Spring sends empty response (we'll fix this in Ch 9)
    }

    // POST /api/books — create a new book
    @PostMapping
    public Book createBook(@RequestBody Book book) {
        book.setId(idCounter.getAndIncrement());
        books.add(book);
        return book;
    }

    // PUT /api/books/{id} — update a book
    @PutMapping("/{id}")
    public Book updateBook(@PathVariable Long id, @RequestBody Book updatedBook) {
        for (int i = 0; i < books.size(); i++) {
            if (books.get(i).getId().equals(id)) {
                updatedBook.setId(id);
                books.set(i, updatedBook);
                return updatedBook;
            }
        }
        return null;  // Not found — we'll fix this in Chapter 9
    }

    // DELETE /api/books/{id} — delete a book
    @DeleteMapping("/{id}")
    public void deleteBook(@PathVariable Long id) {
        books.removeIf(b -> b.getId().equals(id));
    }

    // GET /api/books/search?title=dune — search by title
    @GetMapping("/search")
    public List<Book> searchBooks(@RequestParam String title) {
        return books.stream()
                .filter(b -> b.getTitle().toLowerCase().contains(title.toLowerCase()))
                .toList();
    }
}
```

### Testing Your API

Run the app (`mvn spring-boot:run`) and test with curl:

```bash
# 1. List all books (empty at first)
curl http://localhost:8080/api/books
# Response: []

# 2. Create a book
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "author": "Frank Herbert", "pages": 412}'
# Response: {"id":1,"title":"Dune","author":"Frank Herbert","pages":412}

# 3. Create another book
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "1984", "author": "George Orwell", "pages": 328}'
# Response: {"id":2,"title":"1984","author":"George Orwell","pages":328}

# 4. List all books (now has 2)
curl http://localhost:8080/api/books
# Response: [{"id":1,...},{"id":2,...}]

# 5. Get one book
curl http://localhost:8080/api/books/1
# Response: {"id":1,"title":"Dune","author":"Frank Herbert","pages":412}

# 6. Update a book
curl -X PUT http://localhost:8080/api/books/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune (Revised)", "author": "Frank Herbert", "pages": 420}'
# Response: {"id":1,"title":"Dune (Revised)","author":"Frank Herbert","pages":420}

# 7. Search by title
curl "http://localhost:8080/api/books/search?title=dune"
# Response: [{"id":1,"title":"Dune (Revised)","author":"Frank Herbert","pages":420}]

# 8. Delete a book
curl -X DELETE http://localhost:8080/api/books/1

# 9. Verify deletion
curl http://localhost:8080/api/books
# Response: [{"id":2,"title":"1984","author":"George Orwell","pages":328}]
```

**You just built a working CRUD API!** It stores data in memory (it disappears when you restart), but the HTTP interface is real.

---

## Exercise: Build BookShelf v1

**Goal**: Build the complete BookShelf v1 controller from the code examples above.

### Tasks

1. Create the `Book` class with fields: `id`, `title`, `author`, `pages`
2. Create the `BookController` with all 6 endpoints (list, get, create, update, delete, search)
3. Run the app and test every endpoint with curl
4. Verify the full lifecycle: create → read → update → read → delete → read

### Stretch Goals

Add these extra endpoints:

```java
// Count total books
@GetMapping("/count")
public int countBooks() {
    return books.size();
}

// Get books by author
@GetMapping("/by-author")
public List<Book> getByAuthor(@RequestParam String author) {
    return books.stream()
            .filter(b -> b.getAuthor().toLowerCase().contains(author.toLowerCase()))
            .toList();
}
```

### Observe and Note

After running your tests, answer these questions:
1. What happens when you `GET /api/books/999` (a book that doesn't exist)?
2. What HTTP status code does your create endpoint return? Is it the right one?
3. What happens if you restart the server? Are your books still there?
4. What happens if you POST without the `Content-Type: application/json` header?

These are all problems we'll fix in upcoming chapters.

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Forgetting `@RequestBody` on POST/PUT methods | Without it, Spring won't parse the JSON body. Your object will have all null/default values. |
| Forgetting `Content-Type: application/json` header in curl | Spring won't know the body is JSON and will reject it with a 415 (Unsupported Media Type). |
| Using `@RequestParam` when you mean `@PathVariable` | `@RequestParam` reads from `?key=value`. `@PathVariable` reads from `/path/{value}`. They're different sources. |
| Putting business logic in the controller | Controllers should only receive requests and send responses. Logic belongs in the service layer. We'll refactor in Chapter 10. |
| Missing the no-arg constructor on your data class | Jackson needs it to create the object before setting fields. Without it: `HttpMessageNotReadableException`. |

---

## Key Takeaways

- [ ] A controller is a class annotated with `@RestController` that handles HTTP requests
- [ ] `@RequestMapping` sets a base path for all endpoints in a controller
- [ ] `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` map methods to HTTP verbs
- [ ] `@PathVariable` reads from the URL path, `@RequestParam` reads from query parameters
- [ ] `@RequestBody` reads the JSON body and converts it to a Java object
- [ ] Return values are automatically converted to JSON
- [ ] I built a working CRUD API for books

---

## Quick Quiz

1. What's the difference between `@Controller` and `@RestController`?
2. Write the annotation for a method that handles `DELETE /api/users/42`.
3. How do you make a query parameter optional?
4. What does Jackson do? When does it run?
5. Why should a controller NOT contain business logic like "a book must have at least 10 pages"?

---

*Next: `08-dependency-injection.md` — The most important concept in Spring →*
