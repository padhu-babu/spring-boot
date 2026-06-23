# Chapter 18: Security Basics

> ⏱ Estimated time: 60 minutes

## What You'll Learn

- The difference between authentication and authorization
- How Spring Security works at a high level
- How to add basic authentication to your API
- How to protect specific endpoints (write operations) while leaving others open (read operations)
- What CORS is and why it matters

---

## Concepts

### Authentication vs. Authorization

Two different questions:

| Concept | Question | Example |
|---------|----------|---------|
| **Authentication** | **Who are you?** | "I'm Alice, here's my password" |
| **Authorization** | **What are you allowed to do?** | "Alice has the ADMIN role, so she can delete books" |

Authentication comes first. You must know WHO someone is before you can decide WHAT they're allowed to do.

```
Request arrives → Is the user authenticated? → NO → 401 Unauthorized
                                             → YES → Is the user authorized? → NO → 403 Forbidden
                                                                              → YES → Process request
```

**Analogy**: A building security desk.
- **Authentication**: Showing your ID badge (proving who you are)
- **Authorization**: Your badge only opens doors to floors 1-3, not the executive suite (what you're allowed to access)

### Spring Security: The Filter Chain

Spring Security works by inserting **filters** into the request pipeline, before your controller is reached:

```
HTTP Request
    ↓
┌──────────────────────┐
│ Spring Security      │
│ Filter Chain         │
│                      │
│ 1. Is this request   │
│    to a secured URL? │
│ 2. Does it have      │
│    credentials?      │
│ 3. Are credentials   │
│    valid?            │
│ 4. Does the user     │
│    have the right    │
│    role?             │
└──────────┬───────────┘
           ↓
     Your Controller
     (only reached if all checks pass)
```

### Authentication Methods

| Method | How It Works | Use Case |
|--------|-------------|----------|
| **Basic Auth** | Username + password in every request (Base64 encoded in header) | Simple APIs, internal tools |
| **Session-based** | Login once, server stores session, client sends session cookie | Traditional web apps |
| **Token-based (JWT)** | Login once, receive a token, send token in every request | Modern APIs, mobile apps |
| **OAuth 2.0** | Delegated auth via a provider (Google, GitHub) | "Sign in with Google" |

We'll use **Basic Auth** for learning (simplest to understand). The concepts apply to all methods.

---

## Code Examples

### Step 1: Add Spring Security Dependency

Add to `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Warning**: The moment you add this dependency, **ALL endpoints are locked down**. Every request gets a 401 until you configure security. This is Spring Security's "secure by default" philosophy.

If you run the app now, check the console for a generated password:
```
Using generated security password: a1b2c3d4-e5f6-7890-abcd-ef1234567890
```
The default username is `user`.

### Step 2: Create Security Configuration

Create `src/main/java/com/bookshelf/config/SecurityConfig.java`:

```java
package com.bookshelf.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

import static org.springframework.security.config.Customizer.withDefaults;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF (not needed for stateless REST APIs)
            .csrf(csrf -> csrf.disable())
            
            // Stateless session (no cookies)
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            
            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                // Public endpoints — no authentication needed
                .requestMatchers(HttpMethod.GET, "/api/books/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/authors/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .requestMatchers("/h2-console/**").permitAll()
                
                // Protected endpoints — authentication required
                .requestMatchers(HttpMethod.POST, "/api/**").authenticated()
                .requestMatchers(HttpMethod.PUT, "/api/**").authenticated()
                .requestMatchers(HttpMethod.DELETE, "/api/**").authenticated()
                
                // Everything else requires authentication
                .anyRequest().authenticated()
            )
            
            // Use HTTP Basic authentication
            .httpBasic(withDefaults())
            
            // Allow H2 console frames
            .headers(headers -> headers.frameOptions(frame -> frame.disable()));
        
        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService(PasswordEncoder encoder) {
        // In-memory users for learning — in production, use a database
        var admin = User.builder()
                .username("admin")
                .password(encoder.encode("admin123"))
                .roles("ADMIN")
                .build();

        var user = User.builder()
                .username("user")
                .password(encoder.encode("user123"))
                .roles("USER")
                .build();

        return new InMemoryUserDetailsManager(admin, user);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### What This Configuration Does

```
GET  /api/books          → Public (anyone can read)
GET  /api/books/1        → Public
GET  /api/authors        → Public
POST /api/books          → Requires authentication (must be logged in)
PUT  /api/books/1        → Requires authentication
DELETE /api/books/1      → Requires authentication
GET  /swagger-ui/**      → Public (documentation is open)
GET  /actuator/health    → Public (health check is open)
```

### Step 3: Testing Secured Endpoints

```bash
# Public endpoint — works without credentials
curl http://localhost:8080/api/books
# Response: [] (200 OK)

# Protected endpoint — without credentials
curl -v -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "authorId": 1, "pages": 412}'
# Response: 401 Unauthorized

# Protected endpoint — with credentials (Basic Auth)
curl -u admin:admin123 -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "authorId": 1, "pages": 412}'
# Response: 201 Created

# Wrong credentials
curl -u admin:wrongpassword -X POST http://localhost:8080/api/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Dune", "authorId": 1, "pages": 412}'
# Response: 401 Unauthorized
```

The `-u username:password` flag in curl sends Basic Auth credentials.

### Password Encoding

**NEVER store plain-text passwords.** Always hash them:

```java
BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String hashed = encoder.encode("admin123");
// Result: "$2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy"
// This is a one-way hash — you cannot reverse it to get "admin123"
```

When a user logs in, Spring Security:
1. Takes the provided password
2. Hashes it with BCrypt
3. Compares the hash with the stored hash
4. If they match → authenticated

### CORS (Cross-Origin Resource Sharing)

If your API is at `api.bookshelf.com` and your frontend is at `www.bookshelf.com`, the browser blocks the request by default. This is a security feature — browsers don't let JavaScript make requests to different domains.

**CORS** tells the browser: "It's okay, this frontend is allowed to call my API."

Add CORS configuration:

```java
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.CorsConfigurationSource;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(List.of("http://localhost:3000"));  // Frontend URL
    configuration.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE"));
    configuration.setAllowedHeaders(List.of("*"));
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", configuration);
    return source;
}
```

And enable it in the filter chain:
```java
http.cors(withDefaults())  // Add this to your HttpSecurity configuration
```

---

## Exercise: Secure BookShelf (v5)

**Goal**: Add authentication to protect write operations.

### Tasks

1. Add `spring-boot-starter-security` dependency
2. Create `SecurityConfig` with the rules above
3. Define at least two users (admin and regular user)
4. Configure public GET endpoints and protected POST/PUT/DELETE endpoints
5. Test that GET works without credentials
6. Test that POST/PUT/DELETE requires credentials
7. Test that wrong credentials return 401

### Test Matrix

| Endpoint | No Auth | With Auth | Wrong Auth |
|----------|---------|-----------|------------|
| GET /api/books | 200 ✅ | 200 ✅ | 200 ✅ |
| POST /api/books | 401 ❌ | 201 ✅ | 401 ❌ |
| PUT /api/books/1 | 401 ❌ | 200 ✅ | 401 ❌ |
| DELETE /api/books/1 | 401 ❌ | 204 ✅ | 401 ❌ |

---

## Common Mistakes

| Mistake | Reality |
|---------|---------|
| Storing plain-text passwords | ALWAYS hash passwords with BCrypt (or Argon2). Even in development. Build the habit now. |
| Disabling security "because it's annoying" | Instead, configure it properly. Security is part of your application, not an afterthought. |
| Forgetting to disable CSRF for REST APIs | CSRF protection is for browser forms. Stateless REST APIs with token/basic auth don't need it. |
| Not allowing Swagger through the filter | If Swagger UI returns 401, add `.requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()` |
| `permitAll()` on all endpoints during development | Fix security rules from day one. "I'll add security later" means "I'll forget and ship insecure code." |

---

## Key Takeaways

- [ ] Authentication = who are you? Authorization = what can you do?
- [ ] Spring Security uses a filter chain that runs before your controller
- [ ] Adding `spring-boot-starter-security` locks down ALL endpoints by default
- [ ] Configure which endpoints are public (`permitAll()`) and which require auth (`authenticated()`)
- [ ] Always hash passwords — never store them in plain text
- [ ] CORS is needed when your frontend and backend are on different domains

---

## Quick Quiz

1. What's the difference between a 401 and a 403 response?
2. Why do we disable CSRF for REST APIs?
3. What does `BCryptPasswordEncoder.encode("password")` return?
4. If a user's GET /api/books works but POST /api/books returns 401, what does that tell you about the security configuration?
5. Why is CORS a browser restriction, not a server restriction?

---

## Day 6 Summary

```
✓ Unit tests verify services in isolation using mocks
✓ Integration tests verify the full request-response cycle with MockMvc
✓ Good tests verify behavior (output, state), not implementation (method calls)
✓ Logging with SLF4J: ERROR > WARN > INFO > DEBUG levels
✓ Actuator provides /health and monitoring endpoints
✓ springdoc-openapi generates interactive API documentation
✓ Authentication (who?) and Authorization (what?) are different concepts
✓ Spring Security filter chain runs before your controller
✓ Always hash passwords, never store plain text
```

Tomorrow is the final day — you'll package your application and look at what comes next!

---

*Next: `day-7/19-deploying-your-app.md` — Ship it! →*
