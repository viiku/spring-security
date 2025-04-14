# spring-security

1. why do we implements serializable generally for entity class or Dto classes?
What does Serializable do?

Serializable is a marker interface in Java (it has no methods) that tells the JVM:

"This object can be converted into a stream of bytes and later restored (deserialized)."

Why do we use it for Entity/DTO classes?
1. Caching (e.g., Redis, EhCache)
Most caching libraries (like Redis or EhCache) require objects to be serializable so they can be stored in memory/disk or across a cluster.

```java
@Cacheable("users")
public UserDto getUserById(String id) { ... }
```

2. HTTP Session Replication (for Stateful Apps)
When using distributed session management (like in a clustered Tomcat), session-scoped beans or entities stored in HTTP sessions must be serializable to be passed between servers.

```java
request.getSession().setAttribute("user", userDto);
```

3. JPA/Hibernate Entities (Best Practice)
Although JPA doesn’t require it, making entities serializable is a good idea:

Entities may be passed across layers (controller ↔ service ↔ DAO).

Some frameworks/tools expect them to be serializable for proxying, persistence context detachment, or remote calls.

4. DTOs over Network (e.g., RMI, JMS, Microservices)
If you’re:

Sending DTOs through Java RMI

Using JMS (Java Messaging Service)

Writing to files/sockets

...serialization is needed to send the DTO as a byte stream.