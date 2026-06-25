# Backend Web Development with Spring Boot — A 7-Day Guide

> *From "I know Java" to "I can build a real backend application" in one week.*

---

## Who Is This For?

You know basic Java — classes, objects, methods, loops, collections. That's all you need. This guide assumes **zero knowledge** of web development, HTTP, databases, APIs, or any framework. We'll build everything from the ground up.

## What Will You Build?

By the end of this week, you'll have built **BookShelf** — a complete book library management API. It will:

- Accept HTTP requests from any client (browser, mobile app, Postman)
- Store book and author data in a database
- Validate incoming data and return meaningful errors
- Require authentication for sensitive operations
- Be fully tested and documented
- Run as a standalone application you can deploy anywhere

## Weekly Overview

| Day | Theme | What You'll Learn |
|-----|-------|-------------------|
| **1** | Understanding the Web | How the internet works, HTTP protocol, what a backend actually does |
| **2** | From Concepts to Code | JSON, REST APIs, setting up your first Spring Boot project |
| **3** | Spring Boot Core | Controllers, Dependency Injection, request lifecycle |
| **4** | Databases & Architecture | Layered architecture, JPA, persisting data to a real database |
| **5** | Robustness | Input validation, error handling, entity relationships, configuration |
| **6** | Quality & Security | Testing, logging, API documentation, securing your endpoints |
| **7** | Ship It | Packaging, deployment, and the road ahead |

## How to Use This Guide

### Reading Order

Follow the chapters **in order**. Each builds on the previous one. Skipping ahead will leave gaps.

```
README.md          ← You are here
day-1/
  01-how-the-internet-works.md
  02-speaking-http.md
  03-what-is-a-backend.md
day-2/
  04-json-and-apis.md
  05-why-frameworks-exist.md
  06-your-first-spring-boot-app.md
day-3/
  07-controllers.md
  08-dependency-injection.md
  09-request-response-lifecycle.md
day-4/
  10-thinking-in-layers.md
  11-entities-and-dtos.md
  12-databases-with-jpa.md
day-5/
  13-validation-and-error-handling.md
  14-relationships-and-queries.md
  15-configuration-and-profiles.md
day-6/
  16-testing.md
  17-logging-and-documentation.md
  18-security-basics.md
day-7/
  19-deploying-your-app.md
  20-whats-next.md
appendices/
  A-annotation-reference.md
  B-http-status-codes.md
  C-maven-cheatsheet.md
  D-glossary.md
  E-coding-exercises.md
  F-exercise-solutions.md
```

### Each Chapter Has

- **What You'll Learn** — 3-5 bullet points so you know what's coming
- **Concepts** — Detailed explanations with analogies and diagrams
- **Code Examples** — Runnable snippets you can type and experiment with
- **Exercise** — A hands-on task that builds the BookShelf project
- **Common Mistakes** — Pitfalls that trip up every beginner
- **Key Takeaways** — A checklist to confirm your understanding
- **Quick Quiz** — Self-check questions

### The BookShelf Project

The exercises aren't random — they form a single project that grows with you:

```
Chapter 7  → BookShelf v1: Hardcoded responses, no database
Chapter 9  → BookShelf v1 improved: Proper HTTP status codes
Chapter 10 → BookShelf v2: Clean layered architecture
Chapter 12 → BookShelf v3: Real database with full CRUD
Chapter 14 → BookShelf v4: Authors + relationships
Chapter 18 → BookShelf v5: Secured with authentication
Chapter 19 → BookShelf v5: Packaged and deployable
```

### Time Commitment

Each day requires roughly **3-5 hours**:
- ~1 hour reading concepts
- ~1 hour studying code examples
- ~1-3 hours on the exercise

Day 1 is lighter (theory focused). Days 4-6 are heavier (most coding).

## Prerequisites

### What You Need to Know
- Java basics: classes, interfaces, inheritance, methods, constructors
- Collections: List, Map, Set
- Basic OOP: what an object is, how to create one

### What You Need Installed
- **Java 17+** (run `java -version` to check)
- **Maven** (run `mvn -version` to check)
- **A code editor**: IntelliJ IDEA Community Edition (recommended) or VS Code with Java extensions
- **A terminal**: Terminal (macOS), PowerShell/CMD (Windows), or your IDE's built-in terminal
- **curl** (comes pre-installed on macOS/Linux; Windows users can use Git Bash)

### Nice to Have (Not Required)
- **Postman** — a GUI tool for testing APIs (we'll use curl, but Postman is friendlier)
- **Git** — version control (not covered, but useful for tracking your BookShelf progress)

## A Note on Learning

You will feel confused. That's normal and expected. Backend development involves many concepts working together — HTTP, Java, frameworks, databases, networking — and it takes time for the mental model to click.

Two things that help:
1. **Type the code yourself.** Don't copy-paste. Typing forces your brain to process each line.
2. **Break things on purpose.** After an exercise works, change something and see what error you get. The errors teach you more than the successes.

---

*Let's begin. Open `day-1/01-how-the-internet-works.md` →*
