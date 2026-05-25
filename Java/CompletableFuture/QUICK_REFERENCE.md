# CompletableFuture Quick Reference

## Creation Methods

```java
// Returns a value asynchronously
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "value");

// Executes without returning value
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> { /* code */ });

// Already completed future
CompletableFuture<String> future = CompletableFuture.completedFuture("value");

// Manual creation
CompletableFuture<String> future = new CompletableFuture<>();
future.complete("value");
```

## Transformation Methods

| Method | Input | Output | Description |
|--------|-------|--------|-------------|
| `thenApply(fn)` | `Function<T,R>` | `CF<R>` | Transform result |
| `thenAccept(consumer)` | `Consumer<T>` | `CF<Void>` | Consume result |
| `thenRun(runnable)` | `Runnable` | `CF<Void>` | Run after completion |

```java
future.thenApply(s -> s.toUpperCase())        // Transform
      .thenAccept(s -> System.out.println(s)) // Consume
      .thenRun(() -> System.out.println("Done")); // Action
```

## Combining Methods

### Sequential Chaining (Dependent)
```java
// Use thenCompose for chaining futures (flatMap equivalent)
future.thenCompose(id -> fetchUser(id))
      .thenCompose(user -> fetchOrders(user));
```

### Parallel Combination (Independent)
```java
// Combine two independent futures
CF<User> userFuture = fetchUser();
CF<Order> orderFuture = fetchOrder();

userFuture.thenCombine(orderFuture, (user, order) -> {
    return new UserOrder(user, order);
});
```

## Exception Handling

```java
future
    .exceptionally(ex -> "default")  // Provide fallback value
    .handle((result, ex) -> {        // Handle both success and error
        return ex != null ? "error" : result;
    })
    .whenComplete((result, ex) -> {  // Observe (doesn't transform)
        if (ex != null) log(ex);
    });
```

## Multiple Futures

```java
// Wait for all to complete
CF<String> f1 = ..., f2 = ..., f3 = ...;
CF<Void> allOf = CompletableFuture.allOf(f1, f2, f3);

// Use first to complete
CF<Object> anyOf = CompletableFuture.anyOf(f1, f2, f3);

// Collect results from all
List<CF<String>> futures = Arrays.asList(f1, f2, f3);
CF<List<String>> results = CF.allOf(futures.toArray(new CF[0]))
    .thenApply(v -> futures.stream()
        .map(CF::join)
        .collect(Collectors.toList()));
```

## Timeout (Java 9+)

```java
// Complete exceptionally on timeout
future.orTimeout(5, TimeUnit.SECONDS);

// Complete with default value on timeout
future.completeOnTimeout("default", 5, TimeUnit.SECONDS);
```

## Either/Both Variants

| Method | Waits for | Returns |
|--------|-----------|---------|
| `applyToEither(other, fn)` | First complete | Transformed result |
| `acceptEither(other, consumer)` | First complete | Void |
| `runAfterEither(other, action)` | First complete | Void |
| `thenCombine(other, bifn)` | Both complete | Combined result |
| `thenAcceptBoth(other, biconsumer)` | Both complete | Void |
| `runAfterBoth(other, action)` | Both complete | Void |

## Async Variants

Every method has 3 variants:

```java
// Runs in completing thread (may be main or ForkJoinPool)
.thenApply(fn)

// Runs in ForkJoinPool.commonPool()
.thenApplyAsync(fn)

// Runs in custom executor
.thenApplyAsync(fn, executor)
```

## Custom Executor

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "value", executor)
    .thenApplyAsync(s -> s.toUpperCase(), executor);

// Don't forget to shutdown!
executor.shutdown();
```

## Retrieving Results

```java
// Blocking - throws CompletionException (unchecked)
String result = future.join();

// Blocking - throws ExecutionException (checked)
String result = future.get();

// Blocking with timeout
String result = future.get(5, TimeUnit.SECONDS);

// Check if completed
if (future.isDone()) {
    String result = future.join();
}
```

## Common Patterns

### 1. Parallel API Calls
```java
CF<A> cf1 = fetchA();
CF<B> cf2 = fetchB();
CF<C> cf3 = fetchC();

CF.allOf(cf1, cf2, cf3).thenRun(() -> {
    A a = cf1.join();
    B b = cf2.join();
    C c = cf3.join();
    combine(a, b, c);
});
```

### 2. Fallback Pattern
```java
fetchFromPrimary()
    .exceptionally(ex -> fetchFromBackup())
    .exceptionally(ex -> defaultValue());
```

### 3. Timeout with Fallback
```java
fetchData()
    .orTimeout(2, TimeUnit.SECONDS)
    .exceptionally(ex -> getCachedData());
```

### 4. Sequential Pipeline
```java
fetchUser(userId)
    .thenCompose(user -> fetchProfile(user.id))
    .thenCompose(profile -> enrichProfile(profile))
    .thenApply(profile -> convertToDTO(profile))
    .thenAccept(dto -> sendResponse(dto));
```

### 5. Fire and Forget
```java
CompletableFuture.runAsync(() -> {
    sendNotification();
    logEvent();
});
// Don't wait - continues immediately
```

## Best Practices ✓

### DO
- ✓ Use custom executor for blocking I/O
- ✓ Always handle exceptions
- ✓ Use `thenCompose` for chaining futures
- ✓ Set timeouts for external calls
- ✓ Use `join()` over `get()` for cleaner code
- ✓ Shutdown custom executors

### DON'T
- ✗ Don't block common pool with I/O
- ✗ Don't swallow exceptions silently
- ✗ Don't use `get()` in production (blocks thread)
- ✗ Don't create nested futures (use `thenCompose`)
- ✗ Don't forget to complete manually created futures

## Thread Pool Sizing

```java
// CPU-bound tasks
int cpuPoolSize = Runtime.getRuntime().availableProcessors();

// I/O-bound tasks (rule of thumb)
int ioPoolSize = cpuPoolSize * 2; // or higher based on blocking factor

// Separate pools for different workloads
ExecutorService cpuPool = Executors.newFixedThreadPool(cpuPoolSize);
ExecutorService ioPool = Executors.newCachedThreadPool();
```

## Testing

```java
// Use synchronous executor for tests
ExecutorService syncExecutor = Executors.newSingleThreadExecutor();

class Service {
    private final ExecutorService executor;
    
    Service(ExecutorService executor) {
        this.executor = executor;
    }
    
    CF<String> fetchAsync() {
        return CF.supplyAsync(() -> fetch(), executor);
    }
}

// In test: inject synchronous executor
Service service = new Service(syncExecutor);
String result = service.fetchAsync().join(); // Completes immediately
```

## Error Handling Strategy

```java
CompletableFuture.supplyAsync(() -> riskyOperation())
    .exceptionally(ex -> {
        logger.error("Operation failed", ex);
        metrics.incrementErrorCount();
        return fallbackValue();
    })
    .whenComplete((result, ex) -> {
        if (ex == null) {
            metrics.recordSuccess();
        }
        cleanup();
    });
```

## Performance Tips

1. **Minimize async boundaries** - Thread switching has overhead
2. **Batch operations** - Process multiple items together
3. **Use appropriate pool size** - Based on workload type
4. **Cache frequently accessed data** - Reduce remote calls
5. **Set timeouts** - Prevent hanging operations
6. **Monitor thread pools** - Track utilization and queue sizes

## Common Errors

### Nested Futures
```java
// WRONG
CF<CF<String>> nested = future.thenApply(x -> asyncOperation(x));

// RIGHT
CF<String> flat = future.thenCompose(x -> asyncOperation(x));
```

### Blocking Common Pool
```java
// WRONG
CF.supplyAsync(() -> {
    Thread.sleep(5000); // Blocks common pool thread!
    return database.query();
});

// RIGHT
CF.supplyAsync(() -> {
    return database.query();
}, ioExecutor);
```

### Ignoring Exceptions
```java
// WRONG
CF.supplyAsync(() -> riskyOperation()); // Exception lost!

// RIGHT
CF.supplyAsync(() -> riskyOperation())
  .exceptionally(ex -> {
      logger.error("Failed", ex);
      return defaultValue();
  });
```

## Method Comparison

| Scenario | Use This |
|----------|----------|
| Transform result | `thenApply()` |
| Chain async operations | `thenCompose()` |
| Just consume result | `thenAccept()` |
| Execute after completion | `thenRun()` |
| Combine two futures | `thenCombine()` |
| Wait for all | `allOf()` |
| Use fastest | `anyOf()` |
| Handle errors | `exceptionally()` or `handle()` |
| Observe completion | `whenComplete()` |
| Set timeout | `orTimeout()` (Java 9+) |

## Complete Example

```java
ExecutorService ioPool = Executors.newCachedThreadPool();

try {
    String result = CompletableFuture
        // Fetch user from database (blocking I/O)
        .supplyAsync(() -> fetchUser(userId), ioPool)
        
        // Transform to DTO (lightweight)
        .thenApply(user -> toDTO(user))
        
        // Fetch additional data (async operation)
        .thenCompose(dto -> enrichDTO(dto))
        
        // Timeout after 5 seconds
        .orTimeout(5, TimeUnit.SECONDS)
        
        // Handle any errors
        .exceptionally(ex -> {
            logger.error("Failed to fetch user", ex);
            return getDefaultDTO();
        })
        
        // Observe completion
        .whenComplete((result, ex) -> {
            metrics.recordLatency();
        })
        
        // Wait for result
        .join();
        
    return result;
    
} finally {
    ioPool.shutdown();
}
```

---

**Remember**: CompletableFuture is powerful but with great power comes great responsibility. Always handle exceptions, use appropriate thread pools, and test thoroughly!
