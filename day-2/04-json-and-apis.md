# Chapter 4: JSON and REST APIs

> ⏱ Estimated time: 50 minutes

## What You'll Learn

- What JSON is and why it's the universal data format for APIs
- JSON syntax rules and how it maps to Java objects
- What REST means and its core principles
- How to design clean API endpoints
- The difference between an API and a website

---

## Concepts

### The Problem: How Do You Send Data Over the Internet?

In Java, you work with objects:

```java
Book book = new Book("Dune", "Frank Herbert", 412);
```

But HTTP only transmits **text**. You can't send a Java object over the wire. You need a text format that both the sender and receiver understand.

That format is **JSON**.

### What Is JSON?

**JSON** stands for **JavaScript Object Notation**. Despite the name, it has nothing to do with JavaScript for our purposes — it's just a text format that every programming language can read and write.

JSON represents data as key-value pairs:

```json
{
    "title": "Dune",
    "author": "Frank Herbert",
    "pages": 412,
    "available": true
}
```

That's it. Curly braces, keys in quotes, values after colons, commas between pairs.

### JSON Syntax Rules

#### Data Types

JSON supports exactly these types:

| Type | Example | Java Equivalent |
|------|---------|-----------------|
| String | `"hello"` | `String` |
| Number | `42`, `3.14` | `int`, `double` |
| Boolean | `true`, `false` | `boolean` |
| Null | `null` | `null` |
| Object | `{ "key": "value" }` | A class/object |
| Array | `[1, 2, 3]` | `List` / array |

#### Objects (Curly Braces)

An object is a collection of key-value pairs:

```json
{
    "name": "Alice",
    "age": 30,
    "isStudent": false
}
```

Rules:
- Keys **must** be strings (in double quotes)
- Use double quotes `"`, not single quotes `'`
- No trailing comma after the last pair

#### Arrays (Square Brackets)

An array is an ordered list:

```json
["fiction", "science", "history"]
```

Arrays can contain any type, including other objects:

```json
[
    { "title": "Dune", "pages": 412 },
    { "title": "1984", "pages": 328 }
]
```

#### Nesting

Objects and arrays can nest inside each other:

```json
{
    "name": "Frank Herbert",
    "born": 1920,
    "books": [
        {
            "title": "Dune",
            "year": 1965,
            "genres": ["science fiction", "adventure"]
        },
        {
            "title": "Dune Messiah",
            "year": 1969,
            "genres": ["science fiction"]
        }
    ],
    "awards": {
        "hugo": true,
        "nebula": true
    }
}
```

### JSON ↔ Java Mapping

This is the mental model that will save you hours:

```
JSON                          Java
──────────────────────────    ──────────────────────
{ }                           class / object
"key": "value"                String field
"key": 42                     int / long field
"key": true                   boolean field
"key": null                   null reference
[ ]                           List<> / array
{ nested object }             another class

Example:

{                             public class Book {
    "title": "Dune",             String title;
    "pages": 412,                int pages;
    "available": true            boolean available;
}                             }
```

Spring Boot converts between JSON and Java objects **automatically**. You write Java classes, and Spring Boot handles the translation. This is called **serialization** (Java → JSON) and **deserialization** (JSON → Java).

### What Is an API?

**API** stands for **Application Programming Interface**. It's a contract that says:

> "If you send me a request in *this* format, I'll send you a response in *that* format."

Think of it as a menu at a restaurant:
- The menu lists what you can order (endpoints)
- Each item has a name and description (path and method)
- You order using the menu's format (request body)
- You get back what's described (response body)

An API is NOT a visual interface. There's no HTML, no web page, no buttons. It's data in, data out.

### REST: A Way to Design APIs

**REST** stands for **Representational State Transfer**. It's a set of conventions for designing APIs that make them predictable and easy to use.

REST isn't a technology or a library — it's a style. An opinion on how things should be organized.

#### The Core Ideas of REST

**1. Everything is a Resource**

A "resource" is any thing your API manages: a book, a user, an order, a comment.

Each resource has a URL:
```
/api/books       → the collection of all books
/api/books/42    → a specific book (ID 42)
/api/authors     → the collection of all authors
/api/authors/7   → a specific author (ID 7)
```

**2. Use HTTP Methods for Actions**

Don't put verbs in your URLs. Use HTTP methods instead:

```
WRONG (verb in URL):
  GET  /api/getBooks
  POST /api/createBook
  POST /api/deleteBook/42

RIGHT (HTTP methods express the action):
  GET    /api/books          → get all books
  POST   /api/books          → create a book
  GET    /api/books/42       → get one book
  PUT    /api/books/42       → update a book
  DELETE /api/books/42       → delete a book
```

**3. Use Standard Status Codes**

```
200 OK         → successful GET/PUT
201 Created    → successful POST (something was created)
204 No Content → successful DELETE
400 Bad Request → client sent invalid data
404 Not Found   → resource doesn't exist
500 Internal Server Error → server broke
```

**4. Stateless**

Each request contains all the information needed. The server doesn't remember previous requests.

**5. Use JSON for Data**

Request and response bodies are JSON.

#### A Complete REST API Design

Here's a full REST API for books:

| Action | Method | Path | Request Body | Response Body | Status |
|--------|--------|------|-------------|---------------|--------|
| List all books | GET | `/api/books` | — | Array of books | 200 |
| Get one book | GET | `/api/books/{id}` | — | One book | 200 / 404 |
| Create a book | POST | `/api/books` | Book data | Created book (with ID) | 201 |
| Update a book | PUT | `/api/books/{id}` | Full book data | Updated book | 200 / 404 |
| Delete a book | DELETE | `/api/books/{id}` | — | — | 204 / 404 |
| Search books | GET | `/api/books?title=dune` | — | Filtered array | 200 |

> 🧠 **Think Like a Backend Engineer**: Notice the pattern. Two URLs (`/api/books` and `/api/books/{id}`) handle five different operations by using different HTTP methods. The method tells you *what* to do, the path tells you *what to do it to*.

### API vs Website

| Aspect | Website | API |
|--------|---------|-----|
| Returns | HTML (visual) | JSON (data) |
| Consumer | Humans via browsers | Programs (apps, other services) |
| Rendering | Server generates the page | Client decides how to display |
| Example response | `<h1>Books</h1><ul>...` | `[{"title":"Dune"},...]` |

Modern architecture: the backend is an API that returns JSON, and the frontend (React, mobile app, etc.) calls that API and handles the visual display.

---

## Code Examples

### Writing JSON by Hand

Practice reading and writing JSON. No tools needed — just a text editor.

#### A single book:
```json
{
    "id": 1,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "978-0132350884",
    "pages": 464,
    "available": true,
    "genres": ["programming", "software engineering"]
}
```

#### A list of books:
```json
[
    {
        "id": 1,
        "title": "Clean Code",
        "pages": 464
    },
    {
        "id": 2,
        "title": "Dune",
        "pages": 412
    }
]
```

#### An author with nested books:
```json
{
    "id": 1,
    "name": "Frank Herbert",
    "nationality": "American",
    "books": [
        { "id": 10, "title": "Dune", "year": 1965 },
        { "id": 11, "title": "Dune Messiah", "year": 1969 }
    ]
}
```

#### An error response:
```json
{
    "status": 400,
    "error": "Bad Request",
    "message": "Title must not be empty",
    "timestamp": "2025-01-15T10:30:00"
}
```

### Calling a REST API with curl

```bash
# GET all posts (read a collection)
curl https://jsonplaceholder.typicode.com/posts

# GET one post (read a specific resource)
curl https://jsonplaceholder.typicode.com/posts/1

# GET with query parameter (filter)
curl "https://jsonplaceholder.typicode.com/posts?userId=1"

# POST (create a new resource)
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My New Post",
    "body": "This is the content",
    "userId": 1
  }'

# PUT (update/replace a resource)
curl -X PUT https://jsonplaceholder.typicode.com/posts/1 \
  -H "Content-Type: application/json" \
  -d '{
    "id": 1,
    "title": "Updated Title",
    "body": "Updated content",
    "userId": 1
  }'

# DELETE (remove a resource)
curl -X DELETE https://jsonplaceholder.typicode.com/posts/1
```

---

## Exercise: Design the BookShelf API

**Goal**: Design the REST API for the BookShelf application you'll build this week.

### Task

On paper or in a text file, design the complete API for a book library system. Include:

#### Part 1: Resource Design

List all the resources (things) your API will manage:
- Books (what fields?)
- Authors (what fields?)
- Anything else?

Write the JSON shape for each resource.

#### Part 2: Endpoint Design

Create a table like this:

```
| Action | Method | Path | Request Body | Response | Status |
```

Design at least 8 endpoints covering:
- CRUD for books
- CRUD for authors
- At least one relationship endpoint (e.g., "get all books by author")
- At least one search/filter endpoint

#### Part 3: Write Sample JSON

Write the complete JSON for:

1. A request body to create a new book
2. A response body for "get all books" (array with 3 books)
3. An error response for "book not found"
4. A request body to create an author with their books nested

#### Part 4: Identify Bad API Design

What's wrong with each of these endpoints?

```
1. GET    /api/getAllBooks
2. POST   /api/books/delete/42
3. GET    /api/books/42   → returns 200 with body: { "error": "not found" }
4. POST   /api/books      → returns 200 when creating a book
5. DELETE /api/books/42   → returns the deleted book's full data
```

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Using single quotes in JSON | JSON requires double quotes `"`. `{'key': 'value'}` is invalid JSON. |
| Putting verbs in REST URLs | Use HTTP methods for actions. `/api/books` + `DELETE` method, not `/api/deleteBook`. |
| Returning 200 for errors | Use proper status codes. 404 for not found, 400 for bad input, 500 for server errors. |
| Making everything a POST | Use GET for reading, POST for creating, PUT for updating, DELETE for deleting. Each method has semantic meaning. |
| Forgetting that JSON is text | JSON looks like code, but it's plain text sent over HTTP. It's just a format — a way to structure text so both sides can parse it. |

---

## Key Takeaways

- [ ] I can read and write JSON fluently (objects, arrays, nesting)
- [ ] I understand how JSON maps to Java classes (object → class, array → List)
- [ ] I know what REST means: resources with URLs, HTTP methods for actions, JSON for data
- [ ] I can design a REST API for a given application
- [ ] I know the difference between a website (returns HTML) and an API (returns JSON)
- [ ] Spring Boot automatically converts between Java objects and JSON

---

## Quick Quiz

1. Convert this Java class to its JSON representation: `class User { String name; int age; List<String> hobbies; }`
2. Is this valid JSON? `{ name: "Alice", 'age': 30, }` — If not, why?
3. You have a `User` resource. What are the 5 standard REST endpoints?
4. Why is `POST /api/users/create` not RESTful?
5. An API returns status 200 with body `{"error": "User not found"}`. What's wrong with this approach?

---

*Next: `05-why-frameworks-exist.md` — Why you don't want to build a server from scratch →*
