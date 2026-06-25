# Chapter 19: Deploying Your Application

> ⏱ Estimated time: 45 minutes

## You Built It. Now You're SHIPPING It.

You've spent days building BookShelf. Endpoints? Working. Database? Hooked up. Validation? Locked down. Tests? Green.

But right now, your app lives on ONE machine. Yours. If you close your laptop lid, *poof* — gone. That's not an application. That's a really expensive demo.

**It's time to set your code free.**

In this chapter, you'll learn how to take everything you've built and package it into a single, self-contained file that can run on ANY machine with Java. No IDE required. No Maven required. No "well, it works on my machine" excuses.

### What You'll Learn

- How to package your Spring Boot app as a runnable JAR
- How to run the JAR on any machine with Java
- How to use environment variables for production configuration
- What Docker is (conceptual) and why it matters
- The path from "it works on my laptop" to "it runs in production"

---

## So... What IS Deployment, Really?

Deployment sounds scary. Enterprise-y. Like something that involves pagers and 3 AM phone calls.

But at its core? **Deployment is just making your application available to real users.** That's it. You've been running on `localhost` this whole time — which means the only person who can use your app is YOU. Deployment puts your app on a server that's accessible over the internet — so *other people* can actually use the thing you built.

Here's the journey, step by step:

```
Your laptop (development)
    ↓ package
JAR file (self-contained executable)
    ↓ deploy
Server (cloud or physical)
    ↓ run
Application accessible at https://your-domain.com
```

🎯 **Key Point:** Deployment = your code, running on someone else's computer, accessible to the world.

---

## Meet the Fat JAR

Spring Boot packages your application as a **"fat JAR"** (also called an "uber JAR"). And no, that's not a body-shaming term — it just means the JAR file is *stuffed* with everything it needs to run. Your code, all your dependencies, even a web server. ONE file. That's it.

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

### 🗣️ Overheard at the Coffee Shop

> *"Wait, Tomcat is INSIDE the JAR? I thought you had to install Tomcat separately and deploy WAR files and configure XML and—"*
>
> *"That's the old way. Spring Boot embeds it. One file. `java -jar`. Done."*
>
> *"...I wasted two weeks of my life configuring standalone Tomcat."*
>
> *"We all did. We all did."*

---

## 🎤 Fireside Chat: Interview with the Fat JAR

**Interviewer:** So, Fat JAR — tell us about yourself. What makes you so special?

**Fat JAR:** I contain EVERYTHING. Tomcat, your code, all dependencies. Just add `java -jar` and I run anywhere. No installation wizards. No dependency hunting. No "did you install the right version of..." nonsense. I'm self-contained. I'm portable. I'm the whole show in a single file.

**Interviewer:** That sounds... kind of arrogant?

**Fat JAR:** Look, I weigh about 30-50 MB. That's nothing. And in return, you get a guarantee: if it runs on your laptop, it runs on the server. Same file. Same behavior. No surprises.

**Interviewer:** What do you say to the traditional WAR file?

**Fat JAR:** *[leans back]* I say: enjoy configuring your external Tomcat instance, managing your deployment descriptors, and debugging why the server's Tomcat version doesn't match what you developed against. I'll be over here... just running. With one command. On any machine.

**Interviewer:** Any weaknesses?

**Fat JAR:** I'm a bit chunky, sure. I pack in libraries you might not even use at runtime. And if you're doing microservices, you might want to look into layered JARs or GraalVM native images down the road. But for getting started? I'm your best friend.

**Interviewer:** Thanks, Fat JAR.

**Fat JAR:** `java -jar me.jar` — anytime.

---

## Docker: The 30-Second Concept

You know that friend who says "it works on MY machine"? Docker exists because of that friend.

**Docker** packages your application + its entire runtime environment into a **container** — a lightweight, isolated environment that runs the same way everywhere.

```
Without Docker:
  "It works on my laptop" → Deploy to server → "It doesn't work"
  (Different Java version? Missing config? OS differences?)

With Docker:
  Build a container image (includes Java, your JAR, everything)
  → Run the same image anywhere → It works the same everywhere
```

Think of a Docker container like a shipping container at a port. It doesn't matter what ship carries it, what truck loads it, or what crane lifts it — the *container* is standardized. Everything inside stays exactly the same.

⚠️ **Watch it!** You don't need Docker for this chapter, and you don't need it to deploy a Spring Boot app. But it IS the industry standard, and it's a natural next step once you're comfortable with the basics. We'll show you a reference Dockerfile at the end, just so it's not mysterious.

---

## Let's Actually DO This

Enough theory. Time to package and run your app. For real.

### Step 1: Build the JAR

```bash
# Clean previous builds and create a new JAR
mvn clean package

# Skip tests during build (faster, use when you've already tested)
mvn clean package -DskipTests
```

The JAR is created at: `target/bookshelf-0.0.1-SNAPSHOT.jar`

🧠 **Brain Power:** Go look at that file. Check its size. It's probably 30-50 MB. That single file contains your entire application, a web server, an ORM, a JSON library, and about 30 other libraries. Think about what it would take to set all of that up manually on a server.

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

That's it. No Tomcat installation. No WAR files. No 47-step deployment guide. **One command.**

🎯 **Key Point:** `java -jar` is ALL you need. The target server only needs Java installed. That's the only prerequisite.

### Step 3: Production Configuration

Here's where things get serious. In development, you hardcoded database URLs and used H2. In production, you use REAL databases with REAL passwords — and those passwords should NEVER be in your source code.

Your `application-prod.properties`:

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

### 💡 There Are No Dumb Questions

**Q: What does `${PORT:8080}` mean?**

A: It means "use the `PORT` environment variable if it exists, otherwise default to 8080." The part after the colon is the fallback value. It's Spring Boot's way of saying "I'll be flexible, but I've got a sensible default."

**Q: Why `ddl-auto=validate` instead of `update` or `create`?**

A: Because `create-drop` DESTROYS your database on every restart. `update` can make unexpected schema changes. `validate` just checks that the database matches your entities — if something's wrong, it fails fast instead of silently mangling your data. In production, you want control. Use a migration tool like Flyway or Liquibase to manage schema changes.

**Q: Why not just put the real password in `application-prod.properties`?**

A: Because properties files get committed to git. And once a password is in git, it's there FOREVER (yes, even if you delete it later — git remembers everything). Environment variables live on the server, not in your codebase. That's the right place for secrets.

**Q: Can I run `mvn spring-boot:run` in production?**

A: You *can*, but please don't. That requires Maven AND your source code on the production server. The whole point of the fat JAR is that you don't need any of that. Just Java and the JAR file. Clean. Simple. Secure.

### Step 4: Verify the JAR Works

Never deploy something you haven't tested locally. Here's the drill:

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

⚠️ **Watch it!** Always build, run, and test the JAR locally BEFORE you put it on a server. "It worked in the IDE" is not the same as "it works from the JAR." Sometimes things get missed in packaging — better to catch it on your machine than at 2 AM in production.

### Basic Dockerfile (Reference)

When you're ready to take the next step and containerize:

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

This is just for reference — Docker is beyond this chapter's scope but is the natural next step.

---

## 🏋️ Exercise: Package and Run BookShelf

**Goal**: Create a standalone JAR and run your application independently. No IDE. No Maven at runtime. Just Java and the JAR.

### Tasks

1. Build the JAR: `mvn clean package`
2. Run the JAR: `java -jar target/bookshelf-0.0.1-SNAPSHOT.jar`
3. Test all endpoints work as before
4. Run with a different port: `--server.port=9090`
5. Run with prod profile: `--spring.profiles.active=prod`
6. Check the file size of the JAR — everything is in there

> 📝 **Solution**: See [Appendix E — Coding Exercises](../../appendices/E-coding-exercises.md)

### 🧠 Brain Power

- How big is the JAR file? What's inside it? (Hint: you can run `jar tf target/bookshelf-0.0.1-SNAPSHOT.jar | head -50` to peek inside.)
- What would you need on the target server to run this? (Spoiler: Just. Java.)
- How does this compare to deploying a Python app (need Python + pip + virtualenv + requirements.txt) or a Node.js app (need Node + npm + node_modules + package.json)?

---

## Common Mistakes (a.k.a. "I Learned This the Hard Way")

| Mistake | Reality |
|---------|---------|
| Deploying with `ddl-auto=create-drop` | This deletes your production database on every restart. Use `validate`. |
| Hardcoding production passwords in properties files | Use environment variables. Properties files get committed to git — passwords shouldn't be in source control. |
| Running `mvn spring-boot:run` in production | Use `java -jar`. `mvn spring-boot:run` requires Maven and the source code on the server. |
| Not verifying the JAR before deploying | Always build and test the JAR locally before deploying to a server. |

🗣️ **Overheard at the Coffee Shop:**

> *"So I deployed with `ddl-auto=create-drop` and..."*
>
> *"Oh no."*
>
> *"...yeah. The CEO's account got deleted. Along with everyone else's."*
>
> *"How are you still employed?"*
>
> *"Backups. Always have backups."*

---

## 🎯 Key Takeaways

These are the big ideas from this chapter. Don't move on until each one clicks:

- [ ] `mvn clean package` creates a self-contained JAR with everything included
- [ ] `java -jar app.jar` runs the application — no Tomcat installation needed
- [ ] Use environment variables for production secrets (database credentials, API keys)
- [ ] Command-line arguments override properties: `--server.port=9090`
- [ ] Docker is the industry standard for deploying Spring Boot apps (learn it next)

---

## Quick Quiz

*Grab a pen. No peeking at the answers until you've tried.*

1. What's inside a "fat JAR"?
2. How do you change the port without editing code?
3. Why shouldn't you commit database passwords to `application-prod.properties`?
4. What's the minimum requirement to run a Spring Boot JAR on a server?
5. What does `${PORT:8080}` mean in a properties file?

🧠 **Brain Power:** If you can answer all five without scrolling up, you've got this chapter locked down. If not, re-read the Fat JAR interview — it covers most of them.

---

*Next: `20-whats-next.md` — Your roadmap for continued learning. You've come a LONG way. Let's talk about where to go from here. →*
