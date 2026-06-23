# Appendix C: Maven Cheat Sheet

## What Is Maven?

Maven is a build tool for Java projects. It manages dependencies, compiles code, runs tests, and packages your application.

## Essential Commands

| Command | What It Does |
|---------|-------------|
| `mvn spring-boot:run` | Run the application (development mode) |
| `mvn clean package` | Clean previous builds + compile + test + create JAR |
| `mvn clean package -DskipTests` | Same but skip running tests (faster) |
| `mvn test` | Run all tests |
| `mvn test -Dtest=BookServiceTest` | Run a specific test class |
| `mvn clean compile` | Clean and compile (no tests, no JAR) |
| `mvn dependency:tree` | Show all dependencies and their versions |
| `mvn clean` | Delete the `target/` directory |

## Project Structure

```
my-project/
├── pom.xml                     ← Maven configuration file
├── src/
│   ├── main/
│   │   ├── java/               ← Your application code
│   │   └── resources/          ← Configuration files (application.properties)
│   └── test/
│       ├── java/               ← Your test code
│       └── resources/          ← Test-specific configuration
├── target/                     ← Built artifacts (created by Maven)
│   ├── classes/                ← Compiled .class files
│   └── myapp-0.0.1.jar        ← Packaged application
└── mvnw / mvnw.cmd            ← Maven wrapper (run Maven without installing it)
```

## pom.xml Essentials

```xml
<project>
    <!-- Inherit Spring Boot defaults -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
    </parent>

    <!-- Your project identity -->
    <groupId>com.bookshelf</groupId>     <!-- Organization/company -->
    <artifactId>bookshelf</artifactId>    <!-- Project name -->
    <version>0.0.1-SNAPSHOT</version>     <!-- Version -->

    <!-- Java version -->
    <properties>
        <java.version>17</java.version>
    </properties>

    <!-- Dependencies -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- No <version> needed — inherited from parent -->
        </dependency>
    </dependencies>
</project>
```

## Common Spring Boot Starters

| Starter | What It Includes |
|---------|-----------------|
| `spring-boot-starter-web` | Tomcat, Spring MVC, Jackson (JSON) |
| `spring-boot-starter-data-jpa` | Hibernate, JPA, Spring Data |
| `spring-boot-starter-security` | Spring Security, authentication/authorization |
| `spring-boot-starter-validation` | Bean Validation (JSR 380) |
| `spring-boot-starter-test` | JUnit 5, Mockito, Spring Test, MockMvc |
| `spring-boot-starter-actuator` | Health checks, metrics, monitoring |

## Adding a Dependency

Add inside the `<dependencies>` section of `pom.xml`:

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>    <!-- Only needed at runtime, not compile time -->
</dependency>
```

### Dependency Scopes

| Scope | When Available | Example |
|-------|---------------|---------|
| `compile` (default) | Compile + runtime + test | Spring Web |
| `runtime` | Runtime + test only | H2 database driver |
| `test` | Test only | JUnit, Mockito |
| `provided` | Compile only (server provides at runtime) | Servlet API |

## Maven Wrapper (mvnw)

Spring Initializr includes `mvnw` (Unix) and `mvnw.cmd` (Windows). These let you run Maven without installing it:

```bash
./mvnw spring-boot:run      # Unix/macOS
mvnw.cmd spring-boot:run    # Windows
```

The wrapper downloads the correct Maven version automatically.

## Troubleshooting

| Problem | Solution |
|---------|---------|
| "Cannot resolve dependency" | Run `mvn clean install` or check your internet connection |
| "Plugin not found" | Make sure `spring-boot-maven-plugin` is in `<build><plugins>` |
| "Target directory is stale" | Run `mvn clean` to delete and rebuild |
| "Tests fail during package" | Fix the tests, or use `-DskipTests` temporarily |
| "Java version mismatch" | Ensure `<java.version>` in pom.xml matches your installed Java |
