سأجهز الملف Markdown كاملاً وأعطيك أوامر PowerShell لوضعه في المشروع — لا أستطيع تعديل الملفات مباشرة هنا، لكن الأوامر أدناه ستنشئ الملف في المشروع على جهازك.

Checklist (what I'll deliver)
- [x] Full, very-detailed `PROJECT_DEEP_DOCUMENTATION.md` content (meets requested structure).
- [x] Exact path suggestion where to place the file in your project.
- [x] PowerShell commands you can run to create the file (UTF-8).
- [x] Quick verification steps to open/inspect the file.

Where to place the file
- Recommended path (project root for this module):
  `C:\Users\aligh\IdeaProjects\spring-boot-rest-apis\section-06-spring-boot-rest-project-4\04-todos\PROJECT_DEEP_DOCUMENTATION.md`

How to create the file (PowerShell)
- Option A — using a here-string piped to Out-File (preferred):
```powershell
@'
<PASTE_MARKDOWN_CONTENT_HERE>
'@ | Out-File -FilePath 'C:\Users\aligh\IdeaProjects\spring-boot-rest-apis\section-06-spring-boot-rest-project-4\04-todos\PROJECT_DEEP_DOCUMENTATION.md' -Encoding utf8
```

- Option B — using Set-Content with a here-string variable:
```powershell
$md = @'
<PASTE_MARKDOWN_CONTENT_HERE>
'@

Set-Content -Path 'C:\Users\aligh\IdeaProjects\spring-boot-rest-apis\section-06-spring-boot-rest-project-4\04-todos\PROJECT_DEEP_DOCUMENTATION.md' -Value $md -Encoding UTF8
```

Paste the full Markdown content below in place of `<PASTE_MARKDOWN_CONTENT_HERE>` when running either command.

You can also create the file inside IntelliJ IDEA: right-click the `04-todos` folder → New → File → name it `PROJECT_DEEP_DOCUMENTATION.md`, then paste the content.

PROJECT_DEEP_DOCUMENTATION.md
------------------------------
(Paste everything below as the file content)

```markdown
# PROJECT_DEEP_DOCUMENTATION.md

Last updated: 2026-05-01

This document explains, in exhaustive detail, the entire Spring Boot project located in:
`section-06-spring-boot-rest-project-4/04-todos`

It is written for a backend developer joining the team — it explains architecture, every package and class, request flows, security, transactions, and how to run and test the application.

---

1️⃣ Project Overview
--------------------

What this project does
- A REST API for managing TODO items with user accounts and role-based access control.
- Users can:
  - Register and log in (JWT-based authentication).
  - Create, list, toggle completion, and delete their own todos.
  - Update password and delete their own account (with business rules to prevent last-admin deletion).
- Admins (users with `ROLE_ADMIN`) can:
  - List all users.
  - Promote a user to admin.
  - Delete non-admin users.

Type of application
- A stateless REST API built with Spring Boot (web MVC).

Technologies used
- Java (pom uses Java 25)
- Spring Boot (Web MVC, Security, Data JPA)
- Spring Security (stateless JWT authentication)
- Spring Data JPA (repositories)
- Hibernate (JPA provider)
- MySQL (persistence)
- JJWT (io.jsonwebtoken) for JWT handling
- Springdoc OpenAPI (Swagger UI)
- Jakarta Validation (request validation)
- Maven (build)

High-level architecture (ASCII)
- Simplified:
```
Client → DispatcherServlet → Security Filter Chain (JwtAuthenticationFilter) → Controller → Service → Repository → Database
                     ↑
                   SecurityConfig
```

- Expanded:
```
Client (Postman/Browser)
  ↓ HTTP Request (Authorization: Bearer <token> for protected)
Embedded Servlet Container → DispatcherServlet
  ↓
Security Filter Chain:
  - JwtAuthenticationFilter
  - Spring Security filters
  - AuthenticationEntryPoint (for unauthorized)
  - Session policy: STATELESS
  ↓
Controller Layer (REST endpoints)
  ↓
Service Layer (business logic, transactions)
  ↓
Repository Layer (Spring Data JPA)
  ↓
JPA (Hibernate) → MySQL
```

---

2️⃣ Project Structure Walkthrough
--------------------------------

Top-level structure of the module `04-todos` (relevant folders & files):
- `pom.xml`
- `src/main/java/com/luv2code/springboot/todos/`
  - `TodosApplication.java`
  - `config/`
  - `controller/`
  - `entity/`
  - `exception/`
  - `repository/`
  - `request/`
  - `response/`
  - `service/`
  - `util/`
- `src/main/resources/application.properties`
- `src/test/...` (tests)

Detailed package responsibilities, classes and how they communicate:

- `controller` (`com.luv2code.springboot.todos.controller`)
  - Purpose: HTTP endpoints. Validate inputs (Jakarta Validation) and delegate to the `service` layer.
  - Files:
    - `AuthenticationController.java` — registration & login
    - `UserController.java` — current user info, delete account, change password
    - `TodoController.java` — CRUD-like endpoints for todos for signed-in user
    - `AdminController.java` — admin-only user management
  - Interaction:
    - Controllers accept `request` DTOs and return `response` DTOs.
    - Controllers call `service` beans injected via constructor.

- `service` (`com.luv2code.springboot.todos.service`)
  - Purpose: Business logic and transaction management.
  - Interfaces:
    - `AuthenticationService`, `JwtService`, `UserService`, `TodoService`, `AdminService`
  - Implementations:
    - `AuthenticationServiceImpl`, `JwtServiceImpl`, `UserServiceImpl`, `TodoServiceImpl`, `AdminServiceImpl`
  - Responsibilities:
    - Validate business rules.
    - Use `repository` layer for persistence.
    - Convert entities to response DTOs.
    - Use `FindAuthenticatedUser` utility to get the current authenticated `User`.

- `repository` (`com.luv2code.springboot.todos.repository`)
  - Purpose: Data access using Spring Data.
  - Files:
    - `UserRepository.java` — `findByEmail`, `countAdminUsers` (custom query).
    - `TodoRepository.java` — `findByOwner`, `findByIdAndOwner`.
  - Spring Data auto-generates implementations at runtime.

- `entity` (`com.luv2code.springboot.todos.entity`)
  - Purpose: JPA entities mapping domain model to DB.
  - Files:
    - `User.java` — `@Entity` (table `users`), implements `UserDetails` for Spring Security.
    - `Authority.java` — `@Embeddable`, models an authority string (role).
    - `Todo.java` — `@Entity` (table `todos`), many-to-one owner -> User.
  - Notes:
    - `User.authorities` stored as `@ElementCollection` in `user_authorities`.
    - `User.todos` is `@OneToMany` mapped by `owner`, cascade all and orphan removal.

- `request` (`com.luv2code.springboot.todos.request`)
  - DTOs for incoming requests with validation:
    - `RegisterRequest`, `AuthenticationRequest`, `PasswordUpdateRequest`, `TodoRequest`

- `response` (`com.luv2code.springboot.todos.response`)
  - DTOs for outgoing responses:
    - `AuthenticationResponse`, `UserResponse`, `TodoResponse`

- `config` (`com.luv2code.springboot.todos.config`)
  - Purpose: security configuration, JWT filter, swagger definition
  - Files:
    - `SecurityConfig.java` — configures security filter chain, beans, session policy, allows swagger and auth endpoints to be public.
    - `JwtAuthenticationFilter.java` — validates JWT per request and sets authentication in SecurityContext.
    - `SwaggerConfig.java` — OpenAPI annotations.

- `util` (`com.luv2code.springboot.todos.util`)
  - `FindAuthenticatedUser.java` — interface
  - `FindAuthenticatedUserImpl.java` — component that obtains the current authenticated `User` from `SecurityContextHolder` and throws `AccessDeniedException` when unauthenticated.

- `exception` (`com.luv2code.springboot.todos.exception`)
  - `ExceptionHandlers.java` — `@ControllerAdvice` mapping exceptions to `ExceptionResponses`.
  - `ExceptionResponses.java` — POJO for structured error responses (status, message, timeStamp).

Why these packages exist
- Clear separation of concerns:
  - Controllers handle HTTP, Services handle logic/transactions, Repositories handle persistence, Entities map to DB, Config sets up security and other beans, Util handles repeated concerns (getting current user), Exception centralizes error responses.

---

3️⃣ Application Flow (VERY IMPORTANT)
-------------------------------------

This section documents the full lifecycle of requests in great detail.

Common components participating in the request flow:
- `DispatcherServlet` — Spring's front controller that dispatches to controllers after filters.
- Spring Security filters — configured via `SecurityConfig`.
- `JwtAuthenticationFilter` — custom filter that authenticates requests using JWT.
- `Controller` classes — map endpoints to service methods.
- `Service` classes — business logic, transactions, repository usage.
- `Repository` classes — Spring Data JPA repositories that talk to DB.
- `Hibernate/JPA` — ORM translating entity operations to SQL.
- `MySQL` — persistence store.

A. Authenticated request (e.g., GET /api/todos)
1. Client sends HTTP request with `Authorization: Bearer <token>`.
2. Request hits the embedded servlet container and reaches `DispatcherServlet`.
3. The Servlet Filter chain executes:
   - Spring Security filters run; `JwtAuthenticationFilter` (registered before the `UsernamePasswordAuthenticationFilter`) executes.
4. `JwtAuthenticationFilter#doFilterInternal`:
   - Reads `Authorization` header.
   - If missing or not `Bearer `, does not authenticate — proceeds.
   - Else extracts JWT and asks `JwtService.extractUsername(jwt)`.
   - If username present and `SecurityContext` has no authentication:
     - Uses `UserDetailsService` (bean in `SecurityConfig`) to load `UserDetails` for that username (calls `UserRepository.findByEmail`).
     - Calls `jwtService.isTokenValid(jwt, userDetails)` — verifies token signature and expiration and username match.
     - If valid, constructs `UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities())` and sets it into `SecurityContextHolder`.
5. Authorization:
   - Based on `SecurityConfig`:
     - `/api/auth/**` and swagger endpoints are permitAll.
     - `/api/admin/**` requires `ROLE_ADMIN`.
     - Other routes require authenticated user.
   - If access not allowed, the request is rejected (401/403) by Spring Security.
6. Controller method invoked (if authorized).
7. Controller delegates to Service layer.
8. Service calls `FindAuthenticatedUser.getAuthenticatedUser()` to get the `User` entity from the authentication principal.
9. Service calls repository methods (for queries/updates). Repositories use JPA/Hibernate to execute SQL against MySQL.
10. Service returns DTO(s) to controller.
11. Controller returns DTO(s) to the client (Spring converts to JSON).
12. Response sent back to client.

B. Login request (POST /api/auth/login)
1. Client posts `{email, password}` to `/api/auth/login`.
2. Filter chain runs (no JWT required), request reaches `AuthenticationController.login`.
3. Controller calls `authenticationService.login(authRequest)`.
4. `AuthenticationServiceImpl.login`:
   - Calls `authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(email, password))`.
   - `AuthenticationManager` uses `UserDetailsService` to load the `User` (which implements `UserDetails`).
   - `DaoAuthenticationProvider` checks the password via `PasswordEncoder.matches`.
   - If authentication succeeds, `AuthenticationServiceImpl` obtains user entity and calls `jwtService.generateToken()` to create a JWT and returns it.
5. Controller returns `AuthenticationResponse` with the token.

C. Registration (POST /api/auth/register)
1. Controller receives `RegisterRequest` (validated).
2. Calls `authenticationService.register`.
3. `AuthenticationServiceImpl.register` checks if email is taken (via `userRepository.findByEmail`), builds `User` with roles:
   - Every new user gets `ROLE_EMPLOYEE`.
   - If `userRepository.count() == 0` (first user ever), also adds `ROLE_ADMIN`.
4. Password encoded via `passwordEncoder.encode`.
5. Save to DB via `userRepository.save`.

Transaction management
- Services use `@Transactional` on write operations; read-only operations may use `@Transactional(readOnly=true)`.

---

4️⃣ Controllers Deep Dive
-------------------------

We go controller-by-controller and endpoint-by-endpoint. For each method: URL, HTTP method, request body, response, called service, and internal behavior.

A. `AuthenticationController` (`/api/auth`)
- `register` — POST `/api/auth/register`
  - Request: `RegisterRequest` (`firstName`, `lastName`, `email`, `password`) validated.
  - Response: HTTP 201 CREATED, empty body.
  - Service: `authenticationService.register(registerRequest)`
  - Details:
    - Checks duplicated email, constructs `User`, sets hashed password and initial authorities, persists user.
    - If email exists throws `Exception("Email already taken")` → global exception handler maps to 400.

- `login` — POST `/api/auth/login`
  - Request: `AuthenticationRequest` (`email`, `password`) validated.
  - Response: `AuthenticationResponse` JSON `{ "token": "<jwt>" }` with HTTP 200 OK.
  - Service: `authenticationService.login(authRequest)`
  - Details:
    - Authenticate credentials via `AuthenticationManager`.
    - If successful, generate JWT and return.

B. `UserController` (`/api/users`)
- `getUserInfo` — GET `/api/users/info`
  - Response: `UserResponse` `{ id, fullName, email, authorities }`, HTTP 200.
  - Service: `userService.getUserInfo()`
  - Details:
    - `UserServiceImpl.getUserInfo` uses `FindAuthenticatedUser` to get `User` entity and returns `UserResponse`.

- `deleteUser` — DELETE `/api/users`
  - Response: HTTP 200 OK or 403 if last admin.
  - Service: `userService.deleteUser()`
  - Details:
    - Prevent deletion of the last admin via `userRepository.countAdminUsers()`.
    - If allowed, `userRepository.delete(user)` — cascades to todos due to `User.todos` cascade configuration.

- `passwordUpdate` — PUT `/api/users/password`
  - Request: `PasswordUpdateRequest` (`oldPassword`, `newPassword`, `newPassword2`) validated.
  - Response: HTTP 200 OK if success.
  - Service: `userService.updatePassword(passwordUpdateRequest)`
  - Details:
    - Verify old password matches (via `passwordEncoder.matches`).
    - Verify confirmation matches and new password differs from old.
    - Encode new password and save user.

C. `TodoController` (`/api/todos`)
- `getAllTodos` — GET `/api/todos`
  - Response: `List<TodoResponse>`, HTTP 200.
  - Service: `todoService.getAllTodos()`
  - Details:
    - Gets current user, fetches all todos via `todoRepository.findByOwner(currentUser)`, maps to `TodoResponse`.

- `createTodo` — POST `/api/todos`
  - Request: `TodoRequest` (`title`, `description`, `priority`) validated.
  - Response: `TodoResponse` for created item, HTTP 201 CREATED.
  - Service: `todoService.createTodo(todoRequest)`
  - Details:
    - Builds `Todo` with `complete=false`, owner=currentUser, saves and returns DTO.

- `toggleTodoCompletion` — PUT `/api/todos/{id}`
  - Request: path `id` (min 1).
  - Response: `TodoResponse` updated, HTTP 200.
  - Service: `todoService.toggleTodoCompletion(id)`
  - Details:
    - Uses `todoRepository.findByIdAndOwner(id, currentUser)` to verify ownership and toggle completion. If not found, throws 404.

- `deleteTodo` — DELETE `/api/todos/{id}`
  - Request: path `id` (min 1).
  - Response: HTTP 204 NO CONTENT.
  - Service: `todoService.deleteTodo(id)`
  - Details:
    - Uses `findByIdAndOwner` to ensure only owner can delete. If not found, throws 404.

D. `AdminController` (`/api/admin`) — secured to `ROLE_ADMIN`
- `getAllUsers` — GET `/api/admin`
  - Response: `List<UserResponse>`, HTTP 200.
  - Service: `adminService.getAllUsers()`
  - Details:
    - Lists all users and maps to `UserResponse`.

- `promoteToAdmin` — PUT `/api/admin/{userId}/role`
  - Response: `UserResponse` updated, HTTP 200.
  - Service: `adminService.promoteToAdmin(userId)`
  - Details:
    - Finds user by id; if not found or already admin, throws `ResponseStatusException(BAD_REQUEST)`.
    - Replaces authorities list with `ROLE_EMPLOYEE` and `ROLE_ADMIN`, saves and returns DTO.

- `deleteUser` — DELETE `/api/admin/{userId}`
  - Response: HTTP 204 NO CONTENT.
  - Service: `adminService.deleteNonAdminUser(userId)`
  - Details:
    - Only allows deletion of non-admin users.

---

5️⃣ Service Layer Deep Dive
---------------------------

Detailed per-service responsibilities and methods.

A. `AuthenticationService` / `AuthenticationServiceImpl`
- Methods:
  - `register(RegisterRequest input) throws Exception`
    - Validates uniqueness of email (`userRepository.findByEmail`).
    - Builds `User` entity with `passwordEncoder.encode(input.getPassword())`.
    - Calls `initialAuthority()` to assign `ROLE_EMPLOYEE` and optionally `ROLE_ADMIN` for first user.
    - Saves `User` via `userRepository.save`.
  - `AuthenticationResponse login(AuthenticationRequest request)`
    - Calls `authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(email, password))`.
    - Loads `User` entity and returns JWT via `jwtService.generateToken`.
- Dependencies:
  - `UserRepository`, `PasswordEncoder`, `AuthenticationManager`, `JwtService`.

B. `JwtService` / `JwtServiceImpl`
- Methods:
  - `extractUsername(token)` — extracts subject from token claims.
  - `isTokenValid(token, userDetails)` — checks subject match and expiration.
  - `generateToken(Map<String, Object> claims, UserDetails userDetails)` — builds JWT with subject = username, issuedAt, expiration, signs with secret.
- Implementation details:
  - Uses `io.jsonwebtoken.Jwts`.
  - Reads `spring.jwt.secret` and `spring.jwt.expiration` from `application.properties`.
  - `getSigningKey()` decodes base64 secret to bytes and creates HMAC key.

C. `UserService` / `UserServiceImpl`
- Methods:
  - `UserResponse getUserInfo()` — returns current user details (id, fullName, email, authorities).
  - `void deleteUser()` — deletes the current user, prevents deletion if last admin.
    - Uses `userRepository.countAdminUsers()` to check.
  - `void updatePassword(PasswordUpdateRequest request)` — verifies old password and confirmation, encodes new password and saves.
- Dependencies:
  - `UserRepository`, `FindAuthenticatedUser`, `PasswordEncoder`.

D. `TodoService` / `TodoServiceImpl`
- Methods:
  - `List<TodoResponse> getAllTodos()` — fetch todos by owner using `todoRepository.findByOwner`.
  - `TodoResponse createTodo(TodoRequest)` — create and persist a new `Todo` assigned to current user.
  - `TodoResponse toggleTodoCompletion(long id)` — ensure todo belongs to user via `findByIdAndOwner`, toggle `complete` and save.
  - `void deleteTodo(long id)` — ensure ownership and delete the todo.
- Dependencies:
  - `TodoRepository`, `FindAuthenticatedUser`.

E. `AdminService` / `AdminServiceImpl`
- Methods:
  - `List<UserResponse> getAllUsers()` — returns a list of all users converted to `UserResponse`.
  - `UserResponse promoteToAdmin(long userId)` — checks user existence and admin status, sets admin authority and saves.
  - `void deleteNonAdminUser(long userId)` — deletes user if not admin.
- Dependencies:
  - `UserRepository`.

Why service layer is needed
- Keeps controllers thin.
- Encapsulates business rules (e.g., last admin check, ownership checks).
- Maintains transaction boundaries with `@Transactional`.
- Coordinates repository calls and DTO transformations.

Dependency injection usage
- Constructor injection used across controllers, services, and config beans.
- This supports immutability and easier unit testing.

---

6️⃣ Repository Layer Deep Dive
------------------------------

A. `UserRepository` (`com.luv2code.springboot.todos.repository.UserRepository`)
- Extends: `CrudRepository<User, Long>`
- Methods:
  - `Optional<User> findByEmail(String email)` — derived query from method name; used in authentication.
  - `@Query("SELECT COUNT(u) FROM User u JOIN u.authorities a WHERE a.authority = 'ROLE_ADMIN'") long countAdminUsers();`
    - Custom JPQL to count admin users (used to prevent last-admin deletion).

B. `TodoRepository` (`com.luv2code.springboot.todos.repository.TodoRepository`)
- Extends: `CrudRepository<Todo, Long>`
- Methods:
  - `List<Todo> findByOwner(User owner)` — fetch all todos for a given owner.
  - `Optional<Todo> findByIdAndOwner(Long id, User owner)` — used to ensure todo belongs to the current user when updating/deleting.

Auto-implementation by Spring Data
- Spring Data inspects interface methods and provides run-time proxies implementing derived queries and custom `@Query` methods.
- Repositories are injected into services by Spring.

---

7️⃣ Entity Layer Deep Dive
--------------------------

A. `User` (`com.luv2code.springboot.todos.entity.User`)
- Annotations:
  - `@Entity`, `@Table(name = "users")`
  - Id: `@Id`, `@GeneratedValue(strategy = GenerationType.AUTO)`
- Fields:
  - `id`: primary key (long)
  - `firstName`, `lastName` (not null)
  - `email` (unique, length 100, not null)
  - `password` (not null)
  - `createdAt` (`@CreationTimestamp`, column name `created_at`)
  - `updatedAt` (`@UpdateTimestamp`)
  - `authorities`:
    - `@ElementCollection(fetch = FetchType.EAGER)`
    - `@CollectionTable(name = "user_authorities", joinColumns = @JoinColumn(name = "user_id"))`
    - Type: `List<Authority>`
    - Each `Authority` is `@Embeddable` (single field `authority`)
    - Eager fetching ensures authorities are available for authentication.
  - `todos`:
    - `@OneToMany(mappedBy = "owner", cascade = CascadeType.ALL, orphanRemoval = true)` — owning side is `Todo.owner`.
    - When a `User` is deleted, todos cascade deleted.
- Implements `UserDetails`:
  - `getAuthorities()` returns the authorities collection.
  - `getPassword()` returns password; `getUsername()` returns email.
  - Default `isAccountNonExpired`, `isAccountNonLocked`, etc., delegated to `UserDetails.super`.

B. `Authority` (`com.luv2code.springboot.todos.entity.Authority`)
- `@Embeddable`
- Implements `GrantedAuthority` (method `getAuthority()`).
- Stored in `user_authorities` collection table with `user_id` link.

C. `Todo` (`com.luv2code.springboot.todos.entity.Todo`)
- `@Entity`, `@Table(name = "todos")`
- Fields:
  - `id` (primary key)
  - `title`, `description`, `priority` (not null)
  - `complete` (boolean)
  - `owner`:
    - `@ManyToOne(fetch = FetchType.LAZY)`
    - `@JoinColumn(name = "owner_id", nullable = false)` — stores FK to user.
- Behavior:
  - `Todo` constructors: no-arg (JPA) and convenience constructor used by `TodoServiceImpl`.

How Hibernate manages them
- Hibernate will create/alter tables according to `spring.jpa.hibernate.ddl-auto=update`.
- `@CreationTimestamp` and `@UpdateTimestamp` are managed by Hibernate on insert/update.
- Lazy/Eager settings control whether associations are fetched automatically.

---

8️⃣ 🔐 Security Deep Dive (VERY IMPORTANT)
-----------------------------------------

This app implements stateless JWT authentication with role-based authorization.

Key classes:
- `SecurityConfig.java` — defines the security filter chain and beans
- `JwtAuthenticationFilter.java` — custom OncePerRequestFilter to process JWTs
- `JwtServiceImpl.java` — token generation, parsing, validation
- `User.java` — implements `UserDetails`, returned by `UserDetailsService`
- `AuthenticationServiceImpl.java` — handles login via `AuthenticationManager` and issues JWTs
- `FindAuthenticatedUserImpl.java` — helper to retrieve `User` principal from SecurityContext

Detailed explanation

1) `SecurityConfig`
- Annotations: `@Configuration`, `@EnableWebSecurity`
- Beans:
  - `UserDetailsService userDetailsService()`:
    - Implementation: lambda that calls `userRepository.findByEmail(username).orElseThrow(...)`
    - Because `User` implements `UserDetails`, this returns a `User` object.
  - `PasswordEncoder passwordEncoder()`:
    - `BCryptPasswordEncoder` — recommended for password hashing.
  - `AuthenticationManager authenticationManager(AuthenticationConfiguration config)`
    - Uses Spring's `AuthenticationConfiguration` to expose the default `AuthenticationManager`, which will use the `UserDetailsService` and `PasswordEncoder` for authentication.
  - `AuthenticationEntryPoint`:
    - Custom lambda that writes JSON `{"error": "Unauthorized access"}` with status 401 when incoming request lacks authentication for a protected endpoint.
  - `SecurityFilterChain filterChain(HttpSecurity http)`:
    - Permit all: `/api/auth/**`, swagger UI and docs paths.
    - Require `ROLE_ADMIN` for `/api/admin/**`.
    - Any other request: authenticated.
    - Disable CSRF (common for API-only apps).
    - Set `sessionCreationPolicy` to `STATELESS`.
    - Add `jwtAuthenticationFilter` before `UsernamePasswordAuthenticationFilter`.

2) `JwtAuthenticationFilter`
- A `OncePerRequestFilter`.
- Behavior per request:
  - Reads `Authorization` header. If not present or doesn't start with `"Bearer "`, filter allows request to continue unmodified.
  - Extracts token string (strip `"Bearer "`).
  - Calls `jwtService.extractUsername(jwt)`.
  - Loads `UserDetails` via `userDetailsService.loadUserByUsername(userEmail)`.
  - Calls `jwtService.isTokenValid(jwt, userDetails)` to verify signature and expiration and username match.
  - If valid, creates `UsernamePasswordAuthenticationToken` with principal = userDetails and authorities and sets it into `SecurityContextHolder`.
- This gives the rest of the chain access to authentication/authorities as if the user had been authenticated via username/password for that request.

3) `JwtServiceImpl`
- Reads properties:
  - `@Value("${spring.jwt.secret}") private String SECRET_KEY;`
  - `@Value("${spring.jwt.expiration}") private long JWT_EXPIRATION;`
- `generateToken(claims, userDetails)`:
  - Builds JWT with `subject = userDetails.getUsername()` (email), `issuedAt = now`, `expiration = now + JWT_EXPIRATION`.
  - Signs with HMAC-SHA key created from base64-decoded `SECRET_KEY`.
- `extractUsername(token)`:
  - Extracts `Claims::getSubject`.
- `isTokenValid(token, userDetails)`:
  - Ensures token subject equals `userDetails.getUsername()` and that token is not expired.

4) Authentication vs Authorization
- Authentication:
  - Performed during login via `AuthenticationManager` in `AuthenticationServiceImpl.login`.
  - Per-request authentication reconstructed from JWT in `JwtAuthenticationFilter`.
- Authorization:
  - Configured in `SecurityConfig` via request matchers:
    - Admin endpoints require `ROLE_ADMIN`.
    - Other endpoints require at least authenticated user.
  - `User.getAuthorities()` supplies authorities used by Spring for access decisions.

5) Password encoding
- `BCryptPasswordEncoder` used everywhere:
  - At registration: `passwordEncoder.encode(password)` before saving to DB.
  - At authentication: `AuthenticationManager` uses the encoder to match raw password to stored hash.

6) `UserDetailsService`
- Defined in `SecurityConfig` to lookup `User` by email.
- Returns the `User` entity implementing `UserDetails`.

7) How login works internally step-by-step
- Client posts credentials to `/api/auth/login`.
- Controller delegates to `AuthenticationServiceImpl.login`.
- `AuthenticationManager.authenticate` is called with a `UsernamePasswordAuthenticationToken` containing credentials.
- Authentication manager uses `UserDetailsService` to load `User` and `PasswordEncoder` to match password.
- On success, `jwtService.generateToken` is used to create a token which is returned to client.

8) How requests are secured step-by-step
- For protected endpoints, the client sends `Authorization: Bearer <token>`.
- `JwtAuthenticationFilter` validates the token and sets `SecurityContext` with a valid Authentication.
- `SecurityConfig` authorization rules (`hasRole`, `authenticated`) decide whether the request proceeds to controller.
- Controller and service code may call `FindAuthenticatedUser.getAuthenticatedUser()` to get the `User` principal and enforce additional business rules.

9) Roles and authorities
- Stored on `User` as `List<Authority>` where `Authority` is `@Embeddable` with single String `authority`.
- Typical values: `ROLE_EMPLOYEE`, `ROLE_ADMIN`.
- `SecurityConfig` uses `hasRole("ADMIN")` which checks for `ROLE_ADMIN` in authorities.

Security notes for maintainers
- `spring.jwt.secret` currently placeholder — must be a secure base64 string in production; do not commit secrets.
- Consider placing secret and DB credentials in environment variables or secret manager.
- Consider token revocation strategy (e.g., blacklisting) if needed.
- Consider longer key lengths and secure random generation for signing keys.

---

9️⃣ Exception Handling
-----------------------

Global handler
- `ExceptionHandlers` is `@ControllerAdvice`:
  - Handles `ResponseStatusException`:
    - Uses exception's status and message to build an `ExceptionResponses` object.
  - Handles generic `Exception`:
    - Maps to HTTP 400 with `ExceptionResponses` object containing status, message, timeStamp.

Error payload: `ExceptionResponses`
- Fields:
  - `int status` — HTTP status code
  - `String message` — error message
  - `long timeStamp` — epoch millis
- This ensures consistent JSON error responses across controllers.

Common exception usage in code
- `ResponseStatusException` used in services for 400/404/403 status-level errors.
- `AccessDeniedException` thrown by `FindAuthenticatedUserImpl` if unauthenticated.
- Authentication failures are handled by Spring Security's mechanisms and `AuthenticationEntryPoint`.

---

🔟 Configuration Files
----------------------

`src/main/resources/application.properties`
- Key properties:
  - `spring.application.name=todos`
  - Database:
    - `spring.datasource.url=jdbc:mysql://localhost:3307/tododb?serverTimezone=UTC&allowPublicKeyRetrieval=true&useSSL=false`
    - `spring.datasource.username=root`
    - `spring.datasource.password=secret`
  - JPA/Hibernate:
    - `spring.jpa.hibernate.ddl-auto=update`
    - `spring.jpa.open-in-view=false`
  - Swagger:
    - `springdoc.swagger-ui.path=/docs`
  - JWT:
    - `spring.jwt.secret=<<TODO-INSERT-YOUR-SECRET>>` (replace with secure base64)
    - `spring.jwt.expiration=900000` (ms — 15 minutes)

`pom.xml` — key dependencies & purpose
- `spring-boot-starter-data-jpa` — JPA and Hibernate integration.
- `spring-boot-starter-security` — security features.
- `spring-boot-starter-webmvc` — web MVC framework for REST endpoints.
- `spring-boot-devtools` — dev tools (runtime).
- `mysql-connector-j` — MySQL JDBC driver.
- `spring-boot-starter-webmvc-test`, `spring-security-test` — testing utilities.
- `springdoc-openapi-starter-webmvc-ui` — OpenAPI/Swagger UI integration.
- `spring-boot-starter-validation` — validation (Jakarta).
- `io.jsonwebtoken:jjwt-api`, `jjwt-impl`, `jjwt-jackson` — JWT creation/parsing.

---

1️⃣1️⃣ How to Run the Project
-----------------------------

Requirements
- JDK compatible with project (pom indicates Java 25 — ensure your environment supports it or adjust `pom.xml` `java.version` if needed).
- Maven.
- MySQL running and reachable. By default config expects `tododb` on `localhost:3307`. Update `application.properties` if different.

Steps (PowerShell)
1. Build:
```powershell
cd 'C:\Users\aligh\IdeaProjects\spring-boot-rest-apis\section-06-spring-boot-rest-project-4\04-todos'
mvn -B clean package
```

2. Run with Maven:
```powershell
mvn spring-boot:run
```

3. Or run the JAR:
```powershell
java -jar target\todos-0.0.1-SNAPSHOT.jar
```

4. Tests:
```powershell
mvn test
```

Postman / curl quick tests
- Register:
```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alice","lastName":"Doe","email":"alice@example.com","password":"secret123"}'
```

- Login:
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"secret123"}'
```

- Use returned token:
  - Add header: `Authorization: Bearer <token>` for subsequent requests.

Swagger UI
- Visit: `http://localhost:8080/docs` (based on `springdoc` config) to interactively test APIs.

Important operational notes
- If you plan to deploy, replace `spring.jpa.hibernate.ddl-auto=update` with a schema migration tool like Flyway or Liquibase.
- Keep secrets out of source control — use environment variables or secret vault.

---

1️⃣2️⃣ Big Picture Summary
-------------------------

How layers connect
- Client → Filters (JwtAuthenticationFilter) → Controller → Service → Repository → JPA/Hibernate → Database
- Security layer (configured in `SecurityConfig`) ensures authentication/authorization; `JwtAuthenticationFilter` rebuilds authentication per request for a stateless architecture.

Final architecture ASCII diagram
```
Client
  ↓ HTTP
Embedded Tomcat / DispatcherServlet
  ↓
Filter chain:
  - JwtAuthenticationFilter (validates JWT, sets SecurityContext)
  - Other Spring Security filters
  - AuthenticationEntryPoint (for unauthorized)
  - SessionCreationPolicy = STATELESS
  ↓
Controller Layer (/api/*)
  ↓
Service Layer (business rules, transactions)
  ↓
Repository Layer (Spring Data JPA)
  ↓
Hibernate/JPA → MySQL (tododb)
```

---

Appendix: Explicit file/class map (all classes)
- `TodosApplication.java` (main)
- config:
  - `SecurityConfig.java`
  - `JwtAuthenticationFilter.java`
  - `SwaggerConfig.java`
- controller:
  - `AuthenticationController.java`
  - `UserController.java`
  - `TodoController.java`
  - `AdminController.java`
- service:
  - `AuthenticationService.java`
  - `AuthenticationServiceImpl.java`
  - `JwtService.java`
  - `JwtServiceImpl.java`
  - `UserService.java`
  - `UserServiceImpl.java`
  - `TodoService.java`
  - `TodoServiceImpl.java`
  - `AdminService.java`
  - `AdminServiceImpl.java`
- repository:
  - `UserRepository.java`
  - `TodoRepository.java`
- entity:
  - `User.java`
  - `Authority.java`
  - `Todo.java`
- request:
  - `RegisterRequest.java`
  - `AuthenticationRequest.java`
  - `PasswordUpdateRequest.java`
  - `TodoRequest.java`
- response:
  - `AuthenticationResponse.java`
  - `UserResponse.java`
  - `TodoResponse.java`
- util:
  - `FindAuthenticatedUser.java`
  - `FindAuthenticatedUserImpl.java` (reads `SecurityContextHolder` and returns `(User) authentication.getPrincipal()`)
- exception:
  - `ExceptionHandlers.java`
  - `ExceptionResponses.java`
- resources:
  - `application.properties`
- build:
  - `pom.xml`

Final recommendations & next steps
- Replace `spring.jwt.secret` placeholder with a secure base64 key (do not commit).
- Use environment variables or externalized config for DB credentials & secret in production.
- Add unit and integration tests for services and security flows (use `spring-security-test`).
- Consider storing roles in a normalized `role` table if role metadata grows.

---

If you want, I can:
- Provide the same document as a shorter README quickstart.
- Give you a ready-to-run PowerShell script that writes the file and opens it in Notepad++ or IntelliJ.
- Generate a set of example cURL/Postman requests for every endpoint.

Tell me which option you prefer and I will provide the exact PowerShell script to create the file and optionally open it automatically.
```