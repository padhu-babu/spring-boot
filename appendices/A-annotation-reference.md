# Appendix A: Spring Boot Annotation Reference

A quick-reference guide to every annotation used in this guide.

---

## Controller Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@RestController` | Marks a class as a REST controller (returns data, not views) | `@RestController public class BookController {}` |
| `@Controller` | Marks a class as a web controller (returns views/HTML) | Rarely used for REST APIs |
| `@RequestMapping("/path")` | Sets base path for all endpoints in the controller | `@RequestMapping("/api/books")` |
| `@GetMapping` | Handles GET requests | `@GetMapping("/{id}")` |
| `@PostMapping` | Handles POST requests | `@PostMapping` |
| `@PutMapping` | Handles PUT requests | `@PutMapping("/{id}")` |
| `@PatchMapping` | Handles PATCH requests | `@PatchMapping("/{id}")` |
| `@DeleteMapping` | Handles DELETE requests | `@DeleteMapping("/{id}")` |

## Request Parameter Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@PathVariable` | Extracts value from URL path | `@PathVariable Long id` → `/books/42` |
| `@RequestParam` | Extracts value from query string | `@RequestParam String title` → `?title=dune` |
| `@RequestBody` | Deserializes JSON body to Java object | `@RequestBody BookRequest request` |
| `@Valid` | Triggers Bean Validation on the parameter | `@Valid @RequestBody BookRequest request` |

## Spring Core / DI Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@SpringBootApplication` | Entry point — combines @Configuration, @EnableAutoConfiguration, @ComponentScan | `@SpringBootApplication public class App {}` |
| `@Component` | Generic Spring-managed bean | `@Component public class MyHelper {}` |
| `@Service` | Service layer bean (business logic) | `@Service public class BookService {}` |
| `@Repository` | Data access layer bean | `@Repository public class BookRepo {}` |
| `@Configuration` | Declares a configuration class with @Bean methods | `@Configuration public class AppConfig {}` |
| `@Bean` | Declares a method that returns a Spring-managed object | `@Bean public PasswordEncoder encoder() {}` |
| `@Autowired` | Injects a dependency (prefer constructor injection) | `@Autowired private BookService service;` |
| `@Value("${prop}")` | Injects a property value | `@Value("${server.port}") int port;` |
| `@ConfigurationProperties` | Binds a group of properties to a class | `@ConfigurationProperties(prefix = "app")` |

## JPA / Database Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Entity` | Marks a class as a JPA entity (maps to a table) | `@Entity public class Book {}` |
| `@Table(name = "...")` | Specifies the database table name | `@Table(name = "books")` |
| `@Id` | Marks the primary key field | `@Id private Long id;` |
| `@GeneratedValue` | Auto-generate the primary key | `@GeneratedValue(strategy = GenerationType.IDENTITY)` |
| `@Column` | Configures a column (nullable, name, length) | `@Column(nullable = false, length = 255)` |
| `@ManyToOne` | Many instances of this entity relate to one of another | `@ManyToOne private Author author;` |
| `@OneToMany` | One instance has many of another entity | `@OneToMany(mappedBy = "author") List<Book> books;` |
| `@JoinColumn` | Specifies the foreign key column | `@JoinColumn(name = "author_id")` |
| `@PrePersist` | Callback before first save | `@PrePersist void onCreate() { createdAt = now(); }` |
| `@Query` | Custom JPQL or SQL query on a repository method | `@Query("SELECT b FROM Book b WHERE b.pages > :min")` |

## Validation Annotations (jakarta.validation)

| Annotation | Purpose | Applies To |
|-----------|---------|-----------|
| `@NotNull` | Must not be null | Any |
| `@NotBlank` | Must not be null, empty, or whitespace | String |
| `@NotEmpty` | Must not be null or empty | String, Collection |
| `@Size(min, max)` | Length/size in range | String, Collection |
| `@Min(value)` | Must be ≥ value | Number |
| `@Max(value)` | Must be ≤ value | Number |
| `@Positive` | Must be > 0 | Number |
| `@Email` | Must be valid email | String |
| `@Pattern(regexp)` | Must match regex | String |

## Error Handling Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@ControllerAdvice` | Global exception handler for all controllers | `@ControllerAdvice public class GlobalExHandler {}` |
| `@ExceptionHandler` | Handles a specific exception type | `@ExceptionHandler(BookNotFoundException.class)` |

## Security Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@EnableWebSecurity` | Enables Spring Security configuration | `@EnableWebSecurity public class SecurityConfig {}` |

## Testing Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Test` | Marks a test method | `@Test void testCreate() {}` |
| `@BeforeEach` | Runs before each test | `@BeforeEach void setUp() {}` |
| `@DisplayName` | Human-readable test name | `@DisplayName("creates a book")` |
| `@SpringBootTest` | Loads full application context for testing | `@SpringBootTest class IntegrationTest {}` |
| `@AutoConfigureMockMvc` | Configures MockMvc for controller tests | Used with `@SpringBootTest` |
| `@Mock` | Creates a Mockito mock | `@Mock BookRepository repo;` |
| `@InjectMocks` | Injects mocks into the class under test | `@InjectMocks BookService service;` |
| `@ExtendWith(MockitoExtension.class)` | Enables Mockito in JUnit 5 | Class-level annotation |

## Swagger / OpenAPI Annotations

| Annotation | Purpose | Example |
|-----------|---------|---------|
| `@Tag` | Groups endpoints in Swagger UI | `@Tag(name = "Books")` |
| `@Operation` | Describes an endpoint | `@Operation(summary = "Get all books")` |
| `@ApiResponse` | Documents a possible response | `@ApiResponse(responseCode = "404")` |
| `@Parameter` | Describes a parameter | `@Parameter(description = "Book ID")` |
