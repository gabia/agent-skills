---
name: java-coding-guideline
description: Fundamental Java coding standards and conventions that apply to all Java code. Load this skill first when starting any Java development task. Covers imports, constructors, nullability, immutability, exceptions, Javadoc, naming conventions, security, resource management, and concurrency patterns.
license: MIT
metadata:
  author: agent-skills
  version: "1.1"
  language: java
---

# Java Coding Guideline

**CRITICAL**: These rules apply to ALL Java code unless explicitly overridden by more specific skills.

## When to use this skill

Load this skill at the start of any Java development task:
- Writing new Java code
- Reviewing Java code
- Refactoring Java code
- Creating Java classes, interfaces, or enums

This skill contains universal Java coding standards that should always be followed.

For domain-specific patterns, also load:
- `java-spring-boot` - Spring Boot application patterns
- `java-testing-deep-dive` - Testing strategies and patterns

---

## Import Statements

Always use explicit imports without wildcards for better code clarity and maintainability.

### ✅ Do

```java
import static org.assertj.core.api.Assertions.assertThat;
import static org.assertj.core.api.Assertions.fail;
```

### ❌ Don't

```java
import static org.assertj.core.api.Assertions.*;
```

---

## Constructors

Use a single primary constructor and delegate auxiliary constructors using `this()`.

### ✅ Do

```java
public final class SomeClass {

    private final int field1;
    private final int field2;

    public SomeClass() {
        this(1, 2);
    }

    public SomeClass(int field1, int field2) {
         this.field1 = field1;
         this.field2 = field2;
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    private final int field1;
    private final int field2;

    public SomeClass() {
        this.field1 = 1;
        this.field2 = 2;
    }

    public SomeClass(int field1, int field2) {
         this.field1 = field1;
         this.field2 = field2;
    }
}
```

---

## Optional Usage

Avoid using `Optional` as return values or parameters. Use `@Nullable` annotations instead.

### ✅ Do

```java
public final class SomeClass {

    @Nullable
    public Integer someNullable() {
        if (something) return null;
        return 1;
    }

    public void func(@Nullable Integer param) {
        // do something
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    @NonNull
    public Optional<Integer> someNullable() {
        if (something) return Optional.ofNullable(null);
        return Optional.ofNullable(1);
    }

    public void func(@NonNull Optional<Integer> param) {
        // do something
    }
}
```

---

## Wrapper vs Primitive Types

- Use wrapper types when `null` is expected
- Use primitive types when `null` should not occur

### ✅ Do

```java
public final class SomeClass {

    private final int field1;

    @Nullable
    private final Long field2;

    public SomeClass(int field1, @Nullable Long field2) {
        this.field1 = field1;
        this.field2 = field2;
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    @NonNull
    private final Integer field1;

    @Nullable
    private final Long field2;

    public SomeClass(@NonNull Integer field1, @Nullable Long field2) {
        this.field1 = field1;
        this.field2 = field2;
    }
}
```

---

## Null Comparison

- Use `A == null` for general cases
- Use `Objects.isNull` for Stream API or functional programming contexts

### ✅ Do

```java
final Integer someVariable = null;

if (someVariable == null) {
    // do something
}

List<Integer> lists = new ArrayList<>();
lists.stream()
  .filter(Objects::isNull)
  .collect(Collectors.toList());
```

### ❌ Don't

```java
final Integer someVariable = null;

if (Objects.isNull(someVariable)) {
    // do something
}

List<Integer> lists = new ArrayList<>();
lists.stream()
  .filter(p -> p == null)
  .collect(Collectors.toList());
```

---

## Nullability Annotations

- Use JSpecify annotations
- Apply `@Nullable` and `@NonNull` to all methods including private ones
- Avoid class-level or package-level annotations (Lombok compatibility issues)

### ✅ Do

```java
public final class SomeClass {

    @Nullable
    private final Integer field1;

    @NonNull
    private final String field2;

    @NonNull
    public String func() {
        return process();
    }

    @Nullable
    private String process() {
        if (field1 == null) return null;
        return "awesome";
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    private final Integer field1;
    private final String field2;

    public String func() {
        return process();
    }

    private String process() {
        if (field1 == null) return null;
        return "awesome";
    }
}
```

---

## Package Naming

Use all lowercase and numbers only, following Google Java Style Guide conventions.

### ✅ Do

```java
package com.example.project.initscript;
```

### ❌ Don't

```java
package com.example.project.initScript;
```

---

## Static Factory Methods

Use only `of` naming convention. For multiple types, use format `of<Postfix>`.

### ✅ Do

```java
public final class SomeClass {

    private final int field1;

    private SomeClass(int field1) {
        this.field1 = field1;
    }

    public static SomeClass ofDefault() {
        return new SomeClass(1);
    }

    public static SomeClass ofSpecial() {
        return new SomeClass(2);
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    private final int field1;

    private SomeClass(int field1) {
        this.field1 = field1;
    }

    public static SomeClass from() {
        return new SomeClass(1);
    }

    public static SomeClass valueOf() {
        return new SomeClass(2);
    }
}
```

## Resource Management

Always use try-with-resources for `AutoCloseable` resources. Never rely on finalizers.

### Try-With-Resources

#### ✅ Do

```java
public String readFile(@NonNull Path path) throws IOException {
    try (var reader = Files.newBufferedReader(path)) {
        return reader.lines().collect(Collectors.joining("\n"));
    }
}

// 다중 리소스
public void copyStream(@NonNull InputStream in, @NonNull OutputStream out) throws IOException {
    try (in; out) {  // Java 9+: 이미 선언된 변수 사용 가능
        in.transferTo(out);
    }
}
```

#### ❌ Don't

```java
public String readFile(@NonNull Path path) throws IOException {
    BufferedReader reader = Files.newBufferedReader(path);
    try {
        return reader.lines().collect(Collectors.joining("\n"));
    } finally {
        reader.close();  // 예외 발생 시 원본 예외가 숨겨질 수 있음
    }
}
```

---

## Composition over Inheritance

Prefer composition over inheritance.

### ✅ Do

```java
public final class SomeFeature {
    public void func() { /* ... */ }
}

public final class SomeClass {

    private final SomeFeature feature;

    public void func2() {
        feature.func();
        // do something
    }
}
```

### ❌ Don't

```java
public class SomeFeature {
    public void func() { /* ... */ }
}

public final class SomeClass extends SomeFeature {

    public void func2() {
        super.func();
        // do something
    }
}
```

---

## Immutable Objects

All objects should use `final` keyword unless needed for extension or framework requirements.

### ✅ Do

```java
public final class SomeClass {
    // ...
}
```

### ❌ Don't

```java
public class SomeClass {
    // ...
}
```

---

## Exception Catching

Catch only the minimum necessary exceptions.

### ✅ Do

```java
try {
    // do something
} catch (IllegalArgumentException e) {
    // do catch
}
```

### ❌ Don't

```java
try {
    // do something
} catch (RuntimeException e) {
    // do catch
}
```


### Implementing AutoCloseable

When implementing `AutoCloseable`:
1. Make `close()` **idempotent** (safe to call multiple times)
2. **Never throw exceptions** from `close()` (suppressed exceptions problem)
3. **Log errors** instead of throwing them

#### ✅ Do

```java
public final class DatabaseConnection implements AutoCloseable {

    @NonNull
    private final Connection connection;

    private volatile boolean closed = false;

    @Override
    public void close() {
        if (closed) return;  // 멱등성: 여러 번 호출해도 안전

        closed = true;
        try {
            connection.close();
        } catch (SQLException e) {
            // 로그만 남기고 예외는 던지지 않음 (close에서 예외 발생 시 원본 예외가 숨겨짐)
            logger.warn("Failed to close connection", e);
        }
    }
}
```

#### ❌ Don't

```java
public final class DatabaseConnection implements AutoCloseable {

    @NonNull
    private final Connection connection;

    @Override
    public void close() throws SQLException {
        connection.close();  // 멱등성 없음, 예외 전파
    }
}
```

---

## Date/Time API

Use `java.time` (JSR-310) exclusively. `java.util.Date` and `java.util.Calendar` are banned in new code.

### Type Selection

| Use Case | Type | Example |
|----------|------|---------|
| 저장/전송 (UTC) | `Instant` | API 타임스탬프, DB 저장 |
| 사용자 표시 | `ZonedDateTime` | UI에 표시할 시간 |
| 날짜만 필요 | `LocalDate` | 생년월일, 만료일 |
| 시간만 필요 | `LocalTime` | 영업시간, 알람 |
| 기간 | `Duration` / `Period` | 경과 시간, 날짜 차이 |

### ✅ Do

```java
public final class Event {

    @NonNull
    private final Instant createdAt;  // UTC 타임스탬프

    @NonNull
    private final LocalDate eventDate;  // 날짜만

    @NonNull
    private final ZoneId timeZone;  // 표시용 타임존

    @NonNull
    public ZonedDateTime displayTime() {
        return createdAt.atZone(timeZone);
    }
}
```

Formatter reuse (thread-safe):

```java
public final class DateFormats {

    // DateTimeFormatter는 thread-safe하므로 상수로 재사용
    public static final DateTimeFormatter ISO_DATE = DateTimeFormatter.ISO_LOCAL_DATE;

    public static final DateTimeFormatter KOREAN_DATE = DateTimeFormatter.ofPattern("yyyy년 MM월 dd일");

    private DateFormats() {}
}
```

### ❌ Don't

```java
public final class Event {

    @NonNull
    private final Date createdAt;  // java.util.Date 사용 금지

    @NonNull
    private final Calendar calendar;  // java.util.Calendar 사용 금지

    @NonNull
    public String format() {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");  // thread-safe하지 않음
        return sdf.format(createdAt);
    }
}
```

### Legacy Interop (경계에서만 허용)

When interfacing with legacy APIs that require `Date`, convert at the boundary:

```java
// Instant → Date (경계 레이어에서만)
Date legacyDate = Date.from(instant);

// Date → Instant (수신 즉시 변환)
Instant instant = legacyDate.toInstant();
```

---

## Additional Guidelines

### Builder Pattern

Use Lombok-style builders for API consistency.

### Variable and Method Naming

- Use `var` keyword from JVM 10+
- Use meaningful variable names
    - But avoid too long variable, function, field names
- Method names should be verbs
- Avoid using 'get' prefix except for getters

### Camel Case Convention

Apply camel case to mixed-case words and abbreviations (e.g., "MultiNIC" becomes "MultiNic").

### Control Flow Statements

Use one-liner syntax for simple if statements without braces.

#### ✅ Do

```java
if (someCondition) doSomething();
```

#### ❌ Don't

```java
if (someCondition) {
    doSomething();
}
```

### Comments

- Use comments only in special cases
- TODO comments should be for immediate tasks
- Avoid unnecessary comments that restate the code

---

## See Also

### Reference Guides

For deeper understanding of specific topics:
- [references/async.md](references/async.md) - CompletableFuture and async programming patterns
- [references/concurrency.md](references/concurrency.md) - Thread safety, JCIP annotations, and concurrent collections
- [references/javadoc.md](references/javadoc.md) - Javadoc conventions and exception documentation
- [references/lombok.md](references/lombok.md) - Allowed and banned Lombok annotations
- [references/security-resources.md](references/security-resources.md) - Security best practices, password handling, SQL injection prevention, PII masking

### Domain-Specific Skills

- Load `java-testing-deep-dive` skill for testing strategies
