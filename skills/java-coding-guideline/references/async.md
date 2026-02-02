# Java Async Deep Dive

**When to use this guide**: Reference when working with multi-threaded code, CompletableFuture  or any async programming in Java.

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
        if (param > 0) return null;  // Never return null
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
        if (param > 0) throw new IllegalArgumentException("param should be zero or negative");  // Do not throw exceptions directly
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
        // I/O work - use a dedicated thread pool
        return blockingIoOperation();
    }, ioExecutor);
}
```

---

### Avoid Blocking in Async Context

**Don't** call `.get()` or `.join()` in async chains:

### ✅ Do

```java
public CompletionStage<String> good() {
    return fetchData()
        .thenCompose(data -> 
            otherAsyncCall().thenApply(result -> process(data, result))
        );
}
```

### ❌ Don't

```java
public CompletionStage<String> bad() {
    return fetchData()
        .thenApply(data -> {
            String result = otherAsyncCall().toCompletableFuture().join();  // Blocking!
            return process(data, result);
        });
}
```
