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

Picture this: You just finished building your beautiful BookShelf API. It works perfectly on your laptop. You high-five yourself, push the code, and deploy it to production.

**Then everything catches fire.**

Why? Because your code is pointing at an H2 in-memory database that evaporates every time the server restarts. Your production server needs PostgreSQL. Your port is wrong. Your logging is drowning the log server in DEBUG noise. And your error messages are cheerfully exposing your entire stack trace to the internet.

*All because you hardcoded your settings.*

Here's the thing your code needs to behave **differently** depending on where it runs, and that's not a bug --- it's reality:

| Setting | Development | Production |
|---------|-------------|------------|
| Database | H2 in-memory | PostgreSQL on a server |
| Port | 8080 | 80 or 443 |
| Logging | DEBUG (verbose) | WARN (errors only) |
| Show SQL | Yes (for debugging) | No (performance) |
| Error details | Show full errors | Hide internals |

Hardcoding these values means changing code every time you deploy. Configuration lets you change behavior without changing code.

> 🎯 **Key Point:** Configuration is the art of making your app flexible without touching the source code. One binary, many environments --- that's the dream.

---

### 🗣️ Overheard at the coffee shop

> *"I used to just change the database URL in my Java file, recompile, and deploy."*
>
> *"That's... that's horrifying."*
>
> *"Yeah, I know that now. My production database got wiped three times before someone told me about `application.properties`."*

---

### application.properties

Meet the single most important file in your Spring Boot project that *isn't* a Java file. It lives at `src/main/resources/application.properties`, and it's where your app goes to learn about itself:

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

Think of it as a conversation between you and Spring Boot. You say "Hey, run on port 8080," and Spring Boot says "Got it, boss."

---

### 🗣️ Fireside Chat: An Interview with application.properties

**Interviewer:** Welcome to the show. Tell us a bit about yourself.

**application.properties:** Thanks for having me. I'm the settings file. Change me, change *everything*. I sit quietly in `src/main/resources/`, and when the application starts up, Spring Boot reads me cover to cover. Every line I contain becomes a configuration value that shapes how the app behaves.

**Interviewer:** So you're basically the app's instruction manual?

**application.properties:** More like its personality file. Want the app to be chatty and log everything? I can do that. Want it to be quiet and production-hardened? Change a few of my lines. The Java code doesn't change at all --- just me.

**Interviewer:** What if someone puts secrets in you, like passwords?

**application.properties:** *\*sighs\** They do that all the time. And then they commit me to Git. Please, for the love of all that is holy, use environment variables for secrets. I can reference them with `${VARIABLE_NAME}` syntax. I *want* to be committed to source control --- just not with passwords in me.

**Interviewer:** Fair enough. Any final words?

**application.properties:** Remember: I'm always loaded. Always. Profile-specific files can override me, but I'm the baseline. Respect the baseline.

---

### application.yml (Alternative)

YAML format --- same content, different syntax. Some people prefer it because nested properties look cleaner:

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

> ⚠️ **Watch it!** YAML is indentation-sensitive. One misaligned space and your config silently breaks. If you're the kind of person who mixes tabs and spaces... stick with `.properties`.

---

### Reading Configuration Values

Okay, you've got settings in a file. Great. But how does your Java code actually *use* them? Two ways, and one is better than the other.

#### @Value --- Simple Properties

The quick-and-dirty approach. Slap `@Value` on a field and Spring injects the property value:

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

See that `:Default Name` after the property key? That's a fallback. If the property doesn't exist, Spring uses "Default Name" instead of blowing up in your face. Handy.

> 💡 **There are no Dumb Questions**
>
> **Q: Why is there a `$` and curly braces around the property name?**
>
> A: That's Spring's expression syntax. `${bookshelf.max-books-per-user}` tells Spring "go look up this key in the configuration and give me the value." Without the `${}`, Spring would literally inject the string `bookshelf.max-books-per-user` into your field. Not helpful.
>
> **Q: What happens if I typo the property name?**
>
> A: If there's no default value (no `:` fallback), your application *refuses to start*. Spring throws an exception during startup. This is actually a good thing --- fail fast, not at 3 AM in production.
>
> **Q: Can `@Value` handle complex types like lists or maps?**
>
> A: It can, but awkwardly. That's exactly why `@ConfigurationProperties` exists. Keep reading.

---

#### @ConfigurationProperties --- Grouped Properties (Better)

Here's the grown-up way to read configuration. When you've got multiple related properties sharing a prefix, bundle them into a dedicated class:

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

> 🧠 **Brain Power:** Why do you think `@ConfigurationProperties` requires *getters and setters* while `@Value` just injects directly into fields? Think about how Spring might populate a `@ConfigurationProperties` object versus a `@Value` field. What design pattern is at play here?

> 🎯 **Key Point:** `@Value` is fine for one or two properties. But the moment you have a *group* of related settings (and you will), reach for `@ConfigurationProperties`. It gives you type safety, IDE autocomplete, defaults in one place, and it's way easier to test.

---

### Profiles: Environment-Specific Configuration

Alright, you understand configuration. But here's the million-dollar question: *what happens when the same property needs different values in different environments?*

Do you make one giant `application.properties` with a bunch of comments like `# UNCOMMENT THIS FOR PRODUCTION`?

**No.** Absolutely not. That's how production databases get wiped.

**Profiles** let you have different configuration files for different environments.

#### How It Works

```
application.properties          ← Always loaded (shared settings)
application-dev.properties      ← Loaded when profile = "dev"
application-prod.properties     ← Loaded when profile = "prod"
application-test.properties     ← Loaded when profile = "test"
```

Profile-specific files **override** the base file. Settings in `application-dev.properties` take priority over `application.properties`.

It's like layers. The base file is your foundation, and each profile paints over the parts it wants to change.

---

### 🗣️ Fireside Chat: An Interview with the Profiles

**Interviewer:** I'm here with two profiles --- Dev and Prod. Thanks for joining.

**Dev Profile:** Hey! Glad to be here. Love talking. Want to see my SQL queries? I log ALL of them.

**Prod Profile:** ... I'd rather not be here. I have uptime to maintain.

**Interviewer:** You two look identical from the outside --- same filename pattern, same structure. But you behave completely differently?

**Dev Profile:** I'm dev. My twin is prod. We look alike but behave *very* differently. I'm the "leave the debugging lights on" guy. Full SQL logging, H2 console wide open, stack traces for everyone. I exist to make developers' lives easy.

**Prod Profile:** And I exist to make sure the app doesn't get hacked or fall over at 2 AM. I hide error details, I use a real database, and I *never* log at DEBUG level. Do you know what DEBUG-level logging does to your disk space in production? Nothing good.

**Interviewer:** How do you decide who gets activated?

**Dev Profile:** Whoever sets `spring.profiles.active` decides. Usually it's `dev` by default so developers don't have to think about it. Then the deployment pipeline switches to `prod`.

**Prod Profile:** And if nobody sets an active profile? *Neither of us loads.* Only the base `application.properties` runs. Which is... usually not enough.

**Interviewer:** Any parting advice?

**Dev Profile:** Don't put real passwords in me!

**Prod Profile:** Don't put `ddl-auto=create-drop` in *me*.

---

#### Setting the Active Profile

You've got three ways to tell Spring Boot which profile to use:

**Option 1: In `application.properties`**

```properties
# In application.properties
spring.profiles.active=dev
```

**Option 2: Command line (overrides the properties file)**

```bash
mvn spring-boot:run -Dspring.profiles.active=dev
java -jar bookshelf.jar --spring.profiles.active=prod
```

**Option 3: Environment variable (great for containers and CI/CD)**

```bash
export SPRING_PROFILES_ACTIVE=prod
```

> 💡 **There are no Dumb Questions**
>
> **Q: What if I set the profile in `application.properties` AND on the command line?**
>
> A: The command line wins. Spring Boot has a priority order: command-line args beat environment variables, which beat properties files. Think of it as a "most specific wins" rule.
>
> **Q: Can I have more than one profile active at the same time?**
>
> A: Yes! Use a comma-separated list: `spring.profiles.active=dev,metrics`. Both `application-dev.properties` and `application-metrics.properties` will load. If they conflict, the *last* one listed wins.
>
> **Q: Do I need to create ALL profile files (dev, prod, test)?**
>
> A: Nope. Only create the ones you actually need. If you only have `dev` and `prod`, that's totally fine.

---

## Code Examples

### Setting Up Dev and Prod Profiles

Let's get practical. Here's exactly what goes where.

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

> ⚠️ **Watch it!** See `ddl-auto=create-drop` in the dev config? That tells Hibernate "nuke the database and rebuild it every time I restart." In development, this is *wonderful* because you get a clean slate. In production, this would literally **delete all your user data on every deploy**. This is not a drill. People have done this. Don't be that person.

---

### Using Environment Variables

In production, never hardcode sensitive values. Use environment variables:

```properties
spring.datasource.url=${DATABASE_URL}
spring.datasource.password=${DB_PASSWORD}
```

Spring Boot automatically resolves `${VARIABLE_NAME}` from the environment.

> 🧠 **Brain Power:** Your team deploys to three environments: dev, staging, and production. Each needs different database URLs, API keys, and feature flags. How would you organize your configuration files and environment variables? Sketch it out on paper before reading on. Where do shared values live? Where do secrets live? What's in source control and what isn't?

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

> ⚠️ **Watch it!** These are the traps that catch almost everybody at least once. Read them now so you can avoid the 2 AM debugging session later.

| Mistake | Reality |
|---------|---------|
| Hardcoding database passwords | Use environment variables or a secrets manager. Never commit passwords to source code. |
| Using `ddl-auto=create-drop` in production | This deletes all data on every restart. Use `validate` or `none` in production. |
| Not having a default profile | Without `spring.profiles.active`, Spring uses no profile --- which means only `application.properties` is loaded. Always set a default. |
| Using `@Value` for many related properties | If you have 5+ properties with the same prefix, use `@ConfigurationProperties` instead. It's typed, validated, and testable. |

> 💡 **There are no Dumb Questions**
>
> **Q: I accidentally committed my database password to Git. What do I do?**
>
> A: First, change the password immediately. Then remove it from the code and use environment variables going forward. Be aware that even if you delete the password from the latest commit, it still exists in your Git history. You'd need to rewrite Git history (using `git filter-branch` or BFG Repo-Cleaner) to truly remove it. This is a pain. Prevention is much easier than the cure.

---

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 48](../../appendices/E-coding-exercises.md#exercise-48) | Profile-Specific Configuration | ⭐⭐ |

Solutions are in [Appendix F](../../appendices/F-exercise-solutions.md).

---

## Key Takeaways

> 🎯 **Key Point:** If you remember nothing else from this chapter, remember these:

- [ ] `application.properties` is the central configuration file
- [ ] `@Value` reads individual properties; `@ConfigurationProperties` groups related properties
- [ ] Profiles (`application-dev.properties`, `application-prod.properties`) let you have environment-specific config
- [ ] Profile-specific properties override the base `application.properties`
- [ ] Never hardcode secrets --- use environment variables
- [ ] `ddl-auto=create-drop` for dev, `validate` for prod

---

## Quick Quiz

*Grab a pen. No peeking at the answers above. Seriously --- the act of recalling cements it in your brain.*

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

*Next: `day-6/16-testing.md` --- Prove your code works →*
