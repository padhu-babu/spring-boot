# Chapter 17: Logging and API Documentation

> **You've built an app that works. Congratulations!** But here's the thing nobody tells you until it's too late: a working app that you can't *observe* and that nobody can *understand* is a ticking time bomb. It's 3 AM, your app is down, and all you've got is `System.out.println("here 2")` in the console. Not helpful. Not even a little.

> ⏱ Estimated time: 50 minutes

## What You'll Learn

- Why logging matters and how to do it in Spring Boot (SLF4J + Logback)
- Log levels and when to use each one
- How to add Spring Boot Actuator for health monitoring
- How to generate API documentation with Swagger/OpenAPI
- How to make your API self-documenting

---

## Let's Talk About the Worst Night of Your Career

Picture this. It's 3 AM. Your phone buzzes. The app is down. Customers are angry. Your manager is Slack-messaging you in all caps.

You SSH into the server. You open the logs. And you see... **nothing useful.** Just a sea of `System.out.println("test")`, `System.out.println("got here")`, and the hauntingly unhelpful `System.out.println("why???")`.

**Without logging**: "The app is broken." --> "I have no idea why."
**With logging**: "The app is broken." --> Check logs --> "BookService.createBook failed at line 42: Author not found for ID 999."

See the difference? One is stumbling through a dark house. The other is walking through with a flashlight, a map, and night-vision goggles.

> **⚠️ Watch it!** `System.out.println` is NOT logging. Stop it. Right now. We're going to say that one more time because some of you skimmed past it: **System.out.println is NOT logging.** It has no log levels, no timestamps, no class names, no way to turn it off in production, and no way to route it to a file. It's the programming equivalent of writing notes on your hand and then washing your hands.

---

## 🎤 Fake Interview: Sitting Down with Logger

**Interviewer:** Logger, thanks for being here. Can you introduce yourself?

**Logger:** Sure. I'm the black box flight recorder of your application. You know those devices in airplanes that survive crashes and tell investigators exactly what happened, second by second? That's me, except for your Spring Boot app.

**Interviewer:** That sounds dramatic.

**Logger:** Production incidents at 3 AM ARE dramatic. And when everything's on fire, I'm the calm voice in the room saying, "Here's exactly what happened, in order, with timestamps."

**Interviewer:** So what makes you different from `System.out.println`?

**Logger:** *long pause* You did NOT just compare me to `System.out.println`. That's like comparing a Swiss Army knife to a butter knife. `System.out` just prints text. I provide timestamps, class names, thread information, configurable severity levels, the ability to write to files, the ability to turn me off for certain packages... should I keep going?

**Interviewer:** I think we get it.

**Logger:** And here's the big one: **I'm lazy in the best way.** If you write `log.debug("Processing book: {}", expensiveMethod())`, and debug logging is turned off? I don't even bother evaluating that message. `System.out` does the string concatenation whether you want it or not. I save you CPU cycles AND make your code cleaner.

**Interviewer:** One more thing -- people say you're already included in Spring Boot?

**Logger:** Yep. Zero extra dependencies. I'm SLF4J on the outside, Logback on the inside. You just import me and go. No Maven drama. No configuration files. I'm ready when you are.

---

## SLF4J: The Logging API

Spring Boot uses **SLF4J** (Simple Logging Facade for Java) as the logging API and **Logback** as the implementation. They're included automatically -- no extra dependencies needed. Zip. Zero. Nada.

Here's the basic pattern -- and yes, you'll use this in EVERY class:

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

> **🎯 Key Point:** Three things to burn into your brain:
> - **One `Logger` per class**, using the class name
> - Use `{}` as placeholders -- SLF4J fills them in (DO NOT use string concatenation!)
> - The logger is `static final` -- created once per class, shared across all instances

---

## Log Levels: A Hierarchy of Panic

Not all messages are created equal. "The server started" and "THE DATABASE IS ON FIRE" should not look the same in your logs. That's why we have log levels.

| Level | When to Use | Example |
|-------|-------------|---------|
| `ERROR` | Something failed that shouldn't have. Needs attention. | `log.error("Failed to save book: {}", ex.getMessage(), ex)` |
| `WARN` | Something unexpected but recoverable. | `log.warn("Author not found for id {}, using default", authorId)` |
| `INFO` | Important business events. | `log.info("Book created: id={}", book.getId())` |
| `DEBUG` | Detailed info for troubleshooting. | `log.debug("Searching for books with title containing: {}", title)` |
| `TRACE` | Very detailed, rarely used. | `log.trace("Entering method getAllBooks()")` |

Here's the real-world translation, because you WILL forget which level to use:

> **🗣️ Overheard at the coffee shop:**
>
> - `ERROR` --> "Wake someone up at 3 AM. Something is actually broken."
> - `WARN` --> "Hey, look at this during business hours. It's weird but not fatal."
> - `INFO` --> "This is what the app is doing. Normal stuff. Good stuff."
> - `DEBUG` --> "I need to troubleshoot a specific issue. Time to put on the detective hat."
> - `TRACE` --> "I want to see EVERYTHING. Even the things I'll regret seeing."

> **🧠 Brain Power:** Here's a scenario. A user requests a book with ID 42. The book doesn't exist, so you return a 404. What log level should this be? Think about it before reading on.
>
> If you said `ERROR`, slow down. A book not being found is a perfectly normal business case -- users ask for things that don't exist all the time. That's `WARN` at most, and many teams would argue it's just `DEBUG`. Reserve `ERROR` for things that indicate a real system problem, not "the user typed a wrong ID."

---

## Configuring Log Levels

You control what gets logged in `application.properties`. This is where you fine-tune the noise level:

```properties
# Set log level for your app
logging.level.com.bookshelf=DEBUG

# Set log level for Spring framework
logging.level.org.springframework.web=INFO

# Set log level for Hibernate SQL
logging.level.org.hibernate.SQL=DEBUG
```

When you set a level, you see that level AND everything above it. Set `DEBUG`? You see DEBUG, INFO, WARN, and ERROR. Set `ERROR`? You ONLY see errors. It's a threshold, not a filter.

> **💡 There are no Dumb Questions:**
>
> **Q: If I set the level to DEBUG, won't I get flooded with messages?**
> A: Yes, which is why you set DEBUG only for YOUR packages (`com.bookshelf`) and leave Spring's packages at INFO. In production, you typically run your app at INFO and bump specific packages to DEBUG only when investigating a problem.
>
> **Q: Can I change the log level without restarting the app?**
> A: Actually, yes! With Spring Boot Actuator (which we're about to cover), you can change log levels at runtime through an HTTP endpoint. Pretty slick, right?
>
> **Q: Where do the logs actually go?**
> A: By default, they go to the console (stdout). In production, you'd configure Logback to write to files, rotate them, and maybe ship them to a log aggregation service. But that's a deployment topic -- for now, console is fine.

---

## Spring Boot Actuator: Your App's Vital Signs

You know how doctors check your heart rate, blood pressure, and temperature? **Actuator** does that for your application. It exposes operational endpoints for monitoring -- think of it as a dashboard for your app's health.

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

Now you get free health checks:
```bash
curl http://localhost:8080/actuator/health
# {"status":"UP","components":{"db":{"status":"UP"},"diskSpace":{"status":"UP"}}}

curl http://localhost:8080/actuator/info
# Application info
```

> **🎯 Key Point:** That `/actuator/health` endpoint is pure gold in production. Load balancers use it to know if your app is alive. Monitoring tools poll it to alert you when something's wrong. Kubernetes uses it for liveness and readiness probes. One dependency, one line of config, and your app goes from "I hope it's running" to "I KNOW it's running."

---

## 🎤 Fake Interview: A Conversation with Swagger UI

**Interviewer:** Swagger UI, welcome. What exactly are you?

**Swagger UI:** I make your API self-documenting. You know how developers hate writing documentation? And you know how other developers hate using undocumented APIs? I solve both problems at once.

**Interviewer:** How?

**Swagger UI:** I read your controller code -- the annotations, the request types, the response types, the URL mappings -- and I generate a beautiful, interactive web page where anyone can see every endpoint your API offers. They can even click "Try it out" and make real requests right from the browser. No Postman needed. No reading source code. No begging the backend team for a Confluence page that's six months out of date.

**Interviewer:** So developers don't have to write any documentation?

**Swagger UI:** Well, the basic docs are generated automatically -- just from your code. But if developers add a few annotations, they can make the documentation *richer*. Descriptions, examples, error codes. Think of it like this: I give you a free house. The annotations are paint and furniture. The house works without them, but it's nicer with them.

**Interviewer:** And you're based on OpenAPI?

**Swagger UI:** Right. **OpenAPI** is the specification -- it's the standard for describing REST APIs. I'm the pretty face that reads an OpenAPI spec and makes it visual and interactive. In the Spring Boot world, a library called `springdoc-openapi` generates the OpenAPI spec from your Java code, and I render it. Teamwork.

**Interviewer:** What's the setup like?

**Swagger UI:** One Maven dependency. That's it. No configuration files, no YAML documents, no handwritten specs. Add the dependency, start your app, visit `/swagger-ui.html`, and see all your endpoints listed with their parameters, request bodies, and response types -- generated from your Java code. It's practically magic.

---

## Setting Up Swagger / OpenAPI

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

That's it. Seriously. Run your app and visit:
- **Swagger UI**: `http://localhost:8080/swagger-ui.html`
- **OpenAPI JSON**: `http://localhost:8080/v3/api-docs`

You'll see all your endpoints listed with their parameters, request bodies, and response types -- generated from your Java code. No extra work required.

> **🗣️ Overheard at the coffee shop:** "I spent three days writing API docs in a Google Doc. Then someone added `springdoc-openapi` and the entire thing was auto-generated in two seconds. Three days. I want those three days back."

---

## Code Examples

### Adding Logging to BookService

Here's the full service with logging. Pay attention to WHICH level is used WHERE -- there's a method to the madness:

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

> **🧠 Brain Power:** Look at the logging in `createBook`. We used `log.info` for the create attempt and success, but `log.error` when the author wasn't found. Why `ERROR` and not `WARN`? Because if someone is calling `createBook` with an invalid author ID, that's likely a bug in the calling code -- the frontend shouldn't be sending bad IDs. Something went wrong upstream. That's alarm-worthy.
>
> Now look at `getBookById` -- when a book isn't found, we used `WARN`. Why not `ERROR`? Because users searching for books that don't exist is *normal* behavior. Not ideal, but not a system failure.
>
> The log level tells a story about *intent*. Choose wisely.

### Adding Logging to GlobalExceptionHandler

Your global exception handler is the last line of defense. It MUST log everything that passes through it:

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

> **⚠️ Watch it!** Notice that `handleGenericException` logs the full exception with `log.error("message", ex)` but returns a GENERIC error to the client. This is critical. Your logs get the gory details (stack traces, internal error messages). The client gets a polite "Something went wrong." Never leak your internal structure to the outside world -- that's a security risk AND a bad user experience.

### Swagger Annotations (Optional Enhancement)

The auto-generated docs are good. But with a few annotations, they become *great*:

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

> **💡 There are no Dumb Questions:**
>
> **Q: Do I HAVE to add these annotations?**
> A: Nope. Swagger generates usable docs without them. The annotations just add polish -- descriptions, parameter explanations, response code documentation. Think of them as comments that actually DO something.
>
> **Q: Will the annotations slow down my app?**
> A: No. They're processed at startup time when the OpenAPI spec is generated. They have zero impact on request processing.
>
> **Q: Can I exclude certain endpoints from the documentation?**
> A: Yes, using `@Hidden` on a method or class. Useful for internal admin endpoints you don't want exposed in the public docs.

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

> **🧠 Brain Power:** After adding logging, create a book, then try to fetch a book with an ID that doesn't exist, then try to create a book with an invalid author ID. Watch the console output. Can you see the different log levels in action? Can you see the timestamps, class names, and messages? Compare this to what you'd see with `System.out.println`. Night and day, right?

---

## Common Mistakes

Stop. Before you go off and start logging everything, read these. Every single one is a mistake we've seen in production. Every. Single. One.

| Mistake | Reality |
|---------|---------|
| Using `System.out.println()` | Use a logger. `System.out` has no levels, no configuration, no context. |
| String concatenation in log messages | Use `{}` placeholders: `log.info("Book: {}", id)` not `log.info("Book: " + id)`. Placeholders are only evaluated if the log level is active. |
| Logging sensitive data | Never log passwords, tokens, credit card numbers, or personal data. |
| Logging at the wrong level | Don't use ERROR for expected business cases (like "book not found"). That's WARN at most. |
| Not including exceptions in error logs | `log.error("Failed", ex)` -- always include the exception as the last argument for the full stack trace. |

> **🗣️ Overheard at the coffee shop:** "We had a security audit, and they found we were logging full credit card numbers. In plain text. To a file. That was readable by the entire dev team. That was a *fun* meeting with the CTO."

---

## The Logging Cheat Sheet

Because you WILL come back to this page when you can't remember which level to use:

```
TRACE  --> "I want to see inside every method call"         (almost never in production)
DEBUG  --> "I'm investigating a specific problem"           (on during development)
INFO   --> "Here's what the app is doing"                   (the default in production)
WARN   --> "Something's off, but we're handling it"         (review during business hours)
ERROR  --> "Something broke. Fix it."                       (page someone)
```

> **🎯 Key Point:** When in doubt, use INFO. You can always turn it down later. It's much worse to have NO logs when you need them than to have too many logs you can filter.

---

## Key Takeaways

- [ ] Use SLF4J for logging -- one Logger per class
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

*Next: `18-security-basics.md` -- Protect your API endpoints -->*
