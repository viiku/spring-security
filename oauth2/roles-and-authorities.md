## Roles and Authorities

### Key Differences
Authorities are fine-grained permissions that specify what actions a user can perform (e.g., READ_USER, CREATE_USER)

Roles are broader groupings of authorities, typically prefixed with "ROLE_" (e.g., ROLE_ADMIN, ROLE_USER)

### How They Work Together
    1. Users are assigned roles
    2. Roles contain one or more authorities
    3. Security decisions can be made based on either roles or authorities
    4. Spring Security automatically adds the "ROLE_" prefix to roles when using role-based methods

***In Spring Security, you can secure endpoints or methods using either:***

    1. Role-based access control (RBAC)
    2. Authority-based access control

```
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")          // Role-based
                .antMatchers("/api/**").hasAuthority("API_ACCESS")  // Authority-based
                .anyRequest().authenticated();
        return http.build();
    }
}
```