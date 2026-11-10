# Step 4: Spring Boot Architecture in Kotlin (Controllers, Services & JPA)

This guide covers building a 3-tier Spring Boot backend in idiomatic Kotlin, replacing boilerplate with primary constructor injection, extension mappers, and safe JPA entity patterns.

---

## Quick Summary

* **Web Layer (`@RestController`):** Uses immutable `data class` DTOs for request/response bodies, `@field:` use-site targets for Jakarta Bean Validation (`@Valid`), and single-expression functions for endpoints.
* **Service Layer (`@Service`):** Uses primary constructor injection (`private val`) in the class header without `@Autowired`, combined with Spring Data's `findByIdOrNull()` and the Elvis operator (`?:`).
* **Persistence Layer (`@Entity`):** Always use a standard `class`—**never a `data class`**—for JPA entities to prevent `StackOverflowError` on bidirectional relationships and broken `HashSet` lookups caused by `@Id` mutation after `.save()`.

---

## 1. Web Layer (`@RestController` & DTOs)

Use immutable `data class` models for API boundaries. Jackson (`jackson-module-kotlin`) automatically enforces non-nullability during JSON deserialization.

### Crucial Kotlin Annotation Rule: `@field:`
In Kotlin, a constructor parameter like `val name: String` compiles into a private backing field, a getter, and a constructor parameter. When applying Jakarta Bean Validation annotations (`@NotBlank`, `@Email`, `@Size`), prefix them with **`@field:`** so Spring Validator inspects the underlying JVM field directly.

### Code Implementation

```kotlin
// Request DTO (Immutable & Null-Safe)
data class CreateWorkspaceRequest(
    @field:NotBlank(message = "Workspace name is required")
    val name: String,

    @field:Email(message = "Must be a valid email address")
    val ownerEmail: String
)

// Response DTO
data class WorkspaceResponse(
    val id: Long,
    val name: String,
    val ownerEmail: String
)

@RestController
@RequestMapping("/api/v1/workspaces")
class WorkspaceController(
    private val workspaceService: WorkspaceService
) {

    @GetMapping("/{id}")
    fun getById(@PathVariable id: Long): WorkspaceResponse =
        workspaceService.getWorkspaceById(id).toResponse()

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    fun create(@Valid @RequestBody request: CreateWorkspaceRequest): WorkspaceResponse =
        workspaceService.createWorkspace(request).toResponse()
}

// Private extension function keeps mapping logic clean without extra Mapper classes
private fun Workspace.toResponse() = WorkspaceResponse(
    id = this.id ?: throw IllegalStateException("Saved workspace must have a non-null ID"),
    name = this.name,
    ownerEmail = this.ownerEmail
)
```

---

## 2. Service Layer (`@Service` & Constructor Injection)

Kotlin eliminates field injection (`@Autowired`) and Lombok's `@RequiredArgsConstructor` by declaring dependencies directly in the primary constructor header using `private val`.

### Code Implementation

```kotlin
@Service
class WorkspaceService(
    private val workspaceRepository: WorkspaceRepository
) {

    @Transactional(readOnly = true)
    fun getWorkspaceById(id: Long): Workspace =
        workspaceRepository.findByIdOrNull(id)
            ?: throw NoSuchElementException("Workspace with ID \$id not found")

    @Transactional
    fun createWorkspace(request: CreateWorkspaceRequest): Workspace {
        // Safely check for duplicates using nullable return + scope function
        workspaceRepository.findByName(request.name)?.let {
            throw IllegalArgumentException("Workspace '\${request.name}' already exists")
        }

        val workspace = Workspace(
            name = request.name,
            ownerEmail = request.ownerEmail
        )
        return workspaceRepository.save(workspace)
    }

    @Transactional
    fun updateWorkspaceName(id: Long, newName: String): Workspace =
        getWorkspaceById(id).apply {
            name = newName
        }
}
```

### Key Idiomatic Highlights
* **`findByIdOrNull(id)`:** An official Spring Data Kotlin extension (`org.springframework.data.repository.findByIdOrNull`) that returns `T?` instead of `Optional<T>`, pairing seamlessly with `?: throw ...`.
* **Dirty Checking with `.apply`:** Inside a `@Transactional` method, mutating `name = newName` triggers Hibernate dirty checking automatically on transaction commit without needing an explicit `.save()` call.

---

## 3. Persistence Layer (`@Entity` & `JpaRepository`)

### Why You Must NEVER Use `data class` for JPA `@Entity`

Using `data class` on Hibernate/JPA entities introduces three severe runtime bugs:

#### Trap 1: `StackOverflowError` in Bidirectional Relationships
A `data class` auto-generates `hashCode()`, `equals()`, and `toString()` using **every property** in the primary constructor. If two entities have a bidirectional `@OneToMany` / `@ManyToOne` relationship:

```kotlin
// BROKEN EXAMPLE: Do NOT use data class on bidirectional entities
@Entity
data class Workspace(
    @Id val id: Long = 1L,
    @OneToMany(mappedBy = "workspace", fetch = FetchType.LAZY)
    val modules: MutableList<Module> = mutableListOf()
)

@Entity
data class Module(
    @Id val id: Long = 100L,
    @ManyToOne(fetch = FetchType.LAZY)
    val workspace: Workspace
)
```

**How the Infinite Loop Happens (Decompiled Java Trace):**
1. Hibernate or a collection calls `workspace.hashCode()`.
2. `Workspace.hashCode()` calls `this.modules.hashCode()`, which iterates through the list and calls `module.hashCode()`.
3. `Module.hashCode()` computes its fields and calls `this.workspace.hashCode()` back on the parent.
4. `Workspace.hashCode()` calls `this.modules.hashCode()` again, repeating infinitely until the JVM crashes with **`java.lang.StackOverflowError`** (or triggers unintended `LazyInitializationException` / N+1 queries by forcing lazy collections to load).

#### Trap 2: Broken `HashSet` / `HashMap` Contract on `@Id` Mutation
When a new entity is instantiated before persistence, its generated `@Id` is `null`. When `repository.save(entity)` executes, Hibernate mutates the `id` field to the database-generated value (e.g., `42L`).
1. Entity created with `id = null` -> added to a `HashSet` (stored in bucket calculated from `id = null`).
2. `repository.save(entity)` runs -> Hibernate mutates `id` from `null` to `42L`.
3. `set.contains(entity)` is called -> `data class` recalculates `hashCode()` using `id = 42L`, checks the wrong hash bucket, and returns **`false`** for the exact same object instance.

#### Trap 3: Immutability Mismatch
`data class` is designed for immutable value objects (`val` + `.copy()`), whereas Hibernate relies on stateful proxy subclassing and post-instantiation field mutation.

---

### The Idiomatic JPA Entity & Repository Pattern

Use a standard **`class`** (relying on `kotlin("plugin.spring")` and `kotlin("plugin.jpa")` for `open` modifiers and no-arg constructors). Place the nullable `id: Long? = null` at the end of the constructor, and implement a constant `hashCode()`.

```kotlin
@Entity
@Table(name = "workspaces")
class Workspace(
    @Column(nullable = false)
    var name: String,

    @Column(nullable = false)
    var ownerEmail: String,

    @OneToMany(mappedBy = "workspace", fetch = FetchType.LAZY)
    val modules: MutableList<Module> = mutableListOf(),

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null
) {
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is Workspace) return false
        return id != null && id == other.id
    }

    // Constant hashCode prevents StackOverflowError and survives @Id mutation after .save()
    override fun hashCode(): Int = javaClass.hashCode()
}

interface WorkspaceRepository : JpaRepository<Workspace, Long> {
    // Spring Data JPA supports Kotlin nullable types natively without Optional<T>
    fun findByName(name: String): Workspace?
    fun findAllByOwnerEmail(ownerEmail: String): List<Workspace>
}
```
