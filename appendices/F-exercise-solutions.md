# Appendix F: Exercise Solutions

> Complete solutions for all 50 exercises in [Appendix E](E-coding-exercises.md).
> Try each exercise yourself before reading the solution!

---

## Section 1: HTTP & REST Fundamentals

### Exercise 1: Exploring HTTP with curl {#exercise-1}

Use curl to make GET, POST, and DELETE requests to JSONPlaceholder and observe the status codes.

```bash
# 1. GET a list of posts
curl -s -o /dev/null -w "%{http_code}" https://jsonplaceholder.typicode.com/posts
# Expected: 200

# 2. GET a single post
curl -s -w "\n%{http_code}" https://jsonplaceholder.typicode.com/posts/1
# Expected: 200, returns JSON with userId, id, title, body

# 3. GET a non-existent post
curl -s -o /dev/null -w "%{http_code}" https://jsonplaceholder.typicode.com/posts/9999
# Expected: 404

# 4. POST a new post
curl -s -w "\n%{http_code}" -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "My Post", "body": "Hello world", "userId": 1}'
# Expected: 201, returns JSON with id: 101

# 5. DELETE a post
curl -s -o /dev/null -w "%{http_code}" -X DELETE https://jsonplaceholder.typicode.com/posts/1
# Expected: 200
```

**Expected output:**

```
# GET /posts        → 200 OK
# GET /posts/1      → 200 OK  (returns the post JSON)
# GET /posts/9999   → 404 Not Found
# POST /posts       → 201 Created (returns new post with id)
# DELETE /posts/1   → 200 OK
```

**Why this works:**
- GET returns 200 for successful retrieval, 404 when the resource does not exist.
- POST returns 201 Created because a new resource was created on the server.
- DELETE returns 200 because JSONPlaceholder acknowledges the deletion (some APIs use 204 No Content instead).

---

### Exercise 2: Design a Todo API {#exercise-2}

Design a RESTful API for managing todo items, specifying endpoints, HTTP methods, and status codes.

| Method | Endpoint | Description | Success Code | Error Codes |
|--------|----------|-------------|:------------:|:-----------:|
| GET | `/api/todos` | List all todos | 200 | - |
| GET | `/api/todos/{id}` | Get a single todo | 200 | 404 |
| POST | `/api/todos` | Create a new todo | 201 | 400 |
| PUT | `/api/todos/{id}` | Replace a todo entirely | 200 | 400, 404 |
| DELETE | `/api/todos/{id}` | Delete a todo | 204 | 404 |
| PATCH | `/api/todos/{id}/complete` | Mark a todo as complete | 200 | 404 |

**Example request/response bodies:**

```json
// POST /api/todos — Request
{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "dueDate": "2026-07-01"
}

// POST /api/todos — Response (201 Created)
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "dueDate": "2026-07-01",
  "completed": false
}

// PATCH /api/todos/1/complete — Response (200 OK)
{
  "id": 1,
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "dueDate": "2026-07-01",
  "completed": true
}
```

**Why this works:**
- Each endpoint maps to a single resource (`todos`) or sub-action (`complete`), following REST naming conventions.
- POST returns 201 to signal creation; DELETE returns 204 because the response body is empty after removal.
- PATCH on `/complete` is an action on a sub-resource, which is a common REST pattern for partial updates.

---

### Exercise 3: Fix Non-RESTful Endpoints {#exercise-3}

Convert verb-based endpoints to proper RESTful resource-based endpoints.

| Original (Non-RESTful) | Fixed (RESTful) | Why |
|------------------------|----------------|-----|
| `GET /api/getAllBooks` | `GET /api/books` | GET already means "retrieve"; use plural noun, drop the verb |
| `POST /api/createUser` | `POST /api/users` | POST already means "create"; use plural noun, drop the verb |
| `POST /api/books/delete/42` | `DELETE /api/books/42` | Use the DELETE HTTP method instead of encoding the action in the URL |
| `GET /api/searchBooks?title=x` | `GET /api/books?title=x` | Filtering is done via query params on the resource collection endpoint |
| `POST /api/updateUser/5` | `PUT /api/users/5` | Use PUT for full replacement; the HTTP method conveys the action |

**Why this works:**
- REST uses HTTP methods (GET, POST, PUT, DELETE) as verbs, so the URL should only contain nouns (resources).
- Query parameters handle filtering and searching on collection endpoints.
- Each resource has a consistent URL pattern: `/api/{resource}` for collections, `/api/{resource}/{id}` for individuals.

---

### Exercise 4: CRUD-to-HTTP Mapping {#exercise-4}

Map each CRUD operation to its HTTP method, URL pattern, and expected status code.

| CRUD Operation | HTTP Method | URL Pattern | Request Body | Success Code | Meaning |
|---------------|-------------|-------------|:------------:|:------------:|---------|
| **C**reate | POST | `/api/books` | Yes | 201 | Resource created |
| **R**ead (all) | GET | `/api/books` | No | 200 | Returns list |
| **R**ead (one) | GET | `/api/books/{id}` | No | 200 | Returns single item |
| **U**pdate (full) | PUT | `/api/books/{id}` | Yes | 200 | Resource replaced |
| **U**pdate (partial) | PATCH | `/api/books/{id}` | Yes | 200 | Resource patched |
| **D**elete | DELETE | `/api/books/{id}` | No | 204 | No content returned |

**Why this works:**
- Each CRUD operation maps naturally to one HTTP method, creating a predictable contract.
- POST targets the collection to create; GET, PUT, PATCH, and DELETE target individual resources by ID.
- 201 signals "something new was created"; 204 signals "success, but nothing to return."

---

### Exercise 5: E-Commerce Cart API Design {#exercise-5}

Design a RESTful API for managing an e-commerce shopping cart.

| Method | Endpoint | Description | Success | Errors |
|--------|----------|-------------|:-------:|:------:|
| POST | `/api/carts` | Create a new cart | 201 | 400 |
| GET | `/api/carts/{cartId}` | Get cart details with items | 200 | 404 |
| POST | `/api/carts/{cartId}/items` | Add item to cart | 201 | 400, 404 |
| PUT | `/api/carts/{cartId}/items/{itemId}` | Update item quantity | 200 | 400, 404 |
| DELETE | `/api/carts/{cartId}/items/{itemId}` | Remove item from cart | 204 | 404 |
| DELETE | `/api/carts/{cartId}` | Clear/delete entire cart | 204 | 404 |
| GET | `/api/carts/{cartId}/total` | Get cart total price | 200 | 404 |
| POST | `/api/carts/{cartId}/checkout` | Checkout the cart | 200 | 400, 404 |

**Example request/response bodies:**

```json
// POST /api/carts/{cartId}/items — Request
{
  "productId": 42,
  "quantity": 2
}

// POST /api/carts/{cartId}/items — Response (201 Created)
{
  "itemId": 1,
  "productId": 42,
  "productName": "Wireless Mouse",
  "unitPrice": 29.99,
  "quantity": 2,
  "subtotal": 59.98
}

// GET /api/carts/{cartId} — Response (200 OK)
{
  "cartId": "abc-123",
  "items": [
    {
      "itemId": 1,
      "productId": 42,
      "productName": "Wireless Mouse",
      "unitPrice": 29.99,
      "quantity": 2,
      "subtotal": 59.98
    }
  ],
  "totalItems": 2,
  "totalPrice": 59.98
}
```

**Why this works:**
- Cart items are a sub-resource of carts, modeled as `/carts/{id}/items`.
- Checkout is modeled as a POST action because it triggers a side effect (creating an order).
- Total is a computed read-only value, so it uses GET on a derived sub-resource.

---

### Exercise 6: curl Command Analysis {#exercise-6}

Analyze the following curl commands and identify the HTTP method, URL, headers, and body.

```bash
# Command 1
curl -X PUT http://localhost:8080/api/books/3 \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Title", "author": "Jane Doe"}'
```

**Analysis:**
- **Method:** PUT (full update/replacement)
- **URL:** `http://localhost:8080/api/books/3` (updating book with ID 3)
- **Headers:** `Content-Type: application/json` (telling the server the body is JSON)
- **Body:** `{"title": "Updated Title", "author": "Jane Doe"}`
- **Expected response:** 200 OK with the updated book, or 404 if book 3 does not exist

```bash
# Command 2
curl -X DELETE http://localhost:8080/api/books/3
```

**Analysis:**
- **Method:** DELETE (remove the resource)
- **URL:** `http://localhost:8080/api/books/3` (deleting book with ID 3)
- **Headers:** None specified (DELETE typically has no body)
- **Body:** None
- **Expected response:** 204 No Content on success, 404 if not found

```bash
# Command 3
curl -s http://localhost:8080/api/books?author=Tolkien&sort=title
```

**Analysis:**
- **Method:** GET (default when no `-X` is specified)
- **URL:** `http://localhost:8080/api/books` with query params `author=Tolkien` and `sort=title`
- **Headers:** None specified
- **Body:** None
- **Expected response:** 200 OK with a filtered, sorted list of books by Tolkien

**Why this works:**
- curl defaults to GET when no `-X` flag is provided.
- `-H` sets request headers; `-d` sets the request body and implies POST unless `-X` overrides.
- Query parameters after `?` filter/sort collection endpoints without changing the resource path.

---

## Section 2: JSON Fundamentals

### Exercise 7: Java to JSON Conversions {#exercise-7}

Convert a Java class to its JSON representation and back.

**Java class:**

```java
public class User {
    private String name;
    private String email;
    private int age;
    private List<String> hobbies;

    // Constructor
    public User(String name, String email, int age, List<String> hobbies) {
        this.name = name;
        this.email = email;
        this.age = age;
        this.hobbies = hobbies;
    }

    // Getters and setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public List<String> getHobbies() { return hobbies; }
    public void setHobbies(List<String> hobbies) { this.hobbies = hobbies; }
}
```

**JSON representation:**

```json
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "age": 28,
  "hobbies": ["reading", "hiking", "photography"]
}
```

**Key rules:**
- Java field names become JSON keys (camelCase is preserved).
- `String` maps to JSON string (with double quotes).
- `int` maps to JSON number (no quotes).
- `List<String>` maps to a JSON array of strings.
- `boolean` maps to JSON `true`/`false`.
- `null` fields can be omitted or included as JSON `null`.

**Why this works:**
- Spring Boot uses Jackson to automatically convert between Java objects and JSON.
- Jackson reads getter methods to determine field names: `getName()` becomes `"name"` in JSON.
- Java `List` and array types serialize to JSON arrays `[]`.

---

### Exercise 8: Fix Broken JSON {#exercise-8}

Each JSON below has exactly one error. Find and fix it.

**1. Trailing comma:**
```json
// BROKEN
{"name": "Alice", "age": 25,}

// FIXED
{"name": "Alice", "age": 25}
```
**Error:** Trailing comma after the last property. JSON does not allow trailing commas.

**2. Single quotes:**
```json
// BROKEN
{'name': 'Alice', 'age': 25}

// FIXED
{"name": "Alice", "age": 25}
```
**Error:** JSON requires double quotes for strings and keys, not single quotes.

**3. Unquoted key:**
```json
// BROKEN
{name: "Alice", "age": 25}

// FIXED
{"name": "Alice", "age": 25}
```
**Error:** All JSON keys must be wrapped in double quotes.

**4. Comment:**
```json
// BROKEN
{
  "name": "Alice", // user's name
  "age": 25
}

// FIXED
{
  "name": "Alice",
  "age": 25
}
```
**Error:** JSON does not support comments. Remove the `// user's name` comment.

**5. Missing colon:**
```json
// BROKEN
{"name" "Alice", "age": 25}

// FIXED
{"name": "Alice", "age": 25}
```
**Error:** Missing colon `:` between the key `"name"` and value `"Alice"`.

**Why this works:**
- JSON is strict: double quotes only, no trailing commas, no comments, colon separators required.
- These are the five most common JSON errors that beginners make.
- Use a JSON validator (like jsonlint.com) to catch these quickly.

---

### Exercise 9: University Nested JSON {#exercise-9}

Model a university with departments, each containing courses, each containing enrolled students.

```json
{
  "universityName": "Springfield University",
  "founded": 1952,
  "departments": [
    {
      "name": "Computer Science",
      "head": "Dr. Sarah Chen",
      "courses": [
        {
          "courseCode": "CS101",
          "title": "Introduction to Programming",
          "credits": 3,
          "students": [
            {
              "studentId": "S1001",
              "name": "Alice Johnson",
              "year": 2
            },
            {
              "studentId": "S1002",
              "name": "Bob Smith",
              "year": 1
            }
          ]
        },
        {
          "courseCode": "CS201",
          "title": "Data Structures",
          "credits": 4,
          "students": [
            {
              "studentId": "S1001",
              "name": "Alice Johnson",
              "year": 2
            },
            {
              "studentId": "S1003",
              "name": "Carol Davis",
              "year": 3
            }
          ]
        }
      ]
    },
    {
      "name": "Mathematics",
      "head": "Dr. James Park",
      "courses": [
        {
          "courseCode": "MATH101",
          "title": "Calculus I",
          "credits": 4,
          "students": [
            {
              "studentId": "S1002",
              "name": "Bob Smith",
              "year": 1
            },
            {
              "studentId": "S1004",
              "name": "Diana Lee",
              "year": 2
            }
          ]
        }
      ]
    }
  ]
}
```

**Why this works:**
- Each nesting level uses an array `[]` of objects `{}`, representing one-to-many relationships.
- A student can appear in multiple courses (Alice is in both CS101 and CS201), reflecting real-world enrollment.
- Each level has its own identifying fields (`courseCode`, `studentId`) for uniqueness.

---

### Exercise 10: Weather API Java Records {#exercise-10}

Given the following weather API JSON response, write Java record classes to deserialize it.

**JSON response:**

```json
{
  "city": "London",
  "country": "UK",
  "temperature": {
    "current": 15.5,
    "feelsLike": 13.2,
    "min": 12.0,
    "max": 18.0,
    "unit": "Celsius"
  },
  "wind": {
    "speed": 24.5,
    "direction": "NW",
    "unit": "km/h"
  },
  "conditions": ["Partly Cloudy", "Windy"],
  "lastUpdated": "2026-06-25T14:30:00Z"
}
```

**Java records:**

```java
import java.time.Instant;
import java.util.List;

public record WeatherResponse(
    String city,
    String country,
    Temperature temperature,
    Wind wind,
    List<String> conditions,
    Instant lastUpdated
) {}

public record Temperature(
    double current,
    double feelsLike,
    double min,
    double max,
    String unit
) {}

public record Wind(
    double speed,
    String direction,
    String unit
) {}
```

**Why this works:**
- Java records are immutable data carriers, perfect for deserializing JSON responses.
- Jackson automatically maps JSON keys to record component names (camelCase matching).
- Nested JSON objects map to nested record types; JSON arrays map to `List<>`.
- `Instant` handles ISO-8601 timestamps like `"2026-06-25T14:30:00Z"` automatically.

---

## Section 3: Spring Boot Controllers

### Exercise 11: Simple GreetingController {#exercise-11}

Create a controller with a single GET endpoint that returns "Hello, World!".

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

    @GetMapping("/greet")
    public String greet() {
        return "Hello, World!";
    }
}
```

**Test with curl:**

```bash
curl http://localhost:8080/greet
# Response: Hello, World!
# Status: 200 OK
```

**Why this works:**
- `@RestController` tells Spring this class handles HTTP requests and returns data (not HTML views).
- `@GetMapping("/greet")` maps GET requests to `/greet` to this method.
- Returning a plain `String` sends it as the response body with a 200 status code.

---

### Exercise 12: Returning a JSON Object {#exercise-12}

Create a controller that returns a Product object, which Spring automatically converts to JSON.

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api")
public class ProductController {

    @GetMapping("/product")
    public Product getProduct() {
        return new Product(1L, "Mechanical Keyboard", 79.99, true);
    }
}

class Product {
    private Long id;
    private String name;
    private double price;
    private boolean available;

    public Product(Long id, String name, double price, boolean available) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.available = available;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }

    public boolean isAvailable() { return available; }
    public void setAvailable(boolean available) { this.available = available; }
}
```

**Test with curl:**

```bash
curl http://localhost:8080/api/product
```

**Expected response:**

```json
{
  "id": 1,
  "name": "Mechanical Keyboard",
  "price": 79.99,
  "available": true
}
```

**Why this works:**
- Spring Boot includes Jackson on the classpath, which auto-converts Java objects to JSON.
- Jackson reads the getter methods (`getName()` becomes `"name"`, `isAvailable()` becomes `"available"`).
- The response automatically gets `Content-Type: application/json` header.

---

### Exercise 13: Full CRUD ProductController {#exercise-13}

Build a complete ProductController with GET, POST, PUT, and DELETE using an in-memory list.

```java
package com.example.demo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final List<Product> products = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    // GET all products
    @GetMapping
    public List<Product> getAllProducts() {
        return products;
    }

    // GET product by ID
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst()
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // POST create a new product
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        product.setId(idCounter.getAndIncrement());
        products.add(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(product);
    }

    // PUT update an existing product
    @PutMapping("/{id}")
    public ResponseEntity<Product> updateProduct(@PathVariable Long id,
                                                  @RequestBody Product updated) {
        for (int i = 0; i < products.size(); i++) {
            if (products.get(i).getId().equals(id)) {
                updated.setId(id);
                products.set(i, updated);
                return ResponseEntity.ok(updated);
            }
        }
        return ResponseEntity.notFound().build();
    }

    // DELETE a product
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        boolean removed = products.removeIf(p -> p.getId().equals(id));
        if (removed) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

**Test with curl:**

```bash
# Create two products
curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Keyboard", "price": 79.99, "available": true}'
# 201 Created

curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Mouse", "price": 29.99, "available": true}'
# 201 Created

# Get all products
curl http://localhost:8080/api/products
# 200 OK — returns array of 2 products

# Update product 1
curl -X PUT http://localhost:8080/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "Mechanical Keyboard", "price": 99.99, "available": true}'
# 200 OK

# Delete product 2
curl -X DELETE http://localhost:8080/api/products/2
# 204 No Content
```

**Why this works:**
- `AtomicLong` generates thread-safe unique IDs for each new product.
- `ResponseEntity` lets us control both the response body and HTTP status code.
- Each HTTP method maps to one CRUD operation, following REST conventions.

---

### Exercise 14: @PathVariable {#exercise-14}

Use `@PathVariable` to extract an ID from the URL path and find a product from a list.

```java
package com.example.demo.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final List<Product> products = List.of(
            new Product(1L, "Laptop", 999.99, true),
            new Product(2L, "Phone", 699.99, true),
            new Product(3L, "Tablet", 449.99, false)
    );

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst()
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
}
```

**Test with curl:**

```bash
curl http://localhost:8080/api/products/1
# 200 OK — {"id":1,"name":"Laptop","price":999.99,"available":true}

curl http://localhost:8080/api/products/99
# 404 Not Found
```

**Why this works:**
- `@PathVariable Long id` extracts the `{id}` segment from the URL and converts it to a `Long`.
- Spring returns 404 automatically when we return `ResponseEntity.notFound().build()`.
- The stream filters the list in memory and wraps the result in the appropriate response.

---

### Exercise 15: @RequestParam for Filtering {#exercise-15}

Use `@RequestParam` to filter products by category and minimum price.

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final List<Product> products = List.of(
            new Product(1L, "Gaming Laptop", 1299.99, "electronics", true),
            new Product(2L, "Office Laptop", 799.99, "electronics", true),
            new Product(3L, "Java Textbook", 49.99, "books", true),
            new Product(4L, "USB Cable", 9.99, "electronics", true),
            new Product(5L, "Spring in Action", 39.99, "books", true)
    );

    @GetMapping
    public List<Product> getProducts(
            @RequestParam(required = false) String category,
            @RequestParam(required = false) Double minPrice) {

        return products.stream()
                .filter(p -> category == null || p.getCategory().equalsIgnoreCase(category))
                .filter(p -> minPrice == null || p.getPrice() >= minPrice)
                .collect(Collectors.toList());
    }
}

class Product {
    private Long id;
    private String name;
    private double price;
    private String category;
    private boolean available;

    public Product(Long id, String name, double price, String category, boolean available) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.category = category;
        this.available = available;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
    public String getCategory() { return category; }
    public void setCategory(String category) { this.category = category; }
    public boolean isAvailable() { return available; }
    public void setAvailable(boolean available) { this.available = available; }
}
```

**Test with curl:**

```bash
# All products
curl "http://localhost:8080/api/products"
# Returns all 5 products

# Filter by category
curl "http://localhost:8080/api/products?category=books"
# Returns 2 books

# Filter by minimum price
curl "http://localhost:8080/api/products?minPrice=100"
# Returns 2 laptops

# Combine filters
curl "http://localhost:8080/api/products?category=electronics&minPrice=500"
# Returns 2 laptops (USB Cable excluded by price)
```

**Why this works:**
- `@RequestParam(required = false)` makes query parameters optional, so the endpoint works with or without filters.
- When a parameter is `null` (not provided), the filter predicate passes all items through.
- Multiple filters compose naturally with chained `.filter()` calls.

---

### Exercise 16: @RequestBody {#exercise-16}

Use `@RequestBody` to accept a JSON body in a POST request and create a product.

```java
package com.example.demo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final List<Product> products = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        product.setId(idCounter.getAndIncrement());
        products.add(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(product);
    }

    @GetMapping
    public List<Product> getAllProducts() {
        return products;
    }
}
```

**Test with curl:**

```bash
curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Wireless Mouse", "price": 29.99, "available": true}'
```

**Expected response (201 Created):**

```json
{
  "id": 1,
  "name": "Wireless Mouse",
  "price": 29.99,
  "available": true
}
```

**Why this works:**
- `@RequestBody` tells Spring to deserialize the JSON request body into a `Product` object using Jackson.
- The `Content-Type: application/json` header is required so Spring knows to use JSON deserialization.
- The server assigns the ID (clients should never set their own IDs in a POST).

---

### Exercise 17: ResponseEntity Status Codes {#exercise-17}

Use `ResponseEntity` to return proper HTTP status codes: 201 for creation, 204 for deletion, 404 for not found.

```java
package com.example.demo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final List<Product> products = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    // 200 OK — successful retrieval
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst()
                .map(product -> ResponseEntity.ok(product))           // 200
                .orElse(ResponseEntity.notFound().build());            // 404
    }

    // 201 Created — resource successfully created
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        product.setId(idCounter.getAndIncrement());
        products.add(product);
        URI location = URI.create("/api/products/" + product.getId());
        return ResponseEntity.created(location).body(product);        // 201
    }

    // 204 No Content — successful deletion, nothing to return
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        boolean removed = products.removeIf(p -> p.getId().equals(id));
        if (removed) {
            return ResponseEntity.noContent().build();                 // 204
        }
        return ResponseEntity.notFound().build();                      // 404
    }
}
```

**Test with curl:**

```bash
# Create — 201 Created with Location header
curl -v -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name": "Keyboard", "price": 79.99, "available": true}'
# HTTP/1.1 201 Created
# Location: /api/products/1

# Get existing — 200 OK
curl -o /dev/null -s -w "%{http_code}" http://localhost:8080/api/products/1
# 200

# Get non-existent — 404 Not Found
curl -o /dev/null -s -w "%{http_code}" http://localhost:8080/api/products/999
# 404

# Delete — 204 No Content
curl -o /dev/null -s -w "%{http_code}" -X DELETE http://localhost:8080/api/products/1
# 204
```

**Why this works:**
- `ResponseEntity.created(location)` returns 201 and sets the `Location` header pointing to the new resource.
- `ResponseEntity.noContent().build()` returns 204 with an empty body, ideal for successful deletions.
- `ResponseEntity.notFound().build()` returns 404 with an empty body for missing resources.

---

### Exercise 18: Complete NotesController {#exercise-18}

Build a full NotesController with in-memory storage, all CRUD operations, proper status codes, and auto-generated IDs.

```java
package com.example.demo.controller;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicLong;

@RestController
@RequestMapping("/api/notes")
public class NotesController {

    private final List<Note> notes = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    // GET /api/notes — list all notes
    @GetMapping
    public List<Note> getAllNotes() {
        return notes;
    }

    // GET /api/notes/{id} — get a single note
    @GetMapping("/{id}")
    public ResponseEntity<Note> getNoteById(@PathVariable Long id) {
        return findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // POST /api/notes — create a new note
    @PostMapping
    public ResponseEntity<Note> createNote(@RequestBody Note note) {
        note.setId(idCounter.getAndIncrement());
        note.setCreatedAt(LocalDateTime.now());
        note.setUpdatedAt(LocalDateTime.now());
        notes.add(note);
        return ResponseEntity.status(HttpStatus.CREATED).body(note);
    }

    // PUT /api/notes/{id} — update an existing note
    @PutMapping("/{id}")
    public ResponseEntity<Note> updateNote(@PathVariable Long id,
                                            @RequestBody Note updated) {
        for (int i = 0; i < notes.size(); i++) {
            Note existing = notes.get(i);
            if (existing.getId().equals(id)) {
                updated.setId(id);
                updated.setCreatedAt(existing.getCreatedAt());
                updated.setUpdatedAt(LocalDateTime.now());
                notes.set(i, updated);
                return ResponseEntity.ok(updated);
            }
        }
        return ResponseEntity.notFound().build();
    }

    // DELETE /api/notes/{id} — delete a note
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteNote(@PathVariable Long id) {
        boolean removed = notes.removeIf(n -> n.getId().equals(id));
        if (removed) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }

    private java.util.Optional<Note> findById(Long id) {
        return notes.stream()
                .filter(n -> n.getId().equals(id))
                .findFirst();
    }
}

class Note {
    private Long id;
    private String title;
    private String content;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    public Note() {}

    public Note(Long id, String title, String content) {
        this.id = id;
        this.title = title;
        this.content = content;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }

    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

**Test with curl:**

```bash
# Create a note
curl -X POST http://localhost:8080/api/notes \
  -H "Content-Type: application/json" \
  -d '{"title": "Meeting Notes", "content": "Discuss Q3 roadmap"}'
# 201 Created — {"id":1,"title":"Meeting Notes","content":"Discuss Q3 roadmap","createdAt":"...","updatedAt":"..."}

# Get all notes
curl http://localhost:8080/api/notes
# 200 OK — [{"id":1,...}]

# Get single note
curl http://localhost:8080/api/notes/1
# 200 OK

# Update the note
curl -X PUT http://localhost:8080/api/notes/1 \
  -H "Content-Type: application/json" \
  -d '{"title": "Meeting Notes (Updated)", "content": "Discuss Q3 roadmap and budget"}'
# 200 OK — updatedAt changes, createdAt stays the same

# Delete the note
curl -X DELETE http://localhost:8080/api/notes/1
# 204 No Content

# Verify deletion
curl -o /dev/null -s -w "%{http_code}" http://localhost:8080/api/notes/1
# 404
```

**Why this works:**
- `AtomicLong` ensures unique, thread-safe ID generation without a database.
- `createdAt` is set once during creation; `updatedAt` is refreshed on every update.
- PUT preserves the original `createdAt` timestamp while updating everything else.
- The `findById` helper method keeps the controller methods clean and DRY.

---

## Section 4: Dependency Injection

### Exercise 19: Extract PriceCalculator {#exercise-19}

Refactor an OrderController that has pricing logic inline, extracting it into a separate PriceCalculator service injected via constructor.

**Before (everything in the controller):**

```java
// BAD — business logic lives in the controller
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        double subtotal = request.getPrice() * request.getQuantity();
        double tax = subtotal * 0.08;
        double shipping = subtotal > 100 ? 0 : 9.99;
        double total = subtotal + tax + shipping;
        // ... save order, return response
    }
}
```

**After (refactored with DI):**

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;

@Service
public class PriceCalculator {

    private static final double TAX_RATE = 0.08;
    private static final double SHIPPING_COST = 9.99;
    private static final double FREE_SHIPPING_THRESHOLD = 100.0;

    public double calculateSubtotal(double price, int quantity) {
        return price * quantity;
    }

    public double calculateTax(double subtotal) {
        return subtotal * TAX_RATE;
    }

    public double calculateShipping(double subtotal) {
        return subtotal > FREE_SHIPPING_THRESHOLD ? 0 : SHIPPING_COST;
    }

    public double calculateTotal(double price, int quantity) {
        double subtotal = calculateSubtotal(price, quantity);
        double tax = calculateTax(subtotal);
        double shipping = calculateShipping(subtotal);
        return subtotal + tax + shipping;
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.service.PriceCalculator;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final PriceCalculator priceCalculator;

    // Constructor injection — Spring automatically injects PriceCalculator
    public OrderController(PriceCalculator priceCalculator) {
        this.priceCalculator = priceCalculator;
    }

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody OrderRequest request) {
        double subtotal = priceCalculator.calculateSubtotal(
                request.getPrice(), request.getQuantity());
        double tax = priceCalculator.calculateTax(subtotal);
        double shipping = priceCalculator.calculateShipping(subtotal);
        double total = priceCalculator.calculateTotal(
                request.getPrice(), request.getQuantity());

        OrderResponse response = new OrderResponse(subtotal, tax, shipping, total);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}

class OrderRequest {
    private double price;
    private int quantity;

    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
    public int getQuantity() { return quantity; }
    public void setQuantity(int quantity) { this.quantity = quantity; }
}

class OrderResponse {
    private double subtotal;
    private double tax;
    private double shipping;
    private double total;

    public OrderResponse(double subtotal, double tax, double shipping, double total) {
        this.subtotal = subtotal;
        this.tax = tax;
        this.shipping = shipping;
        this.total = total;
    }

    public double getSubtotal() { return subtotal; }
    public double getTax() { return tax; }
    public double getShipping() { return shipping; }
    public double getTotal() { return total; }
}
```

**Why this works:**
- The controller is now thin: it only handles HTTP concerns (request/response). Business logic lives in `PriceCalculator`.
- Constructor injection makes the dependency explicit and immutable (`final` field).
- `PriceCalculator` can be unit-tested independently without spinning up a web server.

---

### Exercise 20: Interface with Multiple Implementations {#exercise-20}

Create a `GreetingService` interface with two implementations, using `@Primary` to pick the default.

```java
package com.example.demo.service;

public interface GreetingService {
    String greet(String name);
}
```

```java
package com.example.demo.service;

import org.springframework.context.annotation.Primary;
import org.springframework.stereotype.Service;

@Service
@Primary
public class FormalGreetingService implements GreetingService {

    @Override
    public String greet(String name) {
        return "Good day, " + name + ". How may I assist you?";
    }
}
```

```java
package com.example.demo.service;

import org.springframework.stereotype.Service;

@Service
public class CasualGreetingService implements GreetingService {

    @Override
    public String greet(String name) {
        return "Hey " + name + "! What's up?";
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.service.GreetingService;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/greetings")
public class GreetingController {

    private final GreetingService formalService;
    private final GreetingService casualService;

    public GreetingController(
            GreetingService formalService,
            @Qualifier("casualGreetingService") GreetingService casualService) {
        this.formalService = formalService;   // Gets @Primary (FormalGreetingService)
        this.casualService = casualService;   // Gets the specific @Qualifier bean
    }

    @GetMapping("/formal")
    public String formalGreeting(@RequestParam String name) {
        return formalService.greet(name);
    }

    @GetMapping("/casual")
    public String casualGreeting(@RequestParam String name) {
        return casualService.greet(name);
    }
}
```

**Test with curl:**

```bash
curl "http://localhost:8080/api/greetings/formal?name=Alice"
# "Good day, Alice. How may I assist you?"

curl "http://localhost:8080/api/greetings/casual?name=Alice"
# "Hey Alice! What's up?"
```

**Why this works:**
- `@Primary` marks `FormalGreetingService` as the default when Spring has multiple candidates for `GreetingService`.
- `@Qualifier("casualGreetingService")` overrides the default and injects the specific bean by name.
- Programming to the interface (`GreetingService`) means we can swap implementations without changing the controller.

---

### Exercise 21: Field Injection to Constructor Injection {#exercise-21}

Refactor field injection (anti-pattern) to constructor injection (best practice).

**Before (field injection -- avoid):**

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderService orderService;

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private NotificationService notificationService;

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Order order = orderService.create(request);
        paymentService.charge(order);
        notificationService.sendConfirmation(order);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}
```

**After (constructor injection -- recommended):**

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;
    private final PaymentService paymentService;
    private final NotificationService notificationService;

    // Single constructor — @Autowired is optional when there's only one constructor
    public OrderController(OrderService orderService,
                           PaymentService paymentService,
                           NotificationService notificationService) {
        this.orderService = orderService;
        this.paymentService = paymentService;
        this.notificationService = notificationService;
    }

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Order order = orderService.create(request);
        paymentService.charge(order);
        notificationService.sendConfirmation(order);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
}
```

**Key differences:**

| Aspect | Field Injection | Constructor Injection |
|--------|:---------------:|:--------------------:|
| Fields are `final` | No | Yes |
| `@Autowired` needed | Yes, on every field | No (single constructor) |
| Testability | Hard (need reflection) | Easy (pass mocks in constructor) |
| Immutability | Mutable | Immutable |
| Missing dependency | Runtime NullPointerException | Compile-time / startup error |

**Why this works:**
- `final` fields guarantee dependencies cannot be reassigned after construction.
- Constructor injection fails fast at startup if a dependency is missing, rather than crashing at runtime.
- In unit tests, you simply pass mock objects to the constructor -- no reflection or Spring context needed.

---

### Exercise 22: NotificationController with Profiles {#exercise-22}

Create a `NotificationService` interface with Email and SMS implementations, selectable via Spring profiles.

```java
package com.example.demo.service;

public interface NotificationService {
    String send(String recipient, String message);
}
```

```java
package com.example.demo.service;

import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("email")
public class EmailNotificationService implements NotificationService {

    @Override
    public String send(String recipient, String message) {
        // In production, this would use JavaMailSender
        return "Email sent to " + recipient + ": " + message;
    }
}
```

```java
package com.example.demo.service;

import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Service;

@Service
@Profile("sms")
public class SmsNotificationService implements NotificationService {

    @Override
    public String send(String recipient, String message) {
        // In production, this would use Twilio or similar
        return "SMS sent to " + recipient + ": " + message;
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.service.NotificationService;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/notifications")
public class NotificationController {

    private final NotificationService notificationService;

    public NotificationController(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    @PostMapping
    public ResponseEntity<NotificationResponse> sendNotification(
            @RequestBody NotificationRequest request) {
        String result = notificationService.send(
                request.getRecipient(), request.getMessage());
        return ResponseEntity.ok(new NotificationResponse(result));
    }
}

class NotificationRequest {
    private String recipient;
    private String message;

    public String getRecipient() { return recipient; }
    public void setRecipient(String recipient) { this.recipient = recipient; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
}

class NotificationResponse {
    private String result;

    public NotificationResponse(String result) { this.result = result; }

    public String getResult() { return result; }
}
```

**Activate a profile:**

```properties
# application.properties
spring.profiles.active=email
```

Or via command line:

```bash
java -jar app.jar --spring.profiles.active=sms
```

**Test with curl:**

```bash
curl -X POST http://localhost:8080/api/notifications \
  -H "Content-Type: application/json" \
  -d '{"recipient": "alice@example.com", "message": "Your order shipped!"}'

# With email profile: {"result":"Email sent to alice@example.com: Your order shipped!"}
# With sms profile:   {"result":"SMS sent to alice@example.com: Your order shipped!"}
```

**Why this works:**
- `@Profile("email")` means this bean is only created when the "email" profile is active.
- The controller depends on the `NotificationService` interface, not a specific implementation.
- Switching notification channels requires zero code changes -- just change the active profile.

---

## Section 5: Layered Architecture

### Exercise 23: Refactor Monolithic Controller {#exercise-23}

Refactor a controller that contains business logic and data access into proper Controller, Service, and Repository layers.

**Before (monolithic -- everything in one class):**

```java
// BAD — controller does everything
@RestController
@RequestMapping("/api/products")
public class ProductController {
    private final List<Product> products = new ArrayList<>();
    private final AtomicLong counter = new AtomicLong(1);

    @PostMapping
    public ResponseEntity<Product> create(@RequestBody Product product) {
        // Validation (should be in service)
        if (product.getPrice() < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        // Business logic (should be in service)
        product.setId(counter.getAndIncrement());
        product.setCreatedAt(LocalDateTime.now());
        if (product.getPrice() > 1000) {
            product.setCategory("premium");
        }
        // Data access (should be in repository)
        products.add(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(product);
    }
}
```

**After (three layers):**

```java
package com.example.demo.repository;

import com.example.demo.model.Product;
import org.springframework.stereotype.Repository;

import java.util.*;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class ProductRepository {

    private final List<Product> products = new ArrayList<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    public Product save(Product product) {
        if (product.getId() == null) {
            product.setId(idCounter.getAndIncrement());
            products.add(product);
        } else {
            // Update existing
            for (int i = 0; i < products.size(); i++) {
                if (products.get(i).getId().equals(product.getId())) {
                    products.set(i, product);
                    break;
                }
            }
        }
        return product;
    }

    public Optional<Product> findById(Long id) {
        return products.stream()
                .filter(p -> p.getId().equals(id))
                .findFirst();
    }

    public List<Product> findAll() {
        return Collections.unmodifiableList(products);
    }

    public boolean deleteById(Long id) {
        return products.removeIf(p -> p.getId().equals(id));
    }
}
```

```java
package com.example.demo.service;

import com.example.demo.model.Product;
import com.example.demo.repository.ProductRepository;
import org.springframework.stereotype.Service;

import java.time.LocalDateTime;
import java.util.List;

@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public Product createProduct(Product product) {
        if (product.getPrice() < 0) {
            throw new IllegalArgumentException("Price cannot be negative");
        }
        product.setCreatedAt(LocalDateTime.now());
        if (product.getPrice() > 1000) {
            product.setCategory("premium");
        }
        return productRepository.save(product);
    }

    public Product getProductById(Long id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Product not found with id: " + id));
    }

    public List<Product> getAllProducts() {
        return productRepository.findAll();
    }

    public void deleteProduct(Long id) {
        if (!productRepository.deleteById(id)) {
            throw new RuntimeException("Product not found with id: " + id);
        }
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.model.Product;
import com.example.demo.service.ProductService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping
    public List<Product> getAllProducts() {
        return productService.getAllProducts();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        Product product = productService.getProductById(id);
        return ResponseEntity.ok(product);
    }

    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody Product product) {
        Product created = productService.createProduct(product);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteProduct(@PathVariable Long id) {
        productService.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Why this works:**
- **Controller** handles only HTTP concerns: extracting request data, calling the service, and building the response.
- **Service** holds business logic: validation, categorization, timestamps.
- **Repository** handles data access: storing, finding, and deleting from the data store.
- Each layer can be tested independently. Replacing the in-memory repository with a database requires changing only the repository layer.

---

### Exercise 24: Identify Layer Violations {#exercise-24}

Find 5 layer violations in the following code and explain how to fix each one.

**Code with violations:**

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private OrderRepository orderRepository;  // VIOLATION 1

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        // VIOLATION 2: Business logic in controller
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
        double total = 0;
        for (OrderItem item : order.getItems()) {
            total += item.getPrice() * item.getQuantity();
        }
        order.setTotal(total);

        // VIOLATION 3: Direct repository call from controller
        Order saved = orderRepository.save(order);

        // VIOLATION 4: Infrastructure concern (sending email) in controller
        sendEmail(order.getCustomerEmail(), "Order confirmed: " + saved.getId());

        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }

    // VIOLATION 5: Utility method in controller
    private void sendEmail(String to, String body) {
        System.out.println("Sending email to " + to + ": " + body);
    }
}
```

**Violations and fixes:**

| # | Violation | Layer Rule Broken | Fix |
|---|-----------|------------------|-----|
| 1 | Controller injects `OrderRepository` directly | Controller should only depend on Service | Inject `OrderService` instead; the service injects the repository |
| 2 | Validation and total calculation in the controller | Business logic belongs in the Service layer | Move validation and total calculation to `OrderService.createOrder()` |
| 3 | `orderRepository.save()` called from the controller | Data access should only be called from the Service (or Repository) layer | Call `orderService.createOrder(order)` which internally calls the repository |
| 4 | Email sending in the controller | Infrastructure/integration concerns belong in a dedicated Service | Create `EmailService` and call it from `OrderService` |
| 5 | `sendEmail` utility method in the controller | Controllers should have no private business/utility methods | Move to `EmailService` with `@Service` annotation |

**Fixed code:**

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody Order order) {
        Order created = orderService.createOrder(order);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
}

@Service
public class OrderService {

    private final OrderRepository orderRepository;
    private final EmailService emailService;

    public OrderService(OrderRepository orderRepository, EmailService emailService) {
        this.orderRepository = orderRepository;
        this.emailService = emailService;
    }

    public Order createOrder(Order order) {
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
        double total = order.getItems().stream()
                .mapToDouble(item -> item.getPrice() * item.getQuantity())
                .sum();
        order.setTotal(total);
        Order saved = orderRepository.save(order);
        emailService.sendOrderConfirmation(saved.getCustomerEmail(), saved.getId());
        return saved;
    }
}

@Service
public class EmailService {
    public void sendOrderConfirmation(String to, Long orderId) {
        // In production, use JavaMailSender
        System.out.println("Sending email to " + to + ": Order confirmed: " + orderId);
    }
}
```

**Why this works:**
- The controller is now a thin adapter: it delegates everything to the service.
- Business rules (validation, calculation) and orchestration (email notification) live in the service.
- `EmailService` is a separate service, making it reusable and testable independently.

---

### Exercise 25: Full 3-Layer Employee API {#exercise-25}

Build a complete Employee API with Controller, Service, and Repository layers using in-memory storage.

```java
package com.example.demo.model;

import java.time.LocalDate;

public class Employee {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String department;
    private double salary;
    private LocalDate hireDate;

    public Employee() {}

    public Employee(Long id, String firstName, String lastName, String email,
                    String department, double salary, LocalDate hireDate) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.department = department;
        this.salary = salary;
        this.hireDate = hireDate;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }
    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    public String getDepartment() { return department; }
    public void setDepartment(String department) { this.department = department; }
    public double getSalary() { return salary; }
    public void setSalary(double salary) { this.salary = salary; }
    public LocalDate getHireDate() { return hireDate; }
    public void setHireDate(LocalDate hireDate) { this.hireDate = hireDate; }
}
```

```java
package com.example.demo.repository;

import com.example.demo.model.Employee;
import org.springframework.stereotype.Repository;

import java.util.*;
import java.util.concurrent.atomic.AtomicLong;
import java.util.stream.Collectors;

@Repository
public class EmployeeRepository {

    private final Map<Long, Employee> employees = new LinkedHashMap<>();
    private final AtomicLong idCounter = new AtomicLong(1);

    public Employee save(Employee employee) {
        if (employee.getId() == null) {
            employee.setId(idCounter.getAndIncrement());
        }
        employees.put(employee.getId(), employee);
        return employee;
    }

    public Optional<Employee> findById(Long id) {
        return Optional.ofNullable(employees.get(id));
    }

    public List<Employee> findAll() {
        return new ArrayList<>(employees.values());
    }

    public List<Employee> findByDepartment(String department) {
        return employees.values().stream()
                .filter(e -> e.getDepartment().equalsIgnoreCase(department))
                .collect(Collectors.toList());
    }

    public boolean existsByEmail(String email) {
        return employees.values().stream()
                .anyMatch(e -> e.getEmail().equalsIgnoreCase(email));
    }

    public boolean deleteById(Long id) {
        return employees.remove(id) != null;
    }
}
```

```java
package com.example.demo.service;

import com.example.demo.model.Employee;
import com.example.demo.repository.EmployeeRepository;
import org.springframework.stereotype.Service;

import java.time.LocalDate;
import java.util.List;

@Service
public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(EmployeeRepository employeeRepository) {
        this.employeeRepository = employeeRepository;
    }

    public Employee createEmployee(Employee employee) {
        if (employeeRepository.existsByEmail(employee.getEmail())) {
            throw new IllegalArgumentException(
                    "Employee with email " + employee.getEmail() + " already exists");
        }
        if (employee.getSalary() < 0) {
            throw new IllegalArgumentException("Salary cannot be negative");
        }
        if (employee.getHireDate() == null) {
            employee.setHireDate(LocalDate.now());
        }
        return employeeRepository.save(employee);
    }

    public Employee getEmployeeById(Long id) {
        return employeeRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Employee not found with id: " + id));
    }

    public List<Employee> getAllEmployees() {
        return employeeRepository.findAll();
    }

    public List<Employee> getEmployeesByDepartment(String department) {
        return employeeRepository.findByDepartment(department);
    }

    public Employee updateEmployee(Long id, Employee updated) {
        Employee existing = getEmployeeById(id);
        updated.setId(existing.getId());
        updated.setHireDate(existing.getHireDate()); // preserve original hire date
        return employeeRepository.save(updated);
    }

    public void deleteEmployee(Long id) {
        if (!employeeRepository.deleteById(id)) {
            throw new RuntimeException("Employee not found with id: " + id);
        }
    }
}
```

```java
package com.example.demo.controller;

import com.example.demo.model.Employee;
import com.example.demo.service.EmployeeService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    private final EmployeeService employeeService;

    public EmployeeController(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    @GetMapping
    public List<Employee> getAllEmployees(
            @RequestParam(required = false) String department) {
        if (department != null) {
            return employeeService.getEmployeesByDepartment(department);
        }
        return employeeService.getAllEmployees();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployeeById(@PathVariable Long id) {
        Employee employee = employeeService.getEmployeeById(id);
        return ResponseEntity.ok(employee);
    }

    @PostMapping
    public ResponseEntity<Employee> createEmployee(@RequestBody Employee employee) {
        Employee created = employeeService.createEmployee(employee);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Employee> updateEmployee(@PathVariable Long id,
                                                    @RequestBody Employee employee) {
        Employee updated = employeeService.updateEmployee(id, employee);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteEmployee(@PathVariable Long id) {
        employeeService.deleteEmployee(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Test with curl:**

```bash
# Create an employee
curl -X POST http://localhost:8080/api/employees \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alice","lastName":"Johnson","email":"alice@company.com","department":"Engineering","salary":95000}'
# 201 Created

# Get all employees
curl http://localhost:8080/api/employees
# 200 OK — [{"id":1,...}]

# Filter by department
curl "http://localhost:8080/api/employees?department=Engineering"
# 200 OK — returns only Engineering employees

# Update
curl -X PUT http://localhost:8080/api/employees/1 \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alice","lastName":"Johnson","email":"alice@company.com","department":"Engineering","salary":105000}'
# 200 OK — salary updated

# Delete
curl -X DELETE http://localhost:8080/api/employees/1
# 204 No Content
```

**Why this works:**
- Three clean layers: Controller handles HTTP, Service handles business rules, Repository handles data storage.
- The repository uses a `Map` for O(1) lookups by ID, better than scanning a list.
- Business rules (duplicate email check, salary validation, default hire date) are all in the service layer.

---

### Exercise 26: Feature Design — Book Rental System {#exercise-26}

Design the layer responsibility for: "Users can rent books. Max 3 per user. Late fees: $0.50/day."

**Controller Layer** (HTTP handling only):

```java
@RestController
@RequestMapping("/api/rentals")
public class RentalController {

    private final RentalService rentalService;

    public RentalController(RentalService rentalService) {
        this.rentalService = rentalService;
    }

    @PostMapping
    public ResponseEntity<RentalResponse> rentBook(@RequestBody RentalRequest request) {
        // Delegates entirely to service
        RentalResponse rental = rentalService.rentBook(request.getUserId(), request.getBookId());
        return ResponseEntity.status(HttpStatus.CREATED).body(rental);
    }

    @PostMapping("/{rentalId}/return")
    public ResponseEntity<ReturnResponse> returnBook(@PathVariable Long rentalId) {
        ReturnResponse result = rentalService.returnBook(rentalId);
        return ResponseEntity.ok(result);
    }

    @GetMapping("/user/{userId}")
    public List<RentalResponse> getUserRentals(@PathVariable Long userId) {
        return rentalService.getActiveRentals(userId);
    }
}
```

**Service Layer** (business logic):

```java
@Service
public class RentalService {

    private static final int MAX_RENTALS_PER_USER = 3;
    private static final double LATE_FEE_PER_DAY = 0.50;
    private static final int RENTAL_PERIOD_DAYS = 14;

    private final RentalRepository rentalRepository;
    private final BookRepository bookRepository;
    private final UserRepository userRepository;

    // Constructor injection...

    public RentalResponse rentBook(Long userId, Long bookId) {
        // BUSINESS RULE: Check rental limit
        long activeRentals = rentalRepository.countActiveRentalsByUserId(userId);
        if (activeRentals >= MAX_RENTALS_PER_USER) {
            throw new IllegalStateException(
                "User has reached maximum of " + MAX_RENTALS_PER_USER + " rentals");
        }

        // BUSINESS RULE: Check book availability
        Book book = bookRepository.findById(bookId)
                .orElseThrow(() -> new RuntimeException("Book not found"));
        if (!book.isAvailable()) {
            throw new IllegalStateException("Book is currently not available");
        }

        // Create rental with calculated due date
        Rental rental = new Rental();
        rental.setUserId(userId);
        rental.setBookId(bookId);
        rental.setRentedAt(LocalDate.now());
        rental.setDueDate(LocalDate.now().plusDays(RENTAL_PERIOD_DAYS));
        rental.setStatus("ACTIVE");

        rentalRepository.save(rental);
        book.setAvailable(false);
        bookRepository.save(book);

        return toResponse(rental);
    }

    public ReturnResponse returnBook(Long rentalId) {
        Rental rental = rentalRepository.findById(rentalId)
                .orElseThrow(() -> new RuntimeException("Rental not found"));

        rental.setReturnedAt(LocalDate.now());
        rental.setStatus("RETURNED");

        // BUSINESS RULE: Calculate late fee
        double lateFee = 0;
        if (LocalDate.now().isAfter(rental.getDueDate())) {
            long daysLate = ChronoUnit.DAYS.between(rental.getDueDate(), LocalDate.now());
            lateFee = daysLate * LATE_FEE_PER_DAY;
        }
        rental.setLateFee(lateFee);

        rentalRepository.save(rental);

        // Mark book as available again
        Book book = bookRepository.findById(rental.getBookId()).orElseThrow();
        book.setAvailable(true);
        bookRepository.save(book);

        return new ReturnResponse(rentalId, lateFee, rental.getReturnedAt());
    }
}
```

**Repository Layer** (data access):

```java
@Repository
public class RentalRepository {
    // save(Rental), findById(Long), countActiveRentalsByUserId(Long),
    // findActiveRentalsByUserId(Long)
}
```

**Summary of responsibilities:**

| Concern | Layer | Example |
|---------|-------|---------|
| Parse HTTP request | Controller | `@RequestBody RentalRequest` |
| Return HTTP status codes | Controller | `ResponseEntity.status(CREATED)` |
| Check max 3 rentals | Service | `if (activeRentals >= MAX_RENTALS_PER_USER)` |
| Calculate late fees | Service | `daysLate * LATE_FEE_PER_DAY` |
| Calculate due date | Service | `LocalDate.now().plusDays(14)` |
| Store/retrieve rental records | Repository | `rentalRepository.save(rental)` |
| Query active rental count | Repository | `countActiveRentalsByUserId(userId)` |

**Why this works:**
- All business rules (max 3 rentals, late fee calculation, rental period) are centralized in the service.
- If the late fee formula changes from $0.50/day to a tiered rate, only the service changes.
- The controller never knows about rental limits or fee calculations -- it just passes data through.

---

## Section 6: Entities, DTOs, and JPA

### Exercise 27: Movie Entity {#exercise-27}

Create a Movie JPA entity with proper annotations.

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "movies")
public class Movie {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(nullable = false, length = 100)
    private String director;

    @Column(name = "release_date")
    private LocalDate releaseDate;

    @Column(length = 50)
    private String genre;

    @Column(precision = 2)
    private Double rating;

    @Column(name = "duration_minutes")
    private Integer durationMinutes;

    @Column(length = 1000)
    private String synopsis;

    public Movie() {}

    public Movie(String title, String director, LocalDate releaseDate, String genre,
                 Double rating, Integer durationMinutes, String synopsis) {
        this.title = title;
        this.director = director;
        this.releaseDate = releaseDate;
        this.genre = genre;
        this.rating = rating;
        this.durationMinutes = durationMinutes;
        this.synopsis = synopsis;
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDirector() { return director; }
    public void setDirector(String director) { this.director = director; }

    public LocalDate getReleaseDate() { return releaseDate; }
    public void setReleaseDate(LocalDate releaseDate) { this.releaseDate = releaseDate; }

    public String getGenre() { return genre; }
    public void setGenre(String genre) { this.genre = genre; }

    public Double getRating() { return rating; }
    public void setRating(Double rating) { this.rating = rating; }

    public Integer getDurationMinutes() { return durationMinutes; }
    public void setDurationMinutes(Integer durationMinutes) { this.durationMinutes = durationMinutes; }

    public String getSynopsis() { return synopsis; }
    public void setSynopsis(String synopsis) { this.synopsis = synopsis; }
}
```

**Why this works:**
- `@Entity` tells JPA to manage this class as a database table.
- `@Id` + `@GeneratedValue(strategy = GenerationType.IDENTITY)` creates an auto-incrementing primary key.
- `@Column` customizes column properties: `nullable`, `length`, `name` (for snake_case mapping).
- A no-arg constructor is required by JPA for entity instantiation via reflection.

---

### Exercise 28: Movie DTOs {#exercise-28}

Create a `MovieRequest` DTO (for incoming data, with validation) and a `MovieResponse` DTO (for outgoing data, without sensitive fields).

```java
package com.example.demo.dto;

import jakarta.validation.constraints.*;
import java.time.LocalDate;

public class MovieRequest {

    @NotBlank(message = "Title is required")
    @Size(max = 200, message = "Title must be at most 200 characters")
    private String title;

    @NotBlank(message = "Director is required")
    @Size(max = 100, message = "Director must be at most 100 characters")
    private String director;

    private LocalDate releaseDate;

    @Size(max = 50, message = "Genre must be at most 50 characters")
    private String genre;

    @DecimalMin(value = "0.0", message = "Rating must be at least 0")
    @DecimalMax(value = "10.0", message = "Rating must be at most 10")
    private Double rating;

    @Min(value = 1, message = "Duration must be at least 1 minute")
    @Max(value = 600, message = "Duration must be at most 600 minutes")
    private Integer durationMinutes;

    @Size(max = 1000, message = "Synopsis must be at most 1000 characters")
    private String synopsis;

    // Getters and setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDirector() { return director; }
    public void setDirector(String director) { this.director = director; }

    public LocalDate getReleaseDate() { return releaseDate; }
    public void setReleaseDate(LocalDate releaseDate) { this.releaseDate = releaseDate; }

    public String getGenre() { return genre; }
    public void setGenre(String genre) { this.genre = genre; }

    public Double getRating() { return rating; }
    public void setRating(Double rating) { this.rating = rating; }

    public Integer getDurationMinutes() { return durationMinutes; }
    public void setDurationMinutes(Integer durationMinutes) { this.durationMinutes = durationMinutes; }

    public String getSynopsis() { return synopsis; }
    public void setSynopsis(String synopsis) { this.synopsis = synopsis; }
}
```

```java
package com.example.demo.dto;

import java.time.LocalDate;

public class MovieResponse {

    private Long id;
    private String title;
    private String director;
    private LocalDate releaseDate;
    private String genre;
    private Double rating;
    private Integer durationMinutes;
    // Note: synopsis might be excluded from list responses for brevity

    public MovieResponse() {}

    public MovieResponse(Long id, String title, String director, LocalDate releaseDate,
                         String genre, Double rating, Integer durationMinutes) {
        this.id = id;
        this.title = title;
        this.director = director;
        this.releaseDate = releaseDate;
        this.genre = genre;
        this.rating = rating;
        this.durationMinutes = durationMinutes;
    }

    // Getters
    public Long getId() { return id; }
    public String getTitle() { return title; }
    public String getDirector() { return director; }
    public LocalDate getReleaseDate() { return releaseDate; }
    public String getGenre() { return genre; }
    public Double getRating() { return rating; }
    public Integer getDurationMinutes() { return durationMinutes; }

    // Setters
    public void setId(Long id) { this.id = id; }
    public void setTitle(String title) { this.title = title; }
    public void setDirector(String director) { this.director = director; }
    public void setReleaseDate(LocalDate releaseDate) { this.releaseDate = releaseDate; }
    public void setGenre(String genre) { this.genre = genre; }
    public void setRating(Double rating) { this.rating = rating; }
    public void setDurationMinutes(Integer durationMinutes) { this.durationMinutes = durationMinutes; }
}
```

**Conversion helper (in the service):**

```java
// Entity → Response
public MovieResponse toResponse(Movie movie) {
    return new MovieResponse(
            movie.getId(),
            movie.getTitle(),
            movie.getDirector(),
            movie.getReleaseDate(),
            movie.getGenre(),
            movie.getRating(),
            movie.getDurationMinutes()
    );
}

// Request → Entity
public Movie toEntity(MovieRequest request) {
    Movie movie = new Movie();
    movie.setTitle(request.getTitle());
    movie.setDirector(request.getDirector());
    movie.setReleaseDate(request.getReleaseDate());
    movie.setGenre(request.getGenre());
    movie.setRating(request.getRating());
    movie.setDurationMinutes(request.getDurationMinutes());
    movie.setSynopsis(request.getSynopsis());
    return movie;
}
```

**Why this works:**
- `MovieRequest` has no `id` field because the server generates IDs; it has validation annotations.
- `MovieResponse` has an `id` but excludes sensitive/internal fields (e.g., `synopsis` could be excluded from list responses).
- DTOs decouple the API contract from the database schema; you can change one without affecting the other.

---

### Exercise 29: MovieRepository {#exercise-29}

Create a Spring Data JPA repository for the Movie entity.

```java
package com.example.demo.repository;

import com.example.demo.model.Movie;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {

    // Spring Data JPA generates the query from the method name
    List<Movie> findByGenre(String genre);

    List<Movie> findByDirector(String director);

    List<Movie> findByRatingGreaterThanEqual(Double rating);

    List<Movie> findByReleaseDateAfter(LocalDate date);

    List<Movie> findByTitleContainingIgnoreCase(String keyword);

    List<Movie> findByGenreAndRatingGreaterThanEqual(String genre, Double rating);
}
```

**Inherited methods from JpaRepository (no code needed):**

| Method | Description |
|--------|------------|
| `save(Movie)` | Insert or update a movie |
| `findById(Long)` | Find by primary key (returns `Optional<Movie>`) |
| `findAll()` | Get all movies |
| `deleteById(Long)` | Delete by primary key |
| `count()` | Count total movies |
| `existsById(Long)` | Check if a movie exists |

**Why this works:**
- `JpaRepository<Movie, Long>` provides all CRUD methods automatically -- no implementation code needed.
- Spring Data JPA derives SQL queries from method names: `findByGenre(String genre)` becomes `SELECT * FROM movies WHERE genre = ?`.
- The `@Repository` annotation is optional here (Spring Data auto-detects interfaces extending `JpaRepository`), but it is good practice for clarity.

---

### Exercise 30: MovieService {#exercise-30}

Create a MovieService with complete CRUD operations using the repository and DTOs.

```java
package com.example.demo.service;

import com.example.demo.dto.MovieRequest;
import com.example.demo.dto.MovieResponse;
import com.example.demo.model.Movie;
import com.example.demo.repository.MovieRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class MovieService {

    private final MovieRepository movieRepository;

    public MovieService(MovieRepository movieRepository) {
        this.movieRepository = movieRepository;
    }

    public MovieResponse createMovie(MovieRequest request) {
        Movie movie = toEntity(request);
        Movie saved = movieRepository.save(movie);
        return toResponse(saved);
    }

    public MovieResponse getMovieById(Long id) {
        Movie movie = movieRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Movie not found with id: " + id));
        return toResponse(movie);
    }

    public List<MovieResponse> getAllMovies() {
        return movieRepository.findAll().stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public MovieResponse updateMovie(Long id, MovieRequest request) {
        Movie existing = movieRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Movie not found with id: " + id));

        existing.setTitle(request.getTitle());
        existing.setDirector(request.getDirector());
        existing.setReleaseDate(request.getReleaseDate());
        existing.setGenre(request.getGenre());
        existing.setRating(request.getRating());
        existing.setDurationMinutes(request.getDurationMinutes());
        existing.setSynopsis(request.getSynopsis());

        Movie updated = movieRepository.save(existing);
        return toResponse(updated);
    }

    public void deleteMovie(Long id) {
        if (!movieRepository.existsById(id)) {
            throw new RuntimeException("Movie not found with id: " + id);
        }
        movieRepository.deleteById(id);
    }

    // --- Conversion helpers ---

    private MovieResponse toResponse(Movie movie) {
        return new MovieResponse(
                movie.getId(),
                movie.getTitle(),
                movie.getDirector(),
                movie.getReleaseDate(),
                movie.getGenre(),
                movie.getRating(),
                movie.getDurationMinutes()
        );
    }

    private Movie toEntity(MovieRequest request) {
        Movie movie = new Movie();
        movie.setTitle(request.getTitle());
        movie.setDirector(request.getDirector());
        movie.setReleaseDate(request.getReleaseDate());
        movie.setGenre(request.getGenre());
        movie.setRating(request.getRating());
        movie.setDurationMinutes(request.getDurationMinutes());
        movie.setSynopsis(request.getSynopsis());
        return movie;
    }
}
```

**Why this works:**
- `createMovie` converts DTO to entity, saves, and converts back to response DTO.
- `updateMovie` fetches the existing entity, updates its fields, and saves (JPA detects the existing ID and performs an UPDATE).
- `deleteMovie` checks existence first to throw a meaningful error rather than silently doing nothing.
- Conversion methods keep the DTO-to-entity mapping centralized and consistent.

---

### Exercise 31: Custom Query Methods {#exercise-31}

Write derived query methods in the repository interface using Spring Data JPA naming conventions.

```java
package com.example.demo.repository;

import com.example.demo.model.Movie;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {

    // Find by exact match
    List<Movie> findByGenre(String genre);

    // Find by partial match (case-insensitive LIKE)
    List<Movie> findByTitleContainingIgnoreCase(String keyword);

    // Find by comparison
    List<Movie> findByRatingGreaterThanEqual(Double minRating);

    // Find by date range
    List<Movie> findByReleaseDateBetween(LocalDate start, LocalDate end);

    // Find with multiple conditions (AND)
    List<Movie> findByGenreAndRatingGreaterThanEqual(String genre, Double minRating);

    // Find with OR condition
    List<Movie> findByGenreOrDirector(String genre, String director);

    // Find with ordering
    List<Movie> findByGenreOrderByRatingDesc(String genre);

    // Find top N results
    List<Movie> findTop5ByOrderByRatingDesc();

    // Check existence
    boolean existsByTitleIgnoreCase(String title);

    // Count by field
    long countByGenre(String genre);

    // Delete by field
    void deleteByGenre(String genre);
}
```

**Method naming cheat sheet:**

| Keyword | Example | Generated SQL |
|---------|---------|--------------|
| `findBy` | `findByGenre(String)` | `WHERE genre = ?` |
| `Containing` | `findByTitleContaining(String)` | `WHERE title LIKE '%?%'` |
| `IgnoreCase` | `findByTitleIgnoreCase(String)` | `WHERE LOWER(title) = LOWER(?)` |
| `GreaterThanEqual` | `findByRatingGreaterThanEqual(Double)` | `WHERE rating >= ?` |
| `Between` | `findByDateBetween(Date, Date)` | `WHERE date BETWEEN ? AND ?` |
| `OrderBy...Desc` | `findByGenreOrderByRatingDesc(String)` | `ORDER BY rating DESC` |
| `Top5` | `findTop5ByOrderByRatingDesc()` | `LIMIT 5` |
| `And` / `Or` | `findByGenreAndRating(...)` | `WHERE genre = ? AND rating = ?` |

**Why this works:**
- Spring Data JPA parses method names and generates SQL at startup -- no implementation code needed.
- Keywords like `Containing`, `GreaterThanEqual`, `Between`, `OrderBy` map directly to SQL clauses.
- This eliminates boilerplate while still being type-safe and refactor-friendly.

---

### Exercise 32: @Query Annotation {#exercise-32}

Write custom JPQL queries using the `@Query` annotation for complex searches.

```java
package com.example.demo.repository;

import com.example.demo.model.Movie;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface MovieRepository extends JpaRepository<Movie, Long> {

    // Case-insensitive keyword search in title
    @Query("SELECT m FROM Movie m WHERE LOWER(m.title) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<Movie> searchByTitle(@Param("keyword") String keyword);

    // Search across multiple fields
    @Query("SELECT m FROM Movie m WHERE " +
           "LOWER(m.title) LIKE LOWER(CONCAT('%', :keyword, '%')) OR " +
           "LOWER(m.director) LIKE LOWER(CONCAT('%', :keyword, '%')) OR " +
           "LOWER(m.synopsis) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<Movie> searchByKeyword(@Param("keyword") String keyword);

    // Find top-rated movies by genre
    @Query("SELECT m FROM Movie m WHERE m.genre = :genre AND m.rating >= :minRating " +
           "ORDER BY m.rating DESC")
    List<Movie> findTopRatedByGenre(@Param("genre") String genre,
                                     @Param("minRating") Double minRating);

    // Average rating by genre
    @Query("SELECT m.genre, AVG(m.rating) FROM Movie m GROUP BY m.genre")
    List<Object[]> findAverageRatingByGenre();

    // Count movies per director
    @Query("SELECT m.director, COUNT(m) FROM Movie m GROUP BY m.director " +
           "HAVING COUNT(m) >= :minCount ORDER BY COUNT(m) DESC")
    List<Object[]> findDirectorsWithMinMovies(@Param("minCount") Long minCount);

    // Native SQL query (when JPQL is not enough)
    @Query(value = "SELECT * FROM movies WHERE YEAR(release_date) = :year",
           nativeQuery = true)
    List<Movie> findByReleaseYear(@Param("year") int year);
}
```

**Usage in the service:**

```java
@Service
public class MovieService {

    private final MovieRepository movieRepository;

    public MovieService(MovieRepository movieRepository) {
        this.movieRepository = movieRepository;
    }

    public List<MovieResponse> searchMovies(String keyword) {
        return movieRepository.searchByKeyword(keyword).stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }
}
```

**Why this works:**
- `@Query` provides full control when derived method names become unwieldy or impossible.
- JPQL operates on entities (`Movie m`), not tables -- it is database-agnostic.
- `@Param("keyword")` binds the method parameter to the `:keyword` placeholder in the query.
- `nativeQuery = true` allows raw SQL when you need database-specific features like `YEAR()`.

---

### Exercise 33: Full Task Manager Application {#exercise-33}

Build a complete Task Manager with all layers: Entity, DTOs, Repository, Service, Controller.

**Task Entity:**

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.time.LocalDate;
import java.time.LocalDateTime;

@Entity
@Table(name = "tasks")
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(length = 500)
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private TaskStatus status = TaskStatus.TODO;

    @Column(name = "due_date")
    private LocalDate dueDate;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    public Task() {}

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public TaskStatus getStatus() { return status; }
    public void setStatus(TaskStatus status) { this.status = status; }

    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }

    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

**TaskStatus Enum:**

```java
package com.example.demo.model;

public enum TaskStatus {
    TODO,
    IN_PROGRESS,
    DONE
}
```

**TaskRequest DTO:**

```java
package com.example.demo.dto;

import jakarta.validation.constraints.*;
import java.time.LocalDate;

public class TaskRequest {

    @NotBlank(message = "Title is required")
    @Size(max = 200, message = "Title must be at most 200 characters")
    private String title;

    @Size(max = 500, message = "Description must be at most 500 characters")
    private String description;

    private String status;  // optional — defaults to TODO

    @Future(message = "Due date must be in the future")
    private LocalDate dueDate;

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }

    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }
}
```

**TaskResponse DTO:**

```java
package com.example.demo.dto;

import java.time.LocalDate;
import java.time.LocalDateTime;

public class TaskResponse {

    private Long id;
    private String title;
    private String description;
    private String status;
    private LocalDate dueDate;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    public TaskResponse() {}

    public TaskResponse(Long id, String title, String description, String status,
                        LocalDate dueDate, LocalDateTime createdAt, LocalDateTime updatedAt) {
        this.id = id;
        this.title = title;
        this.description = description;
        this.status = status;
        this.dueDate = dueDate;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }

    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }

    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

**TaskRepository:**

```java
package com.example.demo.repository;

import com.example.demo.model.Task;
import com.example.demo.model.TaskStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.time.LocalDate;
import java.util.List;

@Repository
public interface TaskRepository extends JpaRepository<Task, Long> {

    List<Task> findByStatus(TaskStatus status);

    List<Task> findByDueDateBefore(LocalDate date);

    List<Task> findByTitleContainingIgnoreCase(String keyword);

    List<Task> findByStatusAndDueDateBefore(TaskStatus status, LocalDate date);
}
```

**TaskService:**

```java
package com.example.demo.service;

import com.example.demo.dto.TaskRequest;
import com.example.demo.dto.TaskResponse;
import com.example.demo.model.Task;
import com.example.demo.model.TaskStatus;
import com.example.demo.repository.TaskRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class TaskService {

    private final TaskRepository taskRepository;

    public TaskService(TaskRepository taskRepository) {
        this.taskRepository = taskRepository;
    }

    public TaskResponse createTask(TaskRequest request) {
        Task task = toEntity(request);
        Task saved = taskRepository.save(task);
        return toResponse(saved);
    }

    public TaskResponse getTaskById(Long id) {
        Task task = taskRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Task not found with id: " + id));
        return toResponse(task);
    }

    public List<TaskResponse> getAllTasks() {
        return taskRepository.findAll().stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public List<TaskResponse> getTasksByStatus(String status) {
        TaskStatus taskStatus = TaskStatus.valueOf(status.toUpperCase());
        return taskRepository.findByStatus(taskStatus).stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public TaskResponse updateTask(Long id, TaskRequest request) {
        Task existing = taskRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Task not found with id: " + id));

        existing.setTitle(request.getTitle());
        existing.setDescription(request.getDescription());
        existing.setDueDate(request.getDueDate());
        if (request.getStatus() != null) {
            existing.setStatus(TaskStatus.valueOf(request.getStatus().toUpperCase()));
        }

        Task updated = taskRepository.save(existing);
        return toResponse(updated);
    }

    public void deleteTask(Long id) {
        if (!taskRepository.existsById(id)) {
            throw new RuntimeException("Task not found with id: " + id);
        }
        taskRepository.deleteById(id);
    }

    // --- Conversion helpers ---

    private TaskResponse toResponse(Task task) {
        return new TaskResponse(
                task.getId(),
                task.getTitle(),
                task.getDescription(),
                task.getStatus().name(),
                task.getDueDate(),
                task.getCreatedAt(),
                task.getUpdatedAt()
        );
    }

    private Task toEntity(TaskRequest request) {
        Task task = new Task();
        task.setTitle(request.getTitle());
        task.setDescription(request.getDescription());
        task.setDueDate(request.getDueDate());
        if (request.getStatus() != null) {
            task.setStatus(TaskStatus.valueOf(request.getStatus().toUpperCase()));
        }
        return task;
    }
}
```

**TaskController:**

```java
package com.example.demo.controller;

import com.example.demo.dto.TaskRequest;
import com.example.demo.dto.TaskResponse;
import com.example.demo.service.TaskService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    private final TaskService taskService;

    public TaskController(TaskService taskService) {
        this.taskService = taskService;
    }

    @GetMapping
    public List<TaskResponse> getAllTasks(
            @RequestParam(required = false) String status) {
        if (status != null) {
            return taskService.getTasksByStatus(status);
        }
        return taskService.getAllTasks();
    }

    @GetMapping("/{id}")
    public ResponseEntity<TaskResponse> getTaskById(@PathVariable Long id) {
        TaskResponse task = taskService.getTaskById(id);
        return ResponseEntity.ok(task);
    }

    @PostMapping
    public ResponseEntity<TaskResponse> createTask(@Valid @RequestBody TaskRequest request) {
        TaskResponse created = taskService.createTask(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<TaskResponse> updateTask(@PathVariable Long id,
                                                    @Valid @RequestBody TaskRequest request) {
        TaskResponse updated = taskService.updateTask(id, request);
        return ResponseEntity.ok(updated);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(@PathVariable Long id) {
        taskService.deleteTask(id);
        return ResponseEntity.noContent().build();
    }
}
```

**Test with curl:**

```bash
# Create a task
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn Spring Boot","description":"Complete the 7-day guide","dueDate":"2026-07-15"}'
# 201 Created

# Get all tasks
curl http://localhost:8080/api/tasks
# 200 OK

# Filter by status
curl "http://localhost:8080/api/tasks?status=TODO"
# 200 OK — only TODO tasks

# Update a task
curl -X PUT http://localhost:8080/api/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn Spring Boot","description":"Complete the guide","status":"IN_PROGRESS","dueDate":"2026-07-15"}'
# 200 OK

# Delete a task
curl -X DELETE http://localhost:8080/api/tasks/1
# 204 No Content
```

**Why this works:**
- `@PrePersist` and `@PreUpdate` JPA callbacks automatically manage `createdAt` and `updatedAt` timestamps.
- `@Enumerated(EnumType.STRING)` stores the enum as a readable string ("TODO") in the database, not an integer.
- The full layered architecture means each file has a single responsibility and can be tested independently.

---

### Exercise 34: H2 Database Configuration {#exercise-34}

Configure H2 in-memory database for development in `application.properties`.

```properties
# ===== H2 Database Configuration =====

# Use H2 in-memory database (data is lost on restart)
spring.datasource.url=jdbc:h2:mem:taskdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA / Hibernate settings
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Enable H2 web console (accessible at http://localhost:8080/h2-console)
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# ===== Server Configuration =====
server.port=8080
```

**Maven dependency (pom.xml):**

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

**Key settings explained:**

| Property | Value | Purpose |
|----------|-------|---------|
| `spring.datasource.url` | `jdbc:h2:mem:taskdb` | In-memory database named "taskdb" |
| `spring.jpa.hibernate.ddl-auto` | `create-drop` | Create tables on startup, drop on shutdown |
| `spring.jpa.show-sql` | `true` | Print generated SQL to console (dev only) |
| `spring.h2.console.enabled` | `true` | Enable the browser-based SQL console |
| `spring.h2.console.path` | `/h2-console` | URL path for the console |

**Accessing the H2 Console:**

1. Start the application
2. Open `http://localhost:8080/h2-console` in a browser
3. Set JDBC URL to `jdbc:h2:mem:taskdb`
4. Username: `sa`, Password: (leave empty)
5. Click "Connect" to browse tables and run SQL

**Why this works:**
- H2 is an embedded Java database that requires zero setup -- perfect for development and learning.
- `create-drop` recreates the schema every time, so entity changes are reflected immediately.
- The H2 console lets you inspect data directly in the browser without any external database tools.

---

## Section 7: Validation & Error Handling

### Exercise 35: Validation Annotations {#exercise-35}

Add Bean Validation annotations to the TaskRequest DTO.

```java
package com.example.demo.dto;

import jakarta.validation.constraints.*;
import java.time.LocalDate;

public class TaskRequest {

    @NotBlank(message = "Title is required")
    @Size(min = 3, max = 200, message = "Title must be between 3 and 200 characters")
    private String title;

    @Size(max = 500, message = "Description must be at most 500 characters")
    private String description;

    @Future(message = "Due date must be in the future")
    private LocalDate dueDate;

    @Pattern(regexp = "TODO|IN_PROGRESS|DONE",
             message = "Status must be one of: TODO, IN_PROGRESS, DONE")
    private String status;

    @Min(value = 1, message = "Priority must be at least 1")
    @Max(value = 5, message = "Priority must be at most 5")
    private Integer priority;

    @Email(message = "Assignee email must be valid")
    private String assigneeEmail;

    // Getters and setters
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }

    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }

    public Integer getPriority() { return priority; }
    public void setPriority(Integer priority) { this.priority = priority; }

    public String getAssigneeEmail() { return assigneeEmail; }
    public void setAssigneeEmail(String assigneeEmail) { this.assigneeEmail = assigneeEmail; }
}
```

**Controller (must use @Valid):**

```java
@PostMapping
public ResponseEntity<TaskResponse> createTask(@Valid @RequestBody TaskRequest request) {
    TaskResponse created = taskService.createTask(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);
}
```

**Validation annotations reference:**

| Annotation | Applied To | Rule |
|-----------|-----------|------|
| `@NotBlank` | String | Not null, not empty, not just whitespace |
| `@Size(min, max)` | String, Collection | Length/size within range |
| `@Future` | Date/Time | Must be a future date |
| `@Pattern(regexp)` | String | Must match the regex |
| `@Min` / `@Max` | Number | Minimum/maximum value |
| `@Email` | String | Must be a valid email format |
| `@NotNull` | Any | Must not be null |

**Why this works:**
- `@Valid` on the controller parameter triggers validation before the method body executes.
- If validation fails, Spring throws `MethodArgumentNotValidException` with all field errors.
- The `message` attribute customizes the error message returned to the client.

---

### Exercise 36: GlobalExceptionHandler {#exercise-36}

Create a `@ControllerAdvice` class that handles common exceptions and returns proper error responses.

**Custom exception:**

```java
package com.example.demo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

**Error response class:**

```java
package com.example.demo.exception;

import java.time.LocalDateTime;

public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;

    public ErrorResponse(int status, String error, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    public LocalDateTime getTimestamp() { return timestamp; }
    public int getStatus() { return status; }
    public String getError() { return error; }
    public String getMessage() { return message; }
    public String getPath() { return path; }
}
```

**Global exception handler:**

```java
package com.example.demo.exception;

import jakarta.servlet.http.HttpServletRequest;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {

        ErrorResponse error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                "Not Found",
                ex.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex, HttpServletRequest request) {

        String message = ex.getBindingResult().getFieldErrors().stream()
                .map(fieldError -> fieldError.getField() + ": " + fieldError.getDefaultMessage())
                .reduce((a, b) -> a + "; " + b)
                .orElse("Validation failed");

        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                message,
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgument(
            IllegalArgumentException ex, HttpServletRequest request) {

        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                "Bad Request",
                ex.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(
            Exception ex, HttpServletRequest request) {

        ErrorResponse error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "Internal Server Error",
                "An unexpected error occurred",
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**Expected responses:**

```json
// GET /api/tasks/999 → 404
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 404,
  "error": "Not Found",
  "message": "Task not found with id: 999",
  "path": "/api/tasks/999"
}

// POST /api/tasks with invalid body → 400
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 400,
  "error": "Bad Request",
  "message": "title: Title is required; dueDate: Due date must be in the future",
  "path": "/api/tasks"
}
```

**Why this works:**
- `@ControllerAdvice` intercepts exceptions thrown from any controller in the application.
- Each `@ExceptionHandler` method handles a specific exception type and returns a structured JSON error.
- The generic `Exception` handler is a safety net that prevents stack traces from leaking to clients.

---

### Exercise 37: Structured Validation Error Response {#exercise-37}

Return field-level validation errors as a structured list, not a concatenated string.

**Validation error response class:**

```java
package com.example.demo.exception;

import java.time.LocalDateTime;
import java.util.List;

public class ValidationErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private List<FieldError> fieldErrors;

    public ValidationErrorResponse(int status, String error, String message,
                                    String path, List<FieldError> fieldErrors) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
        this.fieldErrors = fieldErrors;
    }

    public LocalDateTime getTimestamp() { return timestamp; }
    public int getStatus() { return status; }
    public String getError() { return error; }
    public String getMessage() { return message; }
    public String getPath() { return path; }
    public List<FieldError> getFieldErrors() { return fieldErrors; }

    public static class FieldError {
        private String field;
        private String message;
        private Object rejectedValue;

        public FieldError(String field, String message, Object rejectedValue) {
            this.field = field;
            this.message = message;
            this.rejectedValue = rejectedValue;
        }

        public String getField() { return field; }
        public String getMessage() { return message; }
        public Object getRejectedValue() { return rejectedValue; }
    }
}
```

**Updated exception handler method:**

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ValidationErrorResponse> handleValidationErrors(
        MethodArgumentNotValidException ex, HttpServletRequest request) {

    List<ValidationErrorResponse.FieldError> fieldErrors = ex.getBindingResult()
            .getFieldErrors().stream()
            .map(error -> new ValidationErrorResponse.FieldError(
                    error.getField(),
                    error.getDefaultMessage(),
                    error.getRejectedValue()
            ))
            .toList();

    ValidationErrorResponse response = new ValidationErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation Failed",
            "One or more fields have invalid values",
            request.getRequestURI(),
            fieldErrors
    );

    return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
}
```

**Expected response:**

```json
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 400,
  "error": "Validation Failed",
  "message": "One or more fields have invalid values",
  "path": "/api/tasks",
  "fieldErrors": [
    {
      "field": "title",
      "message": "Title is required",
      "rejectedValue": null
    },
    {
      "field": "dueDate",
      "message": "Due date must be in the future",
      "rejectedValue": "2020-01-01"
    }
  ]
}
```

**Why this works:**
- Each field error is its own object with `field`, `message`, and `rejectedValue`, making it easy for frontends to display inline errors next to form fields.
- The `rejectedValue` shows exactly what the client sent, helping with debugging.
- This is the industry-standard format used by most production APIs.

---

### Exercise 38: Custom ErrorResponse with Multiple Exception Types {#exercise-38}

Handle `ResourceNotFoundException` (404), `IllegalArgumentException` (400), and generic `Exception` (500) with a consistent error response format.

```java
package com.example.demo.exception;

import java.time.LocalDateTime;

public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String message;
    private String path;

    public ErrorResponse(int status, String message, String path) {
        this.timestamp = LocalDateTime.now();
        this.status = status;
        this.message = message;
        this.path = path;
    }

    public LocalDateTime getTimestamp() { return timestamp; }
    public int getStatus() { return status; }
    public String getMessage() { return message; }
    public String getPath() { return path; }
}
```

```java
package com.example.demo.exception;

import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    // 404 — Resource not found
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {

        log.warn("Resource not found: {}", ex.getMessage());

        ErrorResponse error = new ErrorResponse(
                HttpStatus.NOT_FOUND.value(),
                ex.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    // 400 — Bad request / invalid argument
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(
            IllegalArgumentException ex, HttpServletRequest request) {

        log.warn("Bad request: {}", ex.getMessage());

        ErrorResponse error = new ErrorResponse(
                HttpStatus.BAD_REQUEST.value(),
                ex.getMessage(),
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }

    // 500 — Unexpected server error (catch-all)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleServerError(
            Exception ex, HttpServletRequest request) {

        log.error("Unexpected error at {}: {}", request.getRequestURI(), ex.getMessage(), ex);

        ErrorResponse error = new ErrorResponse(
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                "An unexpected error occurred. Please try again later.",
                request.getRequestURI()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

**Expected responses:**

```json
// 404 — ResourceNotFoundException
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 404,
  "message": "Task not found with id: 999",
  "path": "/api/tasks/999"
}

// 400 — IllegalArgumentException
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 400,
  "message": "Price cannot be negative",
  "path": "/api/products"
}

// 500 — Generic Exception
{
  "timestamp": "2026-06-25T10:30:00",
  "status": 500,
  "message": "An unexpected error occurred. Please try again later.",
  "path": "/api/tasks"
}
```

**Why this works:**
- Specific exceptions (404, 400) return the actual error message because it is safe for clients to see.
- The generic 500 handler hides the real error message (which might contain sensitive details) and returns a safe generic message.
- The real error is logged server-side with `log.error()` including the full stack trace for debugging.

---

### Exercise 39: curl Commands Triggering Validation {#exercise-39}

Write curl commands that trigger each validation error.

```bash
# 1. Empty title — triggers @NotBlank
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "", "description": "Test task", "dueDate": "2026-12-01"}'

# Expected: 400 Bad Request
# {
#   "fieldErrors": [
#     {"field": "title", "message": "Title is required", "rejectedValue": ""}
#   ]
# }

# 2. Title too short — triggers @Size(min=3)
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Ab", "description": "Test task", "dueDate": "2026-12-01"}'

# Expected: 400 Bad Request
# {
#   "fieldErrors": [
#     {"field": "title", "message": "Title must be between 3 and 200 characters", "rejectedValue": "Ab"}
#   ]
# }

# 3. Description too long — triggers @Size(max=500)
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"Valid Title\", \"description\": \"$(python3 -c "print('x' * 501)")\", \"dueDate\": \"2026-12-01\"}"

# Expected: 400 Bad Request
# {
#   "fieldErrors": [
#     {"field": "description", "message": "Description must be at most 500 characters", ...}
#   ]
# }

# 4. Past due date — triggers @Future
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Valid Title", "description": "Test", "dueDate": "2020-01-01"}'

# Expected: 400 Bad Request
# {
#   "fieldErrors": [
#     {"field": "dueDate", "message": "Due date must be in the future", "rejectedValue": "2020-01-01"}
#   ]
# }

# 5. Multiple validation errors at once
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "", "dueDate": "2020-01-01"}'

# Expected: 400 Bad Request
# {
#   "fieldErrors": [
#     {"field": "title", "message": "Title is required", "rejectedValue": ""},
#     {"field": "dueDate", "message": "Due date must be in the future", "rejectedValue": "2020-01-01"}
#   ]
# }

# 6. Missing body entirely — triggers 400
curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json"

# Expected: 400 Bad Request (HttpMessageNotReadableException — "Required request body is missing")
```

**Why this works:**
- Each curl command targets a specific validation annotation, making it easy to verify that validation is working.
- Spring validates all fields at once and returns all errors together (it does not stop at the first error).
- The `-H "Content-Type: application/json"` header is required for `@RequestBody` deserialization.

---

## Section 8: Relationships & Queries

### Exercise 40: Review Entity with @ManyToOne {#exercise-40}

Create a Review entity that belongs to a Movie (many reviews can reference one movie).

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "reviews")
public class Review {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String reviewerName;

    @Column(nullable = false, length = 1000)
    private String content;

    @Column(nullable = false)
    private Integer rating; // 1-5 stars

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "movie_id", nullable = false)
    private Movie movie;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    public Review() {}

    public Review(String reviewerName, String content, Integer rating, Movie movie) {
        this.reviewerName = reviewerName;
        this.content = content;
        this.rating = rating;
        this.movie = movie;
    }

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getReviewerName() { return reviewerName; }
    public void setReviewerName(String reviewerName) { this.reviewerName = reviewerName; }

    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }

    public Integer getRating() { return rating; }
    public void setRating(Integer rating) { this.rating = rating; }

    public Movie getMovie() { return movie; }
    public void setMovie(Movie movie) { this.movie = movie; }

    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**Database result:**

```
reviews table:
+----+--------------+-----------------+--------+----------+---------------------+
| id | reviewer_name| content         | rating | movie_id | created_at          |
+----+--------------+-----------------+--------+----------+---------------------+
|  1 | Alice        | Great movie!    |      5 |        1 | 2026-06-25 10:00:00 |
|  2 | Bob          | Decent film     |      3 |        1 | 2026-06-25 11:00:00 |
|  3 | Carol        | Masterpiece     |      5 |        2 | 2026-06-25 12:00:00 |
+----+--------------+-----------------+--------+----------+---------------------+
```

**Why this works:**
- `@ManyToOne` creates a foreign key column (`movie_id`) in the `reviews` table pointing to the `movies` table.
- `@JoinColumn(name = "movie_id")` specifies the foreign key column name.
- `FetchType.LAZY` means the movie data is loaded only when `review.getMovie()` is called, improving performance.

---

### Exercise 41: Movie with @OneToMany {#exercise-41}

Add the inverse side of the relationship: a Movie has many Reviews.

```java
package com.example.demo.model;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "movies")
public class Movie {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(nullable = false, length = 100)
    private String director;

    @Column(name = "release_date")
    private LocalDate releaseDate;

    @Column(length = 50)
    private String genre;

    private Double rating;

    @Column(name = "duration_minutes")
    private Integer durationMinutes;

    @Column(length = 1000)
    private String synopsis;

    @OneToMany(mappedBy = "movie", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonIgnore  // Prevents infinite recursion in JSON serialization
    private List<Review> reviews = new ArrayList<>();

    public Movie() {}

    // --- Helper methods for managing the bidirectional relationship ---

    public void addReview(Review review) {
        reviews.add(review);
        review.setMovie(this);
    }

    public void removeReview(Review review) {
        reviews.remove(review);
        review.setMovie(null);
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDirector() { return director; }
    public void setDirector(String director) { this.director = director; }

    public LocalDate getReleaseDate() { return releaseDate; }
    public void setReleaseDate(LocalDate releaseDate) { this.releaseDate = releaseDate; }

    public String getGenre() { return genre; }
    public void setGenre(String genre) { this.genre = genre; }

    public Double getRating() { return rating; }
    public void setRating(Double rating) { this.rating = rating; }

    public Integer getDurationMinutes() { return durationMinutes; }
    public void setDurationMinutes(Integer durationMinutes) { this.durationMinutes = durationMinutes; }

    public String getSynopsis() { return synopsis; }
    public void setSynopsis(String synopsis) { this.synopsis = synopsis; }

    public List<Review> getReviews() { return reviews; }
    public void setReviews(List<Review> reviews) { this.reviews = reviews; }
}
```

**Key annotations explained:**

| Annotation | Purpose |
|-----------|---------|
| `@OneToMany(mappedBy = "movie")` | This is the inverse side; the `movie` field in `Review` owns the relationship |
| `cascade = CascadeType.ALL` | When a movie is saved/deleted, all its reviews are saved/deleted too |
| `orphanRemoval = true` | If a review is removed from the list, it is deleted from the database |
| `@JsonIgnore` | Prevents infinite JSON recursion (Movie -> Reviews -> Movie -> Reviews...) |

**Why this works:**
- `mappedBy = "movie"` tells JPA that the `Review.movie` field owns the foreign key -- this side is read-only.
- `CascadeType.ALL` propagates all operations (persist, merge, remove) from parent to children.
- Helper methods `addReview()` and `removeReview()` keep both sides of the relationship in sync.

---

### Exercise 42: Full Blog API {#exercise-42}

Build a complete Blog API with Post and Comment entities, full CRUD on both, and proper relationships.

**Post Entity:**

```java
package com.example.demo.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "posts")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(nullable = false, columnDefinition = "TEXT")
    private String body;

    @Column(nullable = false, length = 100)
    private String author;

    @OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    public Post() {}

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
        this.updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        this.updatedAt = LocalDateTime.now();
    }

    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }

    public void removeComment(Comment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public List<Comment> getComments() { return comments; }
    public void setComments(List<Comment> comments) { this.comments = comments; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

**Comment Entity:**

```java
package com.example.demo.model;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "comments")
public class Comment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 500)
    private String body;

    @Column(nullable = false, length = 100)
    private String author;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id", nullable = false)
    @JsonIgnore
    private Post post;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    public Comment() {}

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public Post getPost() { return post; }
    public void setPost(Post post) { this.post = post; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**Repositories:**

```java
package com.example.demo.repository;

import com.example.demo.model.Post;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

```java
package com.example.demo.repository;

import com.example.demo.model.Comment;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findByPostId(Long postId);
}
```

**DTOs:**

```java
package com.example.demo.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public class PostRequest {
    @NotBlank(message = "Title is required")
    @Size(max = 200)
    private String title;

    @NotBlank(message = "Body is required")
    private String body;

    @NotBlank(message = "Author is required")
    @Size(max = 100)
    private String author;

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
}
```

```java
package com.example.demo.dto;

import java.time.LocalDateTime;
import java.util.List;

public class PostResponse {
    private Long id;
    private String title;
    private String body;
    private String author;
    private int commentCount;
    private List<CommentResponse> comments;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    public PostResponse() {}

    public PostResponse(Long id, String title, String body, String author,
                        int commentCount, List<CommentResponse> comments,
                        LocalDateTime createdAt, LocalDateTime updatedAt) {
        this.id = id;
        this.title = title;
        this.body = body;
        this.author = author;
        this.commentCount = commentCount;
        this.comments = comments;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public int getCommentCount() { return commentCount; }
    public void setCommentCount(int commentCount) { this.commentCount = commentCount; }
    public List<CommentResponse> getComments() { return comments; }
    public void setComments(List<CommentResponse> comments) { this.comments = comments; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
    public LocalDateTime getUpdatedAt() { return updatedAt; }
    public void setUpdatedAt(LocalDateTime updatedAt) { this.updatedAt = updatedAt; }
}
```

```java
package com.example.demo.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public class CommentRequest {
    @NotBlank(message = "Comment body is required")
    @Size(max = 500)
    private String body;

    @NotBlank(message = "Author is required")
    @Size(max = 100)
    private String author;

    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
}
```

```java
package com.example.demo.dto;

import java.time.LocalDateTime;

public class CommentResponse {
    private Long id;
    private String body;
    private String author;
    private LocalDateTime createdAt;

    public CommentResponse() {}

    public CommentResponse(Long id, String body, String author, LocalDateTime createdAt) {
        this.id = id;
        this.body = body;
        this.author = author;
        this.createdAt = createdAt;
    }

    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getBody() { return body; }
    public void setBody(String body) { this.body = body; }
    public String getAuthor() { return author; }
    public void setAuthor(String author) { this.author = author; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

**PostService:**

```java
package com.example.demo.service;

import com.example.demo.dto.*;
import com.example.demo.exception.ResourceNotFoundException;
import com.example.demo.model.Post;
import com.example.demo.repository.PostRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class PostService {

    private final PostRepository postRepository;

    public PostService(PostRepository postRepository) {
        this.postRepository = postRepository;
    }

    public PostResponse createPost(PostRequest request) {
        Post post = new Post();
        post.setTitle(request.getTitle());
        post.setBody(request.getBody());
        post.setAuthor(request.getAuthor());
        Post saved = postRepository.save(post);
        return toResponse(saved);
    }

    public PostResponse getPostById(Long id) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Post not found with id: " + id));
        return toResponse(post);
    }

    public List<PostResponse> getAllPosts() {
        return postRepository.findAll().stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public PostResponse updatePost(Long id, PostRequest request) {
        Post post = postRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Post not found with id: " + id));
        post.setTitle(request.getTitle());
        post.setBody(request.getBody());
        post.setAuthor(request.getAuthor());
        Post updated = postRepository.save(post);
        return toResponse(updated);
    }

    public void deletePost(Long id) {
        if (!postRepository.existsById(id)) {
            throw new ResourceNotFoundException("Post not found with id: " + id);
        }
        postRepository.deleteById(id);
    }

    private PostResponse toResponse(Post post) {
        List<CommentResponse> commentResponses = post.getComments().stream()
                .map(c -> new CommentResponse(c.getId(), c.getBody(), c.getAuthor(), c.getCreatedAt()))
                .collect(Collectors.toList());

        return new PostResponse(
                post.getId(),
                post.getTitle(),
                post.getBody(),
                post.getAuthor(),
                post.getComments().size(),
                commentResponses,
                post.getCreatedAt(),
                post.getUpdatedAt()
        );
    }
}
```

**CommentService:**

```java
package com.example.demo.service;

import com.example.demo.dto.CommentRequest;
import com.example.demo.dto.CommentResponse;
import com.example.demo.exception.ResourceNotFoundException;
import com.example.demo.model.Comment;
import com.example.demo.model.Post;
import com.example.demo.repository.CommentRepository;
import com.example.demo.repository.PostRepository;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class CommentService {

    private final CommentRepository commentRepository;
    private final PostRepository postRepository;

    public CommentService(CommentRepository commentRepository, PostRepository postRepository) {
        this.commentRepository = commentRepository;
        this.postRepository = postRepository;
    }

    public CommentResponse addComment(Long postId, CommentRequest request) {
        Post post = postRepository.findById(postId)
                .orElseThrow(() -> new ResourceNotFoundException("Post not found with id: " + postId));

        Comment comment = new Comment();
        comment.setBody(request.getBody());
        comment.setAuthor(request.getAuthor());
        post.addComment(comment);

        Comment saved = commentRepository.save(comment);
        return toResponse(saved);
    }

    public List<CommentResponse> getCommentsByPostId(Long postId) {
        if (!postRepository.existsById(postId)) {
            throw new ResourceNotFoundException("Post not found with id: " + postId);
        }
        return commentRepository.findByPostId(postId).stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }

    public void deleteComment(Long postId, Long commentId) {
        Comment comment = commentRepository.findById(commentId)
                .orElseThrow(() -> new ResourceNotFoundException(
                        "Comment not found with id: " + commentId));
        if (!comment.getPost().getId().equals(postId)) {
            throw new ResourceNotFoundException(
                    "Comment " + commentId + " does not belong to post " + postId);
        }
        commentRepository.deleteById(commentId);
    }

    private CommentResponse toResponse(Comment comment) {
        return new CommentResponse(
                comment.getId(),
                comment.getBody(),
                comment.getAuthor(),
                comment.getCreatedAt()
        );
    }
}
```

**PostController:**

```java
package com.example.demo.controller;

import com.example.demo.dto.PostRequest;
import com.example.demo.dto.PostResponse;
import com.example.demo.service.PostService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/posts")
public class PostController {

    private final PostService postService;

    public PostController(PostService postService) {
        this.postService = postService;
    }

    @GetMapping
    public List<PostResponse> getAllPosts() {
        return postService.getAllPosts();
    }

    @GetMapping("/{id}")
    public ResponseEntity<PostResponse> getPostById(@PathVariable Long id) {
        return ResponseEntity.ok(postService.getPostById(id));
    }

    @PostMapping
    public ResponseEntity<PostResponse> createPost(@Valid @RequestBody PostRequest request) {
        PostResponse created = postService.createPost(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public ResponseEntity<PostResponse> updatePost(@PathVariable Long id,
                                                    @Valid @RequestBody PostRequest request) {
        return ResponseEntity.ok(postService.updatePost(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.noContent().build();
    }
}
```

**CommentController:**

```java
package com.example.demo.controller;

import com.example.demo.dto.CommentRequest;
import com.example.demo.dto.CommentResponse;
import com.example.demo.service.CommentService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/posts/{postId}/comments")
public class CommentController {

    private final CommentService commentService;

    public CommentController(CommentService commentService) {
        this.commentService = commentService;
    }

    @GetMapping
    public List<CommentResponse> getComments(@PathVariable Long postId) {
        return commentService.getCommentsByPostId(postId);
    }

    @PostMapping
    public ResponseEntity<CommentResponse> addComment(
            @PathVariable Long postId,
            @Valid @RequestBody CommentRequest request) {
        CommentResponse created = commentService.addComment(postId, request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @DeleteMapping("/{commentId}")
    public ResponseEntity<Void> deleteComment(@PathVariable Long postId,
                                               @PathVariable Long commentId) {
        commentService.deleteComment(postId, commentId);
        return ResponseEntity.noContent().build();
    }
}
```

**Test with curl:**

```bash
# Create a post
curl -X POST http://localhost:8080/api/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"My First Post","body":"Hello from the blog!","author":"Alice"}'
# 201 Created

# Add a comment to post 1
curl -X POST http://localhost:8080/api/posts/1/comments \
  -H "Content-Type: application/json" \
  -d '{"body":"Great post!","author":"Bob"}'
# 201 Created

# Get post with comments
curl http://localhost:8080/api/posts/1
# 200 OK — includes comments array and commentCount

# Get all comments for post 1
curl http://localhost:8080/api/posts/1/comments
# 200 OK — array of comments

# Delete a comment
curl -X DELETE http://localhost:8080/api/posts/1/comments/1
# 204 No Content

# Delete the post (cascades to remaining comments)
curl -X DELETE http://localhost:8080/api/posts/1
# 204 No Content
```

**Why this works:**
- Comments are nested under posts in the URL (`/api/posts/{postId}/comments`), reflecting the ownership relationship.
- `CascadeType.ALL` ensures deleting a post automatically deletes all its comments.
- `@JsonIgnore` on `Comment.post` prevents infinite recursion when serializing to JSON.
- The `deleteComment` method verifies the comment actually belongs to the specified post.

---

### Exercise 43: @Query with JOIN FETCH {#exercise-43}

Write a JPQL query that eagerly fetches a post with all its comments in a single query.

```java
package com.example.demo.repository;

import com.example.demo.model.Post;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface PostRepository extends JpaRepository<Post, Long> {

    // Fetch a post with all comments in one query (avoids N+1 problem)
    @Query("SELECT p FROM Post p JOIN FETCH p.comments WHERE p.id = :id")
    Optional<Post> findByIdWithComments(@Param("id") Long id);

    // Fetch all posts with their comments (avoids N+1 for list pages)
    @Query("SELECT DISTINCT p FROM Post p LEFT JOIN FETCH p.comments")
    List<Post> findAllWithComments();

    // Fetch posts by author with comments
    @Query("SELECT DISTINCT p FROM Post p LEFT JOIN FETCH p.comments WHERE p.author = :author")
    List<Post> findByAuthorWithComments(@Param("author") String author);

    // Count comments per post
    @Query("SELECT p.title, SIZE(p.comments) FROM Post p ORDER BY SIZE(p.comments) DESC")
    List<Object[]> findPostsOrderedByCommentCount();
}
```

**The N+1 problem explained:**

```
// WITHOUT JOIN FETCH (N+1 queries):
SELECT * FROM posts;                          -- 1 query for posts
SELECT * FROM comments WHERE post_id = 1;     -- 1 query per post
SELECT * FROM comments WHERE post_id = 2;     -- 1 query per post
SELECT * FROM comments WHERE post_id = 3;     -- ... N more queries

// WITH JOIN FETCH (1 query):
SELECT p.*, c.* FROM posts p
JOIN comments c ON c.post_id = p.id
WHERE p.id = 1;                               -- Everything in 1 query
```

**Usage in service:**

```java
@Service
public class PostService {

    private final PostRepository postRepository;

    public PostService(PostRepository postRepository) {
        this.postRepository = postRepository;
    }

    public PostResponse getPostWithComments(Long id) {
        Post post = postRepository.findByIdWithComments(id)
                .orElseThrow(() -> new ResourceNotFoundException(
                        "Post not found with id: " + id));
        return toResponse(post); // comments are already loaded
    }

    public List<PostResponse> getAllPostsWithComments() {
        return postRepository.findAllWithComments().stream()
                .map(this::toResponse)
                .collect(Collectors.toList());
    }
}
```

**Why this works:**
- `JOIN FETCH` tells JPA to load the related entities in the same SQL query instead of issuing separate queries.
- `DISTINCT` prevents duplicate posts when a post has multiple comments (the JOIN creates one row per comment).
- `LEFT JOIN FETCH` includes posts that have zero comments (regular `JOIN FETCH` would exclude them).
- This solves the N+1 query problem, which is the most common performance issue in JPA applications.

---

## Section 9: Testing

### Exercise 44: MovieServiceTest {#exercise-44}

Write unit tests for MovieService by mocking the MovieRepository.

```java
package com.example.demo.service;

import com.example.demo.dto.MovieRequest;
import com.example.demo.dto.MovieResponse;
import com.example.demo.model.Movie;
import com.example.demo.repository.MovieRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class MovieServiceTest {

    @Mock
    private MovieRepository movieRepository;

    @InjectMocks
    private MovieService movieService;

    private Movie sampleMovie;
    private MovieRequest sampleRequest;

    @BeforeEach
    void setUp() {
        sampleMovie = new Movie();
        sampleMovie.setId(1L);
        sampleMovie.setTitle("Inception");
        sampleMovie.setDirector("Christopher Nolan");
        sampleMovie.setGenre("Sci-Fi");
        sampleMovie.setRating(8.8);
        sampleMovie.setReleaseDate(LocalDate.of(2010, 7, 16));
        sampleMovie.setDurationMinutes(148);

        sampleRequest = new MovieRequest();
        sampleRequest.setTitle("Inception");
        sampleRequest.setDirector("Christopher Nolan");
        sampleRequest.setGenre("Sci-Fi");
        sampleRequest.setRating(8.8);
        sampleRequest.setReleaseDate(LocalDate.of(2010, 7, 16));
        sampleRequest.setDurationMinutes(148);
    }

    @Test
    @DisplayName("getMovieById returns movie when found")
    void getMovieById_WhenMovieExists_ReturnsMovie() {
        // Arrange
        when(movieRepository.findById(1L)).thenReturn(Optional.of(sampleMovie));

        // Act
        MovieResponse response = movieService.getMovieById(1L);

        // Assert
        assertNotNull(response);
        assertEquals(1L, response.getId());
        assertEquals("Inception", response.getTitle());
        assertEquals("Christopher Nolan", response.getDirector());
        verify(movieRepository, times(1)).findById(1L);
    }

    @Test
    @DisplayName("getMovieById throws exception when not found")
    void getMovieById_WhenMovieDoesNotExist_ThrowsException() {
        // Arrange
        when(movieRepository.findById(999L)).thenReturn(Optional.empty());

        // Act & Assert
        RuntimeException exception = assertThrows(RuntimeException.class,
                () -> movieService.getMovieById(999L));
        assertEquals("Movie not found with id: 999", exception.getMessage());
        verify(movieRepository, times(1)).findById(999L);
    }

    @Test
    @DisplayName("createMovie saves and returns movie")
    void createMovie_WithValidRequest_SavesAndReturnsMovie() {
        // Arrange
        when(movieRepository.save(any(Movie.class))).thenReturn(sampleMovie);

        // Act
        MovieResponse response = movieService.createMovie(sampleRequest);

        // Assert
        assertNotNull(response);
        assertEquals("Inception", response.getTitle());
        assertEquals("Christopher Nolan", response.getDirector());
        verify(movieRepository, times(1)).save(any(Movie.class));
    }

    @Test
    @DisplayName("getAllMovies returns list of movies")
    void getAllMovies_ReturnsAllMovies() {
        // Arrange
        Movie movie2 = new Movie();
        movie2.setId(2L);
        movie2.setTitle("The Dark Knight");
        movie2.setDirector("Christopher Nolan");

        when(movieRepository.findAll()).thenReturn(Arrays.asList(sampleMovie, movie2));

        // Act
        List<MovieResponse> movies = movieService.getAllMovies();

        // Assert
        assertEquals(2, movies.size());
        assertEquals("Inception", movies.get(0).getTitle());
        assertEquals("The Dark Knight", movies.get(1).getTitle());
        verify(movieRepository, times(1)).findAll();
    }

    @Test
    @DisplayName("deleteMovie calls repository when movie exists")
    void deleteMovie_WhenMovieExists_DeletesSuccessfully() {
        // Arrange
        when(movieRepository.existsById(1L)).thenReturn(true);
        doNothing().when(movieRepository).deleteById(1L);

        // Act
        movieService.deleteMovie(1L);

        // Assert
        verify(movieRepository, times(1)).existsById(1L);
        verify(movieRepository, times(1)).deleteById(1L);
    }

    @Test
    @DisplayName("deleteMovie throws exception when movie not found")
    void deleteMovie_WhenMovieDoesNotExist_ThrowsException() {
        // Arrange
        when(movieRepository.existsById(999L)).thenReturn(false);

        // Act & Assert
        RuntimeException exception = assertThrows(RuntimeException.class,
                () -> movieService.deleteMovie(999L));
        assertEquals("Movie not found with id: 999", exception.getMessage());
        verify(movieRepository, never()).deleteById(any());
    }
}
```

**Run the tests:**

```bash
./mvnw test -Dtest=MovieServiceTest
```

**Expected output:**

```
[INFO] Tests run: 6, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Why this works:**
- `@Mock` creates a fake `MovieRepository` that we can program to return specific values.
- `@InjectMocks` creates a real `MovieService` and injects the mock repository.
- Each test follows Arrange-Act-Assert: set up mocks, call the method, verify the result.
- `verify()` ensures the service interacted with the repository exactly as expected.

---

### Exercise 45: MovieControllerTest with MockMvc {#exercise-45}

Test the MovieController's HTTP endpoints using MockMvc.

```java
package com.example.demo.controller;

import com.example.demo.dto.MovieRequest;
import com.example.demo.dto.MovieResponse;
import com.example.demo.exception.ResourceNotFoundException;
import com.example.demo.service.MovieService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.bean.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;

import static org.hamcrest.Matchers.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(MovieController.class)
class MovieControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MovieService movieService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/movies returns 200 with list of movies")
    void getAllMovies_Returns200WithList() throws Exception {
        // Arrange
        List<MovieResponse> movies = Arrays.asList(
                new MovieResponse(1L, "Inception", "Nolan",
                        LocalDate.of(2010, 7, 16), "Sci-Fi", 8.8, 148),
                new MovieResponse(2L, "The Matrix", "Wachowski",
                        LocalDate.of(1999, 3, 31), "Sci-Fi", 8.7, 136)
        );
        when(movieService.getAllMovies()).thenReturn(movies);

        // Act & Assert
        mockMvc.perform(get("/api/movies"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].title", is("Inception")))
                .andExpect(jsonPath("$[0].director", is("Nolan")))
                .andExpect(jsonPath("$[1].title", is("The Matrix")));
    }

    @Test
    @DisplayName("GET /api/movies/{id} returns 200 when movie exists")
    void getMovieById_WhenExists_Returns200() throws Exception {
        // Arrange
        MovieResponse movie = new MovieResponse(1L, "Inception", "Nolan",
                LocalDate.of(2010, 7, 16), "Sci-Fi", 8.8, 148);
        when(movieService.getMovieById(1L)).thenReturn(movie);

        // Act & Assert
        mockMvc.perform(get("/api/movies/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.title", is("Inception")))
                .andExpect(jsonPath("$.rating", is(8.8)));
    }

    @Test
    @DisplayName("GET /api/movies/999 returns 404 when movie not found")
    void getMovieById_WhenNotExists_Returns404() throws Exception {
        // Arrange
        when(movieService.getMovieById(999L))
                .thenThrow(new ResourceNotFoundException("Movie not found with id: 999"));

        // Act & Assert
        mockMvc.perform(get("/api/movies/999"))
                .andExpect(status().isNotFound());
    }

    @Test
    @DisplayName("POST /api/movies returns 201 with created movie")
    void createMovie_WithValidRequest_Returns201() throws Exception {
        // Arrange
        MovieRequest request = new MovieRequest();
        request.setTitle("Inception");
        request.setDirector("Christopher Nolan");
        request.setGenre("Sci-Fi");
        request.setRating(8.8);

        MovieResponse response = new MovieResponse(1L, "Inception", "Christopher Nolan",
                null, "Sci-Fi", 8.8, null);
        when(movieService.createMovie(any(MovieRequest.class))).thenReturn(response);

        // Act & Assert
        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.title", is("Inception")));
    }

    @Test
    @DisplayName("POST /api/movies with missing title returns 400")
    void createMovie_WithMissingTitle_Returns400() throws Exception {
        // Arrange
        MovieRequest request = new MovieRequest();
        request.setDirector("Nolan");
        // title is missing — @NotBlank will fail

        // Act & Assert
        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isBadRequest());
    }

    @Test
    @DisplayName("DELETE /api/movies/{id} returns 204")
    void deleteMovie_Returns204() throws Exception {
        // Act & Assert
        mockMvc.perform(delete("/api/movies/1"))
                .andExpect(status().isNoContent());
    }
}
```

**Run the tests:**

```bash
./mvnw test -Dtest=MovieControllerTest
```

**Expected output:**

```
[INFO] Tests run: 6, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Why this works:**
- `@WebMvcTest` loads only the web layer (controller + exception handlers), not the full application -- tests run fast.
- `@MockBean` replaces the real `MovieService` with a mock, so no database is needed.
- `MockMvc` simulates HTTP requests and lets us assert on status codes, headers, and JSON body content.
- `jsonPath("$.title")` uses JSONPath syntax to assert on specific fields in the response body.

---

### Exercise 46: Integration Test {#exercise-46}

Write an integration test that starts the full application, creates a movie via POST, then retrieves it via GET.

```java
package com.example.demo;

import com.example.demo.dto.MovieRequest;
import com.example.demo.dto.MovieResponse;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import java.time.LocalDate;

import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class MovieIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("Full flow: create a movie, then retrieve it")
    void createAndRetrieveMovie() throws Exception {
        // Step 1: Create a movie via POST
        MovieRequest request = new MovieRequest();
        request.setTitle("Interstellar");
        request.setDirector("Christopher Nolan");
        request.setGenre("Sci-Fi");
        request.setRating(8.6);
        request.setDurationMinutes(169);
        request.setReleaseDate(LocalDate.of(2014, 11, 7));

        MvcResult createResult = mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.title", is("Interstellar")))
                .andExpect(jsonPath("$.id").exists())
                .andReturn();

        // Extract the created movie's ID
        String responseBody = createResult.getResponse().getContentAsString();
        MovieResponse createdMovie = objectMapper.readValue(responseBody, MovieResponse.class);
        Long movieId = createdMovie.getId();

        // Step 2: Retrieve the movie via GET
        mockMvc.perform(get("/api/movies/" + movieId))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id", is(movieId.intValue())))
                .andExpect(jsonPath("$.title", is("Interstellar")))
                .andExpect(jsonPath("$.director", is("Christopher Nolan")))
                .andExpect(jsonPath("$.genre", is("Sci-Fi")))
                .andExpect(jsonPath("$.rating", is(8.6)));

        // Step 3: Verify it appears in the list
        mockMvc.perform(get("/api/movies"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$[?(@.title == 'Interstellar')]").exists());

        // Step 4: Delete the movie
        mockMvc.perform(delete("/api/movies/" + movieId))
                .andExpect(status().isNoContent());

        // Step 5: Verify it's gone
        mockMvc.perform(get("/api/movies/" + movieId))
                .andExpect(status().isNotFound());
    }

    @Test
    @DisplayName("POST then PUT updates the movie")
    void createAndUpdateMovie() throws Exception {
        // Create
        MovieRequest request = new MovieRequest();
        request.setTitle("Tenet");
        request.setDirector("Christopher Nolan");
        request.setGenre("Action");
        request.setRating(7.3);

        MvcResult result = mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andReturn();

        Long movieId = objectMapper.readValue(
                result.getResponse().getContentAsString(), MovieResponse.class).getId();

        // Update the rating
        request.setRating(7.8);

        mockMvc.perform(put("/api/movies/" + movieId)
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.rating", is(7.8)))
                .andExpect(jsonPath("$.title", is("Tenet")));
    }
}
```

**Run the tests:**

```bash
./mvnw test -Dtest=MovieIntegrationTest
```

**Expected output:**

```
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Why this works:**
- `@SpringBootTest` starts the full application context, including all layers (controller, service, repository, database).
- `@AutoConfigureMockMvc` provides `MockMvc` without starting a real HTTP server (faster than `TestRestTemplate`).
- The test exercises the entire stack: HTTP handling, validation, business logic, database persistence.
- Each step verifies the previous step's side effects, creating a realistic end-to-end scenario.

---

### Exercise 47: Validation Test {#exercise-47}

Test that sending invalid data returns 400 with specific validation error messages.

```java
package com.example.demo.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class ValidationIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("POST with empty title returns 400 with validation error")
    void createMovie_WithEmptyTitle_Returns400() throws Exception {
        String requestBody = """
                {
                    "title": "",
                    "director": "Christopher Nolan",
                    "genre": "Sci-Fi"
                }
                """;

        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(requestBody))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.fieldErrors").isArray())
                .andExpect(jsonPath("$.fieldErrors[?(@.field == 'title')].message",
                        hasItem("Title is required")));
    }

    @Test
    @DisplayName("POST with missing required fields returns 400")
    void createMovie_WithMissingFields_Returns400() throws Exception {
        String requestBody = """
                {
                    "genre": "Sci-Fi"
                }
                """;

        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(requestBody))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.fieldErrors", hasSize(greaterThanOrEqualTo(2))))
                .andExpect(jsonPath("$.fieldErrors[*].field",
                        hasItems("title", "director")));
    }

    @Test
    @DisplayName("POST with invalid rating returns 400")
    void createMovie_WithInvalidRating_Returns400() throws Exception {
        String requestBody = """
                {
                    "title": "Test Movie",
                    "director": "Test Director",
                    "rating": 15.0
                }
                """;

        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(requestBody))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.fieldErrors[?(@.field == 'rating')].message",
                        hasItem("Rating must be at most 10")));
    }

    @Test
    @DisplayName("POST with valid data returns 201")
    void createMovie_WithValidData_Returns201() throws Exception {
        String requestBody = """
                {
                    "title": "Valid Movie",
                    "director": "Valid Director",
                    "genre": "Drama",
                    "rating": 8.5,
                    "durationMinutes": 120
                }
                """;

        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(requestBody))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.title", is("Valid Movie")))
                .andExpect(jsonPath("$.id").exists());
    }

    @Test
    @DisplayName("POST with null body returns 400")
    void createMovie_WithNoBody_Returns400() throws Exception {
        mockMvc.perform(post("/api/movies")
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isBadRequest());
    }
}
```

**Run the tests:**

```bash
./mvnw test -Dtest=ValidationIntegrationTest
```

**Expected output:**

```
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS
```

**Why this works:**
- Text blocks (`"""..."""`) make JSON payloads readable directly in the test code.
- JSONPath expressions like `$[?(@.field == 'title')]` filter the field errors array to find specific fields.
- `hasItems("title", "director")` verifies that multiple expected errors are present regardless of order.
- Testing both invalid and valid inputs proves that validation only rejects bad data, not good data.

---

## Section 10: Security & Configuration

### Exercise 48: Application Profiles {#exercise-48}

Create two profiles -- dev (H2, debug logging) and prod (PostgreSQL, info logging).

**application.properties (shared defaults):**

```properties
# Shared configuration for all profiles
spring.application.name=bookshelf-api

# Default profile
spring.profiles.active=dev
```

**application-dev.properties:**

```properties
# ===== Development Profile =====

# H2 in-memory database
spring.datasource.url=jdbc:h2:mem:devdb
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA settings for development
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Enable H2 console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Server
server.port=8080

# Logging — verbose for debugging
logging.level.root=INFO
logging.level.com.example.demo=DEBUG
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

**application-prod.properties:**

```properties
# ===== Production Profile =====

# PostgreSQL database
spring.datasource.url=jdbc:postgresql://localhost:5432/bookshelf
spring.datasource.driver-class-name=org.postgresql.Driver
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}

# JPA settings for production
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

# H2 console disabled in production
spring.h2.console.enabled=false

# Server
server.port=8443

# Logging — minimal for performance
logging.level.root=WARN
logging.level.com.example.demo=INFO
logging.level.org.springframework.web=INFO

# Connection pooling
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=30000
```

**Switching profiles:**

```bash
# Run with dev profile (default)
./mvnw spring-boot:run

# Run with prod profile
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod

# Or via environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar

# Or via command line argument
java -jar app.jar --spring.profiles.active=prod
```

**Key differences:**

| Setting | Dev | Prod |
|---------|-----|------|
| Database | H2 (in-memory) | PostgreSQL |
| DDL auto | `create-drop` | `validate` |
| SQL logging | Enabled | Disabled |
| H2 console | Enabled | Disabled |
| Port | 8080 | 8443 |
| Log level | DEBUG | INFO/WARN |
| DB credentials | Hardcoded `sa` | Environment variables |

**Why this works:**
- Spring Boot automatically loads `application-{profile}.properties` on top of the base `application.properties`.
- Dev profile uses `create-drop` for rapid iteration; prod uses `validate` to prevent accidental schema changes.
- Prod credentials use `${DB_USERNAME}` environment variable placeholders instead of hardcoded values.
- Each profile optimizes for its purpose: dev for developer experience, prod for security and performance.

---

### Exercise 49: SecurityFilterChain {#exercise-49}

Configure Spring Security so GET requests are public, but POST/PUT/DELETE require authentication.

```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF for REST APIs (stateless, no browser forms)
            .csrf(csrf -> csrf.disable())

            // Stateless session management (no server-side sessions)
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                // Public endpoints — no authentication required
                .requestMatchers(HttpMethod.GET, "/api/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/h2-console/**").permitAll()

                // Protected endpoints — authentication required
                .requestMatchers(HttpMethod.POST, "/api/**").authenticated()
                .requestMatchers(HttpMethod.PUT, "/api/**").authenticated()
                .requestMatchers(HttpMethod.DELETE, "/api/**").authenticated()
                .requestMatchers(HttpMethod.PATCH, "/api/**").authenticated()

                // Everything else requires authentication
                .anyRequest().authenticated()
            )

            // Use HTTP Basic authentication
            .httpBasic(Customizer.withDefaults())

            // Allow H2 console frames (dev only)
            .headers(headers -> headers.frameOptions(frame -> frame.sameOrigin()));

        return http.build();
    }
}
```

**Test with curl:**

```bash
# Public GET — works without credentials
curl http://localhost:8080/api/movies
# 200 OK — returns list of movies

# Protected POST — fails without credentials
curl -X POST http://localhost:8080/api/movies \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "director": "Test"}'
# 401 Unauthorized

# Protected POST — works with credentials
curl -X POST http://localhost:8080/api/movies \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "director": "Test"}'
# 201 Created

# Health check — always public
curl http://localhost:8080/actuator/health
# 200 OK — {"status": "UP"}
```

**Why this works:**
- `requestMatchers(HttpMethod.GET, "/api/**").permitAll()` allows all GET requests without authentication (read operations are public).
- `requestMatchers(HttpMethod.POST, "/api/**").authenticated()` requires credentials for write operations.
- `SessionCreationPolicy.STATELESS` ensures no server-side session is created -- every request must include credentials.
- CSRF is disabled because REST APIs are stateless and do not use browser cookies/forms.

---

### Exercise 50: Full SecurityConfig with Users {#exercise-50}

Complete Spring Security configuration with in-memory users, BCrypt passwords, and HTTP Basic authentication.

```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder passwordEncoder) {
        UserDetails user = User.builder()
                .username("user")
                .password(passwordEncoder.encode("user123"))
                .roles("USER")
                .build();

        UserDetails admin = User.builder()
                .username("admin")
                .password(passwordEncoder.encode("admin123"))
                .roles("USER", "ADMIN")
                .build();

        return new InMemoryUserDetailsManager(user, admin);
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF for stateless REST API
            .csrf(csrf -> csrf.disable())

            // Stateless session management
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))

            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers(HttpMethod.GET, "/api/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/h2-console/**").permitAll()

                // USER role can create and update
                .requestMatchers(HttpMethod.POST, "/api/**").hasRole("USER")
                .requestMatchers(HttpMethod.PUT, "/api/**").hasRole("USER")
                .requestMatchers(HttpMethod.PATCH, "/api/**").hasRole("USER")

                // Only ADMIN can delete
                .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")

                // Everything else requires authentication
                .anyRequest().authenticated()
            )

            // HTTP Basic authentication
            .httpBasic(Customizer.withDefaults())

            // Allow H2 console frames
            .headers(headers -> headers.frameOptions(frame -> frame.sameOrigin()));

        return http.build();
    }
}
```

**Maven dependency (pom.xml):**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Test with curl:**

```bash
# 1. Public GET — no credentials needed
curl http://localhost:8080/api/movies
# 200 OK

# 2. POST as regular user — works
curl -X POST http://localhost:8080/api/movies \
  -u user:user123 \
  -H "Content-Type: application/json" \
  -d '{"title": "Inception", "director": "Nolan"}'
# 201 Created

# 3. DELETE as regular user — forbidden
curl -o /dev/null -s -w "%{http_code}" \
  -X DELETE http://localhost:8080/api/movies/1 \
  -u user:user123
# 403 Forbidden (USER role cannot delete)

# 4. DELETE as admin — works
curl -o /dev/null -s -w "%{http_code}" \
  -X DELETE http://localhost:8080/api/movies/1 \
  -u admin:admin123
# 204 No Content

# 5. POST without credentials — unauthorized
curl -o /dev/null -s -w "%{http_code}" \
  -X POST http://localhost:8080/api/movies \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "director": "Test"}'
# 401 Unauthorized

# 6. POST with wrong password — unauthorized
curl -o /dev/null -s -w "%{http_code}" \
  -X POST http://localhost:8080/api/movies \
  -u user:wrongpassword \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "director": "Test"}'
# 401 Unauthorized
```

**Security summary:**

| Endpoint | No Auth | USER Role | ADMIN Role |
|----------|:-------:|:---------:|:----------:|
| GET /api/** | 200 | 200 | 200 |
| POST /api/** | 401 | 201 | 201 |
| PUT /api/** | 401 | 200 | 200 |
| DELETE /api/** | 401 | 403 | 204 |
| GET /actuator/health | 200 | 200 | 200 |

**Why this works:**
- `BCryptPasswordEncoder` hashes passwords with a random salt, making them secure even if the database is compromised.
- `InMemoryUserDetailsManager` stores users in memory -- suitable for learning and demos (use a database in production).
- `hasRole("ADMIN")` for DELETE means only admins can remove data, providing role-based access control.
- The `-u user:password` curl flag sends HTTP Basic authentication credentials (Base64-encoded `Authorization` header).

---

## Quick Reference: Test Commands

Run all exercises from a single terminal session:

```bash
# Run all tests
./mvnw test

# Run a specific test class
./mvnw test -Dtest=MovieServiceTest

# Run a specific test method
./mvnw test -Dtest=MovieServiceTest#getMovieById_WhenMovieExists_ReturnsMovie

# Run tests with verbose output
./mvnw test -Dtest=MovieControllerTest -Dsurefire.useFile=false

# Run integration tests only
./mvnw test -Dtest=*IntegrationTest

# Start the app for manual curl testing
./mvnw spring-boot:run
```

---

*End of solutions. If you completed these exercises on your own before checking, congratulations -- you have a solid foundation in Spring Boot!*
