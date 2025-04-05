## Annotations

1. @AuthenticationPrincipal
***The @AuthenticationPrincipal is a Spring Security annotation that provides convenient access to the currently authenticated user in your controllers. It allows you to directly access the principal object (the user) from the security context without having to manually extract it from the SecurityContextHolder.***

```
@GetMapping("/profile")
public String profile(@AuthenticationPrincipal UserDetails userDetails) {
    String username = userDetails.getUsername();
    // Use the username or other user details
    return "profile";
}
```

2. @Configuration
***The @Configuration annotation is a crucial Spring annotation that indicates a class serves as a source of bean definitions. It's essentially a class-level annotation that tells Spring that the class will be used to configure your application context.***

### Key Points
Marks the class as a configuration class
Allows definition of Spring beans using @Bean annotated methods
Enables full Spring AOP proxying to ensure consistent bean lifecycle management
Replaces traditional XML configuration with Java-based configuration

The `@Configuration` class typically contains:

@Bean definitions
Component scanning directives
Property source configurations
Import statements for other configuration classes

```
@Configuration
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // Security configuration
        return http.build();
    }
}
```

3. @EnableWebSecurity
***@EnableWebSecurity is a marker annotation in Spring Security that enables Spring Security's web security support. This annotation is typically used in conjunction with @Configuration to create a security configuration class.***

Purpose and Functionality
Imports the WebSecurityConfiguration class
Enables Spring Security's web security infrastructure
Activates the SpringWebMvcImportSelector and OAuth2ImportSelector
Sets up essential security filters and infrastructure beans
```
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        // Security configuration
        return http
            .authorizeHttpRequests()
            .anyRequest().authenticated()
            .and().build();
    }
}
```

### key points
Required for web applications using Spring Security
Works with both servlet and reactive applications
Can be customized using @EnableWebSecurity(debug = true) for debugging
Automatically sets up default security configuration if none is provided
Integrates with Spring MVC's security features

### Best Practices
Always combine with @Configuration
Place in a dedicated security configuration class
Consider using with @Order when multiple security configurations exist
Use debug mode carefully in production environments


4. @Bean
***The @Bean annotation is a method-level annotation used in Spring Framework to explicitly declare a single bean. When Spring's application context processes this annotation, it will execute the annotated method and register the return value as a bean within the Spring container.***

### Purpose
Declares a bean to be managed by the Spring container
Provides fine-grained control over bean creation and configuration
Usually used within @Configuration classes
Allows programmatic bean creation logic

### Key Features
Bean Naming: By default, the method name becomes the bean name, but you can specify a custom name
Scope Control: Can specify bean scope (singleton, prototype, etc.)
Dependency Injection: Can inject other beans as method parameters
Initialization/Destruction: Supports lifecycle callbacks