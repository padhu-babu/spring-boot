# Chapter 14: Relationships and Queries

> ⏱ Estimated time: 70 minutes

## What You'll Learn

- How database tables relate to each other
- One-to-Many and Many-to-One relationships in JPA
- Lazy vs. eager loading (and the N+1 problem)
- Custom queries with `@Query`
- How to build BookShelf v4 with Authors

---

## Concepts

### Why Relationships?

In the real world, data is connected:
- A **book** has an **author**
- An **author** has many **books**
- An **order** has many **items**
- A **student** takes many **courses** and a **course** has many **students**

Right now, our Book has `author` as a plain String. But what if we want:
- All books by a specific author?
- Author details (biography, birth year)?
- To update an author's name and have it reflected everywhere?

We need a separate `Author` entity **linked** to `Book`.

### Types of Relationships

| Relationship | Example | JPA Annotation |
|-------------|---------|----------------|
| **One-to-Many** | One author has many books | `@OneToMany` |
| **Many-to-One** | Many books belong to one author | `@ManyToOne` |
| **One-to-One** | One user has one profile | `@OneToOne` |
| **Many-to-Many** | Many students in many courses | `@ManyToMany` |

We'll focus on **One-to-Many / Many-to-One** — the most common relationship.

### Database Representation

In the database, relationships are represented by **foreign keys** — a column in one table that references the ID of another table:

```
Table: authors
┌────┬──────────────────┬─────────────┐
│ id │ name             │ nationality │
├────┼──────────────────┼─────────────┤
│  1 │ Frank Herbert    │ American    │
│  2 │ George Orwell    │ British     │
└────┴──────────────────┴─────────────┘

Table: books
┌────┬───────────┬───────┬───────────┐
│ id │ title     │ pages │ author_id │  ← Foreign key: points to authors.id
├────┼───────────┼───────┼───────────┤
│  1 │ Dune      │   412 │         1 │  ← Belongs to Frank Herbert
│  2 │ 1984      │   328 │         2 │  ← Belongs to George Orwell
│  3 │ Dune M.   │   256 │         1 │  ← Also belongs to Frank Herbert
└────┴───────────┴───────┴───────────┘
```

`author_id` in the books table is the **foreign key** — it creates the link.

### JPA: Modeling Relationships

**The "One" side (Author):**

```java
@Entity
@Table(name = "authors")
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "author")    // "I have many books. The Book entity owns the relationship."
    private List<Book> books = new ArrayList<>();
}
```

**The "Many" side (Book):**

```java
@Entity
@Table(name = "books")
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @ManyToOne                         // "I belong to one author"
    @JoinColumn(name = "author_id")   // "The foreign key column is called author_id"
    private Author author;
}
```

**Key annotations:**
- `@ManyToOne` — on the field that holds the parent (`Book.author`)
- `@OneToMany(mappedBy = "author")` — on the collection that holds the children. `mappedBy` says "the `author` field in `Book` is the owner of this relationship"
- `@JoinColumn(name = "author_id")` — names the foreign key column in the database

### Lazy vs. Eager Loading

When you load a Book, should JPA also load its Author? What about when you load an Author — should it load all their Books?

**Eager loading**: Load everything immediately.
```java
Author author = authorRepo.findById(1);
// author.getBooks() is already loaded — JPA fetched them in the same query
```

**Lazy loading**: Load on demand (when you access the field).
```java
Author author = authorRepo.findById(1);
// author.getBooks() is NOT loaded yet
// JPA only loads the books when you call author.getBooks()
```

**Defaults:**
- `@ManyToOne` → **EAGER** (loading a book loads its author — usually fine, one author per book)
- `@OneToMany` → **LAZY** (loading an author does NOT load their books until you ask — an author might have hundreds of books)

> 🧠 **Think Like a Backend Engineer**: Lazy loading is usually what you want for collections. Imagine an author with 1000 books — you don't want to load all 1000 every time you fetch the author's name.

### The N+1 Problem (Preview)

The N+1 problem is the most common performance pitfall in JPA:

```java
List<Book> books = bookRepo.findAll();  // 1 query: SELECT * FROM books

for (Book book : books) {
    System.out.println(book.getAuthor().getName());  
    // Each call triggers ANOTHER query: SELECT * FROM authors WHERE id = ?
    // If you have 100 books → 100 extra queries!
}
```

1 query for books + N queries for authors = N+1 queries.

**Fix**: Use `JOIN FETCH` in a custom query to load everything in one query. We'll see this below.

---

## Code Examples

### BookShelf v4: Adding Authors

#### Step 1: Create the Author Entity

Create `src/main/java/com/bookshelf/model/Author.java`:

```java
package com.bookshelf.model;

import jakarta.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "authors")
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column
    private String nationality;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL)
    private List<Book> books = new ArrayList<>();

    public Author() {}

    public Author(String name, String nationality) {
        this.name = name;
        this.nationality = nationality;
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getNationality() { return nationality; }
    public void setNationality(String nationality) { this.nationality = nationality; }
    public List<Book> getBooks() { return books; }
    public void setBooks(List<Book> books) { this.books = books; }
}
```

#### Step 2: Update Book Entity

Replace the `String author` field with an `Author` relationship:

```java
package com.bookshelf.model;

import jakarta.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "books")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "author_id")
    private Author author;

    @Column
    private int pages;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    public Book() {}

    @PrePersist
    protected void onCreate() {
        this.createdAt = LocalDateTime.now();
    }

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public Author getAuthor() { return author; }
    public void setAuthor(Author author) { this.author = author; }
    public int getPages() { return pages; }
    public void setPages(int pages) { this.pages = pages; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    public void setCreatedAt(LocalDateTime createdAt) { this.createdAt = createdAt; }
}
```

#### Step 3: Create Author DTOs

Create `src/main/java/com/bookshelf/dto/AuthorRequest.java`:

```java
package com.bookshelf.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

public record AuthorRequest(
    @NotBlank(message = "Name is required")
    @Size(max = 255, message = "Name must be at most 255 characters")
    String name,

    String nationality
) {}
```

Create `src/main/java/com/bookshelf/dto/AuthorResponse.java`:

```java
package com.bookshelf.dto;

public record AuthorResponse(
    Long id,
    String name,
    String nationality
) {}
```

#### Step 4: Update Book DTOs

```java
// BookRequest — now takes authorId instead of author name
package com.bookshelf.dto;

import jakarta.validation.constraints.*;

public record BookRequest(
    @NotBlank(message = "Title is required")
    @Size(max = 255, message = "Title must be at most 255 characters")
    String title,

    @NotNull(message = "Author ID is required")
    Long authorId,

    @Min(value = 1, message = "Pages must be at least 1")
    @Max(value = 10000, message = "Pages must be at most 10,000")
    int pages
) {}
```

```java
// BookResponse — includes author info
package com.bookshelf.dto;

import java.time.LocalDateTime;

public record BookResponse(
    Long id,
    String title,
    AuthorResponse author,
    int pages,
    LocalDateTime createdAt
) {}
```

#### Step 5: Create Author Repository and Service

```java
// AuthorRepository.java
package com.bookshelf.repository;

import com.bookshelf.model.Author;
import org.springframework.data.jpa.repository.JpaRepository;

public interface AuthorRepository extends JpaRepository<Author, Long> {}
```

```java
// AuthorService.java
package com.bookshelf.service;

import com.bookshelf.dto.AuthorRequest;
import com.bookshelf.dto.AuthorResponse;
import com.bookshelf.exception.AuthorNotFoundException;
import com.bookshelf.model.Author;
import com.bookshelf.repository.AuthorRepository;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class AuthorService {

    private final AuthorRepository authorRepository;

    public AuthorService(AuthorRepository authorRepository) {
        this.authorRepository = authorRepository;
    }

    public List<AuthorResponse> getAllAuthors() {
        return authorRepository.findAll().stream()
                .map(this::toResponse)
                .toList();
    }

    public AuthorResponse getAuthorById(Long id) {
        return authorRepository.findById(id)
                .map(this::toResponse)
                .orElseThrow(() -> new AuthorNotFoundException(id));
    }

    public AuthorResponse createAuthor(AuthorRequest request) {
        Author author = new Author(request.name(), request.nationality());
        return toResponse(authorRepository.save(author));
    }

    private AuthorResponse toResponse(Author author) {
        return new AuthorResponse(author.getId(), author.getName(), author.getNationality());
    }
}
```

#### Step 6: Update BookService

```java
package com.bookshelf.service;

import com.bookshelf.dto.*;
import com.bookshelf.exception.BookNotFoundException;
import com.bookshelf.model.Author;
import com.bookshelf.model.Book;
import com.bookshelf.repository.AuthorRepository;
import com.bookshelf.repository.BookRepository;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class BookService {

    private final BookRepository bookRepository;
    private final AuthorRepository authorRepository;

    public BookService(BookRepository bookRepository, AuthorRepository authorRepository) {
        this.bookRepository = bookRepository;
        this.authorRepository = authorRepository;
    }

    public List<BookResponse> getAllBooks() {
        return bookRepository.findAll().stream()
                .map(this::toResponse)
                .toList();
    }

    public BookResponse getBookById(Long id) {
        return bookRepository.findById(id)
                .map(this::toResponse)
                .orElseThrow(() -> new BookNotFoundException(id));
    }

    public BookResponse createBook(BookRequest request) {
        Author author = authorRepository.findById(request.authorId())
                .orElseThrow(() -> new RuntimeException("Author not found with id: " + request.authorId()));

        Book book = new Book();
        book.setTitle(request.title());
        book.setAuthor(author);
        book.setPages(request.pages());

        return toResponse(bookRepository.save(book));
    }

    public BookResponse updateBook(Long id, BookRequest request) {
        Book book = bookRepository.findById(id)
                .orElseThrow(() -> new BookNotFoundException(id));

        Author author = authorRepository.findById(request.authorId())
                .orElseThrow(() -> new RuntimeException("Author not found with id: " + request.authorId()));

        book.setTitle(request.title());
        book.setAuthor(author);
        book.setPages(request.pages());

        return toResponse(bookRepository.save(book));
    }

    public void deleteBook(Long id) {
        if (!bookRepository.existsById(id)) {
            throw new BookNotFoundException(id);
        }
        bookRepository.deleteById(id);
    }

    public List<BookResponse> searchByTitle(String title) {
        return bookRepository.findByTitleContainingIgnoreCase(title).stream()
                .map(this::toResponse)
                .toList();
    }

    private BookResponse toResponse(Book book) {
        AuthorResponse authorResponse = book.getAuthor() != null
                ? new AuthorResponse(book.getAuthor().getId(), book.getAuthor().getName(), book.getAuthor().getNationality())
                : null;

        return new BookResponse(
                book.getId(),
                book.getTitle(),
                authorResponse,
                book.getPages(),
                book.getCreatedAt()
        );
    }
}
```

#### Step 7: Add Custom Query

Update `BookRepository` with a `@Query` for more complex queries:

```java
package com.bookshelf.repository;

import com.bookshelf.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;

public interface BookRepository extends JpaRepository<Book, Long> {

    List<Book> findByTitleContainingIgnoreCase(String title);

    // Custom query: find all books by author ID
    List<Book> findByAuthorId(Long authorId);

    // JPQL query with JOIN FETCH (avoids N+1 problem)
    @Query("SELECT b FROM Book b JOIN FETCH b.author")
    List<Book> findAllWithAuthors();

    // JPQL query: search by author name
    @Query("SELECT b FROM Book b JOIN b.author a WHERE LOWER(a.name) LIKE LOWER(CONCAT('%', :name, '%'))")
    List<Book> findByAuthorNameContaining(@Param("name") String name);
}
```

### Testing

```bash
# First create an author
curl -X POST http://localhost:8080/api/authors \
  -H "Content-Type: application/json" \
  -d '{"name": "Frank Herbert", "nationality": "American"}'
# Response: {"id":1,"name":"Frank Herbert","nationality":"American"}

# Create a book with authorId
curl -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "authorId": 1, "pages": 412}'
# Response: {"id":1,"title":"Dune","author":{"id":1,"name":"Frank Herbert","nationality":"American"},"pages":412,...}

# Get all books — author info is embedded
curl http://localhost:8080/api/books
```

---

## Exercise: Build BookShelf v4

**Goal**: Add Authors to your BookShelf with a proper relationship.

### Tasks

1. Create `Author` entity with `@OneToMany` relationship to Book
2. Update `Book` entity with `@ManyToOne` relationship to Author
3. Create Author DTOs, repository, service, and controller
4. Update Book DTOs to use `authorId` in request and `AuthorResponse` in response
5. Update BookService to look up the author when creating/updating a book
6. Add at least one custom `@Query` method
7. Test the full flow: create author → create book → fetch book (with author info)

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Infinite recursion in JSON serialization | `Author` has `books`, each `Book` has `author`, which has `books`... → stack overflow. Use DTOs (which we do) to break the cycle. Don't return entities directly. |
| Forgetting `mappedBy` on `@OneToMany` | Without it, JPA creates an extra join table instead of using the foreign key. Always specify `mappedBy`. |
| Eager loading `@OneToMany` | Loading an author shouldn't load ALL their books. Keep `@OneToMany` as LAZY (the default). |
| Not using `@JoinColumn` on `@ManyToOne` | JPA can infer it, but being explicit (`name = "author_id"`) makes the database schema clear. |

---

## Key Takeaways

- [ ] Relationships connect entities (One-to-Many, Many-to-One, etc.)
- [ ] `@ManyToOne` + `@JoinColumn` goes on the "many" side (Book has one Author)
- [ ] `@OneToMany(mappedBy = "...")` goes on the "one" side (Author has many Books)
- [ ] Lazy loading is the default for collections — load on demand
- [ ] `@Query` lets you write custom JPQL queries when method names aren't enough
- [ ] Always use DTOs to avoid infinite recursion in JSON serialization

---

## Quick Quiz

1. In a One-to-Many relationship, which entity has the foreign key column?
2. What does `mappedBy = "author"` mean on `@OneToMany`?
3. Explain the N+1 problem in one sentence.
4. Why should `@OneToMany` default to LAZY loading?
5. Write a JPQL query that finds all books with more than 300 pages.

---

*Next: `15-configuration-and-profiles.md` — Managing settings across environments →*
