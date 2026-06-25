# Chapter 6: Your First Spring Boot Application

> ⏱ Estimated time: 75 minutes

## What You'll Learn

- How to generate a Spring Boot project using Spring Initializr
- What every file and folder in a Spring Boot project does
- What `@SpringBootApplication` means
- How Maven works (just enough to be productive)
- How to run your application and verify it works

---

## Concepts

### Spring Initializr: The Project Generator

> **Quick Summary**: Spring Initializr is a website that generates a ready-to-run Spring Boot project. You pick your settings and dependencies, click Generate, and get a `.zip` file with everything configured.

You don't create a Spring Boot project from scratch. You use **Spring Initializr** — a web tool that generates a ready-to-run project with the dependencies you choose.

Think of it as ordering a pre-configured laptop: you pick the specs (RAM, storage, OS), and it arrives ready to use.

Go to: **https://start.spring.io**

**What the page looks like:**

```
┌─────────────────────────────────────────────────────────────────────┐
│  Spring Initializr                                                  │
│                                                                     │
│  Project          Language         Spring Boot                      │
│  ● Maven  ○ Gradle   ● Java  ○ Kotlin   ● 3.3.x  ○ 3.2.x         │
│                                                                     │
│  Project Metadata                                                   │
│  ┌──────────────────────────────────────────────────────┐           │
│  │ Group:        com.bookshelf                          │           │
│  │ Artifact:     bookshelf                              │           │
│  │ Name:         bookshelf                              │           │
│  │ Description:  BookShelf - A book library mgmt API    │           │
│  │ Package name: com.bookshelf                          │           │
│  │ Packaging:    ● Jar  ○ War                           │           │
│  │ Java:         ● 17  ○ 21                             │           │
│  └──────────────────────────────────────────────────────┘           │
│                                                                     │
│  Dependencies                        [ ADD DEPENDENCIES... ]        │
│  ┌──────────────────────┐                                           │
│  │ Spring Web           │                                           │
│  └──────────────────────┘                                           │
│                                                                     │
│               [ GENERATE ]    [ EXPLORE ]    [ SHARE ]              │
└─────────────────────────────────────────────────────────────────────┘
```

The left side has your project settings. The right side has a search box for adding dependencies. The "EXPLORE" button lets you preview the generated files before downloading.

**What Each Setting Means:**

| Setting | What It Means |
|---------|---------------|
| **Group** | Your organization's reversed domain name, just like Java packages. `com.bookshelf` means "bookshelf project at com." If you work at `example.com`, you'd use `com.example`. |
| **Artifact** | Your project's name. This becomes the folder name and the name of the built `.jar` file. |
| **Packaging: Jar vs War** | **Jar** = standalone app with an embedded server (just run it). **War** = deploy to an external server like standalone Tomcat. We use Jar. |
| **Java version: 17** | Java 17 is an **LTS (Long-Term Support)** release, meaning it receives security updates for years. It's the most widely supported version for Spring Boot 3.x. |

### What Is Maven?

> **Quick Summary**: Maven manages your project's dependencies (libraries), compiles your code, and packages it into a runnable `.jar` file. Think of it as npm + webpack for Java.

Before we generate the project, you need to understand Maven at a basic level.

#### What Problem Does Maven Solve?

Before Maven existed, Java developers had to:
- **Download `.jar` files manually** from websites and put them in a `lib/` folder
- **Track versions themselves** — "Did I download Spring 5.3.2 or 5.3.3?"
- **Resolve conflicts by hand** — Library A needs Guava 30, Library B needs Guava 31... which one do you use?
- **Write custom build scripts** for every project to compile and package the code

Maven solves all of this. You declare what you need in one file (`pom.xml`), and Maven downloads it, resolves conflicts, and builds your project.

> **Analogy**: Maven is like a **package manager** (npm for Node.js, pip for Python) **but also a build tool**. It handles both "get me these libraries" and "compile and package my project."

**Maven** is a **build tool** for Java. It does three things:

1. **Manages dependencies** — downloads libraries your project needs (like Spring Boot itself)
2. **Compiles your code** — turns `.java` files into runnable code
3. **Packages your app** — creates a `.jar` file you can run anywhere

Maven uses a file called `pom.xml` (Project Object Model) to know what your project needs. You don't write most of it by hand — Spring Initializr generates it for you.

#### Maven Lifecycle (Quick Overview)

Maven has a build lifecycle — a sequence of steps that run in order:

```mermaid
flowchart LR
    clean["clean\nDelete previous build output"]
    compile["compile\nCompile .java → .class files"]
    test["test\nRun unit tests"]
    package["package\nCreate the .jar file"]
    install["install\nPut the JAR in your\nlocal Maven cache"]

    clean --> compile --> test --> package --> install
```

When you run a later phase, all earlier phases run automatically. So `mvn package` will compile, then test, then package.

**You only need one command for now**: `mvn spring-boot:run` (this compiles and runs your app in one step).

**The only Maven commands you need right now:**

```bash
mvn spring-boot:run     # Run the application
mvn clean package       # Build a runnable JAR file
mvn test                # Run tests
```

> **Windows users**: If you don't have Maven installed globally, use the Maven Wrapper that comes with your project. Instead of `mvn spring-boot:run`, run `mvnw.cmd spring-boot:run`. On macOS/Linux, use `./mvnw spring-boot:run`.

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

> **Windows users**: The `.zip` file will download to your `Downloads` folder. Right-click it → "Extract All" → choose a location like `C:\Users\YourName\Projects\`. Avoid paths with spaces or special characters — they can cause issues with Maven.

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

> **Quick Summary**: Spring Boot follows a standard directory layout. Your Java code goes in `src/main/java`, configuration files go in `src/main/resources`, and tests go in `src/test/java`. All your code must live inside the base package (`com.bookshelf`) or Spring won't find it.

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

#### Why This Structure?

This directory layout isn't arbitrary — it's the **Maven Standard Directory Layout** that every Java project follows:

| Directory | Purpose |
|-----------|---------|
| `src/main/java` | Your **production code** — all the Java classes that run in your application |
| `src/main/resources` | **Configuration files**, NOT code. Properties, YAML, XML config, static assets. These get bundled into your JAR but aren't compiled as Java. |
| `src/test/java` | Your **test code**. Mirrors the `main` structure. Test classes for `com.bookshelf.HelloController` go in `src/test/java/com/bookshelf/HelloControllerTest.java`. |

**Understanding the package-to-folder mapping:**

The package name `com.bookshelf` maps directly to the folder structure `com/bookshelf/`. This is a Java requirement — if your class declares `package com.bookshelf;`, the file must be in a `com/bookshelf/` directory.

```
Package name:           com.bookshelf.controller
Maps to folder:         com/bookshelf/controller/
Full path in project:   src/main/java/com/bookshelf/controller/
```

> **Important**: All your code **MUST** be in `com.bookshelf` or its sub-packages (like `com.bookshelf.controller`, `com.bookshelf.service`, `com.bookshelf.model`). Code placed outside this package — for example, in `com.other` or `org.example` — **will not be found by Spring**. This is because `@ComponentScan` (part of `@SpringBootApplication`) only scans the package where the main class lives and everything below it.

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

> **Quick Summary**: `pom.xml` is Maven's configuration file. It declares your project's identity, the Java version, what libraries (dependencies) you need, and how to build the project. Spring Initializr generates it for you.

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

#### Reading pom.xml Section by Section

Let's walk through each block so nothing feels mysterious:

**1. `<parent>` — The Spring Boot Parent POM**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
</parent>
```
This says: "My project inherits from Spring Boot's parent." The parent POM pre-defines compatible versions for hundreds of libraries. That's why you don't see `<version>` tags on individual dependencies below — the parent already knows which versions work together. This saves you from version conflicts.

**2. `<groupId>`, `<artifactId>`, `<version>` — Your Project's Identity**
```xml
<groupId>com.bookshelf</groupId>
<artifactId>bookshelf</artifactId>
<version>0.0.1-SNAPSHOT</version>
```
Together, these three values uniquely identify your project in the Maven ecosystem. `SNAPSHOT` means "this is a development version, not a release." When you eventually deploy, you'd change it to `1.0.0`.

**3. `<properties>` — Project-Wide Settings**
```xml
<properties>
    <java.version>17</java.version>
</properties>
```
This tells Maven and Spring Boot to compile with Java 17. If you switch to Java 21 later, you only change it here.

**4. `<dependencies>` — Libraries Your Project Uses**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- No <version> needed — inherited from parent -->
</dependency>
```
Each `<dependency>` block declares a library you need. Maven downloads it (and all its sub-dependencies) automatically. The `<scope>test</scope>` on the test dependency means it's only available during testing, not in production.

**5. `<build>` — Build Plugins**
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```
This plugin gives you the `mvn spring-boot:run` command and packages your app as an executable JAR (with an embedded Tomcat server inside it).

**Key parts (the short version):**
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

### Step 7.5: Understanding application.properties

The file `src/main/resources/application.properties` is where you configure your Spring Boot application. Right now it's empty, but this is one of the most important files in your project.

Here are common properties you'll use throughout this guide:

```properties
# Server Configuration
server.port=8080                                  # Which port to run on (default: 8080)
server.servlet.context-path=/api                  # Prefix all URLs with /api (e.g., /api/hello)

# Application Info
spring.application.name=bookshelf                 # Name your app (shows in logs and monitoring)

# Logging
logging.level.root=INFO                           # Log level: ERROR < WARN < INFO < DEBUG < TRACE

# JSON Formatting (makes API responses human-readable during development)
spring.jackson.serialization.indent-output=true   # Pretty-print JSON responses
```

> **Don't add all of these right now.** We'll add properties as we need them throughout the guide. For now, just know that this file exists and that Spring Boot reads it automatically at startup. We'll add database properties in Chapter 12 and security properties in Chapter 18.

**Quick rules about this file:**
- One property per line, in `key=value` format
- Lines starting with `#` are comments
- No quotes around values (unlike JSON or YAML)
- Changes require a restart to take effect

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

1. **`@RestController`** — This annotation tells Spring two things: (a) "this class handles HTTP requests" (it's a controller), and (b) "every method's return value should be written directly to the HTTP response body" (not interpreted as a template name). It combines `@Controller` and `@ResponseBody` into one annotation.

2. **`@GetMapping("/hello")`** — "When someone sends an HTTP **GET** request to the path `/hello`, call this method." The method name (`hello`) doesn't matter to Spring — only the path in the annotation matters. You could name the method `foo()` and it would still respond to `/hello`.

3. **The method returns a `String`** — Spring Boot takes that string and sends it as the HTTP response body with status code `200 OK` and Content-Type `text/plain`.

4. **Spring Boot automatically sets HTTP headers** — You didn't write any HTTP response code. Spring Boot handled the status code, Content-Type header, and response formatting for you.

#### Returning JSON (Automatic Conversion)

Returning a string is fine, but real APIs return **JSON**. Let's add a second endpoint that returns a Java object. Add this method inside `HelloController`:

```java
import java.util.HashMap;
import java.util.Map;

// ... inside the HelloController class:

@GetMapping("/info")
public Map<String, Object> info() {
    Map<String, Object> data = new HashMap<>();
    data.put("app", "BookShelf");
    data.put("version", "1.0");
    data.put("ready", true);
    return data;
}
```

Restart the server and try:

```bash
curl http://localhost:8080/info
```

Response:
```json
{
  "app": "BookShelf",
  "version": "1.0",
  "ready": true
}
```

**Notice what just happened**: you returned a Java `Map`, but the HTTP response is **JSON**. You didn't write any JSON conversion code. Spring Boot (via a library called **Jackson**) automatically converted your Java object to JSON. This works with Maps, Lists, custom classes, records — any Java object. This automatic conversion is one of the reasons Spring Boot is so productive.

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

## Troubleshooting

If something goes wrong, check this section before searching online. These are the most common issues beginners hit.

### "Port 8080 already in use"

This means another process is already using port 8080. You need to find and stop it.

**macOS / Linux:**
```bash
# Find what's using port 8080
lsof -i :8080

# You'll see output like:
# COMMAND   PID   USER   FD   TYPE  DEVICE  SIZE/OFF NODE NAME
# java    12345  youruser  ...

# Kill the process using its PID
kill 12345
```

**Windows:**
```cmd
# Find what's using port 8080
netstat -ano | findstr :8080

# You'll see output with a PID at the end, e.g., 12345
# Kill the process
taskkill /PID 12345 /F
```

**Alternative**: Instead of killing the process, change your app's port in `application.properties`:
```properties
server.port=9090
```

### "JAVA_HOME not set" or "java: command not found"

Spring Boot needs Java installed and the `JAVA_HOME` environment variable set.

```bash
# Check if Java is installed
java -version

# Check if JAVA_HOME is set
echo $JAVA_HOME          # macOS/Linux
echo %JAVA_HOME%         # Windows
```

If `JAVA_HOME` is not set, add it to your shell profile:
- **macOS/Linux**: Add `export JAVA_HOME=$(/usr/libexec/java_home)` to `~/.bashrc` or `~/.zshrc`
- **Windows**: System Properties → Environment Variables → New → `JAVA_HOME` = path to your JDK (e.g., `C:\Program Files\Java\jdk-17`)

### "mvn: command not found"

Maven isn't installed globally on your system. Use the **Maven Wrapper** that comes with your project:

```bash
# macOS / Linux
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

The wrapper (`mvnw` / `mvnw.cmd`) downloads the correct Maven version automatically. You don't need to install Maven separately.

### "Application starts but immediately exits"

If you see the Spring Boot banner but the application shuts down right away (no "Started BookshelfApplication" message), you're probably **missing the `spring-boot-starter-web` dependency**.

Open `pom.xml` and make sure this dependency is present:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Without `spring-boot-starter-web`, there's no embedded Tomcat server, so Spring Boot has nothing to keep running. It starts, finds no web server to launch, and exits.

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

### 📝 Practice Exercises

Ready to test your understanding? These exercises from [Appendix E](../appendices/E-coding-exercises.md) directly apply what you learned in this chapter:

| Exercise | Topic | Difficulty |
|----------|-------|------------|
| [Exercise 11](../appendices/E-coding-exercises.md#exercise-11) | Your First GET Endpoint | ⭐ |
| [Exercise 12](../appendices/E-coding-exercises.md#exercise-12) | Return an Object as JSON | ⭐ |

Solutions are in [Appendix F](../appendices/F-exercise-solutions.md).

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
