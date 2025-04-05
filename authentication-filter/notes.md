## Authentication Filter

What is an Authentication Filter?
An Authentication Filter is a core part of Spring Security’s filter chain that:

✅ Intercepts incoming HTTP requests
✅ Extracts authentication credentials (like username/password or JWT token)
✅ Triggers the authentication logic (usually via AuthenticationManager)
✅ Either allows the request to continue or blocks it (401 Unauthorized)

Where does it live in the flow?

```
Client ---> Security Filter Chain:
    - CorsFilter
    - CsrfFilter
    - AuthenticationFilter  <-- ⚡ You're here
    - AuthorizationFilter
    - FilterSecurityInterceptor
    ...
    --> Controller (if allowed)
```

Types of Authentication Filters
1. UsernamePasswordAuthenticationFilter
Default filter used when you log in with a username and password (form login or basic login).

Activated at /login by default.

You can customize it to handle login at a custom endpoint.

2. Custom Authentication Filter (e.g., JWTAuthenticationFilter)
When you use JWT, OAuth2, or other token-based logins.

You often build a custom filter to:

Read the Authorization header

Extract the token

Validate it

Set the user in the security context