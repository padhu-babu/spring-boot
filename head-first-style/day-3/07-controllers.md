# Chapter 7: Controllers — Handling HTTP Requests

> ⏱ Estimated time: 70 minutes

## What You'll Learn

- What a controller is and how it works
- All the request mapping annotations (`@GetMapping`, `@PostMapping`, etc.)
- How to read path variables, query parameters, and request bodies
- How to return different types of data from your endpoints
- How to build BookShelf v1 with hardcoded data

---

## This Is Where It Gets Real

Up until now, you've been setting things up. Configuring. Booting. Watching logs scroll by. That's all important, but let's be honest — it's not exactly *thrilling*.

That changes **right now.**

Controllers are where you write the code that actually *does something*. A user hits a URL, and YOUR code runs. YOUR method decides what to send back. You're not configuring anymore — you're building an API. A real one. One you can poke with curl and see respond.

So take a deep breath. You're about to write your first real Spring Boot feature.

---

## Concepts

### What Is a Controller?

Think of a restaurant. When you walk in, you don't go straight to the kitchen and start barking orders at the chef. There's a host at the front door who greets you, figures out what you need, and routes you to the right place.

A controller is that host. It's a Java class that sits at the **front door** of your application. Every HTTP request that arrives — every `GET`, every `POST`, every `DELETE` — comes through a controller first.

Here's what happens when a request arrives:

```
GET /api/books/42
    ↓
Spring looks for a method annotated with @GetMapping("/api/books/{id}")
    ↓
Calls that method with id=42
    ↓
Takes the return value, converts it to JSON, sends it back
```

Pretty straightforward, right? The request comes in, Spring matches it to one of your methods, your method runs, and the result goes back. This matching process is called **routing**.

> **🗣️ Overheard at the coffee shop:**
> "So wait, Spring just... *knows* which method to call?"
> "Yep. You stick an annotation on a method, and Spring handles the rest. You say 'this method handles GET requests to /api/books,' and Spring believes you."
> "That feels like magic."
> "It's not magic — it's annotations. And we're about to demystify every single one."

### Annotations: Not Magic, Just Sticky Notes

Before we dive into the specific annotations, let's clear something up. If you're new to Spring, annotations can feel like dark sorcery. You slap `@GetMapping` on a method and suddenly it handles web requests? How?!

Here's the mental model that'll save you:

**An annotation is just a sticky note for Spring.**

That's it. When your application starts, Spring scans your classes, reads those sticky notes, and builds a routing table. `@GetMapping("/api/books")` is you sticking a note on your method that says: "Hey Spring, when a GET request comes in for /api/books, call me."

Spring reads the note. Spring calls you. No magic involved.

Now let's look at the specific sticky notes you'll use.

### The Key Annotations

#### `@RestController`

This is the big one. It marks a class as a controller that returns **data** (not HTML pages). Every public method in this class can handle HTTP requests.

```java
@RestController
public class BookController {
    // methods here handle HTTP requests
}
```

`@RestController` = `@Controller` + `@ResponseBody`. It means "every method's return value is the response body" (usually converted to JSON).

> **💡 There are no Dumb Questions:**
>
> **Q: What's the difference between `@Controller` and `@RestController`?**
>
> A: `@Controller` is the old-school annotation. Methods in a `@Controller` class return *view names* (like "index.html"), and Spring goes looking for an HTML template. `@RestController` says "nope, I'm returning data directly — JSON, plain text, whatever." Since you're building APIs, you'll use `@RestController` almost every time.
>
> **Q: Do I always need `@RestController` on the class?**
>
> A: Yes. Without it, Spring doesn't know this class is supposed to handle requests. It'll just be a regular Java class sitting in a corner, ignored.

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

Think of `@RequestMapping` as putting a sign on the front of the building: "Everything in here is about books." Then each method inside adds its own specific room number.

#### HTTP Method Annotations

| Annotation | HTTP Method | Typical Use |
|------------|-------------|-------------|
| `@GetMapping` | GET | Read/retrieve data |
| `@PostMapping` | POST | Create new data |
| `@PutMapping` | PUT | Update/replace data |
| `@PatchMapping` | PATCH | Partially update data |
| `@DeleteMapping` | DELETE | Remove data |

> **🧠 Brain Power:**
> Why do we have BOTH `@PutMapping` and `@PatchMapping`? They both update data, right? Think about it: if you have a book with title, author, and pages, and you want to change ONLY the title, should you have to send all three fields or just the one you're changing? That's the difference. PUT replaces the whole thing. PATCH updates just the parts you send. Can you think of a real app where this distinction matters?

---

### Fake Interview: The Annotations Speak!

---

**🎤 Interview with @GetMapping**

**Interviewer:** So, @GetMapping, tell us about yourself.

**@GetMapping:** I'm probably the most popular annotation in any REST controller. When someone wants to *read* data — look up a book, list all users, search for products — I'm the one they call. I handle GET requests, which means I'm all about retrieval. I never change data. I never create anything. I just *find things and hand them over.*

**Interviewer:** Do you ever get jealous of @PostMapping?

**@GetMapping:** (laughs) Not at all. @PostMapping gets all the flashy "creating new things" work, but I get called WAY more often. Think about it — for every item a user creates, they probably view it a dozen times. I'm the workhorse. I'm the one making read-heavy applications actually work.

---

**🎤 Interview with @PostMapping**

**Interviewer:** @PostMapping, you're up. What do you do?

**@PostMapping:** I create things. You want a new book in the database? A new user account? A new order? That's me. I always expect a request body — usually JSON — because you can't create something from nothing. You've got to tell me *what* to create.

**Interviewer:** Any pet peeves?

**@PostMapping:** Oh, absolutely. People who forget the `Content-Type: application/json` header when calling me. I get a request, I look at the body, and I have *no idea* what format it's in. I can't parse it. I throw a 415 Unsupported Media Type error and everyone acts surprised. Just send the header, people!

---

**🎤 Interview with @PathVariable**

**Interviewer:** @PathVariable, you seem to always hang out with @GetMapping.

**@PathVariable:** I hang out with ALL the method annotations, thank you very much! My job is to grab values right out of the URL path. See that `{id}` in `@GetMapping("/{id}")`? That's a placeholder for me. When someone requests `/api/books/42`, I reach into the URL, pluck out `42`, and hand it to the method parameter. Clean. Simple.

**Interviewer:** How are you different from @RequestParam?

**@PathVariable:** (sighs) I get this question every single day. Look — I live *in* the path. I'm part of the resource's address. `/books/42` — I'm the `42`. @RequestParam lives *after* the question mark. `?genre=fiction` — that's their territory. We're neighbors, but we're NOT the same. I identify *which* resource. They filter or customize the request.

---

### Reading Data from Requests

Alright, so you've got a controller, you've got your method annotations — but how does data actually GET to your method? Clients send data to your API in three different ways, and you need a different annotation for each one.

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

> **💡 There are no Dumb Questions:**
>
> **Q: When do I use @PathVariable vs @RequestParam? They both get data from the URL, right?**
>
> A: Great question, and yes, they both extract data from the URL — but from *different parts* of it. Here's the rule of thumb:
>
> - **@PathVariable** → identifying a **specific resource** (`/books/42` — "get me book number 42")
> - **@RequestParam** → filtering, sorting, pagination (`/books?genre=fiction&page=2` — "get me books, but only fiction, page 2")
>
> Think of it this way: if you're looking up ONE thing by its ID, that's a path variable. If you're asking "show me things that match these criteria," those criteria are query parameters.
>
> **Q: What happens if a required @RequestParam is missing from the URL?**
>
> A: Spring throws a `MissingServletRequestParameterException` and returns a 400 Bad Request. That's why you can set `required = false` or provide a `defaultValue` for optional parameters.
>
> **Q: Can I use @PathVariable and @RequestParam in the same method?**
>
> A: Absolutely! It's common. Imagine `GET /api/authors/42/books?sort=title` — you'd use `@PathVariable` for the author ID and `@RequestParam` for the sort order.

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

> **⚠️ Watch it!**
> Jackson needs a no-argument constructor on your class to do its thing. It creates an empty object first, then fills in the fields using setters. If you don't have a no-arg constructor, you'll get a scary-looking `HttpMessageNotReadableException` and stare at it for 20 minutes before realizing you just need to add `public BookRequest() {}`. Don't ask how we know this.

### Return Types

What you return from a controller method becomes the response body. And here's the beautiful part — Spring handles the conversion for you:

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

You return a Java object, Spring turns it into JSON. You return a list, Spring turns it into a JSON array. You don't have to think about serialization at all. That's Jackson doing its job behind the scenes.

> **🎯 Key Point:**
> The controller's only job is to **receive the request** and **send the response**. It should NOT contain business logic. That belongs in the service layer (Chapter 10). For now, we'll put everything in the controller to keep things simple, then refactor later. Just know that in a real application, your controller should be thin — a traffic cop, not a detective.

---

## Code Examples

### BookShelf v1: Complete Controller

Stop for a second and let that sink in. You're about to build a complete CRUD API. Create, Read, Update, Delete — the four operations that power basically every application you've ever used. Netflix's watchlist? CRUD. Your bank's transaction history? CRUD. That todo app you downloaded and never opened? CRUD.

**You're building a REAL API!** It's going to store books, let you search them, update them, and delete them. Sure, the data lives in memory for now (it'll vanish when you restart the server), but the HTTP interface? That's the real deal. The same patterns you'll use when you connect a database in Chapter 12.

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

> **🧠 Brain Power:**
> Look at that controller one more time. Count the annotations. Now imagine writing all of that WITHOUT annotations — you'd need to manually parse the URL, check the HTTP method, read the body, convert JSON by hand... Spring is doing an enormous amount of work for you. Can you list at least five things Spring handles automatically in the code above?

---

### Fireside Chat: @PathVariable vs @RequestParam

---

*@PathVariable and @RequestParam sit down by the fire to settle, once and for all, when you should use each of them.*

**@PathVariable:** Look, I think the confusion is totally understandable. We both pull data from URLs. But I'm part of the *address*. I'm the house number on the street. When you say `/api/books/42`, that `42` IS the address of a specific book. Without me, you can't even find it.

**@RequestParam:** And I'm more like... preferences. "Show me books, but make them fiction, sort by title, and only show page 2." I don't change WHICH resource you're looking at — I change HOW you look at it.

**@PathVariable:** Right. If you removed me from `/api/books/42`, the URL is meaningless — `/api/books/` is a totally different endpoint. But if you remove a query parameter from `/api/books?genre=fiction`, the URL still works — you just get unfiltered results.

**@RequestParam:** Exactly. I'm optional by nature (even though Spring makes me required by default — you can change that). You're never optional. If your URL says `/{id}`, there HAS to be an id.

**@PathVariable:** So the takeaway?

**@RequestParam:** Use *you* for identity. Use *me* for customization.

**@PathVariable:** Couldn't have said it better myself.

---

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

**You just built a working CRUD API!** Seriously, take a moment. Lots of developers remember building their first API. This is yours. It stores data in memory (it disappears when you restart), but the HTTP interface is real. The same verbs. The same URL patterns. The same JSON. This is how APIs work in production — just with a database behind them.

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

> **🎯 Key Point:**
> These aren't trick questions — they're real problems with our current code. And every single one of them gets solved in upcoming chapters. The fact that you can *see* these issues now means you're already thinking like a backend engineer.

---

## Common Mistakes

> **⚠️ Watch it!**
> These are the mistakes that trip up almost EVERYONE when they're first writing controllers. Read them now, bookmark this section, and come back when something isn't working (because it will happen).

| Mistake | Reality |
|---------|---------|
| Forgetting `@RequestBody` on POST/PUT methods | Without it, Spring won't parse the JSON body. Your object will have all null/default values. |
| Forgetting `Content-Type: application/json` header in curl | Spring won't know the body is JSON and will reject it with a 415 (Unsupported Media Type). |
| Using `@RequestParam` when you mean `@PathVariable` | `@RequestParam` reads from `?key=value`. `@PathVariable` reads from `/path/{value}`. They're different sources. |
| Putting business logic in the controller | Controllers should only receive requests and send responses. Logic belongs in the service layer. We'll refactor in Chapter 10. |
| Missing the no-arg constructor on your data class | Jackson needs it to create the object before setting fields. Without it: `HttpMessageNotReadableException`. |

> **💡 There are no Dumb Questions:**
>
> **Q: I got a 415 Unsupported Media Type error. What's going on?**
>
> A: You forgot the `-H "Content-Type: application/json"` header in your curl command. Spring sees the body but doesn't know it's JSON, so it refuses to process it. Always include that header with POST and PUT requests.
>
> **Q: My POST endpoint is receiving an object where every field is null. What did I do wrong?**
>
> A: You probably forgot `@RequestBody` on the method parameter. Without it, Spring doesn't know to read the JSON body and parse it into your object. It just gives you an empty one.
>
> **Q: Why does my app return a 200 for creates instead of 201?**
>
> A: By default, Spring returns 200 OK for everything. To return 201 Created (the proper status code for resource creation), you'll need `ResponseEntity` — which we cover in Chapter 9. For now, 200 works fine.

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 13](../../appendices/E-coding-exercises.md#exercise-13) | Full CRUD Controller | ⭐⭐ |
| [Exercise 14](../../appendices/E-coding-exercises.md#exercise-14) | Path Variables | ⭐⭐ |
| [Exercise 15](../../appendices/E-coding-exercises.md#exercise-15) | Query Parameters for Filtering | ⭐⭐ |
| [Exercise 16](../../appendices/E-coding-exercises.md#exercise-16) | Accept JSON with @RequestBody | ⭐⭐ |
| [Exercise 17](../../appendices/E-coding-exercises.md#exercise-17) | ResponseEntity with Status Codes | ⭐⭐ |

Solutions are in [Appendix F](../../appendices/F-exercise-solutions.md).

---

## Key Takeaways

- [ ] A controller is a class annotated with `@RestController` that handles HTTP requests
- [ ] `@RequestMapping` sets a base path for all endpoints in a controller
- [ ] `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` map methods to HTTP verbs
- [ ] `@PathVariable` reads from the URL path, `@RequestParam` reads from query parameters
- [ ] `@RequestBody` reads the JSON body and converts it to a Java object
- [ ] Return values are automatically converted to JSON
- [ ] Annotations aren't magic — they're just sticky notes that tell Spring what to do
- [ ] I built a working CRUD API for books

---

## Quick Quiz

1. What's the difference between `@Controller` and `@RestController`?
2. Write the annotation for a method that handles `DELETE /api/users/42`.
3. How do you make a query parameter optional?
4. What does Jackson do? When does it run?
5. Why should a controller NOT contain business logic like "a book must have at least 10 pages"?

> **🧠 Brain Power:**
> Before you move on, try to answer those quiz questions without scrolling up. If you can answer at least 4 out of 5 from memory, you've got a solid grasp of controllers. If not, re-read the sections that tripped you up — it's worth it, because everything in the next few chapters builds on this foundation.

---

*Next: `08-dependency-injection.md` — The most important concept in Spring →*
