# Chapter 6: Your First Spring Boot Application

> ⏱ Estimated time: 60 minutes

## What You'll Learn

- How to generate a Spring Boot project using Spring Initializr
- What every file and folder in a Spring Boot project does
- What `@SpringBootApplication` means
- How Maven works (just enough to be productive)
- How to run your application and verify it works

---

## Concepts

### Spring Initializr: The Project Generator

You don't create a Spring Boot project from scratch. You use **Spring Initializr** — a web tool that generates a ready-to-run project with the dependencies you choose.

Think of it as ordering a pre-configured laptop: you pick the specs (RAM, storage, OS), and it arrives ready to use.

Go to: **https://start.spring.io**

### What Is Maven?

Before we generate the project, you need to understand Maven at a basic level.

**Maven** is a **build tool** for Java. It does three things:

1. **Manages dependencies** — downloads libraries your project needs (like Spring Boot itself)
2. **Compiles your code** — turns `.java` files into runnable code
3. **Packages your app** — creates a `.jar` file you can run anywhere

Maven uses a file called `pom.xml` (Project Object Model) to know what your project needs. You don't write most of it by hand — Spring Initializr generates it for you.

**The only Maven commands you need right now:**

```bash
mvn spring-boot:run     # Run the application
mvn clean package       # Build a runnable JAR file
mvn test                # Run tests
```

> Don't overthink Maven. It's a tool, like a dishwasher. You load it, press a button, and it does its job. Appendix C has a detailed cheat sheet if you need it later.

---

## Code Examples

### Step 1: Generate the Project

Go to **https://start.spring.io** and set:

| Setting | Value |
|---------|-------|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.3.x (latest 3.x) |
| Group | `com.bookshelf` |
| Artifact | `bookshelf` |
| Name | `bookshelf` |
| Description | `BookShelf - A book library management API` |
| Package name | `com.bookshelf` |
| Packaging | Jar |
| Java | 17 |

**Dependencies to add** (click "Add Dependencies"):
- **Spring Web** — for building REST APIs

That's all you need for now. We'll add more dependencies as we go.

Click **"Generate"** → downloads a `.zip` file.

### Step 2: Unzip and Open

```bash
# Unzip the downloaded file
unzip bookshelf.zip

# Move into the project directory
cd bookshelf

# If using IntelliJ: File → Open → select the bookshelf folder
# If using VS Code: code .
```

### Step 3: Understand the Project Structure

```
bookshelf/
├── pom.xml                          ← Maven configuration (dependencies, build settings)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── bookshelf/
│   │   │           └── BookshelfApplication.java    ← Entry point (main method)
│   │   └── resources/
│   │       ├── application.properties               ← App configuration
│   │       ├── static/                              ← Static files (ignore for APIs)
│   │       └── templates/                           ← HTML templates (ignore for APIs)
│   └── test/
│       └── java/
│           └── com/
│               └── bookshelf/
│                   └── BookshelfApplicationTests.java  ← Test file
├── mvnw                             ← Maven wrapper (run Maven without installing it)
├── mvnw.cmd                         ← Maven wrapper for Windows
└── .gitignore                       ← Files Git should ignore
```

**The files that matter right now:**

| File | Purpose |
|------|---------|
| `pom.xml` | Lists your dependencies and build configuration |
| `BookshelfApplication.java` | The main class — starts the application |
| `application.properties` | Configuration (port, database URL, etc.) |
| `src/main/java/com/bookshelf/` | Where ALL your Java code goes |
| `src/test/java/com/bookshelf/` | Where ALL your test code goes |

**Files you can ignore for now:** `static/`, `templates/` (used for server-side rendering, not REST APIs), `mvnw`/`mvnw.cmd` (Maven wrapper scripts).

### Step 4: Examine the Main Class

Open `src/main/java/com/bookshelf/BookshelfApplication.java`:

```java
package com.bookshelf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BookshelfApplication {

    public static void main(String[] args) {
        SpringApplication.run(BookshelfApplication.class, args);
    }
}
```

This is the entire starting point of your application. Let's break it down:

**`@SpringBootApplication`** — This single annotation does three things:
1. `@Configuration` — "This class can define beans (objects Spring manages)"
2. `@EnableAutoConfiguration` — "Spring Boot, please auto-configure everything based on my dependencies"
3. `@ComponentScan` — "Scan this package and all sub-packages for Spring components"

**`SpringApplication.run(...)`** — This one line:
- Creates the Spring application context (the container that manages all your objects)
- Starts the embedded Tomcat web server
- Auto-configures everything based on your dependencies
- Scans for your controllers, services, and repositories
- Makes the application ready to receive HTTP requests

> 🧠 **Think Like a Backend Engineer**: You will almost never modify this file. It's the ignition key — you turn it once, and the engine runs. Your actual code goes in other classes.

### Step 5: Examine pom.xml

Open `pom.xml` (you don't need to understand every line):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- This project inherits from Spring Boot's parent POM -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <!-- Your project's identity -->
    <groupId>com.bookshelf</groupId>
    <artifactId>bookshelf</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>bookshelf</name>
    <description>BookShelf - A book library management API</description>

    <properties>
        <java.version>17</java.version>
    </properties>

    <!-- Dependencies: libraries your project uses -->
    <dependencies>
        <!-- Spring Web: everything needed for REST APIs -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Testing support -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!-- Build plugin: creates a runnable JAR -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**Key parts:**
- `<parent>` — inherits Spring Boot's managed dependency versions
- `<dependencies>` — the libraries you're using. `spring-boot-starter-web` includes Tomcat, Spring MVC, Jackson (JSON parser), and everything else you need for REST APIs
- `<build>` — the plugin that packages your app as a runnable JAR

**"Starters"** are Spring Boot's bundled dependency packages:
| Starter | What It Includes |
|---------|-----------------|
| `spring-boot-starter-web` | Tomcat, Spring MVC, Jackson (JSON), validation |
| `spring-boot-starter-data-jpa` | Hibernate, JPA, database tools (added in Chapter 12) |
| `spring-boot-starter-security` | Authentication, authorization (added in Chapter 18) |
| `spring-boot-starter-test` | JUnit, Mockito, Spring Test |

### Step 6: Run the Application

Open your terminal, navigate to the project directory, and run:

```bash
mvn spring-boot:run
```

**What you'll see** (condensed):

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v3.3.0)

2025-01-15 10:30:00 INFO  Starting BookshelfApplication...
2025-01-15 10:30:01 INFO  Tomcat initialized with port 8080 (http)
2025-01-15 10:30:02 INFO  Started BookshelfApplication in 1.8 seconds
```

**Your server is now running on port 8080!**

### Step 7: Test It

Open a new terminal window (keep the server running) and try:

```bash
curl http://localhost:8080
```

You'll get:

```json
{"timestamp":"2025-01-15T10:30:05.000+00:00","status":404,"error":"Not Found","path":"/"}
```

This is a **404 Not Found** — and that's correct! You haven't defined any endpoints yet. The server is running, it received your request, but it doesn't know how to handle `/`. Spring Boot's default error handler returned this structured JSON error.

**Congratulations — you have a running web server.**

To stop the server, press `Ctrl+C` in the terminal where it's running.

### Step 8: Add a Quick Hello Endpoint

Create a new file at `src/main/java/com/bookshelf/HelloController.java`:

```java
package com.bookshelf;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello from BookShelf!";
    }
}
```

Now restart the server (`Ctrl+C`, then `mvn spring-boot:run`) and try:

```bash
curl http://localhost:8080/hello
```

Response:
```
Hello from BookShelf!
```

**You just built your first API endpoint.** Let's break down what happened:

1. `@RestController` — tells Spring "this class handles HTTP requests"
2. `@GetMapping("/hello")` — "when someone sends GET to `/hello`, call this method"
3. The method returns a `String` — Spring Boot sends it as the HTTP response body
4. Spring Boot automatically sets status 200 and Content-Type header

We'll dive deep into controllers in Chapter 7.

---

## Exercise: Set Up Your BookShelf Project

**Goal**: Get a running Spring Boot application on your machine.

### Tasks

1. **Generate the project** at https://start.spring.io with the settings from Step 1
2. **Open it** in your IDE
3. **Run it** with `mvn spring-boot:run`
4. **Verify** the 404 response: `curl http://localhost:8080`
5. **Create the HelloController** from Step 8
6. **Verify** hello works: `curl http://localhost:8080/hello`
7. **Experiment** — add two more endpoints:

```java
@GetMapping("/goodbye")
public String goodbye() {
    return "Goodbye from BookShelf!";
}

@GetMapping("/time")
public String time() {
    return "Current time: " + java.time.LocalDateTime.now();
}
```

8. **Try to break it** — what happens if you:
   - Create two methods with the same `@GetMapping("/hello")`?
   - Send a POST request to `/hello`? (`curl -X POST http://localhost:8080/hello`)
   - Access a path that doesn't exist?

### Important Configuration

Add this line to `src/main/resources/application.properties`:

```properties
server.port=8080
```

This is the default, but now you know where to change it. Try changing it to `9090` and accessing `http://localhost:9090/hello`.

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| "I need to install Tomcat" | No. Spring Boot includes an embedded Tomcat. Just run your app. |
| Putting Java files outside the main package | `@ComponentScan` only scans `com.bookshelf` and its sub-packages. If you put a controller in `com.other`, Spring won't find it. |
| Forgetting to restart after changes | Unlike some languages, Java needs a restart to pick up changes. (`Ctrl+C` then `mvn spring-boot:run`). Later you can add Spring DevTools for auto-restart. |
| Trying to run two apps on the same port | You'll get "Port 8080 already in use." Stop the first instance or change the port in `application.properties`. |
| Panicking at the Whitelabel Error Page | That's not a bug — it's Spring Boot telling you "I'm running but you haven't told me what to do at this URL." |

---

## Key Takeaways

- [ ] I can generate a Spring Boot project using Spring Initializr
- [ ] I understand the project structure (src/main/java, resources, pom.xml)
- [ ] I know what `@SpringBootApplication` does (auto-config, component scan, config)
- [ ] I can run the application with `mvn spring-boot:run`
- [ ] I created my first endpoint using `@RestController` and `@GetMapping`
- [ ] I can test my endpoint with `curl`

---

## Quick Quiz

1. What does `spring-boot-starter-web` include?
2. Where does your Java code go in the project structure?
3. What happens when you run `SpringApplication.run(BookshelfApplication.class, args)`?
4. Why does `curl http://localhost:8080` return a 404 for a fresh project?
5. What's the difference between `mvn spring-boot:run` and `mvn clean package`?

---

## Day 2 Summary

```
✓ JSON is the universal data format for APIs (key-value pairs, arrays, nesting)
✓ REST is a convention: resources as URLs, HTTP methods as actions
✓ Frameworks handle boilerplate so you focus on business logic
✓ Spring Boot = Spring + auto-configuration + embedded server
✓ You created and ran your first Spring Boot application!
✓ You built your first endpoint with @RestController and @GetMapping
```

Tomorrow, you'll build real API endpoints, learn dependency injection, and understand the full lifecycle of an HTTP request through your application.

---

*Next: `day-3/07-controllers.md` — Time to build real endpoints for the BookShelf API →*
