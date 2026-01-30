# Java Concurrency Deep Dive

**When to use this guide**: Reference when working with multi-threaded code

---

## Thread Safety

Document thread-safety guarantees explicitly. Use private lock objects and avoid `synchronized(this)`.

### Annotations

Use JCIP annotations to document thread-safety guarantees:
- `@ThreadSafe` - Class is safe for concurrent access
- `@NotThreadSafe` - Class is not safe for concurrent access
- `@GuardedBy("lockName")` - Field must be accessed while holding the named lock

### ✅ Do

```java
@ThreadSafe
public final class Counter {

    private final Object lock = new Object();  // 전용 락 객체 사용

    @GuardedBy("lock")
    private int count;

    public int increment() {
        synchronized (lock) {
            return ++count;
        }
    }

    public int get() {
        synchronized (lock) {
            return count;
        }
    }
}
```

For simple counters, prefer atomic classes:

```java
@ThreadSafe
public final class AtomicCounter {

    private final AtomicInteger count = new AtomicInteger(0);

    public int increment() {
        return count.incrementAndGet();
    }

    public int get() {
        return count.get();
    }
}
```

### ❌ Don't

```java
public class Counter {  // @ThreadSafe 어노테이션 누락

    private int count;  // @GuardedBy 어노테이션 누락

    public synchronized int increment() {  // this를 락으로 사용 - 외부에서 데드락 유발 가능
        return ++count;
    }
}
```

---

### Concurrent Collections

Use `ConcurrentHashMap` instead of `Collections.synchronizedMap()` for better performance.

#### ✅ Do

```java
private final ConcurrentMap<String, User> cache = new ConcurrentHashMap<>();

public void cacheUser(@NonNull User user) {
    cache.putIfAbsent(user.id(), user);
}
```

#### ❌ Don't

```java
private final Map<String, User> cache = Collections.synchronizedMap(new HashMap<>());
```

---

## CompletableFuture

### Function Signature Rules

1. **Return type should be `CompletionStage<T>`** (not `CompletableFuture<T>`)
2. **Return value should never be `null`** - the returned `CompletionStage` itself must not be `null`
3. **Inner value can be `@Nullable`** - the value completed inside the stage may be null if the generic type is annotated as `@Nullable`

### ✅ Do

```java
public final class SomeClass {

    @NonNull
    public CompletionStage<@NonNull String> someAsync() {
        return CompletableFuture.completedFuture("awesome");
    }

    @NonNull
    public CompletionStage<@Nullable Integer> someOptional(int param) {
        if (param > 0) return CompletableFuture.completedFuture(null);

        return CompletableFuture.completedFuture(0);
    } 
}
```

### ❌ Don't

```java
public final class SomeClass {

    @NonNull
    public CompletableFuture<@NonNull String> someAsync() {
        return CompletableFuture.completedFuture("awesome");
    }

    @Nullable
    public CompletionStage<@NonNull Integer> someOptional(int param) {
        if (param > 0) return null;  // 절대 null을 반환하지 않음
        return CompletableFuture.completedFuture(0);
    }
}
```

---

### Exception Handling

All exceptions should be passed through `CompletableFuture`. Never throw exceptions directly.

### ✅ Do

#### Java 9+

```java
public final class SomeClass {

    @NonNull
    public CompletionStage<@Nullable Integer> someOptional(int param) {
        if (param > 0) {
            return CompletableFuture.failedFuture(
                new IllegalArgumentException("param must be zero or negative")
            );
        }
        return CompletableFuture.completedFuture(0);
    }
}
```

#### Java 8

```java
public final class SomeClass {

    @NonNull
    public CompletionStage<@Nullable Integer> someOptional(int param) {
        if (param > 0) {
            CompletableFuture<Integer> future = new CompletableFuture<>();
            future.completeExceptionally(
                new IllegalArgumentException("param must be zero or negative")
            );
            return future;
        }
        return CompletableFuture.completedFuture(0);
    }
}
```

### ❌ Don't

```java
public final class SomeClass {

    @NonNull
    public CompletionStage<@NonNull Integer> someOptional(int param) {
        if (param > 0) throw new IllegalArgumentException("param should be zero or negative");  // 예외를 직접 던지지 않음
        return CompletableFuture.completedFuture(0);
    }
}
```

---

## Async Best Practices

### Chaining CompletableFutures

When chaining async operations, use appropriate methods:

- `thenApply()` - Transform result (sync)
- `thenApplyAsync()` - Transform result (async in thread pool)
- `thenCompose()` - Chain another CompletableFuture
- `thenCombine()` - Combine two independent CompletableFutures
- `exceptionally()` - Handle exceptions
- `handle()` - Handle both success and failure

### Example

```java
@NonNull
public CompletionStage<@NonNull UserProfile> getUserProfile(@NonNull String userId) {
    return fetchUser(userId)
        .thenCompose(user -> fetchUserSettings(user.id()))
        .thenCombine(fetchUserPreferences(userId), (settings, prefs) -> 
            new UserProfile(settings, prefs)
        )
        .exceptionally(ex -> {
            logger.error("Failed to fetch user profile", ex);
            return UserProfile.ofDefault();
        });
}
```

---

### Thread Pool Selection

- Default ForkJoinPool: CPU-bound tasks
- Custom Executor: I/O-bound or blocking operations

```java
private final ExecutorService ioExecutor = Executors.newFixedThreadPool(10);

@NonNull
public CompletionStage<@NonNull Data> fetchData() {
    return CompletableFuture.supplyAsync(() -> {
        // I/O 작업 - 전용 스레드 풀 사용
        return blockingIoOperation();
    }, ioExecutor);
}
```

---

## Common Pitfalls

### 1. Blocking in Async Context

**Don't** call `.get()` or `.join()` in async chains:

```java
// ❌ BAD - 비동기 체인에서 블로킹 호출
public CompletionStage<String> bad() {
    return fetchData()
        .thenApply(data -> {
            String result = otherAsyncCall().toCompletableFuture().join();  // 블로킹!
            return process(data, result);
        });
}

// ✅ GOOD - thenCompose로 비동기 유지
public CompletionStage<String> good() {
    return fetchData()
        .thenCompose(data -> 
            otherAsyncCall().thenApply(result -> process(data, result))
        );
}
```

---

### 2. Exception Handling Mistakes

Always handle exceptions in async chains:

```java
// ❌ BAD - 예외 처리 없음
public CompletionStage<Data> bad() {
    return fetchData()
        .thenApply(this::process);  // 예외 발생 시 전파만 됨
}

// ✅ GOOD - 적절한 예외 처리
public CompletionStage<Data> good() {
    return fetchData()
        .thenApply(this::process)
        .exceptionally(ex -> {
            logger.error("Failed to process data", ex);
            return Data.ofDefault();
        });
}
```

---

### 3. Thread Safety in Async Operations

Shared mutable state requires synchronization even with CompletableFuture:

```java
// ❌ BAD - 동기화되지 않은 공유 상태
private int counter = 0;

public CompletionStage<Integer> bad() {
    return CompletableFuture.supplyAsync(() -> counter++);  // race condition
}

// ✅ GOOD - 동시성 안전 타입 사용
private final AtomicInteger counter = new AtomicInteger(0);

public CompletionStage<Integer> good() {
    return CompletableFuture.supplyAsync(() -> counter.incrementAndGet());
}
```

---

## Debugging Tips

### Enable Async Stack Traces

For better debugging, consider libraries like:
- Spring's `@Async` with proper exception handling
- Project Reactor's debugging features
- Custom CompletableFuture wrappers with context propagation

### Logging in Async Chains

Always log exceptions with context:

```java
return fetchData()
    .thenApply(data -> {
        logger.debug("Processing data: {}", data.id());
        return process(data);
    })
    .whenComplete((result, ex) -> {
        if (ex != null) {
            logger.error("Async operation failed", ex);
        } else {
            logger.debug("Async operation succeeded: {}", result);
        }
    });
```

---

## Summary Checklist

### Thread Safety
- ✅ Document thread-safety with JCIP annotations
- ✅ Use private lock objects, avoid `synchronized(this)`
- ✅ Prefer atomic classes for simple counters
- ✅ Use `ConcurrentHashMap` over synchronized collections

### CompletableFuture
- ✅ Return `CompletionStage<T>`, not `CompletableFuture<T>`
- ✅ Never return `null` for the stage itself
- ✅ Use `CompletableFuture.failedFuture()` for exceptions (Java 9+)
- ✅ Use `thenCompose()` for chaining, not blocking calls
- ✅ Always handle exceptions with `exceptionally()` or `handle()`

### Thread Pools
- ✅ Use default ForkJoinPool for CPU-bound tasks
- ✅ Use custom executor for I/O-bound operations
- ✅ Shut down executors properly in lifecycle methods

### Pitfalls to Avoid
- ❌ Never call `.get()` or `.join()` in async chains
- ❌ Never throw exceptions directly from async methods
- ❌ Never use shared mutable state without synchronization
- ❌ Never forget exception handling in async chains
