# 5.2 Inversion of control and Dependency Injection

## Questions
### What is the ApplicationContext ?
- The ApplicationContext in Spring is the central interface for accessing the Spring IoC container.

In simple terms:
It's the object that:

Creates, manages, and injects all your Spring beans

Holds all your app's configuration
Provides powerful features like:
- Dependency Injection
- Bean lifecycle management
- Internationalization
- Event propagation
- Resource loading

Most common implementation:
```
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

--- 
### What are the tradeoffs of different approaches to injecting beans ?
#### 1. Constructor Injection
✅ Pros:
- Immutable dependencies (good for testability & design)
- Guarantees all dependencies are present at creation time
- Encouraged by Spring team (esp. with @Autowired)

❌ Cons:
- Can get messy with many arguments
- Circular dependencies are harder to resolve

#### 2. Setter Injection
✅ Pros:
- Good for optional dependencies
- Easier to read when there are many fields
- Can reconfigure after bean is created

❌ Cons:
- Risk of beans being in partially-initialized state
- Easier to forget to set something (null errors)

#### Field Injection
✅ Pros:
- Shortest, cleanest-looking code (just @Autowired a field)

Cons:
- Not testable easily (needs reflection or framework)
- No control over immutability
- Not recommended for production code

Summary table:
| Approach     | Required? | Testable? | Reconfigurable?    | Recommended for           |
|--------------|-----------|-----------|---------------------|----------------------------|
| Constructor  | ✅ Yes    | ✅ Yes    | ❌ No               | Most production beans      |
| Setter       | ❌ No     | ✅ Yes    | ✅ Yes              | Optional config            |
| Field        | ❌ No     | ❌ No     | ✅ Yes (but bad)    | Quick demos, not prod      |


--- 

### Why do we need to use @Qualifier when multiple of the same type are defined ?

When Spring finds multiple beans of the same type, it doesn’t know which one to inject.
Using @Qualifier lets you tell Spring exactly which one you want.

Example scenario:
```
@Service
class EmailService implements NotificationService {}
```
```
@Service
class SmsService implements NotificationService {}
```
Now if you do:
```
@Autowired
NotificationService service;
```
Spring will throw:
```
NoUniqueBeanDefinitionException: expected single matching bean but found 2
```
Solution:
Specify the one you want:
```
@Autowired
@Qualifier("emailService")
NotificationService service;
```

Summary:
@Qualifier = disambiguation tool when multiple beans match the same type.

--- 
### How to avoid loading of heavy beans (like caches or other beans with heavy init logic) on startup and decrease startup time?

1. @Lazy annotation
Tells Spring not to create the bean at startup, only when it’s first needed.

```
@Component
@Lazy
public class HeavyCacheLoader {
    public HeavyCacheLoader() {
        System.out.println("Initializing cache...");
        // expensive logic
    }
}
```

2. Use @Bean(lazy = true) for config classes
```
@Configuration
public class AppConfig {
    @Bean(lazy = true)
    public HeavyService heavyService() {
        return new HeavyService();
    }
}
```

3. Lazy at container level (not common) in .properties
```
spring.main.lazy-initialization=true
```
This makes all beans lazy by default (only for special cases like dev/testing)

### 4. Move to SmartLifecycle
For beans that run after full context initialization and can delay logic to post-startup.

### Other
Use @PostConstruct for logic that shouldn’t run on construction, and optionally run it in a background thread.

---
### What are Spring lifecycle stages and methods?
# 🔁 Spring Bean Lifecycle Summary

## 🚦 Stages of a Spring Bean

| Stage                 | What Happens                                      | Related Methods / Annotations                      |
|-----------------------|---------------------------------------------------|----------------------------------------------------|
| 1️⃣ Instantiation      | Bean object is created                            | Constructor                                        |
| 2️⃣ Property Population | Dependencies are injected                         | `@Autowired`, setters, fields                      |
| 3️⃣ Aware Interfaces   | Container injects context/environment info         | `setApplicationContext()`, `setBeanName()`         |
| 4️⃣ Initialization     | Custom init logic runs                             | `@PostConstruct`, `afterPropertiesSet()`, `init-method` |
| 5️⃣ Ready to Use       | Bean is available in the context                   | (Used by the app normally)                         |
| 6️⃣ Destruction        | Cleanup before shutdown                            | `@PreDestroy`, `destroy()`, `destroy-method`       |

---

## ✅ Lifecycle Callback Methods

| Type        | Method / Annotation         | When It Runs              | Notes                                               |
|-------------|-----------------------------|----------------------------|-----------------------------------------------------|
| **Init**    | `@PostConstruct`            | After DI, before use       | ✅ Preferred modern way                             |
|             | `afterPropertiesSet()`      | From `InitializingBean`    | ❌ Coupled to Spring                                |
|             | `init-method="..."`         | XML or Java config         | ✔️ Flexible & decoupled                             |
| **Destroy** | `@PreDestroy`               | Before container shutdown  | ✅ Preferred modern way                             |
|             | `destroy()`                 | From `DisposableBean`      | ❌ Coupled to Spring                                |
|             | `destroy-method="..."`      | XML or Java config         | ✔️ Flexible & decoupled                             |
| **Aware**   | `ApplicationContextAware`   | After instantiation        | Allows access to container (use only if needed)    |
| **Lifecycle** | `start()`, `stop()`       | On context start/stop      | For beans with background tasks or manual control  |

---

## 🧠 Pro Tip:
Use `@PostConstruct` and `@PreDestroy` for most cases.  
Only use interfaces like `InitializingBean`, `DisposableBean`, or `*Aware` for infrastructure or advanced needs.


