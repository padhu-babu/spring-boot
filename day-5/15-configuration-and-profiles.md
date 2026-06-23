# Chapter 15: Configuration and Profiles

> ⏱ Estimated time: 40 minutes

## What You'll Learn

- How Spring Boot configuration works (`application.properties` / `application.yml`)
- How to read configuration values in your code (`@Value`, `@ConfigurationProperties`)
- What profiles are and why you need them (dev, test, prod)
- How to set up environment-specific configuration

---

## Concepts

### Why Configuration Matters

Your application behaves differently depending on where it runs:

| Setting | Development | Production |
|---------|-------------|------------|
| Database | H2 in-memory | PostgreSQL on a server |
| Port | 8080 | 80 or 443 |
| Logging | DEBUG (verbose) | WARN (errors only) |
| Show SQL | Yes (for debugging) | No (performance) |
| Error details | Show full errors | Hide internals |

Hardcoding these values means changing code every time you deploy. Configuration lets you change behavior without changing code.

### application.properties

The primary configuration file. Located at `src/main/resources/application.properties`:

```properties
# Server
server.port=8080

# Database
spring.datasource.url=jdbc:h2:mem:bookshelf
spring.datasource.username=sa
spring.datasource.password=

# JPA
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

# Custom properties
bookshelf.max-books-per-user=100
bookshelf.app-name=BookShelf API
```

### application.yml (Alternative)

YAML format — same content, different syntax:

```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:h2:mem:bookshelf
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

bookshelf:
  max-books-per-user: 100
  app-name: BookShelf API
```

Use whichever you prefer. YAML is more readable for nested properties, but `.properties` is simpler. Don't mix both in the same project.

### Reading Configuration Values

#### @Value — Simple Properties

```java
@Service
public class BookService {

    @Value("${bookshelf.max-books-per-user}")
    private int maxBooksPerUser;

    @Value("${bookshelf.app-name:Default Name}")  // Default value after ':'
    private String appName;

    public Book createBook(BookRequest request) {
        if (bookRepository.count() >= maxBooksPerUser) {
            throw new RuntimeException("Maximum books limit reached: " + maxBooksPerUser);
        }
        // ...
    }
}
```

#### @ConfigurationProperties — Grouped Properties (Better)

For multiple related properties, group them in a class:

```java
package com.bookshelf.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "bookshelf")
public class BookshelfProperties {

    private int maxBooksPerUser = 100;  // Default value
    private String appName = "BookShelf";

    // Getters and setters required
    public int getMaxBooksPerUser() { return maxBooksPerUser; }
    public void setMaxBooksPerUser(int maxBooksPerUser) { this.maxBooksPerUser = maxBooksPerUser; }
    public String getAppName() { return appName; }
    public void setAppName(String appName) { this.appName = appName; }
}
```

Then inject the whole config object:

```java
@Service
public class BookService {

    private final BookshelfProperties properties;

    public BookService(BookRepository repo, BookshelfProperties properties) {
        this.properties = properties;
    }

    // Use properties.getMaxBooksPerUser(), properties.getAppName(), etc.
}
```

### Profiles: Environment-Specific Configuration

**Profiles** let you have different configuration files for different environments.

#### How It Works

```
application.properties          ← Always loaded (shared settings)
application-dev.properties      ← Loaded when profile = "dev"
application-prod.properties     ← Loaded when profile = "prod"
application-test.properties     ← Loaded when profile = "test"
```

Profile-specific files **override** the base file. Settings in `application-dev.properties` take priority over `application.properties`.

#### Setting the Active Profile

```properties
# In application.properties
spring.profiles.active=dev
```

Or via command line:
```bash
mvn spring-boot:run -Dspring.profiles.active=dev
java -jar bookshelf.jar --spring.profiles.active=prod
```

Or via environment variable:
```bash
export SPRING_PROFILES_ACTIVE=prod
```

---

## Code Examples

### Setting Up Dev and Prod Profiles

**`application.properties`** (shared baseline):

```properties
bookshelf.app-name=BookShelf API
bookshelf.max-books-per-user=100
spring.profiles.active=dev
```

**`application-dev.properties`** (development):

```properties
# H2 in-memory database
spring.datasource.url=jdbc:h2:mem:bookshelf
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true

spring.h2-console.enabled=true
spring.h2-console.path=/h2-console

server.port=8080

# Verbose logging in development
logging.level.com.bookshelf=DEBUG
logging.level.org.springframework.web=DEBUG
```

**`application-prod.properties`** (production):

```properties
# Real database (PostgreSQL example)
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USERNAME}
spring.datasource.password=${DATABASE_PASSWORD}

spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

spring.h2-console.enabled=false

server.port=80

# Minimal logging in production
logging.level.com.bookshelf=WARN
logging.level.org.springframework.web=WARN
```

**Key differences:**
- Dev uses H2 with `create-drop` (recreate tables on restart). Prod uses PostgreSQL with `validate` (verify schema, never modify).
- Dev enables verbose SQL logging and H2 console. Prod disables both.
- Prod reads database credentials from **environment variables** (`${DATABASE_URL}`), never hardcoded.

### Using Environment Variables

In production, never hardcode sensitive values. Use environment variables:

```properties
spring.datasource.url=${DATABASE_URL}
spring.datasource.password=${DB_PASSWORD}
```

Spring Boot automatically resolves `${VARIABLE_NAME}` from the environment.

---

## Exercise: Add Profiles to BookShelf

**Goal**: Configure your application for different environments.

### Tasks

1. Create `application-dev.properties` with H2 settings and DEBUG logging
2. Create `application-prod.properties` with different settings
3. Set the default active profile to `dev`
4. Create a `BookshelfProperties` class with `@ConfigurationProperties`
5. Add a `max-books-per-user` property and enforce it in `BookService.createBook()`
6. Run with dev profile and verify behavior
7. Switch to prod profile on the command line and observe the differences (logging level, etc.)

### Verification

```bash
# Run with dev profile (default)
mvn spring-boot:run
# → Look for DEBUG-level log messages

# Run with prod profile
mvn spring-boot:run -Dspring.profiles.active=prod
# → Only WARN-level messages, H2 console disabled
```

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Hardcoding database passwords | Use environment variables or a secrets manager. Never commit passwords to source code. |
| Using `ddl-auto=create-drop` in production | This deletes all data on every restart. Use `validate` or `none` in production. |
| Not having a default profile | Without `spring.profiles.active`, Spring uses no profile — which means only `application.properties` is loaded. Always set a default. |
| Using `@Value` for many related properties | If you have 5+ properties with the same prefix, use `@ConfigurationProperties` instead. It's typed, validated, and testable. |

---

## Key Takeaways

- [ ] `application.properties` is the central configuration file
- [ ] `@Value` reads individual properties; `@ConfigurationProperties` groups related properties
- [ ] Profiles (`application-dev.properties`, `application-prod.properties`) let you have environment-specific config
- [ ] Profile-specific properties override the base `application.properties`
- [ ] Never hardcode secrets — use environment variables
- [ ] `ddl-auto=create-drop` for dev, `validate` for prod

---

## Quick Quiz

1. What's the difference between `application.properties` and `application-dev.properties`?
2. How do you set the active profile from the command line?
3. Why is `@ConfigurationProperties` better than `@Value` for related settings?
4. What does `${DATABASE_URL}` do in a properties file?
5. Why is `ddl-auto=create-drop` dangerous in production?

---

## Day 5 Summary

```
✓ Never trust client input — validate with Bean Validation annotations
✓ @ControllerAdvice centralizes error handling across all controllers
✓ Custom exceptions + exception handlers = clean, consistent error responses
✓ Entities can have relationships: @ManyToOne, @OneToMany
✓ Foreign keys link tables in the database
✓ @Query lets you write custom JPQL queries
✓ Profiles separate dev/test/prod configuration
✓ Never hardcode secrets — use environment variables
```

Tomorrow, you'll learn testing, logging, and how to secure your API!

---

*Next: `day-6/16-testing.md` — Prove your code works →*
