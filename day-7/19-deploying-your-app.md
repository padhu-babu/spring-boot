# Chapter 19: Deploying Your Application

> ⏱ Estimated time: 45 minutes

## What You'll Learn

- How to package your Spring Boot app as a runnable JAR
- How to run the JAR on any machine with Java
- How to use environment variables for production configuration
- What Docker is (conceptual) and why it matters
- The path from "it works on my laptop" to "it runs in production"

---

## Concepts

### What Is Deployment?

Deployment is making your application available for real users. Until now, you've been running on `localhost` — only you can access it. Deployment puts your application on a server that's accessible over the internet.

The deployment journey:
```
Your laptop (development)
    ↓ package
JAR file (self-contained executable)
    ↓ deploy
Server (cloud or physical)
    ↓ run
Application accessible at https://your-domain.com
```

### The Fat JAR

Spring Boot packages your application as a **"fat JAR"** (also called an "uber JAR") — a single `.jar` file that contains:
- Your compiled code
- All your dependencies (Spring, Jackson, Hibernate, H2, etc.)
- An embedded Tomcat web server
- Configuration files

One file. Everything needed. Run it anywhere that has Java.

```
bookshelf-0.0.1-SNAPSHOT.jar
├── Your classes (com/bookshelf/...)
├── Spring Boot framework
├── Tomcat web server
├── Jackson JSON library
├── Hibernate ORM
├── H2 database driver
├── application.properties
└── ~30 other libraries
```

### Docker (Conceptual Overview)

**Docker** packages your application + its entire runtime environment into a **container** — a lightweight, isolated environment that runs the same way everywhere.

```
Without Docker:
  "It works on my laptop" → Deploy to server → "It doesn't work"
  (Different Java version? Missing config? OS differences?)

With Docker:
  Build a container image (includes Java, your JAR, everything)
  → Run the same image anywhere → It works the same everywhere
```

Think of a Docker container as a shipping container — standardized, sealed, works with any ship (server).

You don't need Docker for this guide, but it's the industry standard for deploying Spring Boot apps. It's a natural next step after you're comfortable with the basics.

---

## Code Examples

### Step 1: Build the JAR

```bash
# Clean previous builds and create a new JAR
mvn clean package

# Skip tests during build (faster, use when you've already tested)
mvn clean package -DskipTests
```

The JAR is created at: `target/bookshelf-0.0.1-SNAPSHOT.jar`

### Step 2: Run the JAR

```bash
# Run with default settings
java -jar target/bookshelf-0.0.1-SNAPSHOT.jar

# Run with a specific profile
java -jar target/bookshelf-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod

# Run on a different port
java -jar target/bookshelf-0.0.1-SNAPSHOT.jar --server.port=9090

# Run with environment variables
DATABASE_URL=jdbc:postgresql://db-server:5432/bookshelf \
DATABASE_USERNAME=myuser \
DATABASE_PASSWORD=mypassword \
java -jar target/bookshelf-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

That's it. No Tomcat installation, no WAR files, no complex setup. One command.

### Step 3: Production Configuration

In production, you override settings via environment variables or command-line arguments. Your `application-prod.properties`:

```properties
# These use environment variables — set on the server, not in code
spring.datasource.url=${DATABASE_URL}
spring.datasource.username=${DATABASE_USERNAME}
spring.datasource.password=${DATABASE_PASSWORD}

spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false

server.port=${PORT:8080}

logging.level.com.bookshelf=INFO
logging.level.org.springframework=WARN
```

`${PORT:8080}` means "use the PORT environment variable, or default to 8080 if not set."

### Step 4: Verify the JAR Works

```bash
# Build
mvn clean package

# Run (in the background with &, or in a separate terminal)
java -jar target/bookshelf-0.0.1-SNAPSHOT.jar &

# Test
curl http://localhost:8080/api/books
curl http://localhost:8080/actuator/health

# Stop (find the process ID and kill it)
# On macOS/Linux:
kill $(lsof -t -i:8080)
```

### Basic Dockerfile (Reference)

If you want to containerize later:

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/bookshelf-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build and run:
```bash
docker build -t bookshelf .
docker run -p 8080:8080 bookshelf
```

This is just for reference — Docker is beyond this guide's scope but is the natural next step.

---

## Exercise: Package and Run BookShelf

**Goal**: Create a standalone JAR and run your application independently.

### Tasks

1. Build the JAR: `mvn clean package`
2. Run the JAR: `java -jar target/bookshelf-0.0.1-SNAPSHOT.jar`
3. Test all endpoints work as before
4. Run with a different port: `--server.port=9090`
5. Run with prod profile: `--spring.profiles.active=prod`
6. Check the file size of the JAR — everything is in there

### Reflection

- How big is the JAR file? What's inside it?
- What would you need on the target server to run this? (Just Java!)
- How does this compare to deploying a Python or Node.js application?

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Deploying with `ddl-auto=create-drop` | This deletes your production database on every restart. Use `validate`. |
| Hardcoding production passwords in properties files | Use environment variables. Properties files get committed to git — passwords shouldn't be in source control. |
| Running `mvn spring-boot:run` in production | Use `java -jar`. `mvn spring-boot:run` requires Maven and the source code on the server. |
| Not verifying the JAR before deploying | Always build and test the JAR locally before deploying to a server. |

---

## Key Takeaways

- [ ] `mvn clean package` creates a self-contained JAR with everything included
- [ ] `java -jar app.jar` runs the application — no Tomcat installation needed
- [ ] Use environment variables for production secrets (database credentials, API keys)
- [ ] Command-line arguments override properties: `--server.port=9090`
- [ ] Docker is the industry standard for deploying Spring Boot apps (learn it next)

---

## Quick Quiz

1. What's inside a "fat JAR"?
2. How do you change the port without editing code?
3. Why shouldn't you commit database passwords to `application-prod.properties`?
4. What's the minimum requirement to run a Spring Boot JAR on a server?
5. What does `${PORT:8080}` mean in a properties file?

---

*Next: `20-whats-next.md` — Your roadmap for continued learning →*
