# Chapter 8: Dependency Injection Demystified

> ⏱ Estimated time: 60 minutes

## What You'll Learn

- What a "dependency" is in software
- Why tight coupling is a problem
- What Dependency Injection (DI) is and why it matters
- How Spring's DI container works
- The annotations: `@Component`, `@Service`, `@Repository`, `@Autowired`
- Constructor injection (the right way)

---

## Concepts

### What Is a Dependency?

A **dependency** is any object that another object needs to do its job.

```java
public class BookController {
    private BookService bookService;  // ← BookController DEPENDS on BookService
}
```

`BookController` can't function without `BookService`. Therefore, `BookService` is a **dependency** of `BookController`.

In any real application, objects depend on other objects:
- Controller depends on Service
- Service depends on Repository
- Repository depends on DataSource (database connection)

### The Problem: Tight Coupling

The natural way to handle dependencies in Java is to create them yourself:

```java
public class BookController {
    private BookService bookService = new BookService();  // ← Controller creates its own dependency
}
```

This works, but it creates problems:

**Problem 1: Hard to test**

```java
// In a test, you want to use a fake/mock service. But the controller
// creates the real one internally. You can't swap it.
BookController controller = new BookController();
// controller always uses the real BookService — you can't substitute a test version
```

**Problem 2: Hard to change**

```java
// What if BookService needs a database connection now?
public class BookService {
    private Database db;
    
    public BookService(Database db) {  // Constructor changed!
        this.db = db;
    }
}

// Now BookController needs to change too:
public class BookController {
    private BookService bookService = new BookService(new Database("url", "user", "pass"));  // Yikes
}
```

Every class that creates a `BookService` needs to know how to construct it. If the constructor changes, every class that creates it must be updated.

**Problem 3: Single instances**

You probably want ONE `BookService` shared by everything — not a new one for each controller. Managing this manually is error-prone.

### The Solution: Dependency Injection

Instead of creating dependencies yourself, **someone else creates them and gives them to you**.

```java
public class BookController {
    private final BookService bookService;

    // "I need a BookService. Give me one."
    public BookController(BookService bookService) {
        this.bookService = bookService;
    }
}
```

Notice:
- The controller doesn't create the service — it receives it through the constructor
- The controller doesn't know or care *how* the service was created
- You can pass a real service in production and a mock in tests

**Analogy**: 
- **Without DI**: You're a chef who grows their own vegetables, mills their own flour, and raises their own chickens. Changing ingredients means changing your entire supply chain.
- **With DI**: You're a chef who receives ingredients from a supplier. You ask for "chicken" and it arrives. You don't care which farm it came from. Want to switch to organic? Change the supplier, not the recipe.

### Spring's DI Container

Who creates the dependencies and hands them out? **Spring's IoC Container** (also called the Application Context).

When your Spring Boot app starts:

1. Spring scans your code for classes annotated with `@Component`, `@Service`, `@Repository`, `@Controller`, or `@RestController`
2. It creates an instance of each one (called a **bean**)
3. It looks at each bean's constructor to see what it needs
4. It wires them together — passing the right dependencies to the right constructors

```
Spring Boot starts
    ↓
Scans for annotated classes
    ↓
Finds: BookController, BookService
    ↓
Creates BookService (no dependencies needed)
    ↓
Creates BookController, passes BookService to its constructor
    ↓
Both beans are ready — stored in the container
    ↓
When a request arrives, Spring uses the existing BookController bean
```

### The Stereotype Annotations

Spring provides several annotations to mark a class as a bean. They all do the same thing (register the class with the container), but they communicate **intent**:

| Annotation | Use For | Layer |
|------------|---------|-------|
| `@Component` | Generic — any Spring-managed class | Any |
| `@Service` | Business logic classes | Service layer |
| `@Repository` | Data access classes | Repository layer |
| `@Controller` / `@RestController` | Request handling classes | Controller layer |

```java
@Service                    // ← "I am a service. Spring, please manage me."
public class BookService {
    // Spring creates ONE instance of this and shares it everywhere
}

@RestController             // ← "I am a controller. Spring, please manage me."
public class BookController {
    private final BookService bookService;

    public BookController(BookService bookService) {   // ← Spring injects BookService here
        this.bookService = bookService;
    }
}
```

> 🧠 **Think Like a Backend Engineer**: Use `@Service` for services, `@Repository` for data access, and `@RestController` for controllers. Don't use `@Component` for everything — the specific annotations make your code self-documenting.

### Constructor Injection (The Right Way)

There are three ways to inject dependencies in Spring. Only one is recommended:

#### ✅ Constructor Injection (Recommended)

```java
@RestController
public class BookController {
    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }
}
```

Why this is best:
- `final` fields → immutable, thread-safe
- Clear dependencies → you can see everything a class needs in the constructor
- Testable → just pass mocks to the constructor
- Fails fast → if a dependency is missing, the app won't start

**Note**: When a class has **only one constructor**, Spring automatically uses it for injection — you don't even need `@Autowired`.

#### ❌ Field Injection (Avoid)

```java
@RestController
public class BookController {
    @Autowired
    private BookService bookService;  // Don't do this
}
```

Why this is worse:
- Can't use `final` → mutable, harder to reason about
- Hidden dependencies → you have to read every field to know what's needed
- Hard to test → can't easily construct with mocks
- Fails late → errors at runtime instead of startup

#### ❌ Setter Injection (Rare)

```java
@RestController
public class BookController {
    private BookService bookService;

    @Autowired
    public void setBookService(BookService bookService) {
        this.bookService = bookService;
    }
}
```

Only use this for optional dependencies (which is very rare).

### Beans Are Singletons by Default

Spring creates **one instance** of each bean and reuses it everywhere. If three different classes need `BookService`, they all get the *same* instance.

```
Container:
  bookService (1 instance) ──► used by BookController
                           ──► used by ReportService
                           ──► used by AdminController
```

This is efficient and ensures consistency. It's the right behavior for stateless services (which backend services should be).

---

## Code Examples

### Refactoring BookShelf: Extract a Service

In Chapter 7, all the logic was in the controller. Let's fix that.

**Create `BookService.java`** at `src/main/java/com/bookshelf/BookService.java`:

```java
package com.bookshelf;

import org.springframework.stereotype.Service;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;
import java.util.concurrent.atomic.AtomicLong;

@Service
public class BookService {

    private final List<Book> books = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    public List<Book> getAllBooks() {
        return books;
    }

    public Optional<Book> getBookById(Long id) {
        return books.stream()
                .filter(b -> b.getId().equals(id))
                .findFirst();
    }

    public Book createBook(Book book) {
        book.setId(idCounter.getAndIncrement());
        books.add(book);
        return book;
    }

    public Optional<Book> updateBook(Long id, Book updatedBook) {
        for (int i = 0; i < books.size(); i++) {
            if (books.get(i).getId().equals(id)) {
                updatedBook.setId(id);
                books.set(i, updatedBook);
                return Optional.of(updatedBook);
            }
        }
        return Optional.empty();
    }

    public boolean deleteBook(Long id) {
        return books.removeIf(b -> b.getId().equals(id));
    }

    public List<Book> searchByTitle(String title) {
        return books.stream()
                .filter(b -> b.getTitle().toLowerCase().contains(title.toLowerCase()))
                .toList();
    }
}
```

**Update `BookController.java`** — now it uses the service:

```java
package com.bookshelf;

import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/books")
public class BookController {

    private final BookService bookService;

    // Constructor injection — Spring provides the BookService automatically
    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping
    public List<Book> getAllBooks() {
        return bookService.getAllBooks();
    }

    @GetMapping("/{id}")
    public Book getBookById(@PathVariable Long id) {
        return bookService.getBookById(id).orElse(null);
    }

    @PostMapping
    public Book createBook(@RequestBody Book book) {
        return bookService.createBook(book);
    }

    @PutMapping("/{id}")
    public Book updateBook(@PathVariable Long id, @RequestBody Book updatedBook) {
        return bookService.updateBook(id, updatedBook).orElse(null);
    }

    @DeleteMapping("/{id}")
    public void deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
    }

    @GetMapping("/search")
    public List<Book> searchBooks(@RequestParam String title) {
        return bookService.searchByTitle(title);
    }
}
```

**What changed:**
- The `List<Book>` and `AtomicLong` moved from the controller to the service
- The controller now delegates ALL work to the service
- Spring creates the `BookService` and injects it into the controller

**The controller is now thin** — it only translates HTTP to method calls and method results back to HTTP.

---

## Exercise: Refactor BookShelf with Dependency Injection

**Goal**: Apply DI by separating your controller and service.

### Tasks

1. Create `BookService` with the `@Service` annotation
2. Move all the data management logic from `BookController` to `BookService`
3. Update `BookController` to use constructor injection for `BookService`
4. Run the app and verify all endpoints still work exactly the same

### Verification

```bash
# Everything should work identically to Chapter 7
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "author": "Frank Herbert", "pages": 412}'

curl http://localhost:8080/api/books
```

### Thought Experiment

After refactoring, imagine you want to write a test for `BookController`. With DI, you can do this:

```java
// In a test (you'll learn testing in Chapter 16, but preview the concept)
BookService mockService = new FakeBookService();  // A simple test implementation
BookController controller = new BookController(mockService);

// Now you can test the controller without a real database, without HTTP, 
// without anything — just pure Java
```

Without DI, this would be impossible — the controller would create its own service internally.

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Using `@Autowired` on fields instead of constructors | Field injection hides dependencies and prevents `final`. Use constructor injection. |
| Forgetting `@Service` / `@Component` on the class | Spring won't detect it. You'll get: `NoSuchBeanDefinitionException: No qualifying bean of type 'BookService'` |
| Putting the service in a different top-level package | `@ComponentScan` only scans sub-packages of the main class. If `BookshelfApplication` is in `com.bookshelf`, your service must be in `com.bookshelf` or `com.bookshelf.something`. |
| Creating dependencies with `new` inside Spring beans | If you `new BookService()` inside a controller, that instance is NOT managed by Spring. It won't have its own dependencies injected. Always let Spring create your beans. |
| Circular dependencies | If A depends on B and B depends on A, Spring can't create either. This usually means your design needs restructuring. |

---

## Key Takeaways

- [ ] A dependency is any object another object needs to do its job
- [ ] Dependency Injection means receiving dependencies instead of creating them
- [ ] Spring's container creates beans (objects) and wires them together automatically
- [ ] Use `@Service`, `@Repository`, `@RestController` to register beans — they communicate intent
- [ ] Always use constructor injection with `final` fields
- [ ] Beans are singletons by default — one instance, shared everywhere
- [ ] DI makes code testable, flexible, and loosely coupled

---

## Quick Quiz

1. What's the difference between `@Service` and `@Component`?
2. Why is constructor injection better than field injection (`@Autowired` on a field)?
3. What error do you get if you forget `@Service` on your service class?
4. Can two different controllers inject the same `BookService`? Do they get the same instance or different ones?
5. In the restaurant analogy, who is the "supplier" that provides ingredients (dependencies) to the chef (your class)?

---

*Next: `09-request-response-lifecycle.md` — Trace the full journey of a request through your Spring Boot app →*
