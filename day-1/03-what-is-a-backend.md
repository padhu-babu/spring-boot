# Chapter 3: What Is a Backend Application?

> ⏱ Estimated time: 40 minutes

## What You'll Learn

- What "frontend" and "backend" actually mean
- The three jobs every backend application does
- Why the backend is stateless (and what that means for you)
- Where your Spring Boot code fits in the bigger picture

---

## Concepts

### Frontend vs. Backend

When someone builds a web application, there are two halves:

**Frontend (Client-Side)**
- What the user sees and interacts with
- HTML, CSS, JavaScript in a browser — or a mobile app on a phone
- Handles layout, buttons, animations, user input
- Runs on the *user's* device

**Backend (Server-Side)**
- What the user never sees
- The logic, rules, and data behind the application
- Handles business rules, data storage, authentication, security
- Runs on a *server* (a computer you control)

```
┌─────────────────────────────────────────────────┐
│                   THE USER                       │
│              (types, clicks, taps)               │
└────────────────────┬────────────────────────────┘
                     │
         ┌───────────▼───────────┐
         │      FRONTEND         │
         │                       │
         │  - What you see       │
         │  - Buttons, forms     │
         │  - Runs in browser    │
         │    or mobile app      │
         └───────────┬───────────┘
                     │ HTTP requests
                     │ (over the internet)
         ┌───────────▼───────────┐
         │      BACKEND          │  ← This is what you're learning to build
         │                       │
         │  - Business logic     │
         │  - Data validation    │
         │  - Authentication     │
         │  - Database access    │
         │  - Runs on a server   │
         └───────────┬───────────┘
                     │
         ┌───────────▼───────────┐
         │      DATABASE         │
         │                       │
         │  - Stores all data    │
         │  - Books, users,      │
         │    orders, etc.       │
         └───────────────────────┘
```

**Analogy — The Restaurant**:
- **Frontend** = the dining room. Menus, tables, plates, the waiter. Everything the customer sees.
- **Backend** = the kitchen. Recipes, cooking, food prep, inventory management. The customer never sees this.
- **Database** = the pantry/fridge. Where all the ingredients are stored.

The waiter (HTTP) carries orders (requests) from the dining room to the kitchen, and food (responses) from the kitchen to the dining room.

### Why Separate Them?

You might wonder: "Why not just put everything in one place?"

**Separation gives you flexibility:**

1. **Multiple frontends, one backend**: Your bookstore can have a website, a mobile app, and a tablet app — all talking to the *same* backend. You write the logic once.

```
  Website ──────┐
                │
  Mobile App ───┼───► Same Backend API ───► Same Database
                │
  Partner App ──┘
```

2. **Independent updates**: You can redesign the frontend without touching the backend. You can optimize the backend without touching the frontend.

3. **Security**: Sensitive logic (payments, passwords, authorization) stays on the server where users can't tamper with it. If the logic were in the frontend (JavaScript in a browser), anyone could modify it.

4. **Scalability**: If your app gets popular, you can add more backend servers without changing anything about the frontend.

### The Three Jobs of a Backend

Every backend application, regardless of language or framework, does three things:

#### Job 1: Receive Requests and Send Responses

The backend is an HTTP server. It listens on a port, receives HTTP requests, and sends HTTP responses. This is the "plumbing" — the mechanism by which the outside world communicates with your application.

```
Client sends:    POST /books  { "title": "Clean Code" }
Backend does:    (Job 2 and 3)
Backend sends:   201 Created  { "id": 1, "title": "Clean Code" }
```

#### Job 2: Execute Business Logic

This is the *brain* of your application — the rules, calculations, validations, and decisions.

Examples:
- "A book must have a title and at least one page"
- "Only admin users can delete books"
- "When a book is borrowed, decrease available copies by 1"
- "If copies reach 0, mark the book as unavailable"
- "Calculate late fees: $0.50 per day past the due date"

Business logic is the *reason* the backend exists. Without it, you'd just be a database with a door.

#### Job 3: Store and Retrieve Data

Most backend applications need to persist data — save it so it survives server restarts. This is done through a **database**.

- User signs up → backend stores their name, email, hashed password
- User adds a book → backend stores the book data
- User searches for books → backend queries the database and returns results

```
The Three Jobs — Visualized:

  HTTP Request ──► [Job 1: Receive] ──► [Job 2: Think] ──► [Job 3: Store/Fetch]
                                                                    │
  HTTP Response ◄── [Job 1: Send] ◄── [Job 2: Think] ◄────────────┘
```

### A Concrete Example

Let's trace what happens when someone uses a bookstore app to add a new book:

```
1. USER ACTION
   User fills out a form: Title="Dune", Author="Frank Herbert", Pages=412
   Clicks "Add Book"

2. FRONTEND
   Converts the form data to JSON
   Sends HTTP request:
   POST /api/books
   Content-Type: application/json
   { "title": "Dune", "author": "Frank Herbert", "pages": 412 }

3. BACKEND (Job 1 — Receive)
   Spring Boot receives the request on port 8080
   Routes it to the BookController.createBook() method
   Parses the JSON body into a Java object

4. BACKEND (Job 2 — Business Logic)
   Validates: Is the title non-empty? ✓
   Validates: Are pages > 0? ✓
   Checks: Does this book already exist? (queries database) → No
   Generates: Assigns ID = 42

5. BACKEND (Job 3 — Store)
   Saves the book to the database:
   INSERT INTO books (id, title, author, pages) VALUES (42, 'Dune', 'Frank Herbert', 412)

6. BACKEND (Job 1 — Send Response)
   HTTP/1.1 201 Created
   Content-Type: application/json
   { "id": 42, "title": "Dune", "author": "Frank Herbert", "pages": 412 }

7. FRONTEND
   Receives the response
   Shows a success message: "Book added!"
   Navigates to the book detail page for ID 42
```

### Statelessness: The Backend Has Amnesia

Remember from Chapter 2: HTTP is stateless. Your backend should be too.

**Stateless** means the server does not remember anything between requests. Each request must contain *everything* the server needs to process it.

**Why?** Because in production, you often have multiple copies of your server running behind a **load balancer**:

```
                    ┌──► Server A
Client ──► Load ───┼──► Server B
           Balancer └──► Server C
```

Request 1 might go to Server A. Request 2 might go to Server C. If Server A remembered something from Request 1 that Server C doesn't know, things break.

**What does this mean for you?**
- Don't store user data in Java variables that persist between requests
- Use a database for anything that needs to survive longer than a single request
- Use tokens (like JWTs) for authentication — the client sends the token with *every* request

> 🧠 **Think Like a Backend Engineer**: Whenever you're about to store something in a regular Java field in your controller or service, ask yourself: "Will this break if the next request is handled by a different server instance?" If yes, put it in the database instead.

### What a Backend Is NOT

Clearing up common misconceptions:

| Myth | Reality |
|------|---------|
| "The backend generates HTML pages" | It *can*, but modern backends usually return **data** (JSON). The frontend handles the visuals. |
| "The backend is one giant program" | It's often split into **layers** (controller, service, repository) for organization. We'll cover this in Chapter 10. |
| "The backend directly connects to users" | Usually a **web server** or **load balancer** sits in front. But for learning, your app connects directly. |
| "You need a separate frontend" | You can test your backend with `curl` or Postman. No frontend needed. |

### Where Your Spring Boot Code Lives

Here's the architecture of what you'll build in this guide:

```
┌─────────────────────────────────────────────┐
│              YOUR SPRING BOOT APP            │
│                                              │
│  ┌──────────────┐                           │
│  │  Controller   │ ← Receives HTTP requests  │
│  │  (Chapter 7)  │   Routes to the right     │
│  └──────┬───────┘   service method           │
│         │                                    │
│  ┌──────▼───────┐                           │
│  │   Service     │ ← Business logic          │
│  │  (Chapter 10) │   Validation, rules,      │
│  └──────┬───────┘   calculations             │
│         │                                    │
│  ┌──────▼───────┐                           │
│  │  Repository   │ ← Data access             │
│  │  (Chapter 12) │   Talks to the database   │
│  └──────┬───────┘                           │
│         │                                    │
└─────────┼────────────────────────────────────┘
          │
   ┌──────▼───────┐
   │   Database    │ ← Stores data permanently
   │  (Chapter 12) │
   └──────────────┘
```

Don't worry about understanding every layer yet. Just note that your Spring Boot application has clear sections, and each chapter will teach you one.

---

## Code Examples

No code to write in this chapter — it's the last theory chapter before we start building. But here's a preview of what your BookShelf backend will look like by the end of this guide:

```java
// This is what you'll write in Chapter 7 — a real HTTP endpoint in Spring Boot
@RestController
@RequestMapping("/api/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping
    public List<BookResponse> getAllBooks() {
        return bookService.getAllBooks();    // Job 2 & 3: logic + data access
    }

    @PostMapping
    public ResponseEntity<BookResponse> createBook(@Valid @RequestBody BookRequest request) {
        BookResponse created = bookService.createBook(request);  // Job 2 & 3
        return ResponseEntity.status(HttpStatus.CREATED).body(created);  // Job 1: respond
    }
}
```

This might look intimidating now. By Day 3, you'll understand every line.

---

## Exercise: Architect an Application

**Goal**: Practice thinking about what a backend does — before writing any code.

### Task

Pick **one** of these applications and answer the questions below:

- **A** — A social media app (like Twitter/X)
- **B** — An online food ordering app (like DoorDash)
- **C** — A music streaming app (like Spotify)

For your chosen app:

1. **List the data** the backend needs to store. (Example for a bookstore: books, authors, users, borrowing records)

2. **List 5 API endpoints** the backend should have. For each, specify:
   - HTTP method (GET, POST, PUT, DELETE)
   - Path (e.g., `/api/songs`)
   - What it does
   - What status code it returns on success

3. **Describe 3 business rules** the backend must enforce. (Example: "A user can't borrow more than 5 books at once")

4. **Draw the architecture**: Client → Backend → Database. Label what data flows in each direction for one specific request.

5. **Identify what should NOT be in the backend**: What parts of this application belong to the frontend?

### Example Answer (for a Bookstore)

```
Data: books, authors, users, borrow_records

Endpoints:
  GET    /api/books          → List all books         → 200
  POST   /api/books          → Add a new book         → 201
  GET    /api/books/42       → Get book by ID         → 200 (or 404)
  DELETE /api/books/42       → Remove a book          → 204
  POST   /api/borrow         → Borrow a book          → 201

Business Rules:
  1. A book must have a title (non-empty)
  2. Can't borrow a book with 0 available copies
  3. Maximum 5 active borrows per user

Not backend:
  - Displaying the book cover image layout
  - Animating the "add to cart" button
  - The color scheme of the app
```

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "The backend does everything" | The backend handles data and logic. The frontend handles user interface. Drawing buttons, animations, and page layouts are NOT the backend's job. |
| "I need a frontend to test my backend" | No! You can test entirely with `curl` or Postman. This is how professional backend developers work daily. |
| "Each request is tied to a specific server" | In stateless architectures, any server instance can handle any request. This is a feature, not a limitation. |
| "The backend stores state in memory" | For anything that needs to persist, use a database. In-memory state disappears when the server restarts and doesn't work with multiple server instances. |

---

## Key Takeaways

- [ ] I can explain the difference between frontend and backend
- [ ] I know the three jobs of a backend: receive/send HTTP, business logic, data storage
- [ ] I understand why backends are stateless and what that means for my code
- [ ] I can look at a real application and identify what the backend is responsible for
- [ ] I understand the high-level architecture: Client → Backend (Controller → Service → Repository) → Database

---

## Quick Quiz

1. A designer says "make the book list page look nicer." Is this a frontend or backend task?
2. A product manager says "users should only be able to borrow 3 books at a time." Where does this rule live?
3. Why is it dangerous to put business logic (like "only admins can delete books") in the frontend?
4. Your app has 10,000 simultaneous users. What architectural strategy lets the backend handle the load?
5. Why can't you just use a Java `ArrayList` in your controller to store books permanently?

---

## Day 1 Summary

You've completed Day 1! Here's what you now understand:

```
✓ The internet is clients talking to servers using IP addresses and ports
✓ DNS translates domain names to IP addresses
✓ HTTP is the language of the web (methods, status codes, headers, bodies)
✓ A backend application receives requests, applies logic, and stores/retrieves data
✓ Backends are stateless — each request is independent
✓ You'll build a Controller → Service → Repository layered application
```

Tomorrow, you'll learn about JSON data format, REST API design principles, and set up your very first Spring Boot project.

---

*Next: `day-2/04-json-and-apis.md` — Time to learn the data format that every modern API speaks →*
