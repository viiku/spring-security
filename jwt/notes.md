# JWT Authentication Flow (Spring Security)

---

## 1. User Registers / Logs In

- **Client** (frontend app or Postman) sends:
  ```http
  POST /auth/login
  {
    "username": "...",
    "password": "..."
  }
  ```
- **Backend:**
  - Authenticates user against DB (via `AuthenticationManager` + `CustomUserDetailsService`).
  - If valid, `JwtService.generateToken(user)` creates a signed JWT.
  - Returns response:
    ```json
    {
      "accessToken": "jwt_here",
      "tokenType": "Bearer",
      "expiresIn": 3600
    }
    ```
- **Client** stores JWT (e.g., in localStorage, cookie, or memory).

---

## 2. Client Calls Protected Endpoint

- **Client** sends request:
  ```http
  GET /api/v1/files
  Authorization: Bearer <jwt_here>
  ```

---

## 3. Spring Security Filter Chain

- Request hits Spring Security’s filter chain.
- `JwtAuthenticationFilter` (custom filter) runs once per request (`OncePerRequestFilter`).

---

## 4. JwtAuthenticationFilter: Extract & Validate

- Inside `doFilterInternal()`:
  - Reads `Authorization` header.
  - If missing or doesn’t start with `"Bearer "`, request continues unauthenticated.
  - Extracts JWT:  
    `token = authHeader.substring(7)`
  - Uses `JwtService.extractUsername(token)` to decode & validate.
  - If token is invalid, request continues unauthenticated (Spring later sends 401).
  - Loads user details (`CustomUserDetailsService`).
  - If token valid:
    - Builds `UsernamePasswordAuthenticationToken`
    - Sets it in `SecurityContextHolder`
    - ✅ Request is now authenticated.

---

## 5. Controller Execution

- If user has required role/authority, controller executes and returns data.
- If authorization fails (e.g., needs `ROLE_ADMIN` but user has `ROLE_USER`):
  - `RestAccessDeniedHandler` responds:
    ```json
    {
      "error": "forbidden",
      "message": "Access is denied"
    }
    ```
    - HTTP 403 FORBIDDEN

---

## 6. Authentication Errors

- If token is missing or invalid:
  - Spring invokes `RestAuthenticationEntryPoint`
  - Responds:
    ```json
    {
      "error": "unauthorized",
      "message": "Full authentication is required to access this resource"
    }
    ```
    - HTTP 401 UNAUTHORIZED

---

## Request Paths Summary

- **Valid request:**  
  Request → JwtAuthenticationFilter (extracts JWT) → JwtService validates → SecurityContext updated → Controller executes → Response 200

- **Invalid/expired/missing token:**  
  Request → JwtAuthenticationFilter (invalid token) → No authentication set → RestAuthenticationEntryPoint → Response 401

- **Insufficient roles/permissions:**  
  Request → JwtAuthenticationFilter (valid token, user authenticated) → Controller security check fails → RestAccessDeniedHandler → Response 403

---

## Key Components

- **JwtService** — Handles token generation/validation.
- **JwtAuthenticationFilter** — Extracts and authenticates per request.
- **RestAuthenticationEntryPoint** — Handles 401 Unauthorized.
- **RestAccessDeniedHandler** — Handles 403 Forbidden.
- **Controllers** — Secured via `@PreAuthorize`, `@Secured`, or config-based rules.

---

## What Happens on `/register` and `/login`?

### 1. Unprotected Endpoints (`/register`, `/login`)
### 2. Protected Endpoints (`/user`, `/admin`, etc.)

---

### What Happens on `/register`

- Spring Security filter chain is still active, but since `/register` is marked as `permitAll()` in your `SecurityFilterChain`, Spring does not try to authenticate it.
- Your controller/service code will:
  - Take the incoming request (username, password, roles, etc.).
  - Hash the password (usually with `BCryptPasswordEncoder`).
  - Save user info into Postgres (via `UserEntity` + `UserRepository`).
  - Return a `201 Created` response with a message or `RegisterResponse`.
  - 👉 No `AuthenticationManager` or security context involved here.

---

### What Happens on `/login`

- Even though `/login` is also `permitAll()`, it’s the entry point for authentication.

**Flow:**
1. Client sends:
    ```json
    { "username": "user", "password": "pass" }
    ```
2. Controller receives request → calls a custom `AuthenticationService`.
3. That service uses the `AuthenticationManager` to check if the credentials are valid:
    ```java
    Authentication authentication = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(username, password)
    );
    ```
4. `AuthenticationManager` delegates to your `AuthenticationProvider` (by default, `DaoAuthenticationProvider`), which:
    - Calls `UserDetailsService.loadUserByUsername(username)`.
    - Loads the user from Postgres.
    - Compares stored hashed password with incoming password (via `PasswordEncoder.matches()`).
5. If authentication succeeds:
    - You generate a JWT containing user’s roles/username.
    - Return `{ accessToken: "jwt..." }`.
6. If authentication fails:
    - Spring throws `AuthenticationException` → handled by your `RestAuthenticationEntryPoint`.

---


When you call /login (or your custom login endpoint), Spring Security uses an AuthenticationManager to verify credentials.

The flow is like this:

Client sends username & password (e.g., JSON body { "username": "vikku", "password": "mypassword" }).

Spring Security wraps it in a UsernamePasswordAuthenticationToken.

AuthenticationManager delegates to a list of AuthenticationProviders.
By default, with DaoAuthenticationProvider, it:

Calls your UserDetailsService.loadUserByUsername(username).

Retrieves the UserDetails object (from DB, in your case).

Gets the stored hashed password from the DB.

Compares the stored password with the provided password using a PasswordEncoder (e.g., BCrypt).

If it matches → authentication success → security context is populated.
If not → BadCredentialsException.

Step 2: Where password check happens

👉 In your code:
```
@Override
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    UserEntity userEntity = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found"));

    return new User(
            userEntity.getUsername(),
            userEntity.getPassword(),   // <-- stored hashed password
            userEntity.isEnabled(),
            true, true, true,
            List.of(new SimpleGrantedAuthority("ROLE_" + userEntity.getRole().name()))
    );
}

```

This does NOT check the password itself.
It only returns the user details with the stored (hashed) password.

➡️ The password check happens inside DaoAuthenticationProvider (Spring’s default authentication provider).

It uses:

```
passwordEncoder.matches(rawPassword, storedPassword)
```

Example:

Raw password from request = "mypassword"

Stored password in DB = "$2a$10$wGeN1K..." (BCrypt hash)

Spring calls BCryptPasswordEncoder.matches("mypassword", "$2a$10$wGeN1K...")

If true → authenticated, else → fail.


🔹 Step 3: Why use AuthenticationManager?

It orchestrates the authentication flow.

You don’t need to manually check passwords — Spring Security centralizes that.

You can plug in custom AuthenticationProviders if you want to handle authentication differently (e.g., LDAP, OAuth2, custom DB logic).

For example, your /login service might look like:

```
@Autowired
private AuthenticationManager authenticationManager;

@Autowired
private JwtService jwtService;

public String login(LoginRequest request) {
    Authentication authentication = authenticationManager.authenticate(
        new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword())
    );

    // If we reach here, password was checked internally and is valid
    UserDetails user = (UserDetails) authentication.getPrincipal();
    return jwtService.generateToken(user);
}
```
So in short:

loadUserByUsername() → loads user + stored password.

DaoAuthenticationProvider → does the password match check using a PasswordEncoder.

AuthenticationManager → orchestrates all providers.

/login endpoint just delegates → if success, you generate JWT and return it.


### What Happens on `/user` or `/admin`

- These are protected endpoints.

**Flow:**
1. Client sends:
    ```
    Authorization: Bearer <jwt>
    ```
2. Your `JwtAuthenticationFilter` intercepts the request before it hits the controller:
    - Extracts JWT from header.
    - Validates it (signature, expiry, etc.).
    - Parses roles & subject.
    - Creates an `Authentication` object → stores in `SecurityContext`.
3. When controller has `@PreAuthorize("hasRole('USER')")`:
    - Spring Security checks the `Authentication` object in `SecurityContext`.
    - If role matches → request proceeds.
    - Else → AccessDeniedHandler (your `RestAccessDeniedHandler`) returns 403


1. Login

User authenticates (username + password).
Server issues:
Access Token (short-lived, e.g., 15 min)
Refresh Token (longer-lived, e.g., 7 days or 30 days)
👉 Access token is used for requests; refresh token is only for getting new access tokens.

2. Using the API

Client sends access token in the Authorization header (Bearer).
Backend checks validity (signature + expiry).
If valid → proceed.
If expired → reject with 401.

3. Refreshing

When client gets a 401 due to expired access token, it calls /refresh endpoint:
```
POST /refresh
Authorization: Bearer <refresh_token>
```
Server:
Validates refresh token (checks DB/Redis or signature).
If valid, issues new Access Token (and optionally a new Refresh Token).
If invalid/expired → client must re-login.

**We do NOT send access token to refresh endpoint.**

4. Logout

Client calls /logout with refresh token.
Server deletes that refresh token from DB (or marks as revoked).
Access token will naturally expire in a few minutes.
Any attempt to refresh after logout → rejected.

❌ Wrong Assumption

“We send access token to refresh endpoint, server looks it up in DB, and if invalid, issues refresh token.”

That’s not how it works in production systems.
Access token is never used to generate refresh token.
Refresh token is the only credential to get a new access token.
If refresh token is gone/expired, user must log in again.

