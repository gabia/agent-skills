# Javadoc

**When to use this guide**: Reference when writing Javadoc comments for Java code

---

## When to Write Javadoc

Document public APIs thoroughly. Package-private and private members need less documentation.

| Visibility | Requirement | Notes |
|------------|-------------|-------|
| `public` | **Required** | External API - must be documented |
| `protected` | **Recommended** | Subclass API - should be documented |
| `package-private` | Optional | Internal API - document complex logic |
| `private` | Optional | Implementation detail - document if non-obvious |

### ✅ Do

```java
/**
 * Service that handles user authentication.
 *
 * <p>This service provides login, logout, and token refresh functionality.</p>
 *
 * @since 1.0
 */
public interface AuthService {

    /**
     * Validates user credentials and returns an authentication token.
     *
     * @param credentials the user credentials
     * @return authentication token
     * @throws AuthenticationException if the credentials are invalid
     */
    AuthToken authenticate(Credentials credentials);
}
```

### ❌ Don't

```java
// No Javadoc for public API
public interface AuthService {

    AuthToken authenticate(Credentials credentials);
}
```

---

## First Sentence (Summary)

The first sentence is the **summary fragment** - it appears in method tables and package summaries. End with a period.

### Rules

1. **Start with a verb phrase** (3rd person declarative): "Returns", "Gets", "Sets", "Creates"
2. **Be concise** - one sentence, one main point
3. **End with a period** - the parser stops at the first period followed by whitespace

### ✅ Do

```java
/**
 * Returns the user's display name.
 *
 * <p>The display name is formatted as "First Last" if both names are available,
 * otherwise returns the username.</p>
 *
 * @return the formatted display name, never null
 */
public String getDisplayName() {
    // Format user name
}
```

```java
/**
 * Creates a new user with the specified email address.
 *
 * @param email the email address for the new user
 * @return the created user
 */
public User createUser(String email) {
    // User creation logic
}
```

### ❌ Don't

```java
/**
 * This method gets the user's display name  // "This method" is unnecessary; it should start with a verb
 */
public String getDisplayName() {
    // ...
}

/**
 * Get the display name  // Use third-person "Gets"
 */
public String getDisplayName() {
    // ...
}

/**
 * Returns the user's display name (e.g. John Doe).  // The period after "e.g." ends the summary
 * This is used for UI display.
 */
public String getDisplayName() {
    // ...
}
```

---

## Block Tags

Use block tags in this standard order:

```
@param      (parameters - in declaration order)
@return     (return value)
@throws     (exceptions - alphabetical order)
@see        (references)
@since      (introduced version)
@deprecated (scheduled for removal)
```

### @param

Document all parameters. Describe valid values and constraints.

```java
/**
 * Searches for users matching the given criteria.
 *
 * @param query the search query, must not be blank
 * @param limit the maximum number of results to return, must be positive
 * @param offset the starting position for pagination, must be non-negative
 * @return list of matching users, empty list if no matches found
 */
public List<User> search(String query, int limit, int offset) {
    // User search logic
}
```

### @return

Describe what is returned, including edge cases.

```java
/**
 * Finds a user by their unique identifier.
 *
 * @param id the user identifier
 * @return an Optional containing the user if found, or empty if not found
 */
public Optional<User> findById(Long id) {
    // User lookup logic
}
```

### @throws

Document all exceptions that can be thrown. Follow Design by Contract principles.

```java
/**
 * Transfers money between accounts.
 *
 * @param from the source account
 * @param to the destination account
 * @param amount the amount to transfer
 * @throws IllegalArgumentException if amount is not positive
 * @throws InsufficientFundsException if source account has insufficient balance
 * @throws AccountNotFoundException if either account does not exist
 */
public void transfer(Account from, Account to, BigDecimal amount) {
    // Account transfer logic
}
```

### @see

Link to related classes, methods, or external resources.

```java
/**
 * A thread-safe counter implementation.
 *
 * @see java.util.concurrent.atomic.AtomicInteger
 * @see <a href="https://docs.oracle.com/javase/specs/jls/se17/html/jls-17.html">JLS Chapter 17</a>
 */
public final class Counter {
    // Counter implementation
}
```

### @since

Indicate the version when a feature was introduced.

```java
/**
 * Validates the user's email address using RFC 5322 rules.
 *
 * @param email the email address to validate
 * @return true if the email is valid
 * @since 2.1
 */
public boolean validateEmail(String email) {
    // Email validation logic
}
```

### @deprecated

Always include `@deprecated` tag AND `@Deprecated` annotation. Explain why and suggest alternatives.

```java
/**
 * Returns the user's full name.
 *
 * @return the full name
 * @deprecated Use {@link #getDisplayName()} instead. This method will be removed in version 3.0.
 */
@Deprecated(since = "2.5", forRemoval = true)
public String getFullName() {
    // Legacy implementation
}
```

---

## Inline Tags

Use inline tags within Javadoc text for linking and formatting.

### {@link} and {@linkplain}

Link to other classes, methods, or fields.

```java
/**
 * Converts this user to a {@link UserDTO} for API responses.
 *
 * <p>For batch conversions, use {@link UserMapper#toDto(List)} instead.</p>
 *
 * @return the DTO representation
 * @see UserMapper#toDto(User)
 */
public UserDTO toDto() {
    // DTO conversion logic
}
```

- `{@link}` - renders in code font
- `{@linkplain}` - renders in plain text font

### {@code}

Format inline code. Escapes HTML and renders in monospace.

```java
/**
 * Returns {@code true} if this collection contains no elements.
 *
 * <p>Equivalent to {@code size() == 0}.</p>
 *
 * @return {@code true} if empty, {@code false} otherwise
 */
public boolean isEmpty() {
    // Check if empty
}
```

### {@literal}

Escape HTML characters without code formatting.

```java
/**
 * Compares using the {@literal <} operator.
 *
 * <p>Returns {@literal true} if {@literal a < b}.</p>
 */
public boolean isLessThan(int a, int b) {
    return a < b;
}
```

### {@value}

Display the value of a constant.

```java
/**
 * The default timeout value is {@value #DEFAULT_TIMEOUT} milliseconds.
 */
public static final int DEFAULT_TIMEOUT = 5000;
```

---

## Code Examples

Use `<pre>{@code ...}</pre>` for multi-line code blocks.

### ✅ Do

```java
/**
 * Parses a JSON string into a User object.
 *
 * <p>Example usage:</p>
 * <pre>{@code
 * String json = """
 *     {"name": "John", "email": "john@example.com"}
 *     """;
 * User user = UserParser.parse(json);
 * }</pre>
 *
 * @param json the JSON string to parse
 * @return the parsed User object
 * @throws ParseException if the JSON is malformed
 */
public static User parse(String json) {
    // JSON parsing logic
}
```

### ❌ Don't

```java
/**
 * Example usage:
 *
 * User user = UserParser.parse(json);  // Code is not formatted without <pre>
 *
 * <pre>
 * Map<String, User> map = new HashMap<>();  // Generics are interpreted as HTML without {@code}
 * </pre>
 */
public static User parse(String json) {
    // ...
}
```

---

## HTML in Javadoc

Use HTML sparingly for formatting.

### Allowed Tags

| Tag | Purpose |
|-----|---------|
| `<p>` | Separate paragraphs |
| `<ul>`, `<ol>`, `<li>` | Lists |
| `<pre>` | Preformatted text (with `{@code}`) |
| `<table>`, `<tr>`, `<th>`, `<td>` | Tables |
| `<h2>`, `<h3>`, `<h4>` | Headings (never `<h1>`) |
| `<em>`, `<strong>` | Emphasis |
| `<a href="...">` | External links |

### ✅ Do

```java
/**
 * Validates user input according to the following rules:
 *
 * <ul>
 *   <li>Username must be 3-20 characters</li>
 *   <li>Email must be a valid RFC 5322 address</li>
 *   <li>Password must contain at least one uppercase letter</li>
 * </ul>
 *
 * <p>For custom validation rules, implement {@link Validator}.</p>
 *
 * @param input the user input to validate
 * @return validation result
 */
public ValidationResult validate(UserInput input) {
    // Input validation logic
}
```

### ❌ Don't

```java
/**
 * <h1>User Validator</h1>  // Javadoc already uses h1
 *
 * <br><br>  // Use <p> to separate paragraphs
 *
 * <font color="red">Important!</font>  // Deprecated HTML tag
 */
public class UserValidator {
    // ...
}
```

---

## Headings

Use maximum `<h2>` tags for headings (as `<h1>` is already used by Javadoc).

### ✅ Do

```java
/**
 * <h2>SomeClass</h2>
 *
 * <h3>Usage</h3>
 * <p>Example code here...</p>
 *
 * <h3>Thread Safety</h3>
 * <p>This class is thread-safe.</p>
 */
public final class SomeClass {
    // Class implementation
}
```

### ❌ Don't

```java
/**
 * <h1>SomeClass</h1>  // Do not use h1
 */
public final class SomeClass {
    // ...
}
```

---

## Exception Documentation

Document exceptions with `@throws` following Design by Contract principles.

### ✅ Do

```java
public final class SomeClass {

    /**
     * Processes the input value and returns the result.
     *
     * <p>The input value must be within a valid range.</p>
     *
     * @param param input value to process, must be 0 or less
     * @return processing result
     * @throws IllegalArgumentException if param is greater than 0
     */
    public int func(int param) {
        if (param > 0) {
            throw new IllegalArgumentException("param must be <= 0, but was: " + param);
        }
        return 0;
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    /**
     * <p>Brief feature description</p>
     * <p>Detailed feature description</p>
     */
    public int func(int param) throws IllegalArgumentException {  // @throws declared only in signature without documentation
        if (param > 0) throw new IllegalArgumentException();
        return 0;
    }
}
```

---

## Inherited Documentation

Use `{@inheritDoc}` to inherit documentation from superclass or interface.

### ✅ Do

```java
public interface Repository<T> {
    /**
     * Finds an entity by its unique identifier.
     *
     * @param id the entity identifier, must not be null
     * @return an Optional containing the entity, or empty if not found
     * @throws IllegalArgumentException if id is null
     */
    Optional<T> findById(Long id);
}

public class UserRepository implements Repository<User> {

    /**
     * {@inheritDoc}
     *
     * <p>This implementation uses a database query with caching.</p>
     */
    @Override
    public Optional<User> findById(Long id) {
        // Database query logic
    }
}
```

### ❌ Don't

```java
public class UserRepository implements Repository<User> {

    // No Javadoc on overridden method - IDE shows interface docs, but
    // generated Javadoc HTML has no description
    @Override
    public Optional<User> findById(Long id) {
        // ...
    }
}
```

---

## Null Handling Documentation

Document null behavior explicitly. Prefer using nullability annotations.

### ✅ Do

```java
/**
 * Finds a user by username.
 *
 * @param username the username to search for, must not be null
 * @return the user, or {@code null} if not found
 */
@Nullable
public User findByUsername(@NonNull String username) {
    // User search logic
}
```

Better approach using Optional:

```java
/**
 * Finds a user by username.
 *
 * @param username the username to search for, must not be null
 * @return an Optional containing the user, or empty if not found
 */
public Optional<User> findByUsername(@NonNull String username) {
    // User search logic
}
```

---

## Common Anti-Patterns

### Empty or Useless Javadoc

```java
// ❌ Empty Javadoc
/**
 */
public void process() { }

// ❌ Repeating the method name is meaningless
/**
 * Gets the name.
 */
public String getName() { return name; }

// ✅ Add useful information
/**
 * Returns the user's display name, formatted as "First Last".
 *
 * @return the display name, never null or empty
 */
public String getName() { return name; }
```

### Missing First Sentence Period

```java
// ❌ No period at the end of the first sentence
/**
 * Returns the user name
 * This is used for display purposes.
 */

// ✅ Correct format
/**
 * Returns the user name.
 *
 * <p>This is used for display purposes.</p>
 */
```

### Incorrect Tag Order

```java
// ❌ Incorrect tag order
/**
 * @return the result
 * @since 1.0
 * @param input the input  // @param should come before @return
 * @throws Exception if error  // @throws should come after @return
 */

// ✅ Correct order
/**
 * @param input the input
 * @return the result
 * @throws Exception if error
 * @since 1.0
 */
```

---

## Package Documentation

Document packages using `package-info.java`.

### ✅ Do

```java
// package-info.java
/**
 * Provides user management functionality.
 *
 * <p>This package contains classes for creating, updating, and querying users.</p>
 *
 * <h2>Key Classes</h2>
 * <ul>
 *   <li>{@link com.example.user.UserService} - Main service for user operations</li>
 *   <li>{@link com.example.user.UserRepository} - Data access layer</li>
 * </ul>
 *
 * <h2>Usage Example</h2>
 * <pre>{@code
 * UserService service = new UserService(repository);
 * User user = service.createUser("john@example.com");
 * }</pre>
 *
 * @since 1.0
 */
package com.example.user;
```

---

## See Also

- [Concurrency](./concurrency.md) - Thread safety documentation patterns (@ThreadSafe, @GuardedBy)
- [Security & Resources](./security-resources.md) - Security documentation patterns
