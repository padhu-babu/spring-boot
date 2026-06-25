# Chapter 3: What Is a Backend Application?

> ⏱ Estimated time: 60 minutes

## What You'll Learn

- What "frontend" and "backend" actually mean — and why they're separated
- The three jobs every backend application does (receive, think, store)
- What an API is and why it's the backend's public contract
- Why the backend is stateless (and what breaks when it isn't)
- How real-world backends are structured beyond just "one server"
- Where your Spring Boot code fits in the bigger picture

> **Quick Summary**: A backend is a program that sits on a server, waits for HTTP requests, runs business logic, talks to a database, and sends responses. It never draws buttons or renders pages — it deals in *data* and *rules*. The frontend handles everything the user sees; the backend handles everything the user doesn't.

---

## Concepts

### Frontend vs. Backend

When someone builds a web application, there are two halves:

**Frontend (Client-Side)**
- What the user **sees and interacts with**
- HTML, CSS, JavaScript in a browser — or a mobile app on a phone
- Handles layout, buttons, animations, user input
- Runs on the *user's* device (their phone, their laptop, their tablet)

**Backend (Server-Side)**
- What the user **never sees**
- The logic, rules, and data behind the application
- Handles business rules, data storage, authentication, security
- Runs on a *server* (a computer **you** control)

Here's the key insight: **the frontend and backend are completely separate programs running on different computers**. The frontend runs on the user's device. The backend runs on your server. They communicate over the internet using HTTP — the protocol you learned in Chapter 2.

```mermaid
flowchart TD
    USER["THE USER\n(types, clicks, taps)"]
    FRONTEND["FRONTEND\n\nRuns on the USER'S device\n- What you see\n- Buttons, forms\n- Runs in browser or mobile app"]
    INTERNET(["--- The internet (network boundary) ---"])
    BACKEND["BACKEND\n\nRuns on YOUR server\n- Business logic\n- Data validation\n- Authentication\n- Database access\n\nThis is what you're learning to build"]
    DATABASE[("DATABASE\n\nAlso on YOUR server\n(or a dedicated db server)\n- Stores all data\n- Books, users, orders, etc.")]

    USER --> FRONTEND
    FRONTEND -- "HTTP requests\n(over the internet)" --> INTERNET
    INTERNET --> BACKEND
    BACKEND --> DATABASE
```

**Analogy — The Restaurant**:
- **Frontend** = the dining room. Menus, tables, plates, the waiter. Everything the customer sees.
- **Backend** = the kitchen. Recipes, cooking, food prep, inventory management. The customer never sees this.
- **Database** = the pantry/fridge. Where all the ingredients are stored.

The waiter (HTTP) carries orders (requests) from the dining room to the kitchen, and food (responses) from the kitchen to the dining room.

**What does this look like in practice?**

Think about Amazon.com:

| What you see (Frontend) | What you don't see (Backend) |
|-------------------------|------------------------------|
| The product page with images and a price | The code that looks up the product in a database of millions of items |
| The "Add to Cart" button | The code that checks inventory, calculates tax, applies discounts |
| The "Place Order" button | The code that charges your card, reserves stock, triggers shipping |
| A spinning loading indicator | The backend is doing work — the frontend is just waiting |
| "Sorry, out of stock" message | The backend checked inventory and found 0 available |

The frontend only knows how to *display* things. All decisions about what to display, whether to allow a purchase, and how to process payments happen in the backend.

### Why Separate Them?

You might wonder: "Why not just put everything in one place?"

**Separation gives you flexibility:**

#### 1. Multiple frontends, one backend

Your bookstore can have a website, a mobile app, and a tablet app — all talking to the *same* backend. You write the logic once.

```mermaid
flowchart LR
    WEB["Website\n(React/JS)"]
    MOBILE["Mobile App\n(iOS/Android)"]
    PARTNER["Partner API\n(their code)"]
    BACKEND["Backend API\n(Spring Boot)\n\nSame code\nSame rules\nSame database"]
    DB[("Database")]

    WEB --> BACKEND
    MOBILE --> BACKEND
    PARTNER --> BACKEND
    BACKEND <--> DB
```

Without this separation, you'd write the same "add a book" logic three times — once in the website, once in the mobile app, once for the partner. When a rule changes ("max 500 pages"), you'd have to update all three. With a backend, you update the rule once.

#### 2. Independent updates

You can completely redesign the website (new colors, new layout, new framework) without changing a single line of backend code. You can rewrite the backend in a different language without the mobile app noticing — as long as the HTTP API stays the same.

#### 3. Security

Sensitive logic (payments, passwords, authorization) stays on the server where users **can't tamper with it**. This is critical.

```
❌ BAD: Business logic in the frontend (JavaScript in the browser)

  User opens browser DevTools → sees JavaScript code → changes
  "if (user.hasDiscount)" to "if (true)" → gets free stuff

✅ GOOD: Business logic in the backend

  User sends request → backend checks discount rules on the server →
  user can't see or modify the code → rules are enforced
```

If authorization logic ("only admins can delete books") runs in the browser, anyone can bypass it by modifying the JavaScript. If it runs on the server, the user never sees the code and can't tamper with it.

#### 4. Scalability

If your app gets popular, you can add more backend servers without changing anything about the frontend. We'll see this in the Statelessness section below.

### The Three Jobs of a Backend

Every backend application, regardless of language or framework, does three things. Whether it's Netflix handling 200 million users or your BookShelf app on localhost — the same three jobs, every time.

> **Quick Summary**: ① Receive the request → ② Think (apply rules and logic) → ③ Store/fetch data → send the response back.

#### Job 1: Receive Requests and Send Responses

The backend is an HTTP server. It listens on a port, receives HTTP requests, and sends HTTP responses. This is the "plumbing" — the mechanism by which the outside world communicates with your application.

```
Client sends:    POST /books  { "title": "Clean Code" }
Backend does:    (Job 2 and 3)
Backend sends:   201 Created  { "id": 1, "title": "Clean Code" }
```

In Spring Boot, this job is handled by the **Controller** layer. You write methods that say "when a GET request comes to `/books`, run this code." Spring Boot handles all the low-level HTTP parsing — reading bytes off the network, parsing headers, converting JSON to Java objects — so you don't have to.

#### Job 2: Execute Business Logic

This is the *brain* of your application — the rules, calculations, validations, and decisions. Business logic is what makes **your** application different from every other application.

Examples for a bookshelf app:
- "A book must have a title and at least one page"
- "Only admin users can delete books"
- "When a book is borrowed, decrease available copies by 1"
- "If copies reach 0, mark the book as unavailable"
- "Calculate late fees: $0.50 per day past the due date"

Examples for other apps:
- Uber: "Find the nearest available driver within 5 km"
- Twitter/X: "A tweet can't exceed 280 characters"
- Banking: "Don't allow a transfer if the account balance would go negative"

Business logic is the *reason* the backend exists. Without it, you'd just be a database with a door — anyone could read, write, or delete anything. The business logic is the **gatekeeper** that decides what's allowed and what isn't.

In Spring Boot, business logic lives in the **Service** layer.

#### Job 3: Store and Retrieve Data

Most backend applications need to **persist** data — save it so it survives server restarts. If your book data only lived in your Java program's memory, it would vanish every time the server restarted. That's useless.

This is done through a **database** — a program designed specifically for storing, organizing, and quickly retrieving data.

- User signs up → backend stores their name, email, hashed password
- User adds a book → backend stores the book data
- User searches for books → backend queries the database and returns results

```
Without a database:
  Server starts → data is empty → users add books → server restarts → ALL DATA GONE

With a database:
  Server starts → connects to database → data is still there from yesterday
```

In Spring Boot, data access lives in the **Repository** layer. You'll learn about JPA (Java Persistence API), which lets you write Java code instead of raw SQL queries.

```mermaid
flowchart TD
    subgraph BACKEND ["YOUR BACKEND"]
        RECEIVE["Job 1: Receive"]
        THINK["Job 2: Think\n(Business rules, Validation, Calculations)"]
        STORE["Job 3: Store/Fetch"]
        SEND["Job 1: Send"]
        RECEIVE --> THINK --> STORE --> SEND
    end

    REQ["HTTP Request"] --> RECEIVE
    STORE <--> DB[("DATABASE")]
    SEND --> RESP["HTTP Response"]
```

### What Is an API?

You'll hear the term **API** constantly. Let's make sure it's crystal clear.

**API** stands for **Application Programming Interface**. It's the set of rules and endpoints that your backend exposes to the outside world. Think of it as a **menu** at a restaurant.

```mermaid
graph LR
    subgraph API ["BookShelf API (the 'menu')"]
        direction LR
        E1["GET /api/books → Returns a list of all books"]
        E2["GET /api/books/42 → Returns book with ID 42"]
        E3["POST /api/books → Creates a new book"]
        E4["PUT /api/books/42 → Replaces book 42"]
        E5["DELETE /api/books/42 → Deletes book 42"]
        R1["Rules:\n- POST requires JSON body with 'title' and 'pages' (> 0)\n- Returns JSON responses\n- Returns 404 if book ID doesn't exist\n- Returns 400 if request body is invalid\n- Requires authentication for POST/PUT/DELETE"]
    end
```

The API is a **contract**: "If you send me a request in *this* format, I'll send you a response in *that* format." The client doesn't need to know how the backend works internally — it just needs to know the menu.

This is why you can have multiple frontends talking to the same backend. They all read the same menu. A website, a mobile app, and a partner company's system all send the same `GET /api/books` request and get the same JSON response.

> **API vs Website**: A traditional website returns **HTML** (visual pages for humans to read). An API returns **data** (usually JSON) for programs to consume. When you build a Spring Boot backend in this guide, you're building an **API**, not a website. You'll use `curl` to talk to it, not a browser.

### A Concrete Example: Tracing a Full Request

Let's trace what happens when someone uses a bookstore app to add a new book. Follow every step — this is the exact flow you'll be building:

```
1. USER ACTION
   ┌─────────────────────────────────────┐
   │  Add a Book                         │
   │                                     │
   │  Title:  [ Dune                  ]  │
   │  Author: [ Frank Herbert         ]  │
   │  Pages:  [ 412                   ]  │
   │                                     │
   │           [ Add Book ]              │
   └─────────────────────────────────────┘
   User fills out the form and clicks "Add Book"

2. FRONTEND (runs in the user's browser)
   The browser JavaScript:
   a) Reads the form fields
   b) Converts them to a JSON object
   c) Sends an HTTP request to the backend:

   POST /api/books HTTP/1.1
   Host: www.bookshelf.com
   Content-Type: application/json
   Authorization: Bearer user-token-abc

   { "title": "Dune", "author": "Frank Herbert", "pages": 412 }

   ─── Request travels over the internet to the server ───►

3. BACKEND (Job 1 — Receive)
   Spring Boot receives the request on port 8080
   DispatcherServlet reads the HTTP method (POST) and path (/api/books)
   Finds the matching handler: BookController.createBook()
   Parses the JSON body into a Java object (BookRequest)
   Validates the Authorization token → user is authenticated ✓

4. BACKEND (Job 2 — Business Logic)
   The BookService runs the business rules:
   ├── Is the title non-empty?                    → ✓ "Dune"
   ├── Is the title under 200 characters?          → ✓ (4 chars)
   ├── Are pages > 0?                              → ✓ (412)
   ├── Does this book already exist in database?   → Queries DB → No ✓
   └── All checks pass → proceed to save

5. BACKEND (Job 3 — Store)
   The BookRepository saves to the database:
   INSERT INTO books (id, title, author, pages)
   VALUES (42, 'Dune', 'Frank Herbert', 412)
   Database confirms: row inserted ✓

6. BACKEND (Job 1 — Send Response)
   Constructs the HTTP response:

   HTTP/1.1 201 Created
   Content-Type: application/json
   Location: /api/books/42

   { "id": 42, "title": "Dune", "author": "Frank Herbert", "pages": 412 }

   ◄── Response travels back over the internet ───

7. FRONTEND (runs in the user's browser)
   The browser JavaScript:
   a) Receives the response
   b) Sees status 201 → success!
   c) Reads the response body → gets the new book's ID (42)
   d) Shows a success message: "Book added!"
   e) Navigates to the book detail page: /books/42
```

**What would happen at step 4 if the title were empty?**

```
4. BACKEND (Job 2 — Business Logic)
   ├── Is the title non-empty?  → ✗ FAIL!
   └── Stop processing. Don't touch the database.

6. BACKEND (Job 1 — Send Error Response)
   HTTP/1.1 400 Bad Request
   { "error": "Title must not be empty" }

7. FRONTEND
   Sees status 400 → something was wrong with the data
   Shows error: "Title must not be empty"
```

Notice: the backend **never reached the database**. The business logic caught the problem early and returned an error. This is a pattern you'll implement in Chapter 13 (Validation).

### Statelessness: The Backend Has Amnesia

Remember from Chapter 2: HTTP is stateless. Your backend should be too.

**Stateless** means the server does not remember anything between requests. Each request must contain *everything* the server needs to process it.

**Why?** Because in production, you often have multiple copies of your server running behind a **load balancer** — a traffic cop that distributes incoming requests across servers to share the workload:

```mermaid
flowchart LR
    CLIENT["Client"]
    LB["Load\nBalancer"]
    A["Server A\n(your app)"]
    B["Server B\n(your app)"]
    DB[("Database\n(shared)")]

    CLIENT <--> LB
    LB --> A
    LB --> B
    A --> DB
    B --> DB
```

```
Request 1 → might go to Server A
Request 2 → might go to Server B
Request 3 → might go to Server A again
(Client has no control over which server handles its request)
```

#### What Goes Wrong Without Statelessness

Here's a concrete example of what breaks when a backend is stateful:

```java
// ❌ BAD — Storing state in a Java field
@RestController
public class CartController {

    // This list lives in THIS server's memory only
    private List<String> cartItems = new ArrayList<>();

    @PostMapping("/cart/add")
    public String addItem(@RequestBody String item) {
        cartItems.add(item);  // Saved in Server A's memory
        return "Added! Cart has " + cartItems.size() + " items";
    }

    @GetMapping("/cart")
    public List<String> getCart() {
        return cartItems;  // Returns whatever THIS server remembers
    }
}
```

Here's what happens:

```
Request 1: POST /cart/add "Dune"    → goes to Server A
  Server A's cartItems: ["Dune"]
  Response: "Added! Cart has 1 items"     ← looks correct

Request 2: POST /cart/add "1984"    → goes to Server B
  Server B's cartItems: ["1984"]          ← Server B has never seen "Dune"!
  Response: "Added! Cart has 1 items"     ← WRONG! Should be 2

Request 3: GET /cart                → goes to Server A
  Response: ["Dune"]                      ← WHERE DID "1984" GO?!
```

The user added two books but only sees one. The data is split across two servers' memory, and neither has the full picture.

```java
// ✅ GOOD — Using the database (shared between all servers)
@RestController
public class CartController {

    private final CartRepository cartRepository;  // Talks to the database

    @PostMapping("/cart/add")
    public String addItem(@RequestBody String item, @RequestHeader("User-Id") Long userId) {
        cartRepository.addItem(userId, item);  // Saved in the DATABASE
        int count = cartRepository.countItems(userId);
        return "Added! Cart has " + count + " items";
    }
}
```

Now every server reads from and writes to the **same** database. Request 1 goes to Server A, which writes "Dune" to the database. Request 2 goes to Server B, which also writes to the same database and correctly counts 2 items.

#### The Statelessness Checklist

| Store in the database | DON'T store in Java fields |
|----------------------|---------------------------|
| User accounts | Shopping cart items |
| Books, products, orders | "Logged in" status |
| Session data | Request counters |
| Any data a user expects to see next time | Anything that must survive a restart |

**What does this mean for you?**
- Don't store user data in Java variables that persist between requests
- Use a database for anything that needs to survive longer than a single request
- Use tokens (like JWTs) for authentication — the client sends the token with *every* request

> 🧠 **Think Like a Backend Engineer**: Whenever you're about to store something in a regular Java field in your controller or service, ask yourself: "Will this break if the next request is handled by a different server instance?" If yes, put it in the database instead.

### What a Real-World Backend Looks Like

So far we've described a simple setup: one backend server, one database. In reality, production backends are more complex. You don't need to understand all of this yet — but seeing the big picture now will help concepts click faster later.

```mermaid
flowchart LR
    WEB["Website"]
    MOBILE["Mobile App"]
    LB["Load\nBalancer"]

    subgraph SYSTEM ["Your Backend System"]
        A["Server A\n(copy 1)"]
        B["Server B\n(copy 2)"]
        DB[("Database\n(PostgreSQL)")]
        EXTRAS["Also common:\n- Cache (Redis) for speed\n- Message Queue (Kafka) for background tasks\n- File Storage (S3) for images"]

        A --> DB
        B --> DB
    end

    WEB --> LB
    MOBILE --> LB
    LB --> A
    LB --> B
```

**Don't panic.** For this guide, your setup is much simpler:

```
  curl (you) ──► Your Spring Boot app (localhost:8080) ──► H2 Database (in-memory)
```

One client (you, using curl), one server (your laptop), one database (built into your app). That's all you need to learn the concepts. Everything in the big diagram above is just the same three jobs (receive, think, store) — scaled up.

### What a Backend Is NOT

Clearing up common misconceptions:

| Myth | Reality |
|------|---------|
| "The backend generates HTML pages" | It *can*, but modern backends usually return **data** (JSON). The frontend handles the visuals. |
| "The backend is one giant program" | It's often split into **layers** (controller, service, repository) for organization. We'll cover this in Chapter 10. |
| "The backend directly connects to users" | Usually a **web server** or **load balancer** sits in front. But for learning, your app connects directly. |
| "You need a separate frontend" | You can test your backend with `curl` or Postman. No frontend needed. |

### Where Your Spring Boot Code Lives

Here's the architecture of what you'll build in this guide. Each layer has one job, and they stack on top of each other like floors in a building:

```mermaid
flowchart TD
    subgraph APP ["YOUR SPRING BOOT APP"]
        CONTROLLER["Controller Layer (Chapter 7)\n\n'The receptionist'\n- Receives HTTP requests\n- Reads the method, path, and body\n- Calls the right service method\n- Sends the HTTP response\n\nExample: @GetMapping('/api/books') -> getAllBooks()"]
        SERVICE["Service Layer (Chapter 10)\n\n'The brain'\n- Contains all business logic\n- Validates data, enforces rules\n- Converts between DTOs and entities\n- Decides what to do (save? reject? calculate?)\n\nExample: 'Title can't be empty, pages > 0'"]
        REPO["Repository Layer (Chapter 12)\n\n'The filing clerk'\n- Talks to the database\n- Saves, updates, deletes, and queries data\n- Knows nothing about HTTP or business rules\n\nExample: bookRepository.save(book)"]

        CONTROLLER -- "calls" --> SERVICE
        SERVICE -- "calls" --> REPO
    end

    REPO -- "SQL queries" --> DB[("Database (Chapter 12)\n\nStores data permanently")]
```

**Why three layers?** For the same reason restaurants separate the dining room, kitchen, and pantry. Each has a clear responsibility:

| Layer | Knows About | Does NOT Know About |
|-------|-------------|---------------------|
| Controller | HTTP, JSON, request/response | Database queries, SQL |
| Service | Business rules, validation | HTTP, how data is stored |
| Repository | Database, SQL, tables | HTTP, business rules |

This separation means you can change *how* you store data (switch databases) without touching the business logic. Or change *which* HTTP endpoints you support without touching the database code. Each layer is independent.

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

**Goal**: Practice thinking about what a backend does — before writing any code. This is the most important skill in backend development: knowing *what* to build before you build it.

### Task

Pick **one** of these applications and answer the questions below:

- **A** — A social media app (like Twitter/X)
- **B** — An online food ordering app (like DoorDash)
- **C** — A music streaming app (like Spotify)

For your chosen app:

1. **List the data** the backend needs to store. (Example for a bookstore: books, authors, users, borrowing records)

   > Hint: Think about *nouns*. What "things" exist in this app? Users, posts, songs, orders — these become your database tables.

2. **List 5 API endpoints** the backend should have. For each, specify:
   - HTTP method (GET, POST, PUT, DELETE)
   - Path (e.g., `/api/songs`)
   - What it does
   - What status code it returns on success

   > Hint: Think about *verbs*. What can users *do*? Each action maps to an endpoint with a method + path.

3. **Describe 3 business rules** the backend must enforce. (Example: "A user can't borrow more than 5 books at once")

   > Hint: Think about *constraints*. What should the system prevent? What limits exist? What happens in edge cases?

4. **Draw the architecture**: Client → Backend → Database. Label what data flows in each direction for one specific request.

5. **Identify what should NOT be in the backend**: What parts of this application belong to the frontend?

   > Hint: If it involves pixels, colors, animations, or layout — it's frontend. If it involves rules, data, or decisions — it's backend.

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
✓ Frontend = what users see (browser/app). Backend = logic + data (server)
✓ A backend does three jobs: receive HTTP, apply business logic, store/retrieve data
✓ An API is the backend's public "menu" — the contract clients use
✓ Backends are stateless — each request is independent, use a database for persistence
✓ You'll build a Controller → Service → Repository layered application
```

```
Day 1 Concepts Map:

  Chapter 1           Chapter 2              Chapter 3
  ─────────           ─────────              ─────────
  Client/Server  ───► HTTP Protocol  ───►    Backend Architecture
  IP + Ports          Methods (GET/POST)     Three Jobs (receive/think/store)
  DNS                 Status Codes           API as a Contract
                      Headers + Body         Statelessness
                      curl                   Controller → Service → Repository
```

Tomorrow, you'll learn about JSON data format, REST API design principles, and set up your very first Spring Boot project. Day 1 was all theory — Day 2 is where you start building.

---

*Next: `day-2/04-json-and-apis.md` — Time to learn the data format that every modern API speaks →*
