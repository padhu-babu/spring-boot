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

Let's get this tattooed on your brain right now. Rule number one. The golden rule. The sacred covenant of backend development:

> **Every piece of data from the client is potentially wrong, malicious, or missing.**

Never trust user input. NEVER. Not even if the user is you. Not even if you personally built the frontend. Not even if you pinky-swore with yourself that you'd always send clean data.

🗣️ **Overheard at the coffee shop:**
*"My frontend validates everything, so I don't need backend validation."*
*"Yeah, and I don't need a seatbelt because I'm a good driver."*

Think about what the client could send you:

- An empty title: `{"title": "", "pages": 412}`
- Negative pages: `{"title": "Dune", "pages": -5}`
- Missing fields: `{"title": "Dune"}`
- Garbage: `{"title": "x".repeat(10000), "pages": 999999999}`

Your frontend might validate before sending, but here's the uncomfortable truth:

1. Someone can bypass the frontend and call your API directly with curl
2. Bugs in the frontend can send bad data
3. You might have multiple clients (web, mobile, third-party) and can't control them all

⚠️ **Watch it!** Your frontend is a suggestion. Your backend is the law. Someone with `curl` and ten free minutes can send absolutely anything to your API. If you're not validating on the server, you're basically leaving the vault door open and hoping nobody notices.

**The backend is the last line of defense.** Always validate.

### Bean Validation (Jakarta Validation)

Alright, so we need to validate things. But how? Are we going to write a mountain of `if` statements?

```java
// Please, no. Don't do this. We're better than this.
if (request.title() == null || request.title().isBlank()) {
    // cry
}
if (request.pages() < 1) {
    // cry harder
}
```

Spring Boot supports **Bean Validation** — annotations you put on your DTO fields to declare rules. It's declarative, clean, and dare I say... *elegant*:

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

Look at that. You just described your validation rules right next to your fields. No `if` statements. No spaghetti logic. Just annotations that say exactly what they mean.

To activate validation, add `@Valid` to the controller parameter:

```java
@PostMapping
public ResponseEntity<BookResponse> createBook(@Valid @RequestBody BookRequest request) {
    // If validation fails, this method is never called
    // Spring returns 400 Bad Request automatically
}
```

💡 **There are no Dumb Questions**

**Q: Wait, so the method body never executes if validation fails?**

A: Exactly! Think of `@Valid` as a bouncer at a club. If your data doesn't meet the dress code, it never gets inside. Spring intercepts the invalid request, and your controller method never even knows someone knocked.

**Q: What if I forget to add `@Valid`?**

A: Then your beautiful validation annotations sit there doing absolutely nothing. They're just decorations. Like putting a "No Running" sign at a pool but never enforcing it. `@Valid` is the enforcement.

### Common Validation Annotations

Here's your cheat sheet. Bookmark this. Print it. Stick it on your wall.

| Annotation | Applies To | Rule |
|-----------|-----------|------|
| `@NotNull` | Any type | Must not be null |
| `@NotBlank` | String | Must not be null, empty, or whitespace |
| `@NotEmpty` | String, Collection | Must not be null or empty |
| `@Size(min=, max=)` | String, Collection | Length/size must be in range |
| `@Min(value)` | Number | Must be >= value |
| `@Max(value)` | Number | Must be <= value |
| `@Positive` | Number | Must be > 0 |
| `@Email` | String | Must be a valid email format |
| `@Pattern(regexp=)` | String | Must match the regex |
| `@Past` | Date/Time | Must be in the past |
| `@Future` | Date/Time | Must be in the future |

🧠 **Brain Power:** Look at the table above. What's the difference between `@NotNull`, `@NotEmpty`, and `@NotBlank`? If someone sends `"   "` (three spaces), which ones would reject it and which ones would let it through? Think about it before reading on.

---

### 🗣️ Fireside Chat: `@NotNull` vs `@NotBlank` vs `@NotEmpty`

*Three annotations walk into a bar. We asked each one when they'd let a value through.*

**@NotNull:** "I only care about one thing: is it null? If it's `null`, you're out. But `""` ? `"   "`? Fine by me. I'm not picky."

**@NotEmpty:** "I'm a bit stricter. No `null`, and no empty strings either. But `"   "` — sure, that's technically *something*, right? I'll allow it."

**@NotBlank:** "I'm the strictest of the bunch. No `null`, no `""`, and no whitespace-only strings. If you're going to give me data, give me *real* data. I'm not accepting three spaces and calling it a title."

**Bottom line:** For strings, you almost always want `@NotBlank`. It's the one that actually makes sure someone typed something meaningful.

---

### The Problem with Default Error Responses

So you've added validation. Someone sends bad data. Spring rejects it. Victory! But wait — look at what the client actually receives:

```json
{
    "timestamp": "2025-01-15T10:30:00.000+00:00",
    "status": 400,
    "error": "Bad Request",
    "path": "/api/books"
}
```

🗣️ **Overheard at the coffee shop:**
*"The API said 'Bad Request.' Thanks, I guess? But WHAT was bad? Was it the title? The pages? My life choices?"*

This is unhelpful — the client doesn't know *what* was wrong. We need custom error responses. Telling someone "something's wrong" without telling them *what* is like a compiler that just says "error" with no line number. Technically accurate. Practically useless.

### Custom Error Response

Here's what a *helpful* error response looks like. Notice how it tells the client exactly what to fix:

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

🎯 **Key Point:** A good error response answers three questions: What went wrong? Where did it go wrong? How do I fix it? If your error response doesn't answer all three, it's not done yet.

### Exception Handling with @ControllerAdvice

Now we need a way to *catch* all those exceptions flying around and turn them into our beautiful, structured error responses.

Enter `@ControllerAdvice` — the safety net of your application.

```
Request → Controller → throws exception
                           ↓
                    @ControllerAdvice catches it
                           ↓
                    Returns structured error response
```

`@ControllerAdvice` creates a global exception handler — a single class that catches exceptions from ANY controller and converts them to proper HTTP responses.

---

### 🗣️ Fake Interview: A Chat with `@Valid`

**Interviewer:** So `@Valid`, tell us about yourself. What do you do?

**@Valid:** I'm the bouncer. No invalid data gets past me. When I'm standing at the door of a controller method, I check every single field against its validation annotations. `@NotBlank`? I check it. `@Min`? I check it. Everything.

**Interviewer:** And if something fails?

**@Valid:** I throw a `MethodArgumentNotValidException` faster than you can say "bad request." The controller method? Never even sees it. I don't let garbage in the door.

**Interviewer:** What happens if someone forgets to put you on a controller parameter?

**@Valid:** *(sighs)* Then I'm unemployed for that endpoint. The validation annotations are there, sure. But without me, they're just... comments. Fancy, annotated comments. It's honestly a little heartbreaking.

**Interviewer:** Any advice for developers?

**@Valid:** Never forget me. I'm one little annotation. Five characters. I'm small but mighty. And without me, your validation is a lie.

---

### 🗣️ Fake Interview: A Chat with `@ControllerAdvice`

**Interviewer:** `@ControllerAdvice`, you've been described as a "safety net." Fair?

**@ControllerAdvice:** Absolutely fair. I catch every exception so your controllers don't have to. Think about it — do you really want every controller method wrapped in try-catch blocks? That's messy. That's amateur hour. I centralize all of it.

**Interviewer:** How do you know which exceptions to handle?

**@ControllerAdvice:** That's where `@ExceptionHandler` comes in. Each method inside me is tagged with the exception type it handles. `BookNotFoundException`? I've got a method for that. `MethodArgumentNotValidException`? Covered. Generic `Exception`? That's my catch-all, my safety net for the safety net.

**Interviewer:** What happens to exceptions you don't have a handler for?

**@ControllerAdvice:** If I have a generic `Exception` handler — and you *should* always have one — I catch everything. Nothing escapes. No ugly stack traces leak to the client. No unstructured error responses. Just clean, consistent JSON every time.

**Interviewer:** Any pet peeves?

**@ControllerAdvice:** Developers who catch exceptions in their controllers instead of letting them flow to me. That's my job! Let me do it!

---

## Code Examples

Time to build this thing. We're going step by step — and by the end, your API will be a fortress that rejects bad data with style.

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

Notice how each annotation has a `message` parameter. That message is what shows up in your error response. Make it human-readable. "Title is required" is way better than "must not be blank."

### Step 3: Add @Valid to Controller

This is the step people forget. Don't be that person. Update your controller methods that accept request bodies:

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

⚠️ **Watch it!** See that `@Valid` sitting right before `@RequestBody`? That one tiny annotation is the difference between "validates everything" and "validates nothing." If you skip it, your annotations on `BookRequest` are just decoration. Spring will happily accept garbage data and pass it straight to your service layer.

### Step 4: Create Error Response DTO

Now let's define what our error responses look like. Create `src/main/java/com/bookshelf/dto/ErrorResponse.java`:

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

🎯 **Key Point:** Every error your API returns should use this same structure. Consistency is king. Your API consumers should never have to guess what shape an error will come in.

### Step 5: Create a Custom Exception

Time to create an exception that actually means something. No more generic "something went wrong." Create `src/main/java/com/bookshelf/exception/BookNotFoundException.java`:

```java
package com.bookshelf.exception;

public class BookNotFoundException extends RuntimeException {
    
    public BookNotFoundException(Long id) {
        super("Book not found with id: " + id);
    }
}
```

Short. Sweet. Descriptive. When this exception is thrown, everyone reading the stack trace knows exactly what happened: someone asked for a book, and that book doesn't exist.

### Step 6: Create Global Exception Handler

This is the big one. The centerpiece. The grand central station of error handling. Create `src/main/java/com/bookshelf/exception/GlobalExceptionHandler.java`:

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

Let's break down what's happening here. You've got three layers of defense:

1. **Validation errors** — When `@Valid` catches bad input, this handler extracts every single field error and lists them all. The client gets a complete list of everything they need to fix. No "fix this, try again, oh wait, fix that too" ping-pong.

2. **Not found errors** — When your service can't find a book, it throws `BookNotFoundException`, and this handler catches it and returns a clean 404.

3. **The catch-all** — This is your safety net for the safety net. Anything unexpected? Database down? Null pointer? Something you never anticipated? This catches it and returns a generic 500 without leaking any internal details.

⚠️ **Watch it!** See how the generic exception handler says "An unexpected error occurred" instead of passing `ex.getMessage()`? That's deliberate. A 500 error might contain database connection strings, file paths, class names — all the juicy details an attacker would love to have. Log the real error server-side. Send the client a polite, vague message.

### Step 7: Use the Exception in the Service

Now your service doesn't have to return `Optional` and make the controller deal with "what if it's empty?" logic. Just throw the exception and let `@ControllerAdvice` handle the rest:

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

Look how clean that controller is now! No `if (optional.isEmpty())` checks. No manual 404 responses. The controller does its job (handle the HTTP stuff), the service does its job (business logic), and the exception handler does its job (error formatting). Everybody's happy.

### Testing Error Responses

Time to see it in action. Fire up your application and try to break it:

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

🧠 **Brain Power:** Try sending a request with BOTH an empty title AND negative pages. How many errors show up in the `details` array? Does Spring stop at the first error or report all of them? (Spoiler: it reports all of them. Because Spring is thorough like that.)

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

💡 **There are no Dumb Questions**

**Q: What if I add a new entity later, like `Author`? Do I need a separate `GlobalExceptionHandler` for it?**

A: Nope! That's the beauty of `@ControllerAdvice` — it's global. You'd add an `AuthorNotFoundException` and a new `@ExceptionHandler` method inside the same class. One handler class to rule them all.

**Q: Can I have multiple `@ControllerAdvice` classes?**

A: You can, and sometimes it makes sense for organization. But for most applications, one is plenty. Keep it simple until complexity forces your hand.

**Q: What if both `@Valid` fails AND my service throws an exception?**

A: `@Valid` runs first, before your controller method is even called. So if validation fails, your service never executes. The validation error response is returned immediately. No double trouble.

---

## Common Mistakes

Don't feel bad if you make these — everyone does at least once. But now you know to watch out for them.

| Mistake | Reality |
|---------|---------|
| Forgetting `@Valid` on the controller parameter | Validation annotations on the DTO won't do anything without `@Valid` on the controller method. |
| Using `@NotNull` for strings | `@NotNull` allows empty strings `""`. Use `@NotBlank` for strings to also reject empty and whitespace. |
| Exposing stack traces in error responses | Never send `ex.getStackTrace()` to the client. It reveals internal implementation details. Log it, return a generic message. |
| Catching exceptions in the controller | Let exceptions propagate to `@ControllerAdvice`. That's why it exists — centralized error handling. |
| Not handling the generic `Exception` case | Always have a catch-all handler. Unhandled exceptions return ugly Spring defaults. |

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 35](../../appendices/E-coding-exercises.md#exercise-35) | Add Validation Annotations | ⭐⭐ |
| [Exercise 36](../../appendices/E-coding-exercises.md#exercise-36) | Global Exception Handler | ⭐⭐ |
| [Exercise 37](../../appendices/E-coding-exercises.md#exercise-37) | Structured Validation Errors | ⭐⭐ |
| [Exercise 38](../../appendices/E-coding-exercises.md#exercise-38) | Handle Multiple Exception Types | ⭐⭐⭐ |
| [Exercise 39](../../appendices/E-coding-exercises.md#exercise-39) | Test Validation with curl | ⭐⭐ |

Solutions are in [Appendix F](../../appendices/F-exercise-solutions.md).

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

*Next: `14-relationships-and-queries.md` — Connect your entities together ->*
