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
 * 사용자 인증을 처리하는 서비스.
 *
 * <p>이 서비스는 로그인, 로그아웃, 토큰 갱신 기능을 제공한다.</p>
 *
 * @since 1.0
 */
public interface AuthService {

    /**
     * 사용자 자격 증명을 검증하고 인증 토큰을 반환한다.
     *
     * @param credentials 사용자 자격 증명
     * @return 인증 토큰
     * @throws AuthenticationException 자격 증명이 유효하지 않은 경우
     */
    AuthToken authenticate(Credentials credentials);
}
```

### ❌ Don't

```java
// public API에 Javadoc 없음
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
    // 사용자 이름 포맷팅
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
    // 사용자 생성 로직
}
```

### ❌ Don't

```java
/**
 * This method gets the user's display name  // "This method" 불필요, 동사로 시작해야 함
 */
public String getDisplayName() {
    // ...
}

/**
 * Get the display name  // 3인칭 "Gets"를 사용해야 함
 */
public String getDisplayName() {
    // ...
}

/**
 * Returns the user's display name (e.g. John Doe).  // "e.g." 뒤의 마침표로 요약이 끊김
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
@param      (매개변수 - 선언 순서대로)
@return     (반환값)
@throws     (예외 - 알파벳 순서로)
@see        (참조)
@since      (도입 버전)
@deprecated (폐기 예정)
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
    // 사용자 검색 로직
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
    // 사용자 조회 로직
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
    // 계좌 이체 로직
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
    // 카운터 구현
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
    // 이메일 검증 로직
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
    // 레거시 구현
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
    // DTO 변환 로직
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
    // 비어있는지 확인
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
    // JSON 파싱 로직
}
```

### ❌ Don't

```java
/**
 * Example usage:
 *
 * User user = UserParser.parse(json);  // <pre> 없이 코드가 포맷팅되지 않음
 *
 * <pre>
 * Map<String, User> map = new HashMap<>();  // {@code} 없이 제네릭이 HTML로 해석됨
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
    // 입력값 검증 로직
}
```

### ❌ Don't

```java
/**
 * <h1>User Validator</h1>  // h1은 Javadoc이 이미 사용함
 *
 * <br><br>  // 문단 분리에는 <p>를 사용해야 함
 *
 * <font color="red">Important!</font>  // deprecated HTML 태그
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
    // 클래스 구현
}
```

### ❌ Don't

```java
/**
 * <h1>SomeClass</h1>  // h1은 사용하지 않음
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
     * 입력값을 처리하고 결과를 반환한다.
     *
     * <p>입력값이 유효한 범위 내에 있어야 한다.</p>
     *
     * @param param 처리할 입력값, 0 이하여야 함
     * @return 처리 결과
     * @throws IllegalArgumentException param이 0보다 큰 경우
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
     * <p>기능 대략 설명</p>
     * <p>기능 상세 설명</p>
     */
    public int func(int param) throws IllegalArgumentException {  // @throws 문서화 없이 시그니처에만 선언
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
        // 데이터베이스 조회 로직
    }
}
```

### ❌ Don't

```java
public class UserRepository implements Repository<User> {

    // 오버라이드 메서드에 Javadoc 없음 - IDE에서 인터페이스 문서를 보여주지만
    // 생성된 Javadoc HTML에서는 설명이 없음
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
    // 사용자 검색 로직
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
    // 사용자 검색 로직
}
```

---

## Common Anti-Patterns

### Empty or Useless Javadoc

```java
// ❌ 빈 Javadoc
/**
 */
public void process() { }

// ❌ 메서드 이름을 반복하는 것은 무의미함
/**
 * Gets the name.
 */
public String getName() { return name; }

// ✅ 유용한 정보를 추가해야 함
/**
 * Returns the user's display name, formatted as "First Last".
 *
 * @return the display name, never null or empty
 */
public String getName() { return name; }
```

### Missing First Sentence Period

```java
// ❌ 첫 문장 끝에 마침표 없음
/**
 * Returns the user name
 * This is used for display purposes.
 */

// ✅ 올바른 형식
/**
 * Returns the user name.
 *
 * <p>This is used for display purposes.</p>
 */
```

### Incorrect Tag Order

```java
// ❌ 태그 순서가 잘못됨
/**
 * @return the result
 * @since 1.0
 * @param input the input  // @param은 @return 전에 와야 함
 * @throws Exception if error  // @throws는 @return 후에 와야 함
 */

// ✅ 올바른 순서
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
