# lombok

## Allowed

### Logging

- `@Slf4j` (preferred) Use it instead of manually declaring a logger.

### Boilerplate reduction (safe)

- `@Getter` (prefer on fields or class for read-only models)    
- `@RequiredArgsConstructor` (great for DI + `final` fields)
- `@NoArgsConstructor` / `@AllArgsConstructor` **only when required by frameworks**, and document why
- `@Builder` (good for complex constructors / test fixtures)
- `@StandardException` (good for exception)

### Use with caution

- `@ToString`
	- Use cautiously. It can accidentally log secrets or huge object graphs.
	- If object has secrets or has expected huge object graphs, then
		- `@ToString(onlyExplicitlyIncluded = true)`
		- exclude sensitive fields explicitly
- `@EqualsAndHashCode`
	- OK for Value Object
    - `cacheStrategy=lazy` is recommended
	- but when using this for Aggregate Root or Domain objects then ID matches requires
		- `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`

## Ban (Strongly recommended)

The following annotations are explicitly banned:

| Annotation | Reason |
|------------|--------|
| `@Setter` | Breaks immutability, weakens encapsulation |
| `@Data` | Includes `@Setter`; risky auto-generated equals/hashCode |
| `@Value` | Confuses with records; too much implicit behavior |
| `@SneakyThrows` | Bypasses checked exceptions; unclear error handling intent |
| `@Cleanup` | Prefer try-with-resources |
| `@val` / `@var` | Use Java 10+ `var` keyword instead |
| `@Synchronized` | Prefer explicit synchronization code |
| `@UtilityClass` | Prefer explicit `private` constructor and `final class` |
| `@With` | Prefer explicit builders for immutable copies |
| `@Accessors` | Non-standard getter/setter patterns |

**Rule**: Ban any annotation not explicitly listed in the Allowed section.
