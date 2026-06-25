# Appendix E: Coding Exercises

> 50 hands-on exercises organized by topic, from fundamentals to intermediate Spring Boot.
> Solutions are in [Appendix F](F-exercise-solutions.md).

---

## How to Use These Exercises

1. **Try before you peek.** Every exercise has hints, but resist looking at the solution until you've spent at least 15 minutes stuck. Struggling is where learning happens.
2. **Type the code yourself.** Do not copy-paste from the solutions. Typing forces your brain to process every character, every annotation, every semicolon.
3. **Run everything.** If an exercise asks you to write code, actually create a Spring Boot project (or reuse one), paste your code in, and run it. Verify with `curl` or Postman.
4. **Break things on purpose.** After your solution works, change something deliberately and observe the error. Understanding errors is half the skill.
5. **Exercises are independent.** You do not need the BookShelf project to complete any exercise. Many use a book/author domain, but each exercise is self-contained.

Difficulty guide:
- ⭐ **Easy** — Straightforward application of one concept. 10-15 minutes.
- ⭐⭐ **Medium** — Combines two or more concepts, or requires deeper thinking. 20-40 minutes.
- ⭐⭐⭐ **Hard** — Multi-file, multi-concept exercises that simulate real tasks. 45-90 minutes.

---

## Section 1: HTTP & REST Fundamentals (Exercises 1-6)

📖 **Learn the theory first:** [Chapter 2: Speaking HTTP](../day-1/02-speaking-http.md) | [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md)
🎨 **Prefer the fun version?** [Chapter 2 (Head First)](../head-first-style/day-1/02-speaking-http.md) | [Chapter 4 (Head First)](../head-first-style/day-2/04-json-and-apis.md)

### Exercise 1: Exploring HTTP with curl ⭐

**Problem:** Use `curl` to interact with the free public API at `https://jsonplaceholder.typicode.com` and observe HTTP behavior.

**Requirements:**
- Send a `GET` request to `/posts/1` and identify the status code and Content-Type header
- Send a `POST` request to `/posts` with a JSON body containing `title`, `body`, and `userId` fields
- Send a `PUT` request to `/posts/1` with an updated title
- Send a `DELETE` request to `/posts/1`
- For each request, record: the HTTP method used, the status code returned, and whether the response has a body

**Expected behavior:**
- GET returns status `200` with a JSON object containing the post
- POST returns status `201` with the created object (including a new `id`)
- PUT returns status `200` with the updated object
- DELETE returns status `200` with an empty JSON object `{}`

**Hints:**
> Use `curl -v` to see full request/response headers including the status code.
> For POST and PUT, use `-H "Content-Type: application/json"` and `-d '{"key":"value"}'`.
> Example: `curl -v -X POST -H "Content-Type: application/json" -d '{"title":"hello"}' https://jsonplaceholder.typicode.com/posts`

[Solution: Exercise 1](F-exercise-solutions.md#exercise-1)

---

### Exercise 2: Design REST Endpoints ⭐

**Problem:** You are building a Todo application. Design the complete REST API by filling in the table below.

**Requirements:**
- The resource is `todo` (plural: `todos`)
- Support all CRUD operations: create a todo, get all todos, get one todo by ID, update a todo, and delete a todo
- Add an endpoint to mark a todo as complete
- Follow REST conventions for paths and HTTP methods

**Fill in this table:**

| Operation | HTTP Method | Path | Request Body | Success Status Code |
|-----------|------------|------|-------------|-------------------|
| Get all todos | ? | ? | None | ? |
| Get one todo | ? | ? | None | ? |
| Create a todo | ? | ? | ? | ? |
| Update a todo | ? | ? | ? | ? |
| Delete a todo | ? | ? | None | ? |
| Mark todo complete | ? | ? | ? | ? |

**Expected behavior:**
- Your table should have 6 rows with appropriate HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Paths should follow the pattern `/api/todos` and `/api/todos/{id}`
- Status codes should follow convention: 200 for reads/updates, 201 for creation, 204 for deletion

**Hints:**
> "Mark todo complete" is a partial update — which HTTP method is used for partial updates?
> For deletion, `204 No Content` is conventional because there is nothing to return.
> The request body for "mark complete" could be as simple as `{"completed": true}`.

[Solution: Exercise 2](F-exercise-solutions.md#exercise-2)

---

### Exercise 3: Fix the Non-RESTful Endpoints ⭐⭐

**Problem:** The following endpoints violate REST conventions. Identify what is wrong with each one and rewrite them properly.

```
1. GET    /getBookById?id=5
2. POST   /createNewBook
3. GET    /api/books/delete/3
4. PUT    /api/updateBookTitle
5. POST   /api/books/42/getReviews
```

**Requirements:**
- For each endpoint, write one sentence explaining the violation
- Rewrite each endpoint following proper REST conventions
- Include the correct HTTP method for the rewritten version

**Expected behavior:**

Your corrected endpoints should:
- Use nouns instead of verbs in paths
- Use the HTTP method to convey the action
- Use path variables instead of query parameters for resource identification
- Follow the `/api/resource/{id}` pattern consistently

**Hints:**
> REST rule 1: The URL identifies the RESOURCE (noun), the HTTP method identifies the ACTION (verb).
> REST rule 2: Use path variables for resource IDs, not query parameters.
> For #3, think about which HTTP method means "remove."
> For #5, reviews of a book is a sub-resource — GET is the right method.

[Solution: Exercise 3](F-exercise-solutions.md#exercise-3)

---

### Exercise 4: HTTP Methods and CRUD ⭐

**Problem:** Match each HTTP method to the correct CRUD operation, database analogy, and typical use case.

**Requirements:**

Fill in the blanks:

| HTTP Method | CRUD Operation | SQL Equivalent | Example Use |
|-------------|---------------|---------------|-------------|
| GET | ? | ? | Retrieving a user profile |
| POST | ? | ? | ? |
| PUT | ? | ? | ? |
| PATCH | ? | ? | ? |
| DELETE | ? | ? | ? |

Also answer:
- Which methods are **idempotent** (calling them multiple times produces the same result)?
- Which methods are **safe** (they don't modify data)?
- Which method should return `201 Created` on success?

**Expected behavior:**
- All 5 rows filled correctly
- Correct identification of idempotent and safe methods
- POST identified as the method that returns 201

**Hints:**
> Idempotent means: DELETE /books/1 three times still results in book 1 being deleted. The state doesn't change after the first call.
> Safe means: the method only reads data, never modifies it.
> PUT replaces the entire resource. PATCH modifies only the specified fields.

[Solution: Exercise 4](F-exercise-solutions.md#exercise-4)

---

### Exercise 5: Design an E-Commerce Cart API ⭐⭐

**Problem:** You are building the API for an online store's shopping cart. Design the complete REST API.

**Scenario:**
- Users have shopping carts
- Each cart has items (a product reference + quantity)
- Users can add items, update item quantity, remove items, and clear the entire cart
- Users can view their cart with a total price
- Users can "checkout" the cart which creates an order

**Requirements:**
- Design all endpoints with: HTTP method, path, request body (if any), success status code, and response body description
- Handle edge cases: what happens if a user adds an item that is already in the cart?
- What status code should "checkout" return if the cart is empty?

**Expected behavior:**
- At least 7 endpoints covering: view cart, add item, update item quantity, remove item, clear cart, checkout
- Proper use of sub-resources: cart items are a sub-resource of the cart
- Paths like `/api/cart/items` and `/api/cart/items/{productId}`
- Checkout could be `POST /api/cart/checkout` or `POST /api/orders`

**Hints:**
> Cart items are a sub-resource of the cart. The path structure is: `/api/cart/items`.
> "Add an item already in the cart" — you could either return `409 Conflict` or silently increment the quantity (both are valid design choices).
> Checkout creating an order is a new resource creation, so it should return `201 Created`.
> An empty cart checkout could return `400 Bad Request` or `422 Unprocessable Entity`.

[Solution: Exercise 5](F-exercise-solutions.md#exercise-5)

---

### Exercise 6: Decode the curl Command ⭐

**Problem:** Read the following `curl` commands and, without running them, identify the HTTP method, path, headers, request body, and expected status code.

```bash
# Command A
curl -X PUT http://localhost:8080/api/books/7 \
  -H "Content-Type: application/json" \
  -H "Authorization: Basic dXNlcjpwYXNz" \
  -d '{"title": "Clean Code", "author": "Robert Martin", "pages": 464}'

# Command B
curl -s http://localhost:8080/api/books?genre=fiction&page=0&size=10

# Command C
curl -X DELETE http://localhost:8080/api/books/3 \
  -H "Authorization: Basic dXNlcjpwYXNz"

# Command D
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Spring in Action"}'
```

**Requirements:**

For each command, identify:
- HTTP method
- Full URL path (excluding query string)
- Query parameters (if any)
- Headers being sent
- Request body (if any)
- Most likely success status code

**Expected behavior:**
- Command A: PUT, /api/books/7, no query params, 2 headers, JSON body present, status 200
- Command B: GET (default when no -X), /api/books, 3 query params, no explicit headers, no body, status 200
- Command C: DELETE, /api/books/3, no query params, 1 header, no body, status 204
- Command D: POST, /api/books, no query params, 1 header, JSON body present, status 201

**Hints:**
> When `curl` has no `-X` flag, it defaults to GET.
> `-H` sets a header. `-d` sets the request body.
> `-s` is "silent mode" (no progress bar) — it doesn't change the HTTP method.
> `Basic dXNlcjpwYXNz` is Base64-encoded "user:pass" — it's a Basic Auth header.

[Solution: Exercise 6](F-exercise-solutions.md#exercise-6)

---

## Section 2: JSON (Exercises 7-10)

📖 **Learn the theory first:** [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md)
🎨 **Prefer the fun version?** [Chapter 4 (Head First)](../head-first-style/day-2/04-json-and-apis.md)

### Exercise 7: Java to JSON and Back ⭐

📖 *Need a refresher?* See [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md) · [Head First version](../head-first-style/day-2/04-json-and-apis.md) — the "Java and JSON" section.

**Problem:** Convert between Java classes and their JSON representations.

**Part A:** Given this Java class, write the JSON that Jackson would produce:

```java
public class Book {
    private Long id = 1L;
    private String title = "Dune";
    private String author = "Frank Herbert";
    private int pages = 412;
    private boolean available = true;
    private List<String> genres = List.of("Science Fiction", "Adventure");
    // getters and setters
}
```

**Part B:** Given this JSON, write the Java class that would map to it:

```json
{
  "studentId": 2045,
  "firstName": "Alice",
  "lastName": "Chen",
  "email": "alice.chen@university.edu",
  "gpa": 3.87,
  "enrolledCourses": ["CS101", "MATH201", "PHYS150"],
  "active": true
}
```

**Requirements:**
- Part A: Write valid JSON with correct types (strings, numbers, booleans, arrays)
- Part B: Write a complete Java class with fields, correct types, and getters/setters (or use a record)

**Expected behavior:**
- Part A: JSON keys match Java field names exactly. `id` is a number (not a string), `genres` is a JSON array.
- Part B: Java field names match JSON keys exactly. `gpa` is a `double`, `enrolledCourses` is a `List<String>`.

**Hints:**
> Jackson uses the getter names to determine JSON keys. A field `firstName` with getter `getFirstName()` produces `"firstName"` in JSON.
> Java `Long` and `int` both become JSON numbers (no quotes). Java `boolean` becomes JSON `true`/`false`.
> Java `List<String>` becomes a JSON array of strings: `["a", "b"]`.

[Solution: Exercise 7](F-exercise-solutions.md#exercise-7)

---

### Exercise 8: Fix the Broken JSON ⭐

📖 *Need a refresher?* See [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md) · [Head First version](../head-first-style/day-2/04-json-and-apis.md) — the "JSON Syntax Rules" section.

**Problem:** Each of the following JSON snippets contains one or more errors. Find and fix them.

```json
// Snippet 1
{
  title: "Spring Boot in Action",
  "author": "Craig Walls",
  "year": 2016
}

// Snippet 2
{
  "name": "Alice",
  "age": 30,
  "hobbies": ["reading", "coding", "hiking",]
}

// Snippet 3
{
  "product": "Laptop",
  "price": 999.99,
  "inStock": True,
  "specs": {
    "ram": "16GB",
    "storage": "512GB SSD",
  }
}

// Snippet 4
{
  'server': 'localhost',
  'port': 8080,
  'debug': false
}

// Snippet 5
{
  "users": [
    {"id": 1, "name": "Bob"}
    {"id": 2, "name": "Eve"}
  ]
}
```

**Requirements:**
- Identify every error in each snippet
- Write the corrected version
- Explain why each error breaks JSON parsing

**Expected behavior:**
- Snippet 1 has 1 error (unquoted key)
- Snippet 2 has 1 error (trailing comma)
- Snippet 3 has 2 errors (`True` should be `true`, trailing comma in nested object)
- Snippet 4 has 1 error (single quotes instead of double quotes)
- Snippet 5 has 1 error (missing comma between array elements)

**Hints:**
> JSON is stricter than JavaScript. All keys MUST be in double quotes.
> JSON does not allow trailing commas after the last element in arrays or objects.
> JSON booleans are lowercase: `true` and `false` (not `True` or `False` like Python).
> JSON only accepts double quotes `"`, never single quotes `'`.
> Also note: JSON does not support comments (`//`). The comments above each snippet are just labels for this exercise.

[Solution: Exercise 8](F-exercise-solutions.md#exercise-8)

---

### Exercise 9: Model a Complex Object as JSON ⭐⭐

📖 *Need a refresher?* See [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md) · [Head First version](../head-first-style/day-2/04-json-and-apis.md) — the "Nested Objects and Arrays" section.

**Problem:** Model the following real-world structure as a single JSON document.

**Scenario:** A university with the following data:
- University name: "State Technical University"
- Founded: 1965
- Location with city ("Springfield") and state ("IL")
- Two departments:
  - Department 1: "Computer Science", head is "Dr. Smith"
    - Courses: "CS101 - Intro to Programming" (3 credits), "CS201 - Data Structures" (4 credits)
    - Students: Alice (ID 1001, GPA 3.9), Bob (ID 1002, GPA 3.4)
  - Department 2: "Mathematics", head is "Dr. Patel"
    - Courses: "MATH101 - Calculus I" (4 credits), "MATH201 - Linear Algebra" (3 credits)
    - Students: Charlie (ID 1003, GPA 3.7)

**Requirements:**
- Produce a single valid JSON document representing this entire structure
- Use proper nesting: university contains departments, departments contain courses and students
- Use arrays for collections (departments, courses, students)
- Use appropriate data types (strings, numbers, arrays, nested objects)

**Expected behavior:**
- The JSON should be valid (you can verify at jsonlint.com)
- It should be approximately 40-50 lines when formatted
- All data from the scenario above should be present
- Structure: `{ university info, departments: [ { dept info, courses: [...], students: [...] }, ... ] }`

**Hints:**
> Start from the outermost object and work inward. The top level has `name`, `founded`, `location`, and `departments`.
> `location` is a nested object with `city` and `state`.
> `departments` is an array of objects. Each department has `name`, `head`, `courses`, and `students`.
> Each course is an object with `code`, `name`, and `credits`. Each student has `id`, `name`, and `gpa`.

[Solution: Exercise 9](F-exercise-solutions.md#exercise-9)

---

### Exercise 10: JSON Response to Java Classes ⭐⭐

📖 *Need a refresher?* See [Chapter 4: JSON and REST APIs](../day-2/04-json-and-apis.md) · [Head First version](../head-first-style/day-2/04-json-and-apis.md) — the "Deserialization" section.

**Problem:** Given the following JSON API response, write the Java classes needed to deserialize it.

```json
{
  "orderId": "ORD-2024-0042",
  "customer": {
    "id": 155,
    "name": "Jane Doe",
    "email": "jane@example.com"
  },
  "items": [
    {
      "productId": 301,
      "productName": "Wireless Mouse",
      "quantity": 2,
      "unitPrice": 29.99
    },
    {
      "productId": 478,
      "productName": "USB-C Hub",
      "quantity": 1,
      "unitPrice": 49.99
    }
  ],
  "totalAmount": 109.97,
  "status": "SHIPPED",
  "placedAt": "2024-03-15T10:30:00"
}
```

**Requirements:**
- Write all the Java classes needed (you'll need at least 3: an order class, a customer class, and an order item class)
- Use correct Java types: `String` for text, `int`/`Long` for whole numbers, `double`/`BigDecimal` for prices, `LocalDateTime` for timestamps
- Include all necessary fields with matching names
- Add getters, setters, and a no-arg constructor (or use Java records)

**Expected behavior:**
- Jackson should be able to deserialize the JSON into your top-level class without any extra configuration
- Field names must match JSON keys exactly (Java naming convention matches JSON convention when using camelCase)
- Nested `customer` object maps to a separate `Customer` class
- The `items` array maps to `List<OrderItem>`

**Hints:**
> Start from the inner objects and work outward. Write `Customer` first, then `OrderItem`, then `OrderResponse`.
> `"status": "SHIPPED"` could be a `String` or a Java `enum`. For simplicity, use `String`.
> `"placedAt"` can map to `LocalDateTime` — Jackson handles ISO 8601 date strings automatically with the `jackson-datatype-jsr310` module.
> You need a no-arg constructor for Jackson deserialization (records provide this automatically).

[Solution: Exercise 10](F-exercise-solutions.md#exercise-10)

---

## Section 3: Controllers & Request Mapping (Exercises 11-18)

📖 **Learn the theory first:** [Chapter 7: Controllers](../day-3/07-controllers.md) | [Chapter 9: Request-Response Lifecycle](../day-3/09-request-response-lifecycle.md)
🎨 **Prefer the fun version?** [Chapter 7 (Head First)](../head-first-style/day-3/07-controllers.md) | [Chapter 9 (Head First)](../head-first-style/day-3/09-request-response-lifecycle.md)

### Exercise 11: Your First GET Endpoint ⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "@RestController and @GetMapping" section.

**Problem:** Create a Spring Boot controller with a single GET endpoint that returns a greeting message.

**Requirements:**
- Create a class `GreetingController` annotated with `@RestController`
- Add a method `hello()` mapped to `GET /api/hello`
- The method returns the string `"Hello, Spring Boot!"`

**Expected behavior:**
```bash
$ curl http://localhost:8080/api/hello
Hello, Spring Boot!
```

**Hints:**
> You need two annotations: `@RestController` on the class and `@GetMapping("/api/hello")` on the method.
> The return type is `String`. Spring Boot sends it directly as the response body.
> If you want to test this, create a new Spring Boot project with `spring-boot-starter-web` and add this controller.

[Solution: Exercise 11](F-exercise-solutions.md#exercise-11)

---

### Exercise 12: Return an Object as JSON ⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "Returning Objects as JSON" section.

**Problem:** Create a controller that returns a Java object, which Spring Boot automatically converts to JSON.

**Requirements:**
- Create a `Product` class with fields: `id` (Long), `name` (String), `price` (double), `inStock` (boolean)
- Create a `ProductController` with a `GET /api/products/featured` endpoint
- The endpoint returns a single hardcoded `Product` with id=1, name="Mechanical Keyboard", price=79.99, inStock=true
- Include getters and setters in `Product` (Jackson needs them)

**Expected behavior:**
```bash
$ curl http://localhost:8080/api/products/featured
{"id":1,"name":"Mechanical Keyboard","price":79.99,"inStock":true}
```

**Hints:**
> Jackson (included in `spring-boot-starter-web`) automatically converts your Java object to JSON.
> Make sure your `Product` class has a no-arg constructor and getters — Jackson needs both to serialize.
> You don't need `@ResponseBody` because `@RestController` already includes it.

[Solution: Exercise 12](F-exercise-solutions.md#exercise-12)

---

### Exercise 13: Full CRUD Controller ⭐⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "CRUD Endpoints" section.

**Problem:** Create a controller with endpoints for all CRUD operations on a `Product` resource, using an in-memory `List` for storage.

**Requirements:**
- Create a `ProductController` with base path `/api/products`
- Implement these endpoints:
  - `GET /api/products` — return all products
  - `GET /api/products/{id}` — return one product by ID
  - `POST /api/products` — create a new product from request body
  - `PUT /api/products/{id}` — update an existing product
  - `DELETE /api/products/{id}` — delete a product
- Use a `private List<Product> products = new ArrayList<>()` as your data store
- Pre-populate the list with 2-3 products in the constructor

**Expected behavior:**
```bash
# Get all products
$ curl http://localhost:8080/api/products
[{"id":1,"name":"Mouse","price":25.0},{"id":2,"name":"Keyboard","price":75.0}]

# Get one product
$ curl http://localhost:8080/api/products/1
{"id":1,"name":"Mouse","price":25.0}

# Create a product
$ curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Monitor","price":299.99}'
{"id":3,"name":"Monitor","price":299.99}

# Delete a product
$ curl -X DELETE http://localhost:8080/api/products/1
```

**Hints:**
> For ID generation, keep a `private Long nextId` counter and increment it for each new product.
> For `GET /api/products/{id}`, use Java streams: `products.stream().filter(p -> p.getId().equals(id)).findFirst()`.
> For `DELETE`, use `products.removeIf(p -> p.getId().equals(id))`.
> Don't worry about error handling or status codes yet — we'll add those in Exercise 17.

[Solution: Exercise 13](F-exercise-solutions.md#exercise-13)

---

### Exercise 14: Path Variables ⭐⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "Path Variables" section.

**Problem:** Create a controller that uses `@PathVariable` in multiple ways.

**Requirements:**
- Create a `BookController` with base path `/api/books`
- Implement these endpoints:
  - `GET /api/books/{id}` — return a book by its numeric ID
  - `GET /api/books/isbn/{isbn}` — return a book by its ISBN string (e.g., "978-0134685991")
  - `GET /api/books/{id}/chapters/{chapterNum}` — return a specific chapter of a specific book (return a string like "Chapter 3 of Book 1")
- Use hardcoded data (no database needed)

**Expected behavior:**
```bash
$ curl http://localhost:8080/api/books/1
{"id":1,"title":"Effective Java","isbn":"978-0134685991"}

$ curl http://localhost:8080/api/books/isbn/978-0134685991
{"id":1,"title":"Effective Java","isbn":"978-0134685991"}

$ curl http://localhost:8080/api/books/1/chapters/3
"Chapter 3 of Book 1: Lambdas and Streams"
```

**Hints:**
> A method can have multiple `@PathVariable` parameters: `@GetMapping("/{id}/chapters/{chapterNum}")` with `@PathVariable Long id, @PathVariable int chapterNum`.
> The variable name in `{id}` must match the parameter name (or use `@PathVariable("id")`).
> ISBN contains hyphens, so it's a `String`, not a number.

[Solution: Exercise 14](F-exercise-solutions.md#exercise-14)

---

### Exercise 15: Query Parameters for Filtering ⭐⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "Query Parameters" section.

**Problem:** Create an endpoint that uses `@RequestParam` to filter and paginate results.

**Requirements:**
- Create a `ProductController` with a `GET /api/products` endpoint
- Support these optional query parameters:
  - `category` (String) — filter by category
  - `minPrice` (Double) — filter products with price >= minPrice
  - `maxPrice` (Double) — filter products with price <= maxPrice
  - `sortBy` (String, default "name") — sort results by this field
- Pre-populate with at least 6 products across different categories and price ranges
- All parameters are optional — if none are provided, return all products

**Expected behavior:**
```bash
# All products
$ curl http://localhost:8080/api/products
[...all 6+ products...]

# Filter by category
$ curl "http://localhost:8080/api/products?category=electronics"
[...only electronics products...]

# Filter by price range
$ curl "http://localhost:8080/api/products?minPrice=50&maxPrice=200"
[...products between $50 and $200...]

# Combine filters
$ curl "http://localhost:8080/api/products?category=electronics&minPrice=100"
[...electronics over $100...]
```

**Hints:**
> Make parameters optional with `required = false`: `@RequestParam(required = false) String category`.
> For optional numeric types, use `Double` (wrapper) not `double` (primitive), because primitives can't be null.
> Use `defaultValue` for the sort parameter: `@RequestParam(defaultValue = "name") String sortBy`.
> Filter with Java streams: start with `products.stream()`, chain `.filter()` calls for each non-null parameter.

[Solution: Exercise 15](F-exercise-solutions.md#exercise-15)

---

### Exercise 16: Accepting a Request Body ⭐⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First version](../head-first-style/day-3/07-controllers.md) — the "@RequestBody" section.

**Problem:** Create POST and PUT endpoints that read JSON from the request body using `@RequestBody`.

**Requirements:**
- Create a `ProductRequest` class with: `name` (String), `price` (double), `category` (String)
- Create a `Product` class with: `id` (Long), `name` (String), `price` (double), `category` (String)
- In `ProductController`:
  - `POST /api/products` — accepts a `ProductRequest`, creates a `Product` with a generated ID, returns the created `Product`
  - `PUT /api/products/{id}` — accepts a `ProductRequest` and a path variable `id`, updates the product, returns the updated `Product`

**Expected behavior:**
```bash
# Create
$ curl -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Headphones","price":149.99,"category":"electronics"}'
{"id":1,"name":"Headphones","price":149.99,"category":"electronics"}

# Update
$ curl -X PUT http://localhost:8080/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Headphones Pro","price":199.99,"category":"electronics"}'
{"id":1,"name":"Headphones Pro","price":199.99,"category":"electronics"}
```

**Hints:**
> `@RequestBody ProductRequest request` tells Spring to deserialize the JSON body into a `ProductRequest` object.
> Separating `ProductRequest` (what the client sends) from `Product` (what you store and return) is a best practice. The client should never set the `id`.
> Don't forget `Content-Type: application/json` in your curl commands — without it, Spring returns 415 Unsupported Media Type.

[Solution: Exercise 16](F-exercise-solutions.md#exercise-16)

---

### Exercise 17: ResponseEntity and Status Codes ⭐⭐

📖 *Need a refresher?* See [Chapter 9: Request-Response Lifecycle](../day-3/09-request-response-lifecycle.md) · [Head First version](../head-first-style/day-3/09-request-response-lifecycle.md) — the "ResponseEntity" section.

**Problem:** Refactor a controller to return proper HTTP status codes using `ResponseEntity`.

**Requirements:**
- Create a `ProductController` with an in-memory list and these endpoints:
  - `POST /api/products` — returns `201 Created` with the created product
  - `GET /api/products/{id}` — returns `200 OK` with the product, or `404 Not Found` if the ID doesn't exist
  - `PUT /api/products/{id}` — returns `200 OK` with the updated product, or `404 Not Found`
  - `DELETE /api/products/{id}` — returns `204 No Content` if deleted, or `404 Not Found`

**Expected behavior:**
```bash
# Create — 201
$ curl -v -X POST http://localhost:8080/api/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Mouse","price":25.00}'
< HTTP/1.1 201

# Get existing — 200
$ curl -v http://localhost:8080/api/products/1
< HTTP/1.1 200

# Get non-existing — 404
$ curl -v http://localhost:8080/api/products/999
< HTTP/1.1 404

# Delete — 204
$ curl -v -X DELETE http://localhost:8080/api/products/1
< HTTP/1.1 204
```

**Hints:**
> `ResponseEntity.status(HttpStatus.CREATED).body(product)` returns 201 with a body.
> `ResponseEntity.ok(product)` is shorthand for 200 with a body.
> `ResponseEntity.notFound().build()` returns 404 with no body.
> `ResponseEntity.noContent().build()` returns 204 with no body.
> Your return type changes from `Product` to `ResponseEntity<Product>` (or `ResponseEntity<?>` if different endpoints return different types).

[Solution: Exercise 17](F-exercise-solutions.md#exercise-17)

---

### Exercise 18: Complete Notes API ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 7: Controllers](../day-3/07-controllers.md) · [Head First](../head-first-style/day-3/07-controllers.md) | [Chapter 9: Request-Response Lifecycle](../day-3/09-request-response-lifecycle.md) · [Head First](../head-first-style/day-3/09-request-response-lifecycle.md).

**Problem:** Build a complete "Notes" REST API controller with full CRUD operations, proper status codes, and in-memory storage.

**Requirements:**
- Create a `Note` class with: `id` (Long), `title` (String), `content` (String), `createdAt` (LocalDateTime)
- Create a `NoteRequest` class with: `title` (String), `content` (String)
- Create a `NoteController` with base path `/api/notes` and these endpoints:

| Method | Path | Behavior | Status Code |
|--------|------|----------|-------------|
| GET | `/api/notes` | Return all notes | 200 |
| GET | `/api/notes/{id}` | Return one note | 200 or 404 |
| POST | `/api/notes` | Create a note | 201 |
| PUT | `/api/notes/{id}` | Update a note | 200 or 404 |
| DELETE | `/api/notes/{id}` | Delete a note | 204 or 404 |
| GET | `/api/notes/search?keyword=X` | Search notes by title containing keyword | 200 |

- Set `createdAt` automatically when creating a note (don't accept it from the client)
- The search endpoint should be case-insensitive

**Expected behavior:**
```bash
# Create a note
$ curl -X POST http://localhost:8080/api/notes \
  -H "Content-Type: application/json" \
  -d '{"title":"Shopping List","content":"Milk, Eggs, Bread"}'
{"id":1,"title":"Shopping List","content":"Milk, Eggs, Bread","createdAt":"2024-03-15T10:30:00"}

# Search
$ curl "http://localhost:8080/api/notes/search?keyword=shop"
[{"id":1,"title":"Shopping List","content":"Milk, Eggs, Bread","createdAt":"2024-03-15T10:30:00"}]

# Update
$ curl -X PUT http://localhost:8080/api/notes/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Updated Shopping List","content":"Milk, Eggs, Bread, Cheese"}'
{"id":1,"title":"Updated Shopping List","content":"Milk, Eggs, Bread, Cheese","createdAt":"2024-03-15T10:30:00"}

# Not found
$ curl -v http://localhost:8080/api/notes/999
< HTTP/1.1 404
```

**Hints:**
> Use `LocalDateTime.now()` to set `createdAt` in the POST method, not in the request.
> For case-insensitive search: `title.toLowerCase().contains(keyword.toLowerCase())`.
> Store notes in a `Map<Long, Note>` instead of a `List` — lookups by ID are easier.
> This exercise combines everything from Exercises 11-17. If you're stuck, review those solutions first.

[Solution: Exercise 18](F-exercise-solutions.md#exercise-18)

---

## Section 4: Dependency Injection (Exercises 19-22)

📖 **Learn the theory first:** [Chapter 8: Dependency Injection](../day-3/08-dependency-injection.md)
🎨 **Prefer the fun version?** [Chapter 8 (Head First)](../head-first-style/day-3/08-dependency-injection.md)

### Exercise 19: Identify and Fix Tight Coupling ⭐

📖 *Need a refresher?* See [Chapter 8: Dependency Injection](../day-3/08-dependency-injection.md) · [Head First version](../head-first-style/day-3/08-dependency-injection.md) — the "Tight Coupling vs. Loose Coupling" section.

**Problem:** The following code has tight coupling. Identify the problem and refactor it to use dependency injection.

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @GetMapping
    public List<Order> getOrders() {
        OrderService service = new OrderService();  // Problem here
        return service.getAllOrders();
    }

    @PostMapping
    public Order createOrder(@RequestBody OrderRequest request) {
        OrderService service = new OrderService();  // And here
        return service.createOrder(request);
    }
}
```

**Requirements:**
- Explain why creating `new OrderService()` inside the controller is a problem (give at least 3 reasons)
- Refactor to use constructor injection
- Make sure `OrderService` is annotated so Spring can manage it

**Expected behavior:**
- The controller should receive its `OrderService` through the constructor
- `OrderService` should be annotated with `@Service`
- Only one instance of `OrderService` exists (singleton)
- The controller does not use `new` to create any dependencies

**Hints:**
> Three problems with `new`: (1) A new instance is created on every request — wasteful. (2) You can't swap implementations (e.g., for testing). (3) If `OrderService` itself has dependencies, you'd have to construct those too.
> Constructor injection: add a `private final OrderService orderService` field and a constructor that accepts it.
> With only one constructor, Spring automatically injects the dependency — no `@Autowired` needed.

[Solution: Exercise 19](F-exercise-solutions.md#exercise-19)

---

### Exercise 20: Interface with Multiple Implementations ⭐⭐

📖 *Need a refresher?* See [Chapter 8: Dependency Injection](../day-3/08-dependency-injection.md) · [Head First version](../head-first-style/day-3/08-dependency-injection.md) — the "Interfaces and @Primary/@Qualifier" section.

**Problem:** Create a service interface with two implementations and configure Spring to inject the correct one.

**Requirements:**
- Create a `PriceCalculator` interface with one method: `double calculateTotal(double price, int quantity)`
- Create `StandardPriceCalculator` that returns `price * quantity`
- Create `DiscountPriceCalculator` that applies a 10% discount: `price * quantity * 0.9`
- Annotate both with `@Service`
- Create a `ShopController` that injects a `PriceCalculator` and uses it in a GET endpoint
- Use `@Primary` on the `StandardPriceCalculator` to make it the default

**Expected behavior:**
```bash
# Uses StandardPriceCalculator (the @Primary one)
$ curl "http://localhost:8080/api/shop/total?price=100&quantity=3"
{"total":300.0}
```

Also answer: How would you switch to `DiscountPriceCalculator` without changing the controller code?

**Hints:**
> When Spring finds two beans of the same type, it throws `NoUniqueBeanDefinitionException` — unless one is marked `@Primary`.
> Alternative to `@Primary`: use `@Qualifier("discountPriceCalculator")` on the constructor parameter.
> To switch implementations, you could: (1) move `@Primary` to the other class, or (2) use `@Qualifier`, or (3) use `@Profile` to activate different beans in different environments.

[Solution: Exercise 20](F-exercise-solutions.md#exercise-20)

---

### Exercise 21: Refactor Field Injection to Constructor Injection ⭐⭐

📖 *Need a refresher?* See [Chapter 8: Dependency Injection](../day-3/08-dependency-injection.md) · [Head First version](../head-first-style/day-3/08-dependency-injection.md) — the "Constructor Injection vs. Field Injection" section.

**Problem:** The following code uses field injection. Explain why it is discouraged and refactor it to use constructor injection.

```java
@RestController
@RequestMapping("/api/reports")
public class ReportController {

    @Autowired
    private ReportService reportService;

    @Autowired
    private EmailService emailService;

    @Autowired
    private AuditLogger auditLogger;

    @GetMapping("/{id}")
    public Report getReport(@PathVariable Long id) {
        auditLogger.log("Report accessed: " + id);
        return reportService.getReport(id);
    }

    @PostMapping("/{id}/send")
    public void sendReport(@PathVariable Long id) {
        Report report = reportService.getReport(id);
        emailService.send(report);
        auditLogger.log("Report sent: " + id);
    }
}
```

**Requirements:**
- List at least 3 reasons why `@Autowired` field injection is discouraged
- Refactor to constructor injection
- Make all dependency fields `private final`
- Verify that `@Autowired` is not needed on the constructor (explain why)

**Expected behavior:**
- The refactored class has no `@Autowired` annotations
- All dependencies are `private final` fields
- A single constructor accepts all three dependencies
- The class is easier to unit test (you can pass mock objects through the constructor)

**Hints:**
> Reasons against field injection: (1) Fields can't be `final`, so they're mutable. (2) You can't construct the object without Spring (can't pass mocks in a unit test). (3) It hides dependencies — you can't see them from the constructor signature. (4) It allows too many dependencies without it looking wrong (a constructor with 10 parameters screams "refactor me").
> With a single constructor, Spring auto-injects — `@Autowired` is optional since Spring 4.3.

[Solution: Exercise 21](F-exercise-solutions.md#exercise-21)

---

### Exercise 22: Build a Notification System ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 8: Dependency Injection](../day-3/08-dependency-injection.md) · [Head First version](../head-first-style/day-3/08-dependency-injection.md) — the "Injecting Multiple Implementations" section.

**Problem:** Build a notification system using interfaces, multiple implementations, and dependency injection.

**Requirements:**

Create the following components:

1. **`NotificationService` interface** with method: `void send(String recipient, String message)`
2. **`EmailNotificationService`** implements `NotificationService` — prints "EMAIL to {recipient}: {message}"
3. **`SmsNotificationService`** implements `NotificationService` — prints "SMS to {recipient}: {message}"
4. **`NotificationRequest`** class with fields: `recipient` (String), `message` (String), `channel` (String — "email" or "sms")
5. **`NotificationController`** with:
   - `POST /api/notifications` — accepts a `NotificationRequest` and routes to the correct service based on the `channel` field
   - Inject BOTH implementations using a `Map<String, NotificationService>` or a `List<NotificationService>`
6. **`NotificationServiceRouter`** — a `@Service` that receives both implementations and routes based on channel

**Expected behavior:**
```bash
# Send email
$ curl -X POST http://localhost:8080/api/notifications \
  -H "Content-Type: application/json" \
  -d '{"recipient":"user@example.com","message":"Hello!","channel":"email"}'
{"status":"sent","channel":"email","recipient":"user@example.com"}

# Send SMS
$ curl -X POST http://localhost:8080/api/notifications \
  -H "Content-Type: application/json" \
  -d '{"recipient":"+1234567890","message":"Hello!","channel":"sms"}'
{"status":"sent","channel":"sms","recipient":"+1234567890"}

# Invalid channel
$ curl -X POST http://localhost:8080/api/notifications \
  -H "Content-Type: application/json" \
  -d '{"recipient":"user@example.com","message":"Hello!","channel":"pigeon"}'
{"status":"error","message":"Unknown channel: pigeon"}
```

**Hints:**
> Spring can inject all implementations of an interface as a `List<NotificationService>`. You can then iterate to find the right one.
> Alternatively, name your beans `"email"` and `"sms"` using `@Service("email")`, then inject `Map<String, NotificationService>` — Spring populates the map with bean name as key.
> The controller should NOT contain routing logic — delegate that to the `NotificationServiceRouter` service.
> Handle the unknown channel case by returning `ResponseEntity.badRequest()`.

[Solution: Exercise 22](F-exercise-solutions.md#exercise-22)

---

## Section 5: Layered Architecture (Exercises 23-26)

📖 **Learn the theory first:** [Chapter 10: Thinking in Layers](../day-4/10-thinking-in-layers.md)
🎨 **Prefer the fun version?** [Chapter 10 (Head First)](../head-first-style/day-4/10-thinking-in-layers.md)

### Exercise 23: Refactor a Monolith Controller ⭐⭐

📖 *Need a refresher?* See [Chapter 10: Thinking in Layers](../day-4/10-thinking-in-layers.md) · [Head First](../head-first-style/day-4/10-thinking-in-layers.md) — the "Controller, Service, Repository" section.

**Problem:** The following controller has all logic in one class. Refactor it into proper Controller, Service, and Repository layers.

```java
@RestController
@RequestMapping("/api/tasks")
public class TaskController {

    private final List<Task> tasks = new ArrayList<>();
    private Long nextId = 1L;

    @GetMapping
    public List<Task> getAll() {
        return tasks;
    }

    @GetMapping("/{id}")
    public ResponseEntity<Task> getById(@PathVariable Long id) {
        for (Task task : tasks) {
            if (task.getId().equals(id)) {
                return ResponseEntity.ok(task);
            }
        }
        return ResponseEntity.notFound().build();
    }

    @PostMapping
    public ResponseEntity<Task> create(@RequestBody TaskRequest request) {
        // Validation logic
        if (request.getTitle() == null || request.getTitle().isBlank()) {
            return ResponseEntity.badRequest().build();
        }

        // Business logic
        Task task = new Task();
        task.setId(nextId++);
        task.setTitle(request.getTitle().trim());
        task.setDescription(request.getDescription());
        task.setCompleted(false);
        task.setCreatedAt(LocalDateTime.now());

        // Data storage
        tasks.add(task);
        return ResponseEntity.status(HttpStatus.CREATED).body(task);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        boolean removed = tasks.removeIf(t -> t.getId().equals(id));
        if (removed) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}
```

**Requirements:**
- Create a `TaskRepository` class (annotated with `@Repository`) that manages the in-memory list
  - Methods: `findAll()`, `findById(Long id)`, `save(Task task)`, `deleteById(Long id)`
- Create a `TaskService` class (annotated with `@Service`) that contains business logic
  - Methods: `getAllTasks()`, `getTaskById(Long id)`, `createTask(TaskRequest request)`, `deleteTask(Long id)`
- Refactor `TaskController` to only handle HTTP concerns — delegate everything to `TaskService`
- The external behavior (URLs, status codes, responses) must remain exactly the same

**Expected behavior:**
- All four `curl` commands work identically before and after refactoring
- The controller has no `List<Task>`, no `nextId`, no `new Task()`, no `tasks.add()` — just method calls to the service
- The service has no `ResponseEntity`, no `HttpStatus`, no `@GetMapping` — just business logic
- The repository has no business logic — just data storage and retrieval

**Hints:**
> Start with the repository. Move the `List<Task>` and `nextId` there. Create methods that the service will call.
> Then create the service. It receives the repository through constructor injection. Move all object creation and business logic here.
> Finally, slim down the controller. It receives the service through constructor injection. It only translates between HTTP and Java method calls.
> The repository's `findById` should return `Optional<Task>`.

[Solution: Exercise 23](F-exercise-solutions.md#exercise-23)

---

### Exercise 24: Identify Layer Violations ⭐⭐

📖 *Need a refresher?* See [Chapter 10: Thinking in Layers](../day-4/10-thinking-in-layers.md) · [Head First](../head-first-style/day-4/10-thinking-in-layers.md) — the "Layer Responsibilities" section.

**Problem:** The following code has layer violations — each class is doing something that belongs in a different layer. Identify every violation and explain which layer it belongs to.

**Class 1: Controller**
```java
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    private final EmployeeRepository repository; // Violation?

    @PostMapping
    public ResponseEntity<Employee> create(@RequestBody EmployeeRequest request) {
        // Is this in the right place?
        if (request.getSalary() < 30000) {
            throw new IllegalArgumentException("Salary below minimum wage");
        }

        Employee emp = new Employee();
        emp.setName(request.getName());
        emp.setSalary(request.getSalary());
        emp.setDepartment(request.getDepartment());
        emp.setHireDate(LocalDate.now());

        Employee saved = repository.save(emp);
        return ResponseEntity.status(HttpStatus.CREATED).body(saved);
    }
}
```

**Class 2: Service**
```java
@Service
public class EmployeeService {

    private final EmployeeRepository repository;

    public ResponseEntity<List<Employee>> getAllEmployees() {
        List<Employee> employees = repository.findAll();
        return ResponseEntity.ok(employees);  // Violation?
    }

    public ResponseEntity<Employee> getEmployee(Long id) {
        return repository.findById(id)
            .map(emp -> ResponseEntity.ok(emp))
            .orElse(ResponseEntity.notFound().build()); // Violation?
    }
}
```

**Class 3: Repository**
```java
@Repository
public class EmployeeRepository {

    private final List<Employee> employees = new ArrayList<>();

    public Employee save(Employee employee) {
        // Is this in the right place?
        employee.setName(employee.getName().trim().toUpperCase());

        employees.add(employee);
        return employee;
    }
}
```

**Requirements:**
- Identify every layer violation in all three classes (there are at least 5)
- For each violation, state:
  - What the violation is
  - Which layer it's currently in
  - Which layer it belongs to
- Rewrite the corrected version of each class (pseudocode is fine)

**Expected behavior:**
Your analysis should find at least:
- Controller directly uses repository (skipping service layer)
- Business logic (salary check) in the controller
- Object creation/mapping in the controller
- `ResponseEntity` in the service layer (HTTP concern)
- Data transformation (name formatting) in the repository

**Hints:**
> Rule of thumb: Controller knows about HTTP but not about business rules. Service knows about business rules but not about HTTP. Repository knows about data storage but not about business rules.
> `ResponseEntity` is an HTTP class — it should only appear in the controller layer.
> The controller should call the service, never the repository directly.
> Name formatting (`trim().toUpperCase()`) is a business rule, not a storage concern.

[Solution: Exercise 24](F-exercise-solutions.md#exercise-24)

---

### Exercise 25: Build a 3-Layer Employee API ⭐⭐

📖 *Need a refresher?* See [Chapter 10: Thinking in Layers](../day-4/10-thinking-in-layers.md) · [Head First](../head-first-style/day-4/10-thinking-in-layers.md).

**Problem:** Build a complete Employee management API with proper layered architecture from scratch.

**Requirements:**

Create these classes:

1. **`Employee`** (model) — id (Long), name (String), email (String), department (String), salary (double)
2. **`EmployeeRequest`** (DTO) — name, email, department, salary
3. **`EmployeeRepository`** — in-memory storage with: `findAll()`, `findById()`, `save()`, `deleteById()`, `existsById()`
4. **`EmployeeService`** — business logic with: `getAllEmployees()`, `getEmployeeById()`, `createEmployee()`, `updateEmployee()`, `deleteEmployee()`
5. **`EmployeeController`** — HTTP endpoints:

| Method | Path | Description | Status |
|--------|------|-------------|--------|
| GET | `/api/employees` | List all | 200 |
| GET | `/api/employees/{id}` | Get one | 200 / 404 |
| POST | `/api/employees` | Create | 201 |
| PUT | `/api/employees/{id}` | Update | 200 / 404 |
| DELETE | `/api/employees/{id}` | Delete | 204 / 404 |

**Layer rules to follow:**
- Controller: only HTTP concerns (status codes, request/response), delegates to service
- Service: business logic, calls repository, throws exceptions for "not found"
- Repository: data access only (no business logic)

**Expected behavior:**
```bash
$ curl -X POST http://localhost:8080/api/employees \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice","email":"alice@company.com","department":"Engineering","salary":85000}'
{"id":1,"name":"Alice","email":"alice@company.com","department":"Engineering","salary":85000.0}

$ curl http://localhost:8080/api/employees
[{"id":1,"name":"Alice","email":"alice@company.com","department":"Engineering","salary":85000.0}]

$ curl http://localhost:8080/api/employees/999
< HTTP/1.1 404
```

**Hints:**
> Create a custom `ResourceNotFoundException` (extends `RuntimeException`) that the service throws when a resource isn't found.
> In the controller, catch `ResourceNotFoundException` and return `ResponseEntity.notFound().build()` — or use `@ExceptionHandler` (preview of Section 7).
> The repository's `findById` should return `Optional<Employee>`.
> The service converts between `EmployeeRequest` (DTO) and `Employee` (model).

[Solution: Exercise 25](F-exercise-solutions.md#exercise-25)

---

### Exercise 26: Design the Layers ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 10: Thinking in Layers](../day-4/10-thinking-in-layers.md) · [Head First](../head-first-style/day-4/10-thinking-in-layers.md) — the "Layer Responsibilities" section.

**Problem:** You are given a feature description. Without writing the full code, design which code goes in which layer.

**Feature: "Employee Promotion"**
- An admin can promote an employee by ID
- Promotion increases salary by a percentage (configurable, default 10%)
- Promotion cannot happen if the employee has been with the company less than 1 year
- Promotion cannot happen if the employee was already promoted in the last 6 months
- If promoted, the employee's `lastPromotionDate` is updated to today
- The API returns the updated employee with the new salary
- If the employee doesn't exist, return 404
- If promotion is denied (not eligible), return 400 with a reason

**Requirements:**

For each of the following code snippets, identify which layer it belongs to (Controller, Service, or Repository):

1. `@PutMapping("/{id}/promote")`
2. `if (employee.getHireDate().isAfter(LocalDate.now().minusYears(1))) throw new NotEligibleException("Less than 1 year")`
3. `return ResponseEntity.badRequest().body(new ErrorResponse(ex.getMessage()))`
4. `findById(Long id)` returning `Optional<Employee>`
5. `employee.setSalary(employee.getSalary() * (1 + promotionPercentage / 100.0))`
6. `return ResponseEntity.ok(promotedEmployee)`
7. `employee.setLastPromotionDate(LocalDate.now())`
8. `save(Employee employee)`

Also write the method signatures (not full implementations) for:
- The controller method
- The service method
- Any new repository methods needed

**Expected behavior:**
- HTTP annotations and ResponseEntity code are in the controller
- Business rules (eligibility checks, salary calculation) are in the service
- Data operations (find, save) are in the repository
- The service method signature might look like: `Employee promoteEmployee(Long id, double percentage)`

**Hints:**
> The controller method accepts the path variable and maybe a request body (with the percentage), calls the service, and wraps the result in `ResponseEntity`.
> The service loads the employee, checks eligibility, calculates the new salary, updates the date, saves, and returns.
> The repository just needs `findById` and `save` — which you likely already have from CRUD operations.
> `NotEligibleException` is a custom exception. The controller (or a `@ControllerAdvice`) handles it and returns 400.

[Solution: Exercise 26](F-exercise-solutions.md#exercise-26)

---

## Section 6: Entities, DTOs & JPA (Exercises 27-34)

📖 **Learn the theory first:** [Chapter 11: Entities and DTOs](../day-4/11-entities-and-dtos.md) | [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md)
🎨 **Prefer the fun version?** [Chapter 11 (Head First)](../head-first-style/day-4/11-entities-and-dtos.md) | [Chapter 12 (Head First)](../head-first-style/day-4/12-databases-with-jpa.md)

### Exercise 27: Create a JPA Entity ⭐

📖 *Need a refresher?* See [Chapter 11: Entities and DTOs](../day-4/11-entities-and-dtos.md) · [Head First](../head-first-style/day-4/11-entities-and-dtos.md) — the "@Entity and @Id" section.

**Problem:** Create a JPA entity class for a Movie.

**Requirements:**
- Class name: `Movie`
- Fields:
  - `id` — Long, primary key, auto-generated
  - `title` — String, cannot be null, max 255 characters
  - `director` — String, cannot be null
  - `releaseYear` — int
  - `rating` — double (e.g., 8.5 out of 10)
- Add the appropriate JPA annotations: `@Entity`, `@Id`, `@GeneratedValue`, `@Column`
- Include a no-arg constructor, an all-args constructor, and getters/setters
- Map to a table called `movies`

**Expected behavior:**
When Spring Boot starts with this entity and `spring.jpa.hibernate.ddl-auto=create`, Hibernate should create a table:

```sql
CREATE TABLE movies (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    director VARCHAR(255) NOT NULL,
    release_year INT,
    rating DOUBLE
);
```

**Hints:**
> `@Entity` goes on the class. `@Table(name = "movies")` sets the table name.
> `@Id` marks the primary key. `@GeneratedValue(strategy = GenerationType.IDENTITY)` lets the database auto-increment it.
> `@Column(nullable = false, length = 255)` makes a column NOT NULL with a max length.
> By default, `camelCase` field names become `snake_case` column names: `releaseYear` becomes `release_year`.

[Solution: Exercise 27](F-exercise-solutions.md#exercise-27)

---

### Exercise 28: Create DTOs ⭐⭐

📖 *Need a refresher?* See [Chapter 11: Entities and DTOs](../day-4/11-entities-and-dtos.md) · [Head First](../head-first-style/day-4/11-entities-and-dtos.md) — the "DTOs and Why We Separate Them" section.

**Problem:** Create request and response DTOs for the Movie entity from Exercise 27.

**Requirements:**
- Create `MovieRequest` — used when creating or updating a movie. Fields: `title`, `director`, `releaseYear`, `rating`. No `id` field (the client doesn't set it).
- Create `MovieResponse` — used in API responses. Fields: `id`, `title`, `director`, `releaseYear`, `rating`.
- You can use Java records or regular classes
- Explain in a comment: why do we separate entities from DTOs?

**Expected behavior:**

```java
// Creating a movie: client sends MovieRequest
// { "title": "Inception", "director": "Christopher Nolan", "releaseYear": 2010, "rating": 8.8 }

// API responds with MovieResponse
// { "id": 1, "title": "Inception", "director": "Christopher Nolan", "releaseYear": 2010, "rating": 8.8 }
```

**Hints:**
> Java records are perfect for DTOs: `public record MovieRequest(String title, String director, int releaseYear, double rating) {}`
> Why separate entities from DTOs? (1) Entities have JPA annotations and database concerns. (2) You might want different fields in request vs. response. (3) Entities might have fields you don't want to expose (like `createdAt`, `version`). (4) Decoupling: changing the database schema shouldn't break your API contract.

[Solution: Exercise 28](F-exercise-solutions.md#exercise-28)

---

### Exercise 29: Spring Data Repository ⭐⭐

📖 *Need a refresher?* See [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md) — the "JpaRepository" section.

**Problem:** Create a Spring Data JPA repository for the Movie entity.

**Requirements:**
- Create a `MovieRepository` interface extending `JpaRepository<Movie, Long>`
- Do NOT write any method implementations — Spring Data generates them
- List all the methods you get for free from `JpaRepository` (at least 8)

**Expected behavior:**

After creating this one interface, you automatically have:
- `findAll()` — returns all movies
- `findById(Long id)` — returns `Optional<Movie>`
- `save(Movie movie)` — saves or updates a movie
- `deleteById(Long id)` — deletes by ID
- `count()` — returns the number of movies
- `existsById(Long id)` — returns true/false
- And many more

**Hints:**
> The entire interface is just: `public interface MovieRepository extends JpaRepository<Movie, Long> { }` — that's it.
> `JpaRepository` extends `ListCrudRepository` which extends `CrudRepository`. You get full CRUD plus paging and sorting.
> No `@Repository` annotation needed — Spring Data automatically detects interfaces extending `JpaRepository`.

[Solution: Exercise 29](F-exercise-solutions.md#exercise-29)

---

### Exercise 30: Service Layer with JPA Repository ⭐⭐

📖 *Need a refresher?* See [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md) | [Chapter 10](../day-4/10-thinking-in-layers.md) · [Head First](../head-first-style/day-4/10-thinking-in-layers.md).

**Problem:** Create a `MovieService` that uses `MovieRepository` for full CRUD operations.

**Requirements:**
- Create a `MovieService` class annotated with `@Service`
- Inject `MovieRepository` via constructor injection
- Implement these methods:
  - `List<MovieResponse> getAllMovies()` — returns all movies as DTOs
  - `MovieResponse getMovieById(Long id)` — returns one movie or throws `ResourceNotFoundException`
  - `MovieResponse createMovie(MovieRequest request)` — saves a new movie and returns the DTO
  - `MovieResponse updateMovie(Long id, MovieRequest request)` — updates or throws `ResourceNotFoundException`
  - `void deleteMovie(Long id)` — deletes or throws `ResourceNotFoundException`
- Convert between `Movie` entity and DTOs manually (no mapping library)

**Expected behavior:**
- `createMovie` saves the entity and returns a response DTO with the generated ID
- `getMovieById` throws an exception if the movie is not found (don't return null)
- `updateMovie` loads the existing entity, updates its fields, saves it, and returns the updated DTO
- Entity-to-DTO conversion is done in private helper methods

**Hints:**
> Create a `ResourceNotFoundException` that extends `RuntimeException`: `new ResourceNotFoundException("Movie not found with id: " + id)`.
> For entity-to-DTO conversion, create private methods: `private MovieResponse toResponse(Movie movie)` and use the request fields to update the entity.
> `repository.save()` works for both create and update. If the entity has an ID, it updates; if not, it creates.
> `repository.findById(id).orElseThrow(() -> new ResourceNotFoundException("..."))` is a clean pattern.

[Solution: Exercise 30](F-exercise-solutions.md#exercise-30)

---

### Exercise 31: Custom Query Methods ⭐⭐

📖 *Need a refresher?* See [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md) — the "Derived Query Methods" section.

**Problem:** Add custom finder methods to `MovieRepository` using Spring Data's query derivation.

**Requirements:**
- Add these methods to `MovieRepository` (just the method signatures — Spring Data implements them automatically):
  - Find all movies by a specific director
  - Find all movies released after a given year
  - Find all movies with a rating between two values
  - Find all movies whose title contains a keyword (case-insensitive)
  - Find the top 5 movies ordered by rating descending
  - Check if a movie exists with a given title

**Expected behavior:**
```java
// These method calls should work without writing any SQL:
movieRepository.findByDirector("Christopher Nolan");
movieRepository.findByReleaseYearGreaterThan(2000);
movieRepository.findByRatingBetween(7.0, 9.0);
movieRepository.findByTitleContainingIgnoreCase("dark");
movieRepository.findTop5ByOrderByRatingDesc();
movieRepository.existsByTitle("Inception");
```

**Hints:**
> Spring Data parses method names and generates queries: `findBy` + field name + condition keyword.
> Keywords: `GreaterThan`, `LessThan`, `Between`, `Containing`, `IgnoreCase`, `OrderBy...Desc`, `Top5`.
> Return types: `List<Movie>` for multiple results, `Optional<Movie>` for single results, `boolean` for exists.
> You don't write ANY implementation. Just the method signature in the interface.

[Solution: Exercise 31](F-exercise-solutions.md#exercise-31)

---

### Exercise 32: Custom JPQL Queries ⭐⭐

📖 *Need a refresher?* See [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md) — the "@Query and JPQL" section.

**Problem:** Write custom queries using `@Query` annotation with JPQL.

**Requirements:**
- Add these methods to `MovieRepository` using `@Query`:
  1. Search movies by partial title (LIKE query, case-insensitive)
  2. Find the average rating of all movies by a given director
  3. Find all movies released in a specific decade (e.g., 2010-2019)
  4. Find directors who have more than a given number of movies

**Expected behavior:**
```java
// Partial title search
@Query("SELECT m FROM Movie m WHERE LOWER(m.title) LIKE LOWER(CONCAT('%', :keyword, '%'))")
List<Movie> searchByTitle(@Param("keyword") String keyword);

movieRepository.searchByTitle("dark");
// Returns: [The Dark Knight, Dark Waters, ...]

// Average rating by director
movieRepository.getAverageRatingByDirector("Christopher Nolan");
// Returns: 8.4

// Movies in a decade
movieRepository.findByDecade(2010, 2019);
// Returns: [Inception, Interstellar, ...]

// Prolific directors
movieRepository.findProlificDirectors(3);
// Returns: ["Christopher Nolan", "Steven Spielberg"]
```

**Hints:**
> JPQL uses entity/field names, not table/column names. It's `Movie m`, not `movies m`. It's `m.title`, not `m.title`.
> Use `@Param("keyword")` to name parameters in the query: `WHERE m.title LIKE :keyword`.
> For the average, the return type can be `Double`: `@Query("SELECT AVG(m.rating) FROM Movie m WHERE m.director = :director")`.
> For the prolific directors query, use `GROUP BY` and `HAVING COUNT(m) > :minCount`, returning `List<String>`.

[Solution: Exercise 32](F-exercise-solutions.md#exercise-32)

---

### Exercise 33: Complete Task Manager API ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 11: Entities and DTOs](../day-4/11-entities-and-dtos.md) · [Head First](../head-first-style/day-4/11-entities-and-dtos.md) | [Chapter 12](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md).

**Problem:** Build a complete Task Manager REST API from scratch with a real database.

**Requirements:**

1. **Entity: `Task`**
   - `id` (Long, auto-generated)
   - `title` (String, not null, max 100 chars)
   - `description` (String, max 500 chars)
   - `priority` (String — "LOW", "MEDIUM", "HIGH")
   - `completed` (boolean, default false)
   - `createdAt` (LocalDateTime, set automatically)
   - `completedAt` (LocalDateTime, null until completed)

2. **DTOs:**
   - `TaskRequest` — title, description, priority
   - `TaskResponse` — all fields

3. **Repository: `TaskRepository`**
   - Extends `JpaRepository`
   - Custom: `findByPriority(String priority)`, `findByCompleted(boolean completed)`

4. **Service: `TaskService`**
   - Full CRUD
   - `completeTask(Long id)` — sets `completed=true` and `completedAt=now()`
   - `getTasksByPriority(String priority)`
   - `getPendingTasks()` — returns incomplete tasks

5. **Controller: `TaskController`**

   | Method | Path | Status |
   |--------|------|--------|
   | GET | `/api/tasks` | 200 |
   | GET | `/api/tasks/{id}` | 200 / 404 |
   | POST | `/api/tasks` | 201 |
   | PUT | `/api/tasks/{id}` | 200 / 404 |
   | DELETE | `/api/tasks/{id}` | 204 / 404 |
   | PATCH | `/api/tasks/{id}/complete` | 200 / 404 |
   | GET | `/api/tasks?priority=HIGH` | 200 |
   | GET | `/api/tasks/pending` | 200 |

6. **Use H2 in-memory database**

**Expected behavior:**
```bash
# Create a task
$ curl -X POST http://localhost:8080/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Write unit tests","description":"Cover the service layer","priority":"HIGH"}'
{"id":1,"title":"Write unit tests","description":"Cover the service layer","priority":"HIGH","completed":false,"createdAt":"2024-03-15T10:30:00","completedAt":null}

# Complete a task
$ curl -X PATCH http://localhost:8080/api/tasks/1/complete
{"id":1,"title":"Write unit tests",...,"completed":true,"completedAt":"2024-03-15T11:00:00"}

# Get pending tasks
$ curl http://localhost:8080/api/tasks/pending
[...only incomplete tasks...]

# Filter by priority
$ curl "http://localhost:8080/api/tasks?priority=HIGH"
[...only HIGH priority tasks...]
```

**Hints:**
> Start with the entity and DTOs, then repository, then service, then controller. Build layer by layer.
> Use `@PrePersist` to auto-set `createdAt`: `@PrePersist void onCreate() { this.createdAt = LocalDateTime.now(); }`
> For the `complete` endpoint, use `@PatchMapping` since you're partially updating the resource.
> In `application.properties`, set: `spring.datasource.url=jdbc:h2:mem:taskdb`, `spring.h2.console.enabled=true`, `spring.jpa.hibernate.ddl-auto=create`.
> This exercise combines everything from Exercises 27-32. Take it step by step.

[Solution: Exercise 33](F-exercise-solutions.md#exercise-33)

---

### Exercise 34: Configure H2 Database ⭐⭐

📖 *Need a refresher?* See [Chapter 12: Databases with JPA](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md) — the "H2 In-Memory Database" section.

**Problem:** Configure an H2 in-memory database for a Spring Boot application and verify it works.

**Requirements:**
- Add the H2 dependency to `pom.xml`
- Configure `application.properties` with:
  - H2 in-memory database URL
  - H2 console enabled and accessible at `/h2-console`
  - Show SQL in the console (for learning)
  - Auto-create tables from entities
- Create a simple entity (e.g., `Book`) and a `JpaRepository` for it
- Start the application and:
  - Visit `http://localhost:8080/h2-console` in a browser
  - Use the JDBC URL from your properties to connect
  - Verify your table was created

**Expected behavior:**
- Application starts without errors
- H2 console is accessible in the browser
- You can see the auto-created table in the H2 console
- SQL statements are printed in the application logs

**Your `application.properties` should include:**
```properties
# Database URL
spring.datasource.url=???
spring.datasource.driver-class-name=???

# H2 Console
spring.h2.console.enabled=???
spring.h2.console.path=???

# JPA/Hibernate
spring.jpa.hibernate.ddl-auto=???
spring.jpa.show-sql=???
```

**Hints:**
> H2 dependency: `<groupId>com.h2database</groupId>`, `<artifactId>h2</artifactId>`, `<scope>runtime</scope>`.
> JDBC URL for in-memory: `jdbc:h2:mem:testdb`. Driver: `org.h2.Driver`.
> `spring.jpa.hibernate.ddl-auto=create` drops and recreates tables on every startup (fine for development).
> `spring.jpa.show-sql=true` prints every SQL statement — great for learning, noisy in production.
> When you open H2 console in the browser, make sure the JDBC URL matches exactly what's in your properties.

[Solution: Exercise 34](F-exercise-solutions.md#exercise-34)

---

## Section 7: Validation & Error Handling (Exercises 35-39)

📖 **Learn the theory first:** [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md)
🎨 **Prefer the fun version?** [Chapter 13 (Head First)](../head-first-style/day-5/13-validation-and-error-handling.md)

### Exercise 35: Add Validation Annotations ⭐⭐

📖 *Need a refresher?* See [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md) · [Head First](../head-first-style/day-5/13-validation-and-error-handling.md) — the "Bean Validation Annotations" section.

**Problem:** Add Bean Validation annotations to a DTO so that invalid data is rejected automatically.

**Requirements:**
- Create a `RegistrationRequest` DTO with these fields and validation rules:

| Field | Type | Validation Rules |
|-------|------|-----------------|
| `username` | String | Required, 3-20 characters, no spaces |
| `email` | String | Required, must be a valid email |
| `password` | String | Required, at least 8 characters |
| `age` | int | Must be between 13 and 120 |
| `bio` | String | Optional, max 500 characters |

- Add the appropriate validation annotations from `jakarta.validation.constraints`
- Create a controller endpoint `POST /api/register` that accepts `@Valid @RequestBody RegistrationRequest`
- Include custom error messages for each annotation

**Expected behavior:**
```bash
# Valid request — 200
$ curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","email":"alice@example.com","password":"secret123","age":25}'
{"message":"Registration successful","username":"alice"}

# Missing username — 400
$ curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"secret123","age":25}'
400 Bad Request (with validation error details)

# Invalid email — 400
$ curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","email":"not-an-email","password":"secret123","age":25}'
400 Bad Request
```

**Hints:**
> `@NotBlank(message = "Username is required")` — not null, not empty, not whitespace.
> `@Size(min = 3, max = 20, message = "Username must be between 3 and 20 characters")`.
> `@Pattern(regexp = "^\\S+$", message = "Username must not contain spaces")` — `\\S+` means one or more non-whitespace characters.
> `@Email(message = "Must be a valid email address")`.
> `@Min(value = 13)` and `@Max(value = 120)` for age range.
> The `@Valid` annotation on the controller parameter triggers validation. Without it, annotations are ignored.

[Solution: Exercise 35](F-exercise-solutions.md#exercise-35)

---

### Exercise 36: Global Exception Handler ⭐⭐

📖 *Need a refresher?* See [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md) · [Head First](../head-first-style/day-5/13-validation-and-error-handling.md) — the "@ControllerAdvice and @ExceptionHandler" section.

**Problem:** Create a `@ControllerAdvice` class that handles exceptions globally across all controllers.

**Requirements:**
- Create a `ResourceNotFoundException` that extends `RuntimeException`
- Create a custom `ErrorResponse` class with fields: `status` (int), `message` (String), `timestamp` (LocalDateTime)
- Create a `GlobalExceptionHandler` class annotated with `@ControllerAdvice`
- Handle these exceptions:
  - `ResourceNotFoundException` returns `404` with a descriptive message
  - `IllegalArgumentException` returns `400` with the exception message
  - `Exception` (catch-all) returns `500` with a generic message

**Expected behavior:**
```bash
# ResourceNotFoundException
$ curl -v http://localhost:8080/api/movies/999
< HTTP/1.1 404
{"status":404,"message":"Movie not found with id: 999","timestamp":"2024-03-15T10:30:00"}

# IllegalArgumentException
$ curl -v -X POST http://localhost:8080/api/movies \
  -H "Content-Type: application/json" \
  -d '{"title":"","director":"Someone"}'
< HTTP/1.1 400
{"status":400,"message":"Title cannot be empty","timestamp":"2024-03-15T10:30:05"}

# Unexpected error
< HTTP/1.1 500
{"status":500,"message":"An unexpected error occurred","timestamp":"2024-03-15T10:31:00"}
```

**Hints:**
> ```java
> @ControllerAdvice
> public class GlobalExceptionHandler {
>     @ExceptionHandler(ResourceNotFoundException.class)
>     public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
>         ErrorResponse error = new ErrorResponse(404, ex.getMessage(), LocalDateTime.now());
>         return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
>     }
> }
> ```
> Each `@ExceptionHandler` method takes the exception as a parameter and returns a `ResponseEntity<ErrorResponse>`.
> The catch-all `@ExceptionHandler(Exception.class)` should log the real error (for debugging) but return a generic message to the client (for security).

[Solution: Exercise 36](F-exercise-solutions.md#exercise-36)

---

### Exercise 37: Handle Validation Errors Gracefully ⭐⭐

📖 *Need a refresher?* See [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md) · [Head First](../head-first-style/day-5/13-validation-and-error-handling.md) — the "Handling Validation Errors" section.

**Problem:** When `@Valid` fails, Spring throws `MethodArgumentNotValidException`. Handle it to return structured, user-friendly validation errors.

**Requirements:**
- Add a handler for `MethodArgumentNotValidException` in your `GlobalExceptionHandler`
- Create a `ValidationErrorResponse` class with:
  - `status` (int) — always 400
  - `message` (String) — "Validation failed"
  - `errors` (Map<String, String>) — field name to error message
  - `timestamp` (LocalDateTime)
- Extract each field error from the exception and put it in the map
- Test with the `RegistrationRequest` from Exercise 35

**Expected behavior:**
```bash
# Multiple validation errors
$ curl -X POST http://localhost:8080/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"","email":"bad-email","password":"short","age":5}'

{
  "status": 400,
  "message": "Validation failed",
  "errors": {
    "username": "Username is required",
    "email": "Must be a valid email address",
    "password": "Password must be at least 8 characters",
    "age": "Age must be at least 13"
  },
  "timestamp": "2024-03-15T10:30:00"
}
```

**Hints:**
> ```java
> @ExceptionHandler(MethodArgumentNotValidException.class)
> public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
>     Map<String, String> errors = new HashMap<>();
>     ex.getBindingResult().getFieldErrors().forEach(error ->
>         errors.put(error.getField(), error.getDefaultMessage())
>     );
>     // build and return response
> }
> ```
> `ex.getBindingResult().getFieldErrors()` gives you a list of `FieldError` objects.
> Each `FieldError` has `getField()` (the field name) and `getDefaultMessage()` (the message from your annotation).

[Solution: Exercise 37](F-exercise-solutions.md#exercise-37)

---

### Exercise 38: Multiple Exception Types ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md) · [Head First](../head-first-style/day-5/13-validation-and-error-handling.md) — the "Custom Exceptions" section.

**Problem:** Build a robust error handling system that handles different exception types with appropriate HTTP status codes.

**Requirements:**

1. Create these custom exceptions:
   - `ResourceNotFoundException` — when a resource doesn't exist
   - `DuplicateResourceException` — when trying to create a resource that already exists (e.g., duplicate email)
   - `BusinessRuleViolationException` — when a business rule is violated (e.g., can't delete an admin user)

2. Create an `ErrorResponse` class with: `status`, `error` (short code like "NOT_FOUND"), `message`, `path` (the request URI), `timestamp`

3. Create a `GlobalExceptionHandler` that handles all three:

   | Exception | Status Code | Error Code |
   |-----------|------------|------------|
   | `ResourceNotFoundException` | 404 | NOT_FOUND |
   | `DuplicateResourceException` | 409 | CONFLICT |
   | `BusinessRuleViolationException` | 422 | UNPROCESSABLE_ENTITY |
   | `MethodArgumentNotValidException` | 400 | VALIDATION_FAILED |

4. Include the request URI in the error response (use `WebRequest` or `HttpServletRequest`)

**Expected behavior:**
```bash
# 404
$ curl -v http://localhost:8080/api/users/999
{
  "status": 404,
  "error": "NOT_FOUND",
  "message": "User not found with id: 999",
  "path": "/api/users/999",
  "timestamp": "2024-03-15T10:30:00"
}

# 409
$ curl -v -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"email":"existing@example.com"}'
{
  "status": 409,
  "error": "CONFLICT",
  "message": "User with email existing@example.com already exists",
  "path": "/api/users",
  "timestamp": "2024-03-15T10:30:05"
}

# 422
$ curl -v -X DELETE http://localhost:8080/api/users/1
{
  "status": 422,
  "error": "UNPROCESSABLE_ENTITY",
  "message": "Cannot delete user with admin role",
  "path": "/api/users/1",
  "timestamp": "2024-03-15T10:30:10"
}
```

**Hints:**
> To get the request path, add `HttpServletRequest request` as a parameter to your `@ExceptionHandler` method, then call `request.getRequestURI()`.
> Each custom exception should extend `RuntimeException` and accept a message in the constructor.
> Consider creating a helper method that builds the `ErrorResponse` to avoid duplicating code in every handler.
> `HttpStatus.CONFLICT` is 409. `HttpStatus.UNPROCESSABLE_ENTITY` is 422.

[Solution: Exercise 38](F-exercise-solutions.md#exercise-38)

---

### Exercise 39: Test Your Validation with curl ⭐⭐

📖 *Need a refresher?* See [Chapter 13: Validation and Error Handling](../day-5/13-validation-and-error-handling.md) · [Head First](../head-first-style/day-5/13-validation-and-error-handling.md) | [Chapter 2](../day-1/02-speaking-http.md) · [Head First](../head-first-style/day-1/02-speaking-http.md).

**Problem:** Write `curl` commands that trigger every validation error in the `RegistrationRequest` from Exercise 35, and verify the responses.

**Requirements:**

Write curl commands for each scenario and state the expected response:

1. Missing `username` (field is null)
2. `username` too short (1 character)
3. `username` with spaces ("alice chen")
4. Invalid `email` format ("not-an-email")
5. `password` too short (3 characters)
6. `age` below minimum (age = 10)
7. `age` above maximum (age = 150)
8. `bio` exceeding 500 characters
9. Multiple validation errors at once (bad username, bad email, bad age)
10. A completely valid request (should succeed)

**Expected behavior:**

For each invalid request, you should get:
- Status code: `400 Bad Request`
- Body: a `ValidationErrorResponse` (from Exercise 37) with the specific field(s) and message(s)

For the valid request:
- Status code: `200 OK`
- Body: success response

**Hints:**
> For a completely empty body: `curl -X POST http://localhost:8080/api/register -H "Content-Type: application/json" -d '{}'`
> For a long bio, you can use `$(python3 -c "print('a' * 501)")` to generate a string.
> Send `Content-Type: application/json` with every POST request or Spring returns 415.
> The `-v` flag shows the status code in the response.

[Solution: Exercise 39](F-exercise-solutions.md#exercise-39)

---

## Section 8: Relationships & Queries (Exercises 40-43)

📖 **Learn the theory first:** [Chapter 14: Relationships and Queries](../day-5/14-relationships-and-queries.md)
🎨 **Prefer the fun version?** [Chapter 14 (Head First)](../head-first-style/day-5/14-relationships-and-queries.md)

### Exercise 40: @ManyToOne Relationship ⭐⭐

📖 *Need a refresher?* See [Chapter 14: Relationships and Queries](../day-5/14-relationships-and-queries.md) · [Head First](../head-first-style/day-5/14-relationships-and-queries.md) — the "@ManyToOne" section.

**Problem:** Create a `Review` entity that belongs to a `Movie` using `@ManyToOne`.

**Requirements:**
- Use the `Movie` entity from Exercise 27
- Create a `Review` entity with:
  - `id` (Long, auto-generated)
  - `content` (String, not null, max 1000 chars)
  - `rating` (int, 1-10)
  - `reviewerName` (String)
  - `movie` (Movie — the many-to-one relationship)
  - `createdAt` (LocalDateTime)
- Add the `@ManyToOne` annotation on the `movie` field
- Add `@JoinColumn(name = "movie_id")` to specify the foreign key column
- Create a `ReviewRepository` with a method to find all reviews for a movie: `findByMovieId(Long movieId)`

**Expected behavior:**

The resulting database tables should look like:
```
movies table:       id | title        | director         | release_year | rating
                     1 | Inception    | Christopher Nolan| 2010         | 8.8

reviews table:      id | content          | rating | reviewer_name | movie_id | created_at
                     1 | "Masterpiece!"   | 10     | Alice         | 1        | 2024-03-15
                     2 | "Mind-bending"   | 9      | Bob           | 1        | 2024-03-16
```

The `movie_id` column in `reviews` is a foreign key pointing to `movies.id`.

**Hints:**
> `@ManyToOne` means "many reviews can belong to one movie." Put it on the `Review.movie` field.
> `@JoinColumn(name = "movie_id")` explicitly names the foreign key column. Without it, Hibernate generates a default name.
> In `ReviewRepository`, `findByMovieId(Long movieId)` works because Spring Data traverses the `movie` field and looks at its `id`.
> `@ManyToOne` defaults to eager loading — the movie is loaded every time you load a review.

[Solution: Exercise 40](F-exercise-solutions.md#exercise-40)

---

### Exercise 41: @OneToMany Relationship ⭐⭐

📖 *Need a refresher?* See [Chapter 14: Relationships and Queries](../day-5/14-relationships-and-queries.md) · [Head First](../head-first-style/day-5/14-relationships-and-queries.md) — the "@OneToMany and Bidirectional Relationships" section.

**Problem:** Add the other side of the relationship: a `Movie` has many `Reviews`.

**Requirements:**
- Modify the `Movie` entity from Exercise 27 to include a list of reviews
- Add a `List<Review> reviews` field with `@OneToMany(mappedBy = "movie")`
- Prevent infinite recursion in JSON serialization when a Movie contains Reviews which contain a Movie which contains Reviews...
- Create a `MovieResponse` DTO that includes review data without the circular reference

**Expected behavior:**
```bash
# Get a movie — reviews are included
$ curl http://localhost:8080/api/movies/1
{
  "id": 1,
  "title": "Inception",
  "director": "Christopher Nolan",
  "releaseYear": 2010,
  "rating": 8.8,
  "reviews": [
    {"id": 1, "content": "Masterpiece!", "rating": 10, "reviewerName": "Alice"},
    {"id": 2, "content": "Mind-bending", "rating": 9, "reviewerName": "Bob"}
  ]
}
```

Note: the reviews in the response do NOT include the movie object (no circular reference).

**Hints:**
> `@OneToMany(mappedBy = "movie")` tells JPA that the `Review.movie` field owns the relationship. `mappedBy` MUST match the field name in the other entity.
> To prevent infinite recursion, use EITHER: (1) `@JsonIgnore` on the `movie` field in `Review`, or (2) DTOs that don't reference each other circularly (recommended), or (3) `@JsonManagedReference` / `@JsonBackReference`.
> Using DTOs is the cleanest solution: `MovieResponse` contains `List<ReviewSummary>`, and `ReviewSummary` does not contain a movie.
> `@OneToMany` defaults to lazy loading — reviews are NOT loaded until you access the list. This is usually what you want.

[Solution: Exercise 41](F-exercise-solutions.md#exercise-41)

---

### Exercise 42: Blog API with Relationships ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 14: Relationships and Queries](../day-5/14-relationships-and-queries.md) · [Head First](../head-first-style/day-5/14-relationships-and-queries.md) | [Chapter 12](../day-4/12-databases-with-jpa.md) · [Head First](../head-first-style/day-4/12-databases-with-jpa.md).

**Problem:** Build a "Blog" REST API where Posts have many Comments.

**Requirements:**

1. **Entities:**
   - `Post`: id, title (not null), content (not null), author (String), createdAt, comments (List)
   - `Comment`: id, text (not null), commenterName (String), post (ManyToOne), createdAt

2. **DTOs:**
   - `PostRequest`: title, content, author
   - `PostResponse`: id, title, content, author, createdAt, commentCount, comments (list of CommentResponse)
   - `CommentRequest`: text, commenterName
   - `CommentResponse`: id, text, commenterName, createdAt

3. **Repositories:** `PostRepository`, `CommentRepository`

4. **Services:** `PostService`, `CommentService`

5. **Controllers:**

   Post endpoints:

   | Method | Path | Status |
   |--------|------|--------|
   | GET | `/api/posts` | 200 |
   | GET | `/api/posts/{id}` | 200 / 404 |
   | POST | `/api/posts` | 201 |
   | PUT | `/api/posts/{id}` | 200 / 404 |
   | DELETE | `/api/posts/{id}` | 204 / 404 |

   Comment endpoints (sub-resource of a post):

   | Method | Path | Status |
   |--------|------|--------|
   | GET | `/api/posts/{postId}/comments` | 200 |
   | POST | `/api/posts/{postId}/comments` | 201 / 404 (if post not found) |
   | DELETE | `/api/posts/{postId}/comments/{commentId}` | 204 / 404 |

6. When a post is deleted, all its comments should also be deleted (cascade)

**Expected behavior:**
```bash
# Create a post
$ curl -X POST http://localhost:8080/api/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Spring Boot Guide","content":"A complete tutorial...","author":"Alice"}'
{"id":1,"title":"Spring Boot Guide","content":"A complete tutorial...","author":"Alice","createdAt":"...","commentCount":0,"comments":[]}

# Add a comment to the post
$ curl -X POST http://localhost:8080/api/posts/1/comments \
  -H "Content-Type: application/json" \
  -d '{"text":"Great post!","commenterName":"Bob"}'
{"id":1,"text":"Great post!","commenterName":"Bob","createdAt":"..."}

# Get post with comments
$ curl http://localhost:8080/api/posts/1
{"id":1,"title":"Spring Boot Guide",...,"commentCount":1,"comments":[{"id":1,"text":"Great post!","commenterName":"Bob","createdAt":"..."}]}

# Delete post (cascade deletes comments)
$ curl -X DELETE http://localhost:8080/api/posts/1
204 No Content
```

**Hints:**
> For cascade delete: `@OneToMany(mappedBy = "post", cascade = CascadeType.ALL, orphanRemoval = true)`.
> Comments are a sub-resource: `/api/posts/{postId}/comments`. The `CommentController` (or a `PostCommentController`) has `@RequestMapping("/api/posts/{postId}/comments")`.
> When creating a comment, first verify the post exists: `postRepository.findById(postId).orElseThrow(...)`.
> `commentCount` in the DTO can be computed as `post.getComments().size()`.
> Use `@PrePersist` for auto-setting `createdAt` on both entities.

[Solution: Exercise 42](F-exercise-solutions.md#exercise-42)

---

### Exercise 43: JOIN FETCH to Solve N+1 ⭐⭐

📖 *Need a refresher?* See [Chapter 14: Relationships and Queries](../day-5/14-relationships-and-queries.md) · [Head First](../head-first-style/day-5/14-relationships-and-queries.md) — the "N+1 Problem and JOIN FETCH" section.

**Problem:** Identify and fix an N+1 query problem using JOIN FETCH.

**Scenario:** You have `Movie` and `Review` entities (from Exercises 40-41). When you load all movies and access their reviews, Hibernate executes:
- 1 query to load all movies
- N queries to load reviews for each movie (one per movie)

If you have 100 movies, that's 101 queries!

**Requirements:**
- Write a method in `MovieRepository` that loads all movies WITH their reviews in a single query using `JOIN FETCH`
- Write a second query that loads a single movie by ID with its reviews using `JOIN FETCH`
- Explain why `@OneToMany(fetch = FetchType.EAGER)` is NOT a good solution

**Expected behavior:**

```java
// BAD: N+1 problem (1 + N queries)
List<Movie> movies = movieRepository.findAll();
for (Movie movie : movies) {
    System.out.println(movie.getReviews().size()); // Triggers a query for EACH movie
}

// GOOD: JOIN FETCH (1 query total)
List<Movie> movies = movieRepository.findAllWithReviews();
for (Movie movie : movies) {
    System.out.println(movie.getReviews().size()); // Already loaded, no extra query
}
```

**Hints:**
> The JOIN FETCH query:
> ```java
> @Query("SELECT m FROM Movie m LEFT JOIN FETCH m.reviews")
> List<Movie> findAllWithReviews();
> ```
> Use `LEFT JOIN FETCH` to include movies that have zero reviews. A regular `JOIN FETCH` would exclude them.
> Why not `FetchType.EAGER`? Because it loads reviews EVERY time you load a movie, even when you don't need them. `JOIN FETCH` gives you control — load reviews only when you explicitly ask for them.
> For the single movie query: `@Query("SELECT m FROM Movie m LEFT JOIN FETCH m.reviews WHERE m.id = :id")`.
> If you use `JOIN FETCH` with pagination (`Pageable`), Hibernate warns: "Applying offset/limit in memory." To paginate with JOIN FETCH, you need a two-query approach (entity graph or subquery).

[Solution: Exercise 43](F-exercise-solutions.md#exercise-43)

---

## Section 9: Testing (Exercises 44-47)

📖 **Learn the theory first:** [Chapter 16: Testing](../day-6/16-testing.md)
🎨 **Prefer the fun version?** [Chapter 16 (Head First)](../head-first-style/day-6/16-testing.md)

### Exercise 44: Unit Test a Service ⭐⭐

📖 *Need a refresher?* See [Chapter 16: Testing](../day-6/16-testing.md) · [Head First](../head-first-style/day-6/16-testing.md) — the "Unit Testing with Mockito" section.

**Problem:** Write unit tests for a `MovieService` class using JUnit 5 and Mockito.

**Requirements:**
- Test the `MovieService` from Exercise 30 (or write a similar one)
- Use `@ExtendWith(MockitoExtension.class)` and `@Mock` / `@InjectMocks`
- Write tests for:

| Test | What to verify |
|------|---------------|
| `getAllMovies_returnsListOfMovies` | Service returns all movies from the repository |
| `getMovieById_existingId_returnsMovie` | Service returns the correct movie |
| `getMovieById_nonExistingId_throwsException` | Service throws `ResourceNotFoundException` |
| `createMovie_savesAndReturnsMovie` | Service converts DTO, saves entity, returns response |
| `deleteMovie_existingId_deletesSuccessfully` | Service calls `deleteById` on the repository |
| `deleteMovie_nonExistingId_throwsException` | Service throws when movie not found |

- Use `when(...).thenReturn(...)` to stub repository methods
- Use `verify(...)` to ensure repository methods are called
- Use `assertThrows(...)` for exception testing

**Expected behavior:**
```java
@Test
@DisplayName("getMovieById with existing ID returns the movie")
void getMovieById_existingId_returnsMovie() {
    // Given
    Movie movie = new Movie(1L, "Inception", "Christopher Nolan", 2010, 8.8);
    when(movieRepository.findById(1L)).thenReturn(Optional.of(movie));

    // When
    MovieResponse result = movieService.getMovieById(1L);

    // Then
    assertEquals("Inception", result.title());
    assertEquals("Christopher Nolan", result.director());
    verify(movieRepository).findById(1L);  // Verify repository was called
}
```

All 6 tests should pass.

**Hints:**
> Basic setup:
> ```java
> @ExtendWith(MockitoExtension.class)
> class MovieServiceTest {
>     @Mock private MovieRepository movieRepository;
>     @InjectMocks private MovieService movieService;
> }
> ```
> For `assertThrows`: `assertThrows(ResourceNotFoundException.class, () -> movieService.getMovieById(999L))`.
> For `save`, use `when(movieRepository.save(any(Movie.class))).thenReturn(savedMovie)` — `any()` matches any argument.
> For `deleteMovie`, stub `existsById` to return `true` or `false`, then verify `deleteById` was (or wasn't) called.

[Solution: Exercise 44](F-exercise-solutions.md#exercise-44)

---

### Exercise 45: MockMvc Controller Tests ⭐⭐

📖 *Need a refresher?* See [Chapter 16: Testing](../day-6/16-testing.md) · [Head First](../head-first-style/day-6/16-testing.md) — the "MockMvc and @WebMvcTest" section.

**Problem:** Write controller tests using `MockMvc` to test HTTP behavior without starting a real server.

**Requirements:**
- Test a `MovieController` with the following scenarios:

| Test | Endpoint | Expected |
|------|----------|----------|
| `getAllMovies_returns200` | GET /api/movies | Status 200, JSON array |
| `getMovieById_returns200` | GET /api/movies/1 | Status 200, correct movie |
| `getMovieById_notFound_returns404` | GET /api/movies/999 | Status 404 |
| `createMovie_returns201` | POST /api/movies | Status 201, created movie |
| `createMovie_invalidBody_returns400` | POST /api/movies | Status 400, validation errors |
| `deleteMovie_returns204` | DELETE /api/movies/1 | Status 204 |

- Use `@WebMvcTest(MovieController.class)` to test only the controller layer
- Use `@MockBean` to mock `MovieService`
- Use `MockMvc` to perform requests and assert responses

**Expected behavior:**
```java
@Test
@DisplayName("GET /api/movies returns 200 with list of movies")
void getAllMovies_returns200() throws Exception {
    // Given
    List<MovieResponse> movies = List.of(
        new MovieResponse(1L, "Inception", "Nolan", 2010, 8.8)
    );
    when(movieService.getAllMovies()).thenReturn(movies);

    // When & Then
    mockMvc.perform(get("/api/movies"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$[0].title").value("Inception"))
        .andExpect(jsonPath("$[0].director").value("Nolan"));
}
```

**Hints:**
> Setup:
> ```java
> @WebMvcTest(MovieController.class)
> class MovieControllerTest {
>     @Autowired private MockMvc mockMvc;
>     @MockBean private MovieService movieService;
> }
> ```
> For POST: `mockMvc.perform(post("/api/movies").contentType(MediaType.APPLICATION_JSON).content("{\"title\":\"Inception\"}"))`.
> `jsonPath("$.title")` accesses the `title` field in the JSON response. `$[0].title` accesses the first element of an array.
> `@WebMvcTest` loads only the web layer (controller, filters, advice) — no service or repository beans are created.
> For the 404 test, make the mock service throw `ResourceNotFoundException`.

[Solution: Exercise 45](F-exercise-solutions.md#exercise-45)

---

### Exercise 46: Integration Test ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 16: Testing](../day-6/16-testing.md) · [Head First](../head-first-style/day-6/16-testing.md) — the "Integration Testing with @SpringBootTest" section.

**Problem:** Write an integration test that starts the full Spring Boot application and tests the Movie API end-to-end.

**Requirements:**
- Use `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`
- Use `TestRestTemplate` to make real HTTP requests
- Use H2 in-memory database (auto-configured in test)
- Test the full flow:

```
1. POST /api/movies  → create a movie → verify 201 and response body
2. GET /api/movies   → list all       → verify the movie is in the list
3. GET /api/movies/1 → get by ID      → verify the correct movie is returned
4. PUT /api/movies/1 → update         → verify the movie is updated
5. DELETE /api/movies/1 → delete      → verify 204
6. GET /api/movies/1 → get deleted    → verify 404
```

- No mocks — the test hits the real controller, service, repository, and database

**Expected behavior:**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class MovieIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    @DisplayName("Full CRUD lifecycle for movies")
    void movieCrudLifecycle() {
        // 1. Create
        MovieRequest request = new MovieRequest("Inception", "Nolan", 2010, 8.8);
        ResponseEntity<MovieResponse> createResponse =
            restTemplate.postForEntity("/api/movies", request, MovieResponse.class);
        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        assertNotNull(createResponse.getBody().id());

        Long movieId = createResponse.getBody().id();

        // 2. Get all — verify it's there
        // 3. Get by ID — verify correct data
        // 4. Update — verify changes
        // 5. Delete — verify 204
        // 6. Get deleted — verify 404
    }
}
```

**Hints:**
> `TestRestTemplate` is auto-configured when you use `RANDOM_PORT`. It handles the random port automatically.
> For PUT: `restTemplate.put("/api/movies/" + movieId, updatedRequest)` — but `put` returns void. Use `restTemplate.exchange(url, HttpMethod.PUT, new HttpEntity<>(request), MovieResponse.class)` to get the response.
> For DELETE: `restTemplate.exchange(url, HttpMethod.DELETE, null, Void.class)`.
> The H2 database is in-memory, so each test run starts with a clean database. No cleanup needed.
> Add `@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)` if tests share state and interfere with each other.

[Solution: Exercise 46](F-exercise-solutions.md#exercise-46)

---

### Exercise 47: Test Validation Errors ⭐⭐

📖 *Need a refresher?* See [Chapter 16: Testing](../day-6/16-testing.md) · [Head First](../head-first-style/day-6/16-testing.md) — the "Testing Validation" section.

**Problem:** Write MockMvc tests that verify validation is working correctly — send invalid data and assert 400 with the expected error messages.

**Requirements:**
- Test the `POST /api/register` endpoint from Exercise 35
- Write tests for these scenarios:

| Test | Invalid Data | Expected Error Field |
|------|-------------|---------------------|
| Empty username | `{"username":"","email":"a@b.com","password":"12345678","age":20}` | `username` |
| Invalid email | `{"username":"alice","email":"not-email","password":"12345678","age":20}` | `email` |
| Short password | `{"username":"alice","email":"a@b.com","password":"123","age":20}` | `password` |
| Age too low | `{"username":"alice","email":"a@b.com","password":"12345678","age":5}` | `age` |
| Multiple errors | `{"username":"","email":"bad","password":"x","age":0}` | `username`, `email`, `password`, `age` |

- Assert:
  - Status code is 400
  - Response body contains the field name and error message
  - Multiple errors return all error messages (not just the first one)

**Expected behavior:**
```java
@Test
@DisplayName("POST /api/register with blank username returns 400 with error")
void register_blankUsername_returns400() throws Exception {
    String body = """
        {"username":"","email":"alice@example.com","password":"secret123","age":25}
        """;

    mockMvc.perform(post("/api/register")
            .contentType(MediaType.APPLICATION_JSON)
            .content(body))
        .andExpect(status().isBadRequest())
        .andExpect(jsonPath("$.errors.username").exists());
}
```

**Hints:**
> Use text blocks (Java 15+) with `"""` for readable JSON in tests.
> `jsonPath("$.errors.username").exists()` verifies the field is present.
> `jsonPath("$.errors.username").value("Username is required")` verifies the exact message.
> For the "multiple errors" test, chain multiple `jsonPath` assertions.
> Make sure your `GlobalExceptionHandler` (from Exercise 37) is in the same package or component-scanned — `@WebMvcTest` loads `@ControllerAdvice` classes automatically.

[Solution: Exercise 47](F-exercise-solutions.md#exercise-47)

---

## Section 10: Security & Configuration (Exercises 48-50)

📖 **Learn the theory first:** [Chapter 15: Configuration and Profiles](../day-5/15-configuration-and-profiles.md) | [Chapter 18: Security Basics](../day-6/18-security-basics.md)
🎨 **Prefer the fun version?** [Chapter 15 (Head First)](../head-first-style/day-5/15-configuration-and-profiles.md) | [Chapter 18 (Head First)](../head-first-style/day-6/18-security-basics.md)

### Exercise 48: Profile-Specific Configuration ⭐⭐

📖 *Need a refresher?* See [Chapter 15: Configuration and Profiles](../day-5/15-configuration-and-profiles.md) · [Head First](../head-first-style/day-5/15-configuration-and-profiles.md) — the "Profiles" section.

**Problem:** Create profile-specific configuration files for development and production environments.

**Requirements:**
- Create three properties files:

**`application.properties`** (shared defaults):
```properties
app.name=MovieAPI
server.port=8080
```

**`application-dev.properties`** (development overrides):
- H2 in-memory database
- H2 console enabled
- Show SQL in logs
- Log level DEBUG for your package
- A custom property `app.environment=development`

**`application-prod.properties`** (production overrides):
- PostgreSQL database URL (any placeholder URL)
- H2 console disabled
- Don't show SQL
- Log level WARN
- `app.environment=production`

- Create an `AppConfig` class annotated with `@ConfigurationProperties(prefix = "app")` that reads `app.name` and `app.environment`
- Create an endpoint `GET /api/info` that returns the app name and environment

**Expected behavior:**
```bash
# Start with dev profile
$ java -jar app.jar --spring.profiles.active=dev
# or set SPRING_PROFILES_ACTIVE=dev

$ curl http://localhost:8080/api/info
{"name":"MovieAPI","environment":"development"}

# Start with prod profile
$ java -jar app.jar --spring.profiles.active=prod

$ curl http://localhost:8080/api/info
{"name":"MovieAPI","environment":"production"}
```

**Hints:**
> Spring Boot loads `application.properties` first, then overlays `application-{profile}.properties` on top.
> To activate a profile: command line `--spring.profiles.active=dev`, environment variable `SPRING_PROFILES_ACTIVE=dev`, or in `application.properties`: `spring.profiles.active=dev`.
> For `@ConfigurationProperties`:
> ```java
> @Component
> @ConfigurationProperties(prefix = "app")
> public class AppConfig {
>     private String name;
>     private String environment;
>     // getters and setters
> }
> ```
> Add the `spring-boot-configuration-processor` dependency for IDE auto-completion of custom properties.

[Solution: Exercise 48](F-exercise-solutions.md#exercise-48)

---

### Exercise 49: Public and Protected Endpoints ⭐⭐

📖 *Need a refresher?* See [Chapter 18: Security Basics](../day-6/18-security-basics.md) · [Head First](../head-first-style/day-6/18-security-basics.md) — the "SecurityFilterChain" section.

**Problem:** Add Spring Security so that GET endpoints are public but POST, PUT, and DELETE require authentication.

**Requirements:**
- Add the `spring-boot-starter-security` dependency
- Create a `SecurityConfig` class with `@Configuration` and `@EnableWebSecurity`
- Define a `SecurityFilterChain` bean that:
  - Permits all GET requests to `/api/**` (read-only is public)
  - Requires authentication for POST, PUT, DELETE requests
  - Uses HTTP Basic authentication
  - Disables CSRF (since this is a stateless REST API)
- Configure an in-memory user with username `admin` and password `password`

**Expected behavior:**
```bash
# GET — works without authentication (200)
$ curl http://localhost:8080/api/movies
[{"id":1,"title":"Inception",...}]

# POST — without auth → 401 Unauthorized
$ curl -v -X POST http://localhost:8080/api/movies \
  -H "Content-Type: application/json" \
  -d '{"title":"Dune"}'
< HTTP/1.1 401

# POST — with auth → 201 Created
$ curl -X POST http://localhost:8080/api/movies \
  -u admin:password \
  -H "Content-Type: application/json" \
  -d '{"title":"Dune","director":"Denis Villeneuve","releaseYear":2021,"rating":8.0}'
< HTTP/1.1 201

# DELETE — with auth → 204
$ curl -X DELETE http://localhost:8080/api/movies/1 -u admin:password
< HTTP/1.1 204

# DELETE — without auth → 401
$ curl -v -X DELETE http://localhost:8080/api/movies/1
< HTTP/1.1 401
```

**Hints:**
> The `SecurityFilterChain` bean:
> ```java
> @Bean
> public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
>     http
>         .csrf(csrf -> csrf.disable())
>         .authorizeHttpRequests(auth -> auth
>             .requestMatchers(HttpMethod.GET, "/api/**").permitAll()
>             // configure POST, PUT, DELETE...
>             .anyRequest().authenticated()
>         )
>         .httpBasic(Customizer.withDefaults());
>     return http.build();
> }
> ```
> For the in-memory user, create an `InMemoryUserDetailsManager` bean with `UserDetails` built using `User.withDefaultPasswordEncoder()` (deprecated for production, fine for learning).
> `curl -u admin:password` sends a Basic Auth header automatically.
> Disable CSRF for REST APIs because they use tokens, not cookies. CSRF protection is for browser-based form submissions.

[Solution: Exercise 49](F-exercise-solutions.md#exercise-49)

---

### Exercise 50: Complete Security Configuration ⭐⭐⭐

📖 *Need a refresher?* See [Chapter 18: Security Basics](../day-6/18-security-basics.md) · [Head First](../head-first-style/day-6/18-security-basics.md) — the "Role-Based Access Control" section.

**Problem:** Create a complete `SecurityFilterChain` that uses HTTP Basic authentication with an in-memory user, BCrypt password encoding, and role-based access control.

**Requirements:**
- Create a `SecurityConfig` class with:
  1. A `PasswordEncoder` bean that uses BCrypt
  2. An `InMemoryUserDetailsManager` bean with two users:
     - `user` with password `userpass` and role `USER`
     - `admin` with password `adminpass` and roles `USER` and `ADMIN`
  3. A `SecurityFilterChain` bean with these rules:

| Endpoint Pattern | Access Rule |
|-----------------|-------------|
| `GET /api/**` | Permit all (public) |
| `POST /api/**` | Requires `USER` role |
| `PUT /api/**` | Requires `USER` role |
| `DELETE /api/**` | Requires `ADMIN` role |
| `GET /h2-console/**` | Permit all |
| Any other request | Requires authentication |

  4. Disable CSRF
  5. Enable HTTP Basic authentication
  6. Allow H2 console frames (disable frame options headers)

**Expected behavior:**
```bash
# Public read — no auth needed
$ curl http://localhost:8080/api/movies
[...movies...]

# User can create (POST)
$ curl -X POST http://localhost:8080/api/movies \
  -u user:userpass \
  -H "Content-Type: application/json" \
  -d '{"title":"Dune"}'
< HTTP/1.1 201

# User CANNOT delete (needs ADMIN)
$ curl -v -X DELETE http://localhost:8080/api/movies/1 -u user:userpass
< HTTP/1.1 403 Forbidden

# Admin CAN delete
$ curl -v -X DELETE http://localhost:8080/api/movies/1 -u admin:adminpass
< HTTP/1.1 204

# No auth on mutating endpoint → 401
$ curl -v -X POST http://localhost:8080/api/movies \
  -H "Content-Type: application/json" \
  -d '{"title":"Dune"}'
< HTTP/1.1 401
```

**Hints:**
> BCrypt `PasswordEncoder` bean:
> ```java
> @Bean
> public PasswordEncoder passwordEncoder() {
>     return new BCryptPasswordEncoder();
> }
> ```
>
> In-memory users:
> ```java
> @Bean
> public InMemoryUserDetailsManager userDetailsManager(PasswordEncoder encoder) {
>     UserDetails user = User.builder()
>         .username("user")
>         .password(encoder.encode("userpass"))
>         .roles("USER")
>         .build();
>     // ... admin user with roles("USER", "ADMIN")
>     return new InMemoryUserDetailsManager(user, admin);
> }
> ```
>
> For role-based access: `.requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")`.
>
> For H2 console frames: `.headers(headers -> headers.frameOptions(frame -> frame.disable()))`.
>
> Order matters in `authorizeHttpRequests` — more specific rules must come before `.anyRequest()`.
>
> `403 Forbidden` means "authenticated but not authorized." `401 Unauthorized` means "not authenticated."

[Solution: Exercise 50](F-exercise-solutions.md#exercise-50)

---

## What's Next?

You have completed all 50 exercises. If you worked through them all, you now have practical experience with:

- HTTP protocol and REST API design
- JSON structure and Java/JSON mapping
- Spring Boot controllers, request mapping, and response handling
- Dependency injection and interface-based design
- Layered architecture (Controller / Service / Repository)
- JPA entities, DTOs, and database access
- Input validation and global error handling
- Entity relationships and JPQL queries
- Unit testing, controller testing, and integration testing
- Spring Security with role-based access control

These are the foundations. To keep growing:
- Build your own project from scratch (a budget tracker, recipe app, or inventory system)
- Add features not covered here: pagination, file upload, caching, scheduled tasks
- Explore the topics in [Chapter 20: What's Next](../day-7/20-whats-next.md)

---

*Solutions for all exercises are available in [Appendix F: Exercise Solutions](F-exercise-solutions.md).*
