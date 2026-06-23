# Appendix D: Glossary

Terms listed in the order you encounter them in the guide.

---

| Term | Definition | First Used |
|------|-----------|------------|
| **Client** | A device or program that sends requests to a server | Ch 1 |
| **Server** | A computer running a program that listens for and responds to requests | Ch 1 |
| **IP Address** | A unique numerical address identifying a device on a network (e.g., 192.168.1.5) | Ch 1 |
| **Port** | A number identifying a specific program on a computer (e.g., 8080) | Ch 1 |
| **DNS** | Domain Name System — translates domain names (google.com) to IP addresses | Ch 1 |
| **HTTP** | HyperText Transfer Protocol — the language clients and servers use to communicate | Ch 2 |
| **HTTPS** | HTTP over TLS — encrypted HTTP | Ch 2 |
| **HTTP Method** | The action of a request: GET (read), POST (create), PUT (update), DELETE (remove) | Ch 2 |
| **Status Code** | A 3-digit number indicating the result: 200 (OK), 404 (Not Found), 500 (Server Error) | Ch 2 |
| **Header** | Metadata sent with an HTTP request or response (e.g., Content-Type, Authorization) | Ch 2 |
| **Request Body** | Data sent by the client in a POST/PUT request (usually JSON) | Ch 2 |
| **Response Body** | Data sent back by the server (usually JSON) | Ch 2 |
| **Stateless** | Each request is independent — the server remembers nothing between requests | Ch 2 |
| **Frontend** | The user-facing part of an application (browser, mobile app) | Ch 3 |
| **Backend** | The server-side part that handles logic, data, and security | Ch 3 |
| **API** | Application Programming Interface — a contract defining how to interact with a system | Ch 4 |
| **REST** | Representational State Transfer — conventions for designing APIs using HTTP methods and resources | Ch 4 |
| **JSON** | JavaScript Object Notation — a text format for structured data (`{"key": "value"}`) | Ch 4 |
| **CRUD** | Create, Read, Update, Delete — the four basic data operations | Ch 4 |
| **Endpoint** | A specific URL + HTTP method combination that handles a request (e.g., GET /api/books) | Ch 4 |
| **Resource** | A thing your API manages (book, user, order), represented by a URL | Ch 4 |
| **Framework** | Pre-built infrastructure that handles common functionality so you focus on business logic | Ch 5 |
| **Library** | Code you call (you control the flow). Compare with framework (it calls you). | Ch 5 |
| **IoC** | Inversion of Control — the framework controls the flow and calls your code | Ch 5 |
| **Spring** | A comprehensive Java application framework (since 2003) | Ch 5 |
| **Spring Boot** | Spring with auto-configuration and embedded servers — minimal setup | Ch 5 |
| **Maven** | A build tool for Java — manages dependencies, compiles, packages | Ch 6 |
| **pom.xml** | Maven's configuration file listing dependencies and build settings | Ch 6 |
| **Starter** | A Spring Boot dependency bundle (e.g., spring-boot-starter-web includes Tomcat + Spring MVC + Jackson) | Ch 6 |
| **Tomcat** | The embedded web server that Spring Boot uses to handle HTTP | Ch 6 |
| **Controller** | A class that handles HTTP requests, annotated with @RestController | Ch 7 |
| **@RestController** | Annotation marking a class as a REST controller that returns data (not views) | Ch 7 |
| **@GetMapping** | Annotation mapping a method to HTTP GET requests | Ch 7 |
| **@PathVariable** | Annotation extracting a value from the URL path (/books/{id}) | Ch 7 |
| **@RequestParam** | Annotation extracting a value from the query string (?key=value) | Ch 7 |
| **@RequestBody** | Annotation deserializing the JSON request body into a Java object | Ch 7 |
| **Jackson** | The JSON library Spring Boot uses to convert between Java objects and JSON | Ch 7 |
| **Serialization** | Converting a Java object to JSON (for responses) | Ch 7 |
| **Deserialization** | Converting JSON to a Java object (for request bodies) | Ch 7 |
| **Dependency** | An object that another object needs to function | Ch 8 |
| **Dependency Injection (DI)** | Receiving dependencies through the constructor instead of creating them yourself | Ch 8 |
| **Bean** | An object managed by Spring's IoC container | Ch 8 |
| **@Service** | Annotation marking a class as a service-layer bean | Ch 8 |
| **@Repository** | Annotation marking a class as a data-access-layer bean | Ch 8 |
| **@Component** | Generic annotation marking a class as a Spring-managed bean | Ch 8 |
| **Constructor Injection** | Receiving dependencies via the constructor (recommended approach) | Ch 8 |
| **Singleton** | A bean with only one instance, shared across the application (Spring's default) | Ch 8 |
| **DispatcherServlet** | Spring's front controller that routes requests to the correct handler | Ch 9 |
| **ResponseEntity** | A class that lets you control the HTTP status code, headers, and body of a response | Ch 9 |
| **Layered Architecture** | Organizing code into Controller → Service → Repository layers | Ch 10 |
| **Separation of Concerns** | Each layer/class has one responsibility and doesn't mix concerns | Ch 10 |
| **Entity** | A Java class that maps to a database table | Ch 11 |
| **DTO** | Data Transfer Object — a class that carries data between layers (request/response) | Ch 11 |
| **Record** | A Java feature (16+) for creating immutable data classes concisely | Ch 11 |
| **Database** | A program that stores, organizes, and queries data persistently | Ch 12 |
| **SQL** | Structured Query Language — the language for interacting with relational databases | Ch 12 |
| **JPA** | Java Persistence API — a specification for mapping Java objects to database tables | Ch 12 |
| **Hibernate** | The most popular JPA implementation — generates SQL from your Java code | Ch 12 |
| **ORM** | Object-Relational Mapping — the technique of mapping objects to database rows | Ch 12 |
| **H2** | An in-memory Java database, perfect for development and testing | Ch 12 |
| **JpaRepository** | A Spring Data interface providing free CRUD methods for entities | Ch 12 |
| **@Entity** | Annotation marking a class as a JPA entity (maps to a table) | Ch 12 |
| **@Id** | Annotation marking a field as the primary key | Ch 12 |
| **@GeneratedValue** | Annotation telling the database to auto-generate the primary key value | Ch 12 |
| **Bean Validation** | Annotations (@NotBlank, @Min, etc.) that declare validation rules on fields | Ch 13 |
| **@Valid** | Annotation on a controller parameter that triggers validation | Ch 13 |
| **@ControllerAdvice** | A class that handles exceptions globally across all controllers | Ch 13 |
| **@ExceptionHandler** | A method that handles a specific exception type | Ch 13 |
| **Foreign Key** | A column in one table that references the primary key of another table | Ch 14 |
| **@ManyToOne** | Annotation: many instances of this entity relate to one of another | Ch 14 |
| **@OneToMany** | Annotation: one instance has many of another entity | Ch 14 |
| **Lazy Loading** | Loading related data only when accessed (not upfront) | Ch 14 |
| **Eager Loading** | Loading related data immediately when the parent is loaded | Ch 14 |
| **N+1 Problem** | Performance issue: 1 query for parents + N queries for children | Ch 14 |
| **JPQL** | Java Persistence Query Language — SQL-like queries using entity names instead of table names | Ch 14 |
| **Profile** | A named configuration set (dev, prod, test) for environment-specific settings | Ch 15 |
| **@ConfigurationProperties** | Annotation binding a group of properties to a Java class | Ch 15 |
| **Environment Variable** | A value set outside the application (on the OS/server) for runtime configuration | Ch 15 |
| **Unit Test** | A test that verifies one class in isolation, using mocks for dependencies | Ch 16 |
| **Integration Test** | A test that verifies multiple components working together | Ch 16 |
| **Mock** | A fake object that simulates a real dependency for testing | Ch 16 |
| **MockMvc** | A Spring tool that simulates HTTP requests without starting a real server | Ch 16 |
| **JUnit 5** | Java's standard testing framework | Ch 16 |
| **Mockito** | Java's most popular mocking library | Ch 16 |
| **SLF4J** | Simple Logging Facade for Java — the logging API Spring Boot uses | Ch 17 |
| **Logback** | The default logging implementation behind SLF4J in Spring Boot | Ch 17 |
| **Log Level** | Priority of a log message: ERROR > WARN > INFO > DEBUG > TRACE | Ch 17 |
| **Actuator** | Spring Boot module providing operational endpoints (/health, /metrics) | Ch 17 |
| **Swagger / OpenAPI** | A specification and UI for interactive API documentation | Ch 17 |
| **Authentication** | Verifying WHO a user is (identity) | Ch 18 |
| **Authorization** | Verifying WHAT a user is allowed to do (permissions) | Ch 18 |
| **Spring Security** | Spring's security framework for authentication and authorization | Ch 18 |
| **Filter Chain** | A sequence of security checks that run before your controller | Ch 18 |
| **Basic Auth** | Authentication by sending username:password (Base64 encoded) in every request | Ch 18 |
| **BCrypt** | A password hashing algorithm — never store plain-text passwords | Ch 18 |
| **CORS** | Cross-Origin Resource Sharing — browser security that restricts cross-domain requests | Ch 18 |
| **JAR** | Java ARchive — a packaged Java application (zip file with .jar extension) | Ch 19 |
| **Fat JAR** | A JAR containing your code + all dependencies + embedded server | Ch 19 |
| **Docker** | A platform for packaging applications with their runtime environment into containers | Ch 19 |
| **Container** | A lightweight, isolated environment for running an application | Ch 19 |
| **JWT** | JSON Web Token — a token-based authentication method for stateless APIs | Ch 20 |
| **Microservices** | Architecture style: many small, independent services instead of one large application | Ch 20 |
